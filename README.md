# lamp-server-setup
選考課題としてVirtualBoxとAlmaLinuxを使用したLAMP環境構築の記録。Apache VirtualHost、PHP・MySQL連携、WordPress構築まで実施。

# LAMP環境構築

選考課題としてVirtualBox上のAlmaLinuxに、Apache、PHP、MySQL、WordPressを使用したLAMP環境を構築した記録です。

Linuxとサーバ構築の知識がほとんどない状態から取り組み、各ミドルウェアのインストール、設定、動作確認、WordPressでの記事投稿まで実施しました。

## 構築環境

- ホストOS：Windows 11
- 仮想化ソフト：Oracle VM VirtualBox
- ゲストOS：AlmaLinux 9.8
- Webサーバ：Apache httpd 2.4.62
- PHP：PHP 7.4.33
- DB：MySQL 8.0.46
- CMS：WordPress

> PHP 7.4は課題要件に合わせて使用しました。現在の新規環境でPHP 7.4を推奨するものではありません。

## 実施内容

- VirtualBoxへのAlmaLinuxインストール
- タイムゾーンのJST設定
- chronyによるNTP同期
- firewalldの有効化とHTTP許可
- SELinuxの無効化
- Apacheのインストールと自動起動
- Apache VirtualHostの設定
- Windows hostsファイルを利用した名前解決
- PHP 7.4とphpinfoの表示
- MySQL 8.0のインストールと自動起動
- PHPからMySQLのデータを取得して表示
- WordPressのインストールと記事投稿

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

## 動作確認

以下をブラウザから確認しました。

- 2つのVirtualHostで異なるページを表示
- `phpinfo()` によるPHP 7.4.33の確認
- PHPからMySQLの `meibo` テーブルを参照し、登録した氏名を表示
- WordPressでテスト記事を投稿し、ブログ上に表示

動作確認画面は、詳細レポート内に掲載しています。

## 問題解決の例

### VirtualBoxでコピー＆ペーストできない

当初はVirtualBoxのコンソール上ですべてのコマンドを手入力していましたが、入力ミスが増えていました。

そこで、VirtualBoxのNATポートフォワーディングを設定し、Windows PowerShellからSSH接続する方法へ切り替えました。

これにより、入力ミスを減らし、作業速度と正確性を改善しました。

### WordPressでNot Foundが表示された

エラーとファイル配置を確認したところ、WordPressのアーカイブが正常に展開されておらず、DocumentRoot配下にWordPress本体が存在していませんでした。

アーカイブを再展開し、正しい場所へ配置することで解決しました。

## 学んだこと

Webページが表示されるまでには、名前解決、ネットワーク、Webサーバ、PHP、DBなど複数の要素が関係しています。

今回の構築を通じて、エラーの種類によって確認箇所が異なることを学びました。

- `command not found`：コマンド名やパッケージを確認する
- `Permission denied`：権限を確認する
- `Not Found`：URLやファイル配置を確認する
- `Connection refused`：サービスやポートの状態を確認する

未経験の技術であっても、問題を小さく分解し、確認可能な単位で検証することで、成果物まで到達できると学びました。

## AIの利用について

初めて扱う技術が多かったため、エラーの意味や次に確認すべき観点を整理する目的で生成AIも補助的に利用しました。

ただし、提案された内容をそのまま使用するのではなく、実際の環境で以下を確認しました。

- 設定ファイルの内容
- `systemctl status` によるサービス状態
- `httpd -t` によるApache設定の構文
- `httpd -S` によるVirtualHostの認識状況
- `curl` によるLinux内部からの表示
- Windowsブラウザからの最終動作

最終的な設定変更と動作確認は自分の環境で行いました。

## ファイル

- ./TECHNICAL_DETAILS.md
- [LAMP環境構築レポート](./LAMP環境

## 今後試したいこと

- SELinuxを有効にした状態での構築
- HTTPS化
- WebサーバとDBサーバの分離
- WordPressとMySQLのバックアップ・復元
- サーバ監視
