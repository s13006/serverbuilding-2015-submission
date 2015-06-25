# Section 3 Ansibleによる自動化とテスト

毎回毎回手動で

    yum install もにょもにょ

とやるのも非効率なので、それらを自動化してくれるツールを使って今迄の作業を何回でもできるようにします。

今回の講義ではAnsibleを使用します。

## 3-0 Ansibleのインストール

[公式サイト](http://docs.ansible.com/intro_installation.html#latest-releases-via-apt-ubuntu)に手順載ってるのでそのとおりにやってください。

リポジトリ追加してインストール

		sudo apt-get install software-properties-common
		sudo apt-add-repository ppa:ansible/ansible
		sudo apt-get update
		sudo apt-get install ansible

## 3-1 ansibleでWordpressを動かす(2)を行なう

###ansibleの疎通確認

まずsshpassをインストールする。

		sudo aptitude install sshpass

インストールできたらansibleの疎通確認

		ansible -k [ipaddress] -m ping

パスワードを聞かれるのでパスワードを入力(初期のパスワードは'vagrant')


###proxyの超絶楽な設定方法 & playbook実行するための設定

		vagrant plugin install vagrant-proxyconf
		vagrant plugin install vagrant-vbguest

Vaglantfileに書いてあるの全部コピーしてね

###playbook書く
playbook.ymlにいろいろ書いてあるのでコピペしてね
コピペできたら以下のコマンドを実行してplaybookを読み込みましょう

		vagrant provision
