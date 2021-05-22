# Tor使い方ガイド
## vpn vs proxy
### VPN
- Virtual Private Networkの略。
- 通信のすべてを暗号化して情報をやりとりする方法。
- 組織の外から組織のリソースにアクセスするのが目的。
- 仮想的にLANとつながっている状態になる。

### proxy
- VPN同様にユーザと訪問先を仲介する。
- 代理でアクセスするため、すべてが暗号化されるわけではない。
- 暗号化の対象はブラウザのみ。

## Torとは
Torとはproxyの1つであり、ネットワークの**接続経路を匿名化して、接続元の特定を困難にする**ソフトウェアである。→通信に複数のノード(中継地点)を経由しているため。

## 発信者特定の仕組み
ここでは、誹謗中傷をしたユーザを特定する流れを簡単に解説する。
1. 掲示板で誹謗中傷されているのを見つける。
2. 誹謗中傷を行なったユーザのIPアドレスを掲示板に開示してもらう。
3. IPアドレスからどのプロバイダの回線を使っているか調べる。
4. プロバイダからIPアドレスを使っている契約情報を聞き出し、誹謗中傷を行なったユーザの住所等を手に入れる。

プロバイダとは: OCN光やSo-net光のように回線(NTTやSo-net等の回線業者によって提供されている)とインターネットをつなげる接続事業者のこと。
### なぜ、Torは発信者を特定しにくくするのか?
Torは3つの中継地点を経由して目的のウェブサイトに到達するようになっており、中継地点ごとにIPアドレスが変わるようになっている。
そのため、発信者が中継地点を通るたびにIPアドレスが変わる。→開示請求をなんども行わなければならない。
特に、海外のサーバに対してプロバイダ開示請求を行なっても、無視されたり開示請求までに時間がかかりログが消失してしまう可能性もある。

## Torブラウザのインストール
1. [Torproject.org](torproject.org)にアクセス
以降の手順を詳しく書く。

### IPアドレスを確かめる。

### torrc編集してtorの設定を変更する



