# Section 5 DNSサーバーを動作させる

インターネットにおいて、これが無いとほぼ成り立たないのがDNSサーバー。
最近は特に不人気なんだけど、一応基本のDNSサーバーであるbindを設定してみよう。

## 5-1 bindのインストール

yumで入るので適当にインストールしてください。

ansible化しとくと楽ですよ。

## 5-2 bindの設定

chroot化&自分のゾーンを作ってレコードを返すように設定してください。

[RHEL7のNetworking Guide(英語)](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Networking_Guide/sec-BIND.html)にやり方書かれてるのでそれを参考に設定してください。

また、ゾーン転送を行なって、ミラーリングができていることを確認してください(そのためには2台の仮想マシンが必要になります)

###自分のゾーンを作る
chroot化はplaybookで全部やっちゃってるので自分のゾーンを作ります。

落としてきたansibleディレクトリの中にplaybookやらが入ってるのでいろいろ書き換える。
この時Vaglantfileの設定でIPアドレスが今まで設定してたのと違ったらipアドレスの設定を書き換えなければならない。
(例)　192.168.33.14　と書いてあってもともと192.168.56.??を使っていたら33の部分を変更する。(master-named.confとかslave-named.confの中でもipアドレスを記述しているのでそこも書き換える)

###digコマンドでレコードを取得する

ゾーンを作ることができたらdigコマンドを使ってDNSサーバーがちゃんと動いているか確認する。

		dig [@DNSサーバー名(ipaddress)] [ドメイン名]

結果が帰ってきたらおｋ。