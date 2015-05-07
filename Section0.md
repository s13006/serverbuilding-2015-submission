# Section 0 講義の前のセットアップ

## 0-1 VirtualBoxのインストール

公式サイトからdebファイルをダウンロード(Ubuntu14.04)

パッケージが足りないので

    sudo aptitude install libsdl1.2debian:amd64

他にパッケージが足りてなければエラー読んでインストール

パッケージインストールが完了したら

    sudo dpkg -i Downloads/virtualbox-4.3_4.3.26-98988-Ubuntu-raring_amd64.deb 

コマンド叩いて起動確認

    virtualbox


## 0-2 Vagrantのインストール

公式サイト行ってUbuntuの64bitダウンロード

    sudo dpkg -i vagrant_1.7.2_x86_64.deb

コマンド叩いて起動確認

    vargrant
