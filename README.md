# lamp-server-setup
選考課題としてVirtualBoxとAlmaLinuxを使用したLAMP環境構築の記録。Apache VirtualHost、PHP・MySQL連携、WordPress構築まで実施。
# LAMP環境構築

選考課題としてVirtualBox上のAlmaLinuxに、Apache、PHP、MySQL、WordPressを使用したLAMP環境を構築した記録です。

Linuxおよびサーバ構築の知識がほとんどない状態から取り組み、各ミドルウェアのインストール、設定、動作確認まで実施しました。

## 構築環境

- ホストOS：Windows 11
- 仮想化ソフト：Oracle VM VirtualBox
- ゲストOS：AlmaLinux 9.8
- Webサーバ：Apache httpd 2.4.62
- PHP：PHP 7.4.33
- DB：MySQL 8.0.46
- CMS：WordPress

> PHP 7.4は課題要件に合わせて選択しています。現在の新規環境でPHP 7.4を推奨するものではありません。

## 実施内容

- VirtualBoxへのAlmaLinuxインストール
- タイムゾーンのJST設定
- chronyによるNTP同期
- firewalldの有効化とHTTP許可
- SELinuxの無効化
- Apache 2.4のインストールと自動起動
- Apache VirtualHostの設定
- Windows hostsファイルを利用した名前解決
- PHP 7.4とphpinfoの表示
- MySQL 8.0のインストールと自動起動
- PHPからMySQLのデータを取得して表示
- WordPressのインストール
- WordPressでの記事投稿

## 構成

```text
Windowsブラウザ
    |
    | hostsファイルによる名前解決
    v
VirtualBox NAT / ポートフォワーディング
    |
    v
AlmaLinux
    |
    +-- Apache
    |     |
    |     +-- PHP
    |           |
    |           +-- MySQL
    |
    +-- WordPress
```

## Apache VirtualHost

以下の2つのホスト名に対して、異なるDocumentRootを設定しました。

```text
takada-hbtask.local
  -> /var/www/takada-hbtask.local

haruto-hbtask.local
  -> /var/www/haruto-hbtask.local
```

### 表示確認

#### 名字側VirtualHost

![名字/screenshots/01-apache-surname.png

#### 名前側VirtualHost

docs/screenshots/02-apache-firstname.png

## PHP

PHP 7.4をインストールし、Apacheを通して `phpinfo()` が実行されることを確認しました。

![phpscreenshots/03-phpinfo.png

## PHP・MySQL連携

MySQLに以下のデータを登録しました。

```text
number: 1
username: Haruto Takada
```

PHPから次のSQLを実行し、取得した氏名をブラウザに表示しました。

```sql
SELECT username
FROM meibo
WHERE number = 1;
```

![PHPとMySQLの連携](docs/screenshots/04-mysql-php.pngWordPress公式サイトからアーカイブを取得し、ApacheのDocumentRoot配下へ配置しました。

初期設定後、WordPress管理画面から記事を投稿し、ブログ上に表示されることを確認しました。

![WordPress投稿](docs/screenshots/05-wordpress生した問題と対応

### VirtualBoxのコンソールでコピー＆ペーストできない

当初はVirtualBoxのコンソール上ですべてのコマンドを手入力していました。

長いコマンドや設定ファイルで入力ミスが増えたため、VirtualBoxのNATポートフォワーディングを設定し、Windows PowerShellからSSH接続する方式へ変更しました。

これにより、入力ミスを減らし、作業速度と正確性を改善できました。

### `apachectl -S` が利用できない

VirtualHostの確認で `apachectl -S` を実行しましたが、環境上サポートされていませんでした。

代わりに次のコマンドを使用しました。

```bash
sudo httpd -S
```

### WordPressでNot Foundが表示された

WordPressへのアクセス時にNot Foundが表示されました。

コマンドの実行結果とファイル配置を確認したところ、WordPressのアーカイブが正常に展開されておらず、DocumentRoot配下にWordPress本体が存在していませんでした。

アーカイブを再展開し、正しい場所へ配置することで解決しました。

## 学んだこと

今回の環境構築を通じて、Webページが表示されるまでに、名前解決、ネットワーク、Webサーバ、PHP、DBなど複数の要素が関わっていることを学びました。

また、表示されない場合も、以下のように原因を分けて確認する必要があると理解しました。

- `command not found`：コマンド名またはパッケージを確認する
- `Permission denied`：権限を確認する
- `Not Found`：URLやファイル配置を確認する
- `Connection refused`：サーバやポートの状態を確認する

未経験の技術であっても、問題を小さく分解し、確認可能な単位で検証することで、成果物まで到達できると学びました。

## AIの利用について

初めて扱う技術が多かったため、エラーの意味や次に確認すべき観点を整理する目的で生成AIも補助的に使用しました。

ただし、提案されたコマンドをそのまま信用するのではなく、実際の環境で次の確認を行いました。

- 設定ファイルの内容確認
- `systemctl status` によるサービス状態確認
- `httpd -t` によるApache設定の構文確認
- `httpd -S` によるVirtualHostの確認
- `curl` によるLinux内部からの表示確認
- Windowsブラウザからの最終確認

最終的な設定変更と動作確認は、自分の環境で実施しました。

## 詳細レポート

実行したコマンド、発生したエラー、解決方法、参考資料については、以下のレポートにまとめています。

[詳細レポート（PDF）](docs/LAMP環境構築レ## 今後試したいこと

- SELinuxを有効にした状態での構築
- HTTPS化
- WebサーバとDBサーバの分離
- WordPressとMySQLのバックアップ・復元
- ApacheまたはWordPressの監視
