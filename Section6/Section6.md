# Section 6 AWS(Amazon Web Services)

このセクションではAWS(Amazon Web Services)を使用したサーバー構築を行ないます。

## 講義関連リンク

* [AWS公式サイト](http://aws.amazon.com/jp/)
* [Cloud Design Pattern](http://aws.clouddesignpattern.org/index.php/%E3%83%A1%E3%82%A4%E3%83%B3%E3%83%9A%E3%83%BC%E3%82%B8)

## 6-0 AWSコマンドラインインターフェイスのインストール

[公式サイト](http://aws.amazon.com/jp/cli/)参照。

まずはよなしろせんせーからアクセスキーをもらう。
公式サイトを見ながら進める。

aws configureの設定までできたら

###インスタンスの作成

AWS公式サイトへ行ってサインアップをするとマネジメントコンソールが開かれるのでEC2を選択しインスタンスを作成する。
マシンイメージはAmazon Linux、あとはデフォルトのまま作成。
作成時にpemファイルをダウンロードするので権限を400に変更

		chmod 400 [ファイル名].pem

インスタンスの一覧を取得する

		aws ec2 describe-instances 

###インスタンスのセキュリティグループにHTTPを追加

インスタンスをクリックし、セキュリティグループの設定画面に移動する。
編集を選択しHTTPを追加

インスタンスを作成したら今後はコマンドを使って起動や停止を行う

起動

		aws ec2 start-instances --instance-ids [id名]


停止

		aws ec2 stop-instances --instance-ids [id名]

###インスタンスにssh接続

EC2 Management Console画面に行き自分の作成したインスタンスを選択、上の接続ボタンを押して画面に出てきた通りに実行する。
うまく実行できれば終了。

注意：pemファイルのある場所で接続しないとうまく行きません。

## 6-1	AWS EC2 + Ansible

Amazon Elastic Computing Cloud(EC2)を使用してWordpressが動作するサーバーを作ります。

###hostsファイルの作成

以下の文字をhostsファイルに書く。

		vi hosts

		[all]
		[EC2のIPアドレス]

ファイルを落として、playbookを実行

		ansible-playbook -i hosts --private-key ./[pemファイル] playbook.yml

あとは今までどおりブラウザで開いてWordPressの設定して終了。

### AMI(Amazon Machine Image)を作る

環境の構築が終わったら、AMIを作成します。AMIを作成後、同じマシンを2つ起動して、コピーができていることを確認してください。

インスタンスをクリック、イメージ、作成を選択
なんか適当に設定する。

AMIの画面に反映されたらインスタンスを作成。
初回にインスタンスを作成したようにセキュリティグループにHTTPを追加してブラウザで開きWordPressが動いていることを確認。

## 6-2 AWS EC2(AMIMOTO)

6-1では自力(?)で環境構築を行ない、AMIを作成したが、別の人が作ったAMIを使用してサーバーを起動することもできる。

AMIMOTOのWordpressを起動してWordpressが見れることを確認する。

## 6-3 Route53

Route53はAWSが提供するDNSサービス。

5-1で作ったDNSの情報をRoute53に突っ込んでみよう。

## 6-4 S3

S3はSimple Storage Service。その名の通り、ファイルを保存し、(状況によっては)公開するサービス。

てきとーにWebサイトを作り、それをS3にアップロードし、公開してみよう。

S3にアップロードする際にはAWSコマンドラインインターフェイスを使ってね。

## 6-5 CloudFront

CloudFrontはCDNサービスです。CDNって何って?ggrましょう(ちゃんと講義では説明するので聞いてね)。

6-1で作ったAMIを起動し、CloudFrontに登録します。登録して直接アクセスするのとCloudFront経由するのどっちが速いかベンチマークを取ってみましょう。

また、CloudFrondを経由することで、地域ごとにアクセス可能にしたり不可にしたりできるので、それを試してみましょう。

## 6-6 RDS

RDSは…MySQLっぽい奴です。

RDSを立ち上げて、6-1で作ったAMIのWordpressのDBをRDSに向けてみよう。

## 6-7 ELB

ELBはロードバランサーです。すごいよ。

6-1で作ったAMIを3台ぶんくらい立ち上げてELBに登録し、負荷が割り振られているか確認してみよう。

## 6-8 API叩いてみよう

AWSは自分で作ったプログラムからもいろいろ制御できます!
なんでもいいのでがんばってプログラム書いてみてね(おすすめはSES)。