## torの詳しい解説
- ユーザはローカルにSOCKSプロキシ(オニオンプロキシ)立てて、プロキシ経由の通信を行う。
- 接続経路を匿名化するものであり、通信内容の秘匿を保証しているわけではない。
- [tor install guice](https://linuxconfig.org/install-tor-on-ubuntu-18-04-bionic-beaver-linux)
- ubuntuならaptでインストールできる。
- ssコマンドは、ネットワーク通信で利用するソケットについての情報などを出力するコマンド。
- ソケットとは:プログラムとネットワークをつなげる接続口のことを指す場合と、プログラムでネットワーク通信部分を担当する部分という2つの意味がある。

```
ss -nlt #ssコマンドを使ってTorで使用されている9050番ポートのリクエストを確認。
-t, --tcp           display only TCP sockets
-l, --listening     display listening sockets
-n, --numeric       don't resolve service names
```

### Torが使われているか確認

- Torを経由した外部IPアドレスを確認する

```
wget -qO - https://api.ipify.org; echo #外部IPアドレス
torsockes wget -qO - https://api.ipify.org; echo  #Torを経由した外部IPアドレスを確認する。
```
> -qは過程を出力しない、O -はファイルとして出力して結果をSTDOUTに出力する。

### Torを起動,終了

```
source torsocks on  #tor起動
source torsocks off #tor終了
```
> torではなぜ動かないのか？→torがすでに実行されているから。

```
sudo killall tor
```

### torの確実性とは
[Torをつかって個人情報をコントロール](https://qiita.com/syui/items/ebd6734a8102c7906cc1)
- torを起動してもすべてのアプリが自動でTor経由になるわけではない。
- 通常は個々のアプリに対してtorをつかう設定をしなければならない。
- socksプロトコルに対応していないコマンドを使うにはproxychainsという独自のプロキシ経由でコマンドを実行するツールを使う。
- proxychainsを使う際には**一度torsocksをオフ**にする必要がある
- proxychainsの設定ファイルは/etc/proxychains.confにあり、デフォルトはtorの9050番ポートが指定されている。

```
sudo apt install proxychains
proxychains youtube-dl $URL
```

### 外部と通信するコマンドが使う環境変数

```
export http_proxy=socks5://127.0.0.1:9050
export https_proxy=$http_proxy
export ftp_proxy=$http_proxy
export rsync_proxy=$http_proxy
export HTTP_PROXY=$http_proxy
export HTTPS_PROXY=$http_proxy
export FTP_PROXY=$http_proxy
export RSYNC_PROXY=$http_proxy
```


### torrc
- torの場合は/etc/tor/torrc
- tor_Browserの場合は/home/tomita/app/tor_browser/Browser/TorBrowser/Data/Tor/torrc
- UseEntryGuards 0|1 : 1にすると、長期的なエントリーサーバをいくつか選び、それらに固執する。絶えずサーバを変更すると、サーバを所有する敵対者があなたのパスののごく一部を監視する確立が高くなるので望ましい。
- NumEntryGuards: 長期的なエントリーサーバに使う数を指定する。

```
# 14-Eyesと自分の国のノードを通さないようにする。
ExcludeNodes {jp},{us},{gb},{ca},{au},{nz},{dk},{fr},{nl},{no},{de},{be},{it},{es},{il},{sg},{kr},{se},127.0.0.1,SlowServer
# どこの国のノードを出口にしないか
ExcludeExitNodes {jp},{us},{gb},{ca},{au},{nz},{dk},{fr},{nl},{no},{de},{be},{it},{es},{il},{sg},{kr},{se},{bg},{cz},{fi},{hu},{ie},{lv},{lt},{lu},{ro},{se},{ch},{ru},{hk},127.0.0.1
#1の場合は絶対に指定した国のノードに設定しない
StrictNodes 1
# 長期的な回路の個数を指定
NumEntryGuards 4
```
#### 日本のノードだけを使って高速でtorを使用

```
EntryNodes {jp}
MiddleNodes {jp}
ExitNodes {jp}
```


### firefoxでtorを使う
設定→ネットワークの設定→ManulalProxyconfiguration→9050port→SOCKS Hostをlocalhostに
> localhostは自身を表すホスト名。送信元に戻ってくる電子信号であるループバック機能をもち、自らがネットワーク上で提供しているサーバ機能などに自分でアクセスする際などに使われる。
>IPV4では127.0.0.1を表す。

### torへの対策
- IP開示請求は裁判所を介さなくてもできる。→法的拘束力がないので裁判所を介さないと応じられない。
- Firefox17の脆弱性を利用。
- 出口ノードのサーバをハニーポットにする。出口ノードでマルウェアを仕込まれる可能性。
- Tor出口ノードのIPは公開されている。これをブロックすることは可能。
******


## torを使ってサイトを公開する方法
### 公開するサイトを準備

```
mkdir www ; cd www
cat index.html
<html>
    <head>
        <title></title>
        <meta charset="utf-8"/>
    </head>
    <body>
        <a href="https://www.youtube.com/channel/UCesAzmQTTBiY6Yq1vrt2pCw">世界をおおいに盛り上げるためのハッキング霊夢の団</a>
    <body>
</html>
```

```
python -m SimpleHTTPServer #Webサーバを準備
```
- ブラウザを開き、127.0.0.1:8080を検索してサイトが表示できることを確認。
### torを使ってサイトを公開
- onion_serviceディレクトリの作成。
- onion_service.torrcを作成。

```
#onios_service_torrc
HiddenServiceDir /home/tomita/tmp/www/onion_service
HiddenServicePort 80 127.0.0.1:8000 #ローカルループバックアドレス。このPC上の8000番portに80から来た通信を流す。
```
- onion_serviceのパーミションを700に指定
- 動いているtorプロセスをkillする

```
chmod 700 ../onion_service
sudo killall tor
```

### torrcを指定して、torを起動
- pythonによってWebサイトが事前に立ち上げられている必要がある。

```
cd tmp/www
tor -f ../onion_service.torrc
```
- ../onion_service/hostnameにダークウェブ上でのURLが作成されているのでこれをtorブラウザで検索するとサイトが確認できる。



