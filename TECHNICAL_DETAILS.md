# 技術詳細・設定例

本ファイルには、LAMP環境構築で使用した主な設定、SQL、PHPコード、問題と解決方法をまとめています。

実際のパスワードや認証情報は掲載していません。

---

## 1. chrony設定

課題で指定されたNTPサーバを設定しました。

```conf
server ntp1.jst.mfeed.ad.jp iburst
server ntp2.jst.mfeed.ad.jp iburst
server ntp3.jst.mfeed.ad.jp iburst
```

確認には次のコマンドを使用しました。

```bash
systemctl status chronyd
chronyc sources -v
```

---

## 2. firewalld設定

firewalldを起動し、自動起動を有効にしました。

```bash
sudo systemctl enable --now firewalld
```

HTTP通信を許可しました。

```bash
sudo firewall-cmd --add-service=http --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-all
```

---

## 3. SELinux設定

課題要件に従い、SELinuxを無効にしました。

```conf
SELINUX=disabled
```

再起動後、次のコマンドで確認しました。

```bash
getenforce
```

確認結果：

```text
Disabled
```

---

## 4. Apache VirtualHost設定

```apache
<VirtualHost *:80>
    ServerName takada-hbtask.local
    DocumentRoot /var/www/takada-hbtask.local

*   <Directory "/var/www/takada-hbt*sk.local">
        Require all gra*ted
    </Directory>
</VirtualHost*

<Virtual*ost *:80>
*   ServerName haruto-hbtask.local
*   DocumentRoot /var/www/haruto-hb*ask.local

    <Directory "/var/ww*/haruto-hbtask.local">
        Req*ire all granted
    </Directory>
<*VirtualHost>
```

設定確認：

```bash**udo httpd -t
sudo httpd -S
```

--*

## 5. Windows hosts設定

```text
1*7.*.0.1 takada-hbtask.local
*27.0.0.1 haruto-hbtask.local
```

*irtualBoxのNATポートフォワーディングでは、次の対応を設定*ました。

```text
ホストポート 2222 → ゲストポート*22
ホスト*ート 80   → ゲストポ*ト 80
```

SSH接続：

```powershell
ss* -p 2222 haruto@127.0.0.1
```

---*
## 6. VirtualHost用HTML

### 名*側*
```html
<!DOCTYPE html>
<html lan*="ja">
<head>
   *<*eta charset="UTF-8">
    <title>Su*name VirtualHost</title>
</head>
<*ody>
    <p>こんにちは。ありがとう。</p*
*/body>
</html>
```

### 名前側

```ht*l
<!DOCTYPE html>
<html lang="ja">*<head>
*  *<meta charset="UTF-8">
    <title>*irstname VirtualHost</title>
</hea*>
<body*
*  *<*>ありがとう。こんにちは。</p>
</body>
</html>
*``

---

## 7. PHP確認用ファイル

```php
*?php
phpinfo();
?>
```

`php*nfo()` はサーバの詳細情報を表示する*め、本番環境では公開しないことを前提としています。

---

##*8. MySQL設定

### データベースとテーブル

```sq*
CREATE DATABASE hbtask
    CHARAC*ER*SET utf8mb4
*   COLLATE utf8mb4_0900_ai_ci;

US* hbtask;

CREATE TABLE meibo (
   *number INT,
    username VARCHAR(1*0)
);

INSERT INTO*meibo (number, username)
*ALUES (1, 'Haruto Takada');

SELEC* username
FROM meibo
WHERE number * 1;
```

### Webアプリケーション用ユーザー

```*ql
CREATE USER 'hbuser'@'localhost*
    IDENT*FIED BY 'YOUR_PASSWORD_HERE';

*RANT SELECT ON hbtask.*
    TO 'hb*ser'@'localhost';

FLUSH PRIVILEGE*;
```

`YOUR_PASSWORD_HERE` は公開用のプースホルダーです。実際のパスワードは掲載していません。

---

*# 9. PHP・*ySQL連携

*``*hp
<?php
$host = 'localhost';
$use* = 'hbuser';
$password = 'YOUR_PAS*WORD_HERE';
$database = 'hbtask';
*$mysqli = new mysqli($host, $user,*$password, $database);

if ($mysql*->connect_error) {
    die(
      * 'DB接続失敗: ' .
        htmlspecialc*ars(
            $mysqli->connect_*rror,
            ENT_QUOTES,
    *       'UTF-8'
        )
    );
}
*$mysqli->set_charset('utf8mb4');

*stmt = $mysqli->prepare(
    'SELE*T username FROM meibo WHERE number*= ?'
);

$number*= 1;

$stmt->bind_param('i', $numb*r);
$stmt->execute();
$stmt->*ind*result($username);
$stmt->fetch();*
echo '私の名前は ' .
    htmlspecialch*rs($username, ENT_QUOTES, 'UTF-8')*.
    ' です。';

$stmt->close();
$my*qli->close();
?>
```

SQL文と入力値を分離す*ために、プリペアドステートメントを使用しました。

---

## *0. WordPress

WordPress公式サイトからアーカイ*を取得しました。

```bash
cd /tmp
curl -O *ttps://wordpress.org/latest.tar.gz*tar xzf latest.tar.gz
```

Apacheの*ocumentRoot配下へ配置しました。

```bash
sud* mv wordpress /var/www/takada-hbta*k.local/blog
sudo chown -R apache:*pache /var/www/takada-hbtask.local*blog
sudo systemctl restart httpd
*``

WordPress用データベースは、Webアプリケーション用*Bと分離しました。

```sql
CREATE DATABASE *ordpress_hbtask
    CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

CREATE USER 'wpuser'@'localhost'
    IDENTIFIED BY 'YOUR_PASSWORD_HERE';

GRANT ALL PRIVILEGES
    ON wordpress_hbtask.*
    TO 'wpuser'@'localhost';

FLUSH PRIVILEGES;
```

実際の `wp-config.php` は認証情報を含むため公開していません。

---

## 11. 発生した問題と対応

### コマンドと引数の間にスペースがなかった

誤り：

```bash
cat/etc/os-release
```

修正：

```bash
cat /etc/os-release
```

### `$home` を使用してPermission deniedになった

Linuxの環境変数では大文字と小文字が区別されます。

誤り：

```bash
script -a $home/hbtask_work.log
```

修正：

```bash
script -a /home/haruto/hbtask_work.log
```

### VirtualBoxのコンソールでコピー＆ペーストできなかった

SSH接続へ切り替えました。

```powershell
ssh -p 2222 haruto@127.0.0.1
```

### `apachectl -S` が利用できなかった

環境上、次のコマンドは利用できませんでした。

```bash
sudo apachectl -S
```

代わりに以下を使用しました。

```bash
sudo httpd -S
```

### WordPressでNot Foundが表示された

`mv` と `chown` のエラーから、WordPressディレクトリが存在していないことを確認しました。

アーカイブを再展開し、正しい場所へ配置することで解決しました。

```bash
sudo dnf install -y tar
cd /tmp
tar xzf latest.tar.gz
sudo mv wordpress /var/www/takada-hbtask.local/blog
```

---

## 12. 公開時の安全対策

以下は公開していません。

- 実際のDBパスワード
- WordPress管理者パスワード
- Linuxユーザーのパスワード
- `wp-config.php`
- SSH秘密鍵
- 作業ログ
- VirtualBoxの仮想ディスク

コード内のパスワードは、すべて次のプレースホルダーに置き換えています。

```text
YOUR_PASSWORD_HERE
```