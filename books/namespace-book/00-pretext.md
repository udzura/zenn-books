---
title: "はじめに/環境構築"
---

# Hello! unshared world.

本書は、筆者が[コンテナを自作](https://github.com/haconiwa/haconiwa)したり、そのほかシスプロで遊んだり僭越ながら講義をしたりしていろいろと溜まっていった、Linuxにおける名前空間機能（Linux Namespace）に関する知見を書き残すためのオンライン書籍です。

一般にLinuxの名前空間機能は、DockerやLXCを代表としたコンテナ機能の実装に利用されます。また、複数の名前空間をセットで同時に隔離することが多く、例えばホスト名の分離、プロセスの分離などは個別の機能であると意識されにくいです。本書ではそういった個別の名前空間に光を当て、それぞれのclone/unshareに渡すフラグによりどのようなことが起こるのかをなるべく解説します。コンテナ自作をしたい、chrootにちょい足しをしたい、systemdやsnapが変な名前空間を作るのをデバッグしたい、などという読者の要望に応えるものです。

執筆は以下の方針で行います。

* それぞれの名前空間に関して、コマンドラインツールでの動作確認と、プログラミングによる簡単なPoCの作成を行います。
* PoCに用いる言語は **Rust** の予定です。素人ですが頑張ります。
  * ただ、より低レベルなインタフェースで理解したい読者のために、C言語での実装例も可能な限り付記します。
* 将来的にはカーネルの当該コードを読んでいくなどの解説も行いたいですが、筆者の気力と反響次第ですね...。

また、本書は基本的に無料で配信したいと考えています。その上で、Rustでの書き方上のツッコミや、カーネルに関する理解の間違いなどが残った状態で公開されるかもしれません。また、しばらくは未完成な部分が残った原稿となるでしょう。公開することで徐々にブラッシュアップできれば、と甘く考えております。

本書の原稿は、 [GitHub 上で公開](https://github.com/udzura/zenn-books) し、いつでもPull RequestやIssueを作成可能な状態を保ちます。

## 環境構築

本書のコンテンツで遊ぶための環境を構築します。

推奨環境はIntel CPU上で、Vagrant + VirtualBox を用いてx86_64環境のLinuxを立ち上げ、その中で作業をすることです。これは拙作[mrubyシステムプログラミング入門](https://www.c-r.com/book/detail/1364)と同じような開発環境です。

ディストリビューションはUbuntu Groovyを用います。以下のようなVagrantfileを作成し、

```ruby
Vagrant.configure("2") do |c|
  c.vm.define :groovy do |config|
    config.vm.box = "ubuntu/groovy64"
    config.vm.box_check_update = true

    config.vm.synced_folder ".", "/vagrant"

    # 皆様の環境に合わせてください。
    #config.vm.provider "virtualbox" do |vb|
    #  vb.cpus = 15
    #  vb.memory = (1024 * 8).to_s
    #end

    config.disksize.size = '120GB'
  end
end
```

以下のようなコマンドを発行して立ち上げると良いでしょう。

```
$ vagrant plugin install vagrant-disksize
$ vagrant up groovy
```

Linux環境に `vagrant ssh groovy` でログインし、Rustのビルド環境を入れておきましょう。[公式の方法](https://www.rust-lang.org/tools/install)で全く構いません。

```
$ curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
$ echo 'source $HOME/.cargo/env' >> ~/.bashrc
$ . ~/.bashrc
$ cargo --version
cargo 1.51.0 (43b129a20 2021-03-16)
```

本書の手順では複数のターミナルを立ち上げる場合があります。[tmux](https://github.com/tmux/tmux/wiki)のような端末多重化ソフトウェアを用いると便利かもしれません。そのほか、必要なパッケージをその場で入れるような指示があるかもしれません。

```
$ sudo apt install tmux
```

### その他の環境の読者について

まず、ホストマシンがWindowsの場合も、同様の手順でゲストLinuxをインストール、操作ができるはずです。詳細は一旦は割愛します。

また、本書はホストがARM系CPUである場合、ゲストLinuxがARMである場合の両方についてサポートしませんが、基本的には同様に動くはずです...。今後の原稿ではシグナルの番号、システムコールの番号などをハードコードしている箇所があるかもしれません。例えばそれらの番号はアーキテクチャにより違うので、そういう場合にご留意ください。

ホストマシンがLinuxの方々。その場合でも、kvmなどを用いて仮想環境を作り、その内部での作業をお勧めします。ホスト側でNamespaceに関する操作を誤った場合、ホストマシンの制御を失い、再起動をするしか対処法がなくなる場合が考えられますので、そのような面倒な場面を避け心配なく大胆な操作（？）を行えるよう、一つ仮想環境を挟みましょう。[vagrant-libvirt](https://github.com/vagrant-libvirt/vagrant-libvirt)などは便利ではないかと思います。

## 参考となる文献、資料について。

事前に以下の3つのインターネット記事をご覧いただくと、もしかしたら本書が不要になる... いえ、本書の理解が進むでしょう。

### [LXCで学ぶコンテナ入門 －軽量仮想化環境を実現する技術](https://gihyo.jp/admin/serial/01/linux_containers/0001)

加藤さんの連載記事です。コンテナの内部理解に関する日本語の基本文献と言われています（筆者の周りで）。筆者もこの連載記事を読んでコンテナの何たるかを理解したつもりです。

### [Container Security Book](https://container-security.dev/)

森田さんによる、コンテナに関するセキュリティの話題をまとめたウェブ書籍ですが、その理解のために必要な知識として、丁寧にコンテナの内部技術を解説しています。

### [コンテナ技術入門 - 仮想化との違いを知り、要素技術を触って学ぼう](https://eh-career.com/engineerhub/entry/2019/02/05/103000)

今井さんによるコンテナ要素技術の全体的な解説です。非常にわかりやすくまとめられ、分量としても読みやすいかと思います。

また、以下に書籍を2つ紹介しますが、執筆時点で筆者が目を通せていない（本当にすいません）ので、あくまで参考とはなります。非常に評判は良いと聞いています。

### [コンテナ型仮想化概論](https://www.cutt.co.jp/book/978-4-87783-478-4.html)

目次で「濃さ」がわかる本ですね。情報の少ないFreeBSDやSolarisでのコンテナ実装にも言及されています。

### [イラストでわかる DockerとKubernetes](https://gihyo.jp/book/2020/978-4-297-11837-2)

クラウドネイティブコミュニティで活躍される徳永さんによる、DockerやKubernetesの内部技術にも踏み込んだ詳細な本です。

----

ということで、マニアックだけれどいつかどこかで役立つ、個別のLinux Namespaceの世界にunshareされていきましょう。

