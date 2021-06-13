# Kali Linuxのインストール

## 環境
- ホストOS: Windows10
- ゲストOS: Kali-linux-2021.1-vbox-amd64
- Virtualbox 6.1

## Virtualboxのインストール
[Virtualbox](tmp)からVirtualboxをダウンロードする。

- [extension pack](https://download.virtualbox.org/virtualbox/6.1.22/Oracle_VM_VirtualBox_Extension_Pack-6.1.22.vbox-extpack)

## Kali Linuxのイメージダウンロード
- 今回は、Virtualboxを使ってKali Linuxを起動するため、VirtualboxImageを選択する。
- [Kali linux](tmp)からダウンロードできる。

## 仮想アプライアンスのインポートを行う。
VirtualBoxを起動し、ファイル→仮想アプライアンスのインポートを選択する。ここでダウンロードしたKali LinuxのOVAファイルを選択する。
![仮想アプライアンスのインポート](/home/tomita/article/picture/virtualapplyance.png)

このあと、ライセンス等に関する警告がでるのでよく読んでから同意する。

## とりあえず起動してみる。
- いきなり設定を弄って起動しなくなると困るので初回起動は、デフォルト設定のまま行う。
- 初期のユーザ名パスワードはどちらもkaliである。
![初期ユーザ](/home/tomita/article/picture/kalikali.png)

## CPUやメモリの量を変更する。
VirtualBoxでは指定できるCPUの数やメモリの量はユーザのホストPCによって異なってくる。
Virtualboxでは安全な構成は緑色になっている。
今回は、メモリを4 Gb、CPUは全体の半分使う。

![virtualbox](/home/tomita/article/picture/cpumemory.png)
チップセットは個人的な好みICH9を使う。
![ICH9](/home/tomita/article/picture/ICH9.png)
ビデオメモリーは256 Mbに設定する。通常ではビデオメモリーの上限は126 Mbだが、ディスプレイの数を1度8にしてからビデオメモリーを252 Mbに設定し、ディスプレイの数を1に戻すことで設定できる。
![ビデオメモリー](/home/tomita/article/picture/videomemory.png)

## 共有フォルダの作成
仮想マシンで作成したファイルをホストマシンに持ってきたり、逆にホストマシンで作成したファイルを仮想マシンに持っていく方法の一つが共有フォルダを使うことである。

共有フォルダ内にあるファイルはホストマシンと仮想マシンの両方からアクセスできるようになるのでファイルの受け渡しに便利である。
### 共有フォルダの作成方法
1. Virtualboxを立ち上げて共有フォルダをクリック。
2. フォルダのパスの部分にホストマシンの好きなパスを指定する。
3. 自動マウントにチェックを入れる。
![共有フォルダ](/home/tomita/article/picture/sharefolder.png)

共有フォルダはホストマシンのホームディレクトリ以下にマウントされる。

## ソフトウェアアップデート
ここからはターミナルにコマンドを入力する。
ターミナルは左上のアイコンをクリックすることで起動できる。
![terminal](/home/tomita/article/picture/terminalclick.png)

> Linux系のOSはCtrl Alt tでターミナルが起動するものが多い。Kali LInuxでもこのショートカットキーを使ってターミナルを起動できる。
### apt-get update

入力する画面が現れたら、アップデートの確認をするために以下のコマンドを入力する。sudoは管理者権限を使うため、パスワードを入力するように促されるがこれもkaliでOK

```shell
sudo apt-get update
```

### apt-get upgrade
次にソフトウェアのアップデートを行う。
通常ならばsudo apt upgradeと入力するのだが、今回はアップデートの際のダウンロードを並列的に実行するのでアップデートを高速で終わらせるapt-fastを使う。

```shell
cd Desktop/
git clone https://github.com/ilikenwf/apt-fast.git
cd apt-fast
sudo bash quick-install.sh
sudo apt-fast upgrade -y
```

## 日本語化
日本語関係のパッケージをインストールする。

```shell
sudo apt-get install -y task-japanese task-japanese-desktop
sudo dpkg-reconfigure locales
```
![jajpUTF](/home/tomita/article/picture/jajputf.png)
ja_JP.UTF-8 UTF-8をクリックしてtabキーを押してOKの位置に置いてエンターし、次にja_JP.UTF-8を選択する。

```shell
sudo update-locale LANG=ja_JP.UTF-8
```

sudo lsusb #認識されているusb機器を表示
sudo gpasswd -a tomita vboxusers
