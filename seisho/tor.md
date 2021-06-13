# Tor使い方ガイド
## 元動画
この記事は、[ネット匿名化 Torの仕組みと使い方・インストール・高速化・reCAPTUREの回避](https://www.youtube.com/watch?v=RPNWQUx7xrE)に加筆したものである。
## vpn vs proxy
### VPN
- Virtual Private Networkの略。
- 自分の外部ipアドレスがvpnサーバーのipアドレスに変わる。
- LAN外から仮想的にLAN内にいるのと同じ状態にすることができる。
- ex: リモートワークをするために、自宅から会社のLAN内に入りssh等を使って会社のパソコンにアクセスする。
- また、フリーwifiを使う際に通信を盗聴されないようにvpnを使うのが推奨されている。
vpnを使わない通信であればLAN内で通信が直接行われるので暗号化されていない通信は盗聴することができるが、vpnを使った通信を盗聴した場合にはvpnとの通信していることしかわからない。

### proxy
- 代理でアクセスすることを目的としているため、自分のipアドレスを隠して通信ができる。

## Torとは
Torとはproxyの1つであり、ネットワークの**接続経路を匿名化して、接続元の特定を困難にする**ソフトウェアである。

## 発信者特定の仕組み
Torの匿名性について説明するために、まず発信者特定の流れを解説する。

例: 誹謗中傷をしたユーザを訴える場合

1. 掲示板等で誹謗中傷されているのを見つける。
2. 誹謗中傷を行なったユーザのIPアドレスを掲示板に開示してもらう。
3. IPアドレスからどのプロバイダの回線を使っているか調べる。
4. プロバイダから該当するIPアドレスを使っている契約者の契約情報を聞き出し、誹謗中傷を行なったユーザを特定する。**誰が書き込みしたか判明!**

![手順1](/home/tomita/article/picture/hibou.png)
![手順2](/home/tomita/article/picture/ipget.png)
![手順3](/home/tomita/article/picture/provider.png)
![手順4](/home/tomita/article/picture/iptoprovider.png)
> プロバイダとは: OCN光やSo-net光のように回線(NTTやSo-net等の回線業者によって提供されている)とインターネットをつなげる接続事業者のこと。
### なぜ、Torは発信者を特定を難しくするのか?
Torは3つの中継地点を経由して目的のウェブサイトに到達するように設計されており、中継地点ごとにIPアドレスが変わるようになっている。
そのため、発信者が中継地点を通るたびにIPアドレスが変わるようになっている。そのため、開示請求を行う際に全ての中継地点において開示請求を行う必要がある。
また、Torの中継地点は世界中に存在するため、開示請求のお願いをしても無視されたり、時間がかかりログが消失してしまうといったことが起きる可能性がある。
![tor](/home/tomita/article/picture/tor.png)
******


## Torブラウザ
### Torブラウザのダウンロード
Torブラウザとは**Torを組み込んでカスタマイズされたFirefoxブラウザ**である。

1. [Torproject.org](https://www.torproject.org/)にアクセス
2. [Download Tor Browser](https://www.torproject.org/download/)をクリック
3. 自分のOSにあったTor Browserをクリックしてダウンロードする。デフォルトでは英語版がダウンロードされるため、[Download in another language or platform](https://www.torproject.org/download/languages/)をクリックして日本語の列を探せば日本語版がダウンロードできる。
![Tor対応OS](/home/tomita/article/picture/torOS.png)
4. Tor Browserのインストーラーがダウンロードされているのでこれを使ってTor Browserをインストールする。
5. インストールが完了したらTorが起動する。

### IPアドレスを確かめる。
- [ipinfo.io/json](https://ipinfo.io/json)にアクセスすると現在のIPアドレスやアクセスしている国や地域が確認できる。
- 鍵マークをクリックすると、どこのリレーノードを通ってアクセスしているかを表示できる。
![Torリレーノード](/home/tomita/article/picture/torcircuit.png)

### wiresharkを使ってパケットを確認する。
私の場合は、This browser→46.166.161.7→185.117.82.6→176.123.7.102→目的のWebサイトといったようにアクセスが行われている。

そこで、wiresharkを使ってエントリーノード(最初のTorノード)の通信のパケットを観察してみる。
[wiresharkについてはこちら](tmp)
1. wiresharkを起動して、パケットキャプチャを開始。
2. Torブラウザで開いているipinfo.ioのページを更新する。
3. wiresharkによるパケットキャプチャを止める。
4. フィルター(検索欄)にip.addr == 最初のtorノードのipアドレスと入力して検索する。
リレーノードの入り口との通信を見ることができました。
![wiresharkによるパケットキャプチャ](/home/tomita/article/picture/wireshak1.png)
> Torのクライアントやノード間は暗号化されているから盗聴されないが、Exitノード(リレーノードの終わりのノードで今回でいうと176.123.7.102)は暗号化されていない通信の場合は盗聴の恐れがあるので注意が必要。
### torrc編集してtorの設定を変更する
- Torブラウザのデフォルト設定では海外のノードを経由しているため、通信速度がかなりおそい。
- torの設定を編集するにはtorrcというファイルを編集することで設定が変更できる。
- tor_Browserのtorrcはtor_browser/Browser/TorBrowser/Data/Tor/torrcにある。

#### リレーサーキットの日本国内に限定して通信を速くする
torrcに以下を追加する。

```
EntryNodes {jp}
MiddleNodes {jp}
ExitNodes {jp}
```
すると、リレーサーキットがすべて日本国内になり、Torのデメリットである通信の遅さがある程度解消される。
しかし、これはTorとしては脆弱な設定であるため、ブラウジングにのみ留めるべき。

#### 安全な国のノードだけを通るようにする設定
政府によるトラッキングを回避するには、14-Eyesの監視から逃れる必要がある。
これには、自分の国と14-Eyesのノードをリレーサーキットに含まない設定をする。

```
# 14-Eyesと自分の国のノードを通さないようにする。
ExcludeNodes {jp},{us},{gb},{ca},{au},{nz},{dk},{fr},{nl},{no},{de},{be},{it},{es},{il},{sg},{kr},{se},127.0.0.1,SlowServer
# どこの国のノードを出口にしないか
ExcludeExitNodes {jp},{us},{gb},{ca},{au},{nz},{dk},{fr},{nl},{no},{de},{be},{it},{es},{il},{sg},{kr},{se},{bg},{cz},{fi},{hu},{ie},{lv},{lt},{lu},{ro},{se},{ch},{ru},{hk},127.0.0.1
#1の場合は絶対に指定した国のノードに通らない
StrictNodes 1
```
安全な設定であるが通るサーバがすべて海外のサーバとなるため、ブラウジングはすこぶる遅い。
******

### torのExitNodeとRecaputureの回避
TorのExitNodeとはTorの最後の経由サーバであり、サイトと通信する部分である。
TorのExit Nodeの一覧は公開されており、[こちら](https://check.torproject.org/exit-addresses)から見ることができる。

```shell
# コマンドラインからExit Node一覧を見る
curl -s https://check.torproject.org/exit-addresses | grep ExitAddress | cut -d ' ' -f2
51.195.166.197
31.220.0.10
96.66.15.152
# Exit Nodeの数を確認
curl -s https://check.torproject.org/exit-addresses | grep ExitAddress | wc -l
```
そのため、TorのExit Nodeは一部のサービスではブロックされていたり、recapture(私はロボットではありません的なうざいやつ)を要求される場合がある。
![recapture](/home/tomita/article/picture/recapture.png)
これを回避するには、ブラウジングに使う検索エンジンにgoogleではなく、yahooを使えば検索結果は同じまま、reCAPTUREを回避できる。


## Torを使ってサイトを公開する方法
### 元動画
[ハッキング霊夢配信](https://www.youtube.com/watch?v=D5kjxqO0c7E&list=PL6Cp7Rl8AiCWnfp0Epg_5sY6eO-dogWkg)のTorを使ったサイト公開の部分を抜粋して加筆した。

### 公開するサイトを準備
- ディレクトリを作成

```
mkdir html ; cd html
```
- index.htmlを作成する。サンプルなので最低限のindex.htmlを作成した。

```html:index.html
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
- pythonのSimpleHTTPServerという機能を使ってwebサーバを立ち上げる。
- python3の場合はこの機能はhttp.serverに統合されているため、python3の人はこちらを使ってください。

```shell
python -m SimpleHTTPServer #python2系
python3 -m http.server #python3系
```
ブラウザを開きサイトが表示できるか確認する。
- python2系の場合は、127.0.0.1:8080を検索してサイトが表示できることを確認。
- python3系の場合は、0.0.0.0:8000を検索してサイトが表示できることを確認できる。

### torを使ってサイトを公開
#### torをインストール
Torブラウザをインストールしただけでは、tor自体のインストールはできていないので、aptからインストールする。

```shell
sudo apt install tor
```
#### onion_serviceディレクトリの作成。

```shell
mkdir onion_service
chmod 700 onion_service #ディレクトリを自分以外から見えなくする。
```

#### onion_service.torrcを作成。
onion_service.torrcファイルを作成し、以下のように記述する。

```
# python2系
HiddenServiceDir /home/tomita/tmp/onion_service
HiddenServicePort 80 127.0.0.1:8000 #ローカルループバックアドレス。このPC上の8000番portに80から来た通信を流す。
```

```
#python3系
HiddenServiceDir /home/tomita/tmp/onion_service
HiddenServicePort 80 0.0.0.0:8000
```

#### 動いているtorプロセスをkillする
torが既に起動されているとエラーが起きる場合があるので念の為torプロセスをすべてkillしておく。

```
sudo killall tor
```

### torrcを指定して、torを起動しWeb siteを公開する。

```
cd tmp/html
python -m SimpleHTTPServer #python2系
python3 -m http.server #python3系

tor -f ../onion_service.torrc
```
- ../onion_service/hostnameにダークウェブ上でのURLが作成されている。
- Troブラウザを立ち上げてそのURLを入力するとサイトにアクセスできる。
![Website](/home/tomita/article/picture/onionsite.png)
******


## 「Tor over VPN」と「VPN over Tor」
Torと併用されるものとしてVPNがある。ここでは、TorとVPNを併用してより安全に通信する方法を紹介する。

### Tor over VPNとは
![Tor over VPN](/home/tomita/article/picture/ToroverVPN.PNG)
- VPNを接続したパソコンでTorを使う。
- TorのEntry guard(入り口)にIPアドレスを知られない。
- ISP(回線業者)にTorを使用していることがばれない。
- VPNを接続した状態でTorを使うことでTor over VPNは実現できる。

### VPN over Torとは
![VPN over Tor](/home/tomita/article/picture/VPNoverTor.PNG)
- TorのExitNodeとwebサイトの間をVPNで中継する。
- TorのExitNodeはFBI等が運営して監視しているものもあるが、ExitNodeに残るログがVPNとの通信となる。そのため、発信者特定がより難しくなる。
- 仮想マシン内でTorによる通信を行い、ホストOS側でVPNを使うことでVPN over Torは実現できる。
### Whonixを使ってVPN over Torを実現する
Whonixは仮想マシン内で使用するOSで、仮想マシン内の通信をすべてTorによって匿名化することができる。
#### 使用した環境
- ホストOS: Windows10
- 仮想OS: Ubuntu20.04
- VirtualBox6.1
#### Whonixのインストール
- [Whonix Download Page](https://www.whonix.org/wiki/Download)からWhonixのダウンロードする。
- VirtualBoxを立ち上げ、新規→仮想アプライアンスのインポート→からダウンロードしたWhonixをインポートする。

インポートが成功するとWhonix GatewayとWhonix Workstationの2つの仮想OSが作成される。
これらを接続
> Whonix Gateway:Torによるインターネットの接続環境を提供する。

> Whonix Workstation: Gatewayによって構築されたネットワークのみを経由してインターネットに接続する。


#### Ubuntu20.04側の設定
##### 内部ネットワークを設定する
Virtualboxの設定からネットワークの割当を内部ネットワークに変更し、Whonixを指定する。
![内部ネットワーク](/home/tomita/article/picture/whonixsetting.png)

##### Ubuntuのネットワーク設定
仮想マシンを立ち上げてUbuntuのネットワーク設定を変更する。
![設定](/home/tomita/article/picture/setting.png)

![ネットワークの設定](/home/tomita/article/picture/networkset.png)
|             |             |             |             |
|-------------|-------------|-------------|-------------|
|Address      |Netmask      |Gateway      |DNS          |
|10.152.152.50|255.255.192.0|10.152.152.10|10.152.152.10|

#### whonix Gatewayのテスト
仮想マシン内で現在の外部IPアドレスを確認する。

```shell
curl ipinfo.io

{
  "ip": "109.70.100.42",
  "hostname": "tor-exit-anonymizer.appliedprivacy.net",
  "city": "Vienna",
  "region": "Vienna",
  "country": "AT",
  "loc": "48.2085,16.3721",
  "org": "AS208323 Foundation for Applied Privacy",
  "postal": "1010",
  "timezone": "Europe/Vienna",
  "readme": "https://ipinfo.io/missingauth"
}
```
現在の自分の居場所と比べて接続先が代わっていればwhonix Gatewayは正しく動いている。

#### VPN over Torを試す
1. VirtualboxからWhonix Gatewayを起動する。デフォルトユーザはuser,パスワードはchangemeである。
2. Ubuntuを起動する。
3. VPNをホスト側で起動する。

これを見ると接続先が変わっており、Torによる接続ができていることが確認できた。

この状態でホストOS側でVPNを起動すれば、Exitノードと通信するのがVPNサーバとなる。
******

## onionshare
**Torを使ってファイルの送受信やサイトの公開ができるGUIアプリケーション**である。機能としては、
- ファイルの送受信
- 匿名でのチャットサーバの立ち上げ
- サイトの公開
がある。インストールはsnapから行える。


```shell
sudo snap install onionshare
```

### ファイルの送信
onionshareを立ち上げるとスタート画面が表示される。
![onionshare](/home/tomita/article/picture/onionshare.png)
- 送信側は、スタート画面からShareFileを選び、共有したいファイルを選択してstartをクリックすると、.onionのurlを生成できる。
- 受信側は先程のURLを受け取り、Torブラウザやonionshare等を使ってURLにアクセスする(.onionは通常のブラウザからはアクセスできないため)。そしてファイルをダウンロードする。

### Chat Anonymous
- Tor上にチャットサーバを立てて、匿名でチャットできる。
- スタート画面でChat Anonymouslyを選択して"Start chat server"を押すとurlが生成されるので、参加者にURLを共有してチャットを行う。
![anonymouschat](/home/tomita/article/picture/anonymouschat.png)

### サイトの公開
- スタート画面からHost a Websiteを選択して、共有したいhtmlファイルを選択して共有をスタートするとURLが生成されてTorネットワーク上にサイトを公開できる。
******


## コマンドラインでTorを使う
### Torのインストール
Torブラウザをインストールしただけでは、tor自体のインストールはできていないので、aptからインストールする。

```
sudo apt install tor
```
### ipアドレスの確認

source torsocks onを実行するとコマンドラインをtorモードにすることができる。まずは、ipアドレスを確認してみる。

```shell
source torsocks on #コマンドラインをTorモードにする
Tor mode activated. Every command will be torified for this shell.
curl ipinfo.io
{
  "ip": "91.208.52.61",
  "city": "Almelo",
  "region": "Overijssel",
  "country": "NL",
  "loc": "52.3567,6.6625",
  "org": "AS50673 Serverius",
  "postal": "7607",
  "timezone": "Europe/Amsterdam",
  "readme": "https://ipinfo.io/missingauth"
}

source torsocks off
```

### 他のアプリケーションとの組み合わせ
Torを起動してもすべてのアプリが自動でTor経由になるわけではない。
bashrcに以下を追加することでこれらの環境変数を使っているアプリケーションに関してはtorを使った通信にすることができる。

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

また、youtube-dl等のアプリケーションはsocksプロトコルに対応していないため、以下のように実行する必要がある。

```shell
sudo torsocks -i youtube-dl $targetURL
```
