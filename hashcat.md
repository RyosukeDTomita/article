# hashcatを使う
## インストール

```
sudo apt install hashcat
```

## ベンチマーク

```
hashcat -m 0 -b #-mでhash-type0番(md5)を指定。-bはベンチマークモード
> No devices found/left error
hashcat -m 0 -b --force #-b ベンチマークモード、-m 0 ハッシュタイプを指定する。0はMD5
```
## hashについて
- ハッシュ関数: 任意のデータから、別の値(多くの場合は固定長の)値を得るための関数のこと。
- 検索の高速化やデータ比較処理の高速化、改ざんの検出に使われる。

### MD5のhash値を作成
[コマンドライン上でMD5のハッシュを取得](https://qiita.com/aki3061/items/32f61e33a795d2f5d8c7)

```
echo -n "abcd" | md5sum #-nオプションで改行コードを出力しないようにしている。
```

## MD5形式をhashcatを使って解析

```
hashcat -m 0 e2fc714c4727ee9395f324cd2e7f331f -a 3 ?l?l?l?l --force # -a 3 アタックモード3番。ブルートフォースアタック
```
### アタックモード

| |                      |
|-|----------------------|
|0|辞書攻撃              |
|1|Combination           |
|3|Brute-force           |
|6|Hybrid Wordlist + Mask|
|7|Hybrid Mask + Wordlist|


### マスク表

|  |                           |
|--|---------------------------|
|?l|abcdefghijklmnopqrstuvwxyz |
|?u|ABCDEFGHIJKLMNOPQRSTUVWXYZ |
|?d|0123456789                 |
|?h|0123456789abcdef           |
|?H|0123456789ABCDEF           |
|?s|各種記号                   |
|?a|?l?u?d?s(l,u,d,sのパターン)|
|?b|0x00 - 0xff                |

> 各種記号!"#$%&'()+*,-./:;<=>?@[]^_`{|}~*

### 辞書ファイル
- [rockyou.txt](https://www.google.com/search?channel=fs&client=ubuntu&q=rockyou.txt)
- RockYoutと呼ばれるソシャゲ会社から流出した3200万件の平文パスワードリスト。

```
hashcat -m 0 d41e98d1eafa6d6011d3a70f1a5b92f0 -a 0 ~/app/rockyou.txt  --force #-a 0は辞書攻撃モード
```
