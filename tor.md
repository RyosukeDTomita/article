# Tor使い方ガイド
## 元動画
この記事は、[ネット匿名化 Torの仕組みと使い方・インストール・高速化・reCAPTUREの回避](https://www.youtube.com/watch?v=RPNWQUx7xrE)に加筆したものです。
## vpn vs proxy
### VPN
- Virtual Private Networkの略。
- 自分の外から見えるipアドレスがvpnサーバーのipアドレスに変わる。
- LAN外から仮想的にLANとつながっている状態にすることができる。
- リモートワークをするために、自宅から会社のLAN内に入りssh等を使って会社のパソコンにアクセスするなどに使われる。
- また、フリーwifiを使う際に通信を盗聴されないようにvpnを使うのが推奨されている。
vpnを使わない通信であればLAN内で通信が直接行われるので暗号化されていない通信は盗聴することができるが、vpnを使った通信を盗聴した場合にはvpnとの通信しか盗聴することができない。

### proxy
- 代理でアクセスすることを目的としているため、自分のipアドレスを隠して通信ができる。

## Torとは
Torとはproxyの1つであり、ネットワークの**接続経路を匿名化して、接続元の特定を困難にする**ソフトウェアである。通信に複数のノード(中継地点)を経由しているため。

## 発信者特定の仕組み
ここでは、誹謗中傷をしたユーザを特定する流れを簡単に解説する。
1. 掲示板で誹謗中傷されているのを見つける。
2. 誹謗中傷を行なったユーザのIPアドレスを掲示板に開示してもらう。
3. IPアドレスからどのプロバイダの回線を使っているか調べる。
4. プロバイダからIPアドレスを使っている契約情報を聞き出し、誹謗中傷を行なったユーザの住所等を手に入れる。**誰が書き込みしたか判明**

![手順1](/home/tomita/article/picture/hibou.png)
![手順2](/home/tomita/article/picture/ipget.png)
![手順3](/home/tomita/article/picture/provider.png)
![手順4](/home/tomita/article/picture/iptoprovider.png)
> プロバイダとは: OCN光やSo-net光のように回線(NTTやSo-net等の回線業者によって提供されている)とインターネットをつなげる接続事業者のこと。
### なぜ、Torは発信者を特定しにくくするのか?
Torは3つの中継地点を経由して目的のウェブサイトに到達するようになっており、中継地点ごとにIPアドレスが変わるようになっている。
そのため、発信者が中継地点を通るたびにIPアドレスが変わる。→開示請求をなんども行わなければならない。
特に、海外のサーバに対してプロバイダ開示請求を行なっても、無視されたり開示請求までに時間がかかりログが消失してしまう可能性もある。
![tor](/home/tomita/article/picture/tor.png)
******


## Torブラウザのダウンロード
Torブラウザとは**Torを組み込んでカスタマイズされたFirefoxブラウザ**である。

1. [Torproject.org](https://www.torproject.org/)にアクセス
2. [Download Tor Browser](https://www.torproject.org/download/)をクリック
3. 自分のOSにあったTor Browserをクリックしてダウンロードする。デフォルトでは英語版がダウンロードされるため、[Download in another language or platform](https://www.torproject.org/download/languages/)をクリックして日本語の列を探せば日本語版がダウンロードできる。
![Tor対応OS](/home/tomita/article/picture/torOS.png)
4. Tor Browserのインストーラーがダウンロードされているので起動してインストールを選択する。
5. インストールが完了したら起動する。

### IPアドレスを確かめる。
- [ipinfo.io/json](https://ipinfo.io/json)にアクセスすると現在のIPアドレスやアクセスしている国や地域が確認できる。
- 鍵マークをクリックすると、どこのリレーノードを通ってアクセスしているかを表示できる。
![Torリレーノード](/home/tomita/article/torcircuit.png)
#### wiresharkを使ってパケットを確認する。
私の場合は、This browser、46.166.161.79、185.117.82.68、176.123.7.102、ipinfo.ioという順のリレーノードとなっています。
wiresharkを使うと自分のパソコン(local ip)と最初に接続しているリレーノードの通信のパケットを見てみます。
[wiresharkについてはこちら](tmp)
1. wiresharkを起動して、パケットキャプチャを開始。
2. Torブラウザで開いているipinfo.ioのページを更新する。
3. wiresharkによるパケットキャプチャを止める。
4. フィルター(検索欄)にip.addr == 46.166.161.79と入力して検索する。
リレーノードの入り口との通信を見ることができました。
[wiresharkによるパケットキャプチャ](/home/tomita/article/picture/wireshak1.png)
Torのクライアントやノード間は暗号化されているから盗聴されないが、Exitノード(リレーノードの終わりのノードで今回でいうと176.123.7.102)は暗号化されていない通信の場合は盗聴の恐れがあるので注意が必要。
### torrc編集してtorの設定を変更する
- Torブラウザのデフォルト設定ではかなり遅い。
- torの設定を編集するにはtorrcというファイルを編集することで設定ができる。
- tor_Browserの場合はtor_browser/Browser/TorBrowser/Data/Tor/torrcにある。

#### リレーサーキットのノードをすべて日本国内のものにする。
torrcに以下を追加します。

```
EntryNodes {jp}
MiddleNodes {jp}
ExitNodes {jp}
```
すると、リレーサーキットがすべて日本国内になり、快適なブラウジングができるようになる。
しかし、これはtorとしては脆弱な設定であり、開示請求ができてしまうのが注意点。

#### 安全な国のノードだけを通るようにする設定
あなたが、エドワード・スノーデンのように世界レベルの内部告発をする予定があるなら、14-Eyesの監視から逃れる必要がある。
そのため、自分の国と14-Eyesのノードをリレーサーキットに含まない設定がこちら。

```
# 14-Eyesと自分の国のノードを通さないようにする。
ExcludeNodes {jp},{us},{gb},{ca},{au},{nz},{dk},{fr},{nl},{no},{de},{be},{it},{es},{il},{sg},{kr},{se},127.0.0.1,SlowServer
# どこの国のノードを出口にしないか
ExcludeExitNodes {jp},{us},{gb},{ca},{au},{nz},{dk},{fr},{nl},{no},{de},{be},{it},{es},{il},{sg},{kr},{se},{bg},{cz},{fi},{hu},{ie},{lv},{lt},{lu},{ro},{se},{ch},{ru},{hk},127.0.0.1
#1の場合は絶対に指定した国のノードに設定しない
StrictNodes 1
```
ただし、ブラウジングはすこぶる遅い。
******

### torのExitNodeの一覧
TorのExit Nodeの一覧は[こちら](https://check.torproject.org/exit-addresses)

```shell
# コマンドラインからExit Node一覧を見る
curl https://check.torproject.org/exit-addresses | grep ExitAddress

ExitAddress 51.195.166.197 2021-05-23 09:33:14
ExitAddress 31.220.0.10 2021-05-23 07:22:07
ExitAddress 96.66.15.152 2021-05-23 09:23:11
# Exit Nodeの数を確認
curl -s https://check.torproject.org/exit-addresses | grep ExitAddress | wc -l
```
### reCAPTURE画面を回避する。
TorのExit NodeのIPアドレスは公開されていて一部のサービスではブロックされている。
![recapture](/home/tomita/article/picture/recapture.png)
- ブラウジングに使う検索エンジンにgoogleではなく、yahooを使えば検索結果は同じまま、reCAPTUREを回避できる。


## torを使ってサイトを公開する方法
### 元動画
[ハッキング霊夢配信](https://www.youtube.com/watch?v=D5kjxqO0c7E&list=PL6Cp7Rl8AiCWnfp0Epg_5sY6eO-dogWkg)の一部分の内容を記事化しました。
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
- python3の場合はhttp.serverに統合されている。

```shell
python -m SimpleHTTPServer #python2系
python3 -m http.server #python3系
```
ブラウザを開きサイトが表示できるか確認する。
- python2系の場合は、127.0.0.1:8080を検索してサイトが表示できることを確認。
- python3系の場合は、http://0.0.0.0:8000/を検索してサイトが表示できることを確認できる。

### torを使ってサイトを公開
#### torをインストール

```shell
sudo apt install tor
```
#### onion_serviceディレクトリの作成。

```shell
mkdir onion_service
chmod 700 onion_service #ディレクトリを自分以外から見えなくする。
```

#### onion_service.torrcを作成。

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
torが既に起動されているとエラーが起きる場合があるのでtorプロセスをすべてkillする。

```
sudo killall tor
```

### torrcを指定して、torを起動
- pythonによってWebサイトが事前に立ち上げられている必要がある。

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
### Tor over VPN
![Tor over VPN](/home/tomita/article/picture/ToroverVPN.PNG)
- VPNを接続したパソコンでTorを使う。
- TorのEntry guard(入り口)にIPアドレスを知られない。
- ISP(回線業者)にTorを使用していることがばれない。
- VPNサービスを使用した機器でTorを使うことでTor over VPNは実現できる。

### VPN over Tor
![VPN over Tor](/home/tomita/article/picture/VPNoverTor.PNG)
- VPN接続する際のIPアドレスがTorの出口ノード
- 身元の特定はより難しくなる。
- Whonixを使ったり、TailsやParrotのような
### Whonixを使う
Whonixは仮想マシン内で使用するOSで、仮想マシン内の通信をすべてTorによって匿名化することができる。
#### メリット
Torを介した通信のみが許可されるので、ソフトやアプリケーションごとにTorを使うように設定しなくてよい。
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
ネットワークの割当を内部ネットワークに変更し、Whonixを指定する。

##### Ubuntuのネットワーク設定
設定を開いてネットワークを以下のように設定する。
![設定](/home/tomita/article/picture/setting.png)

|             |             |             |             |
|-------------|-------------|-------------|-------------|
|Address      |Netmask      |Gateway      |DNS          |
|10.152.152.50|255.255.192.0|10.152.152.10|10.152.152.10|
![ネットワークの設定](/home/tomita/article/picture/networksetting.png)

#### VPN over Torの接続テスト
1. Whonix Gatewayを起動する。デフォルトユーザはuser,パスワードはchangemeである。
2. Ubuntuを起動してIPアドレスを調べる。
3. VPNをホスト側で起動する。

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
これを見ると接続先が変わっており、Torによる接続ができていることが確認できた。

この状態でVPNを起動すれば、Exitノードと通信するのがVPNサーバとなる。今回はテストなので、Firefoxの拡張機能である、[CyberGhost VPN Free Proxy](https://addons.mozilla.org/ja/firefox/addon/cyberghost-vpn-free-proxy/)を使う。
CyberGhost VPNはツールバーから簡単に使用できるがブラウジングにしか利用できない点に注意する。
[CyberGhost](/home/tomita/article/picture/cyberghost.png)

