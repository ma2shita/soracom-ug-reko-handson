---
layout: default
title: ステップ1. 環境準備
---

# ステップ1. 環境準備（SORACOM回線を使ったスマートフォンとAWSサービスを用いた画像認識サービスを構築する）

*当コンテンツは、エッジデバイスとしてスマートフォン、クラウドサービスとしてAWSを利用し、エッジデバイスとクラウド間とのデータ連携とAWSサービスを利用した画像認識を体験し、IoT/画像認識システムの基礎的な技術の習得を目指す方向けのハンズオン(体験学習)コンテンツ「[SORACOM回線を使ったスマートフォンとAWSサービスを用いた画像認識サービスを構築する](https://iotkyoto.github.io/soracom-ug-reko-handson/)」の一部です。*

# ステップ1. 環境準備

今回のハンズオンでは、エッジデバイスであるスマートフォンで表示するWebアプリケーションと、クラウド側で稼働するWebAPIを構築します。

AWS CLIを使ってのコマンドラインでの操作や、プログラムを読み書きする作業は、ブラウザ上で動く統合開発環境サービスである「AWS Cloud9」を使用します。
今回使用するサンプルプログラムやテストデータはGithubに公開していますので、そちらからファイル一式を取得します。

また、スマートフォンで表示するWebアプリケーションは、ハンズオン参加者全員で同じWebサイトを使用することが可能なため、構築済みのWebアプリケーションを用意しています。

---

### 目的

- AWS Cloud9 環境を立ち上げ、実際に使ってみる
- AWS CLIを使ってみる

### 概要

- AWSコンソールからCloud9環境を構築する
- AWS CLIを使ってS3にファイルをアップロードする

---

### ＜AWS Cloud9とは？＞

AWS Cloud9は、ブラウザのみでコードを記述、実行、デバッグできるクラウドベースの統合開発環境 (IDE) です。
コードエディタ、デバッガー、ターミナルの機能が含まれています。

Cloud9 IDE はクラウドベースのため、インターネットに接続されたマシンを使用して、オフィス、自宅、その他どこからでもプロジェクトに取り組むことができます。

また、JavaScript、Python、Java など４０を超える一般的なプログラム言語に不可欠なツールがあらかじめパッケージ化されているため、新しいプロジェクトを開始するためにファイルをインストールしたり、開発マシンを設定したりする必要はありません。

- ブラウザのみでコーディングが可能
  - ブラウザのみでアプリケーションを作成、実行、デバッグできる
- リアルタイムにコードの共同編集が出来る
  - 容易にペアプログラミングを行うことが出来る
  - IDE内でのチャットも可能
- サーバーレスアプリケーションの開発環境の構築が簡単に出来る
  - サーバーレス開発に必要なSDK、ライブラリ、プラグインが導入済みの環境がすぐに構築できる
  - AWS Lambda 関数をローカルでテストおよびデバッグするための環境を利用できる
- AWSの各種サービスにターミナルからアクセス可能
  - 事前認証されたAWS CLIがセットアップ済みとなっている
  - ターミナル経由でAWSリソースに対する操作が可能
- Cloud9自体の料金は無料
  - Cloud9を稼働させるEC2インスタンス/EBSボリュームに対してのみ[料金](https://aws.amazon.com/jp/cloud9/pricing/)が発生

より詳しく知りたい場合は[公式サイト](https://aws.amazon.com/jp/cloud9/)をご確認ください。

### ＜AWS CLIとは？＞

AWSコマンドラインインターフェースの略で、AWSサービスをコマンドラインから操作し管理するためのオープンソースツールです。
コマンドラインから複数のAWSサービスを制御し、スクリプトを使用してこれらの作業を自動化することができます。

- Windows/Mac/Linuxなどの様々なOSで使用可能
  - Windowsの場合、PowerShell または Windowsコマンドプロンプトでコマンドを実行
  - Mac/Linux/Unixの場合、bash/zsh/tcshなどの一般的なシェルプログラムでコマンドを実行
- 使い方やリファレンスは以下を参照
  - [AWSコマンドラインインターフェースのユーザーガイド](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-chap-welcome.html)
  - [AWS CLIコマンドリファレンス](https://docs.aws.amazon.com/cli/latest/reference/)
- AWS CLI には 「バージョン 1.x」「バージョン 2.x」の 2 つのバージョンがある（2021年2月時点のCloud9(AmazonLinux2)には AWS CLI version1.x がデフォルトでインストールされているため、今回のハンズオンコンテンツは AWS CLI version1.x での実行となります）

より詳しく知りたい場合は[公式サイト](https://aws.amazon.com/jp/cli/)をご確認ください。

---

## 1-1. AWS Cloud9 環境を構築する

AWSコンソールにログインし、AWS Cloud9の環境を構築します。

### 1-1-1. AWSコンソールにログインする
- 各自のアカウントでAWS環境にログインしてください  
 **使用するユーザーは「AdministratorAccess」権限が付与されていることを前提とします**
![1-1-1_1](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-1_1.png)
![1-1-1_2](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-1_2.png)

- リージョンを「東京」言語を「日本語」に切り替えてください

### 1-1-2. Cloud9サービスに移動する

- 画面上部の「サービス」をクリックしてください
- 検索窓に「cloud」と入力し、候補から「Cloud9」を選択してください
![1-1-2](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-2.png)

### 1-1-3. Cloud9環境を構築する

- [create environment]（環境を作成する）をクリックしてください
![1-1-3_1](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-3_1.png)

- Nameに「{名前}-handson-env」を入力し、[Next step] をクリックしてください
例）yamada-handson-env
![1-1-3_3](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-3_3.png)

- 設定内容に以下の設定を行い、画面下部の [Next step] をクリックしてください
  - Environment type：Create a new EC2 instance for environment (direct access)
  - Instance type：t2.micro (1 GiB RAM + 1 vCPU)
  - Platform：Amazon Linux 2 (recommended)
  - Cost-saving setting：After 30 minutes(default)
![1-1-3_4](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-3_4_2.png)
![1-1-3_5](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-3_5_2.png)

- 設定内容確認画面で [create environment] をクリックしてください
![1-1-3_6](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-1-3_6_2.png)

※ 新しいタブが開き、Cloud9 IDEのCreate処理が行われますので、環境が立ち上がるまでしばらくお待ちください(２〜３分程度かかります)

## 1-2. 必要なプログラム・データを取得する
今回使用するプログラムやデータをCloud9環境に取得します
必要なファイルは、ソースコード管理サービスの[Github](https://github.com/IoTkyoto/soracom-ug-reko-handson)に保管しています。

### 1-2-1. ターミナルを開く

- Cloud9 IDE の画面を表示してください
![1-2-1_1](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-2-1_2_2.png)

- 「＋」をクリックして、新しくターミナルを開いてください
![1-2-1_2](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-2-1_1_2.png)

### 1-2-2. Githubからファイル一式をCloneする

- ターミナルで以下のコマンドを実行し、Githubからソースコードを取得する
※ ブラウザの設定によってはコマンド文字列をペーストする際にダイアログが表示される可能性があります

```sh
$ git clone https://github.com/IoTkyoto/soracom-ug-reko-handson
```

- コマンドの実行結果が表示される
```sh
Cloning into 'soracom-ug-reko-handson'...
remote: Enumerating objects: 369, done.
remote: Counting objects: 100% (369/369), done.
remote: Compressing objects: 100% (324/324), done.
remote: Total 369 (delta 131), reused 210 (delta 29), pack-reused 0
Receiving objects: 100% (369/369), 21.30 MiB | 2.04 MiB/s, done.
Resolving deltas: 100% (131/131), done.
```

- 左側のEnvironment部分に「soracom-ug-reko-handson」ディレクトリが出来ていればClone完了です。
![1-2-2_1](https://s3.amazonaws.com/docs.iot.kyoto/img/SoracomUG-Reko-Handson/step1/1-2-2_1.png)

- これらのファイルは以後のステップで使用します

## 1-3. AWS CLIを動かす

ステップ２で使用するため、Cloud9でAWS CLIが動くことを確認します。

### 1-3-1. AWS CLIのバージョンを表示する

- ターミナルから以下のコマンドを実行してください

```sh
$ aws --version
```

- エラーとならずに、AWS CLIのバージョンが表示されれば、AWS CLIは使用可能状態です。

```sh
aws-cli/1.18.212 Python/2.7.18 Linux/4.14.209-160.339.amzn2.x86_64 botocore/1.19.52
```
※ 上記各バージョン表記は一例となります。時期によってバージョンは変更されますので注意ください。

### 次のステップへ進んでください

- ここまでの作業で事前の準備作業は終了です
- [トップページ](https://iotkyoto.github.io/soracom-ug-reko-handson/)に戻って次のステップに進んでください
