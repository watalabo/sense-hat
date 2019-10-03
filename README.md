# RaspberryPi 3 Model B+でSenseHatが動かなかったときのメモ

学部の授業でRaspberryPi 3 Model B+とSenseHatを組み合わせて使っていたのだが、SenseHatが動かずドはまりして解決したのでメモしておく。

## ことの始まり

まず、AmazonでSense HATが何種類か売っていて、深く考えず一番安いやつを買っちゃったのがそもそもの不幸の始まりだった。
Amazonをみて分かる通り、Sense HATの売り手は何種類か存在する([Amazon.co.jpのSenseHat検索結果](https://www.amazon.co.jp/s?k=SenseHat&__mk_ja_JP=%E3%82%AB%E3%82%BF%E3%82%AB%E3%83%8A&ref=nb_sb_noss_2))。
僕は
[一番安いやつ](https://www.amazon.co.jp/dp/B014N3RAUC)
を買ってしまった。

レビューを読めば分かるのだが、思いっきり同じ症状で困っている人がいた。

> LEDが全部つくので通電はしているようだが、Pythonで動かしても「モジュールが見つからない」とエラーになるだけで、全く機能しない。ちなみに、Sense Hatを外して同じコードを動かしてもまったく同じメッセージが出る。つまり、このSense Hatが壊れているということ。返品しました。でもこのセンサーモジュールでやりたいことがあるので返金後にもう一度買うと思います。こんどは頼みますよ。

現象は
* 起動時にSense HATが光りっぱなしになってしまう(本当は一度光った後消えるのが正しい)
* ```sense = SenseHat()```すると```OSError: Cannot detect RPi-Sense FB device```

などなど。
とにかくRaspberry Piにつなげば動くはずのSense HATがちゃんと動かない。
環境はRaspbian BusterでもStretchでも同じ。

## 原因

いろいろ調べた結果、SenseHATのfirmwareに問題があるらしいということが分かった。

## 解決方法

SenseHATのfirmwareを焼き直したら直った。
参考にしたのは↓
* [Sense HAT - Raspberry Pi Documentation](https://www.raspberrypi.org/documentation/hardware/sense-hat/)

### /boot/config.txt

ファームを焼き直すには

/boot/config.txtに以下の2行が必要らしい。

```
dtparam=i2c_arm=on
dtparam=i2c_vc=on
```

```dtparam=i2c_arm=on```はもう書いてあったので```dtparam=i2c_vc=on```を追加した。

そしてreboot。

### SenseHATのfirmwareの入手とコンパイル

githubのRaspberryPiのところから入手できる。
* [raspberrypi/hats](https://github.com/raspberrypi/hats/)

僕の場合は
```
wget https://github.com/raspberrypi/hats/archive/master.zip
```
でダウンロード後解凍。

eepromのソースコードらしきものをダウンロード。

```
wget https://github.com/raspberrypi/rpi-sense/raw/master/eeprom/eeprom_settings.txt -O sense_eeprom.txt
```

eepromをmake。

```
./eepmake sense_eeprom.txt sense.eep /boot/overlays/rpi-sense.dtbo
```

### SenseHATへの書き込み

RapsberryPiに刺しておいて以下のコマンドを打てばOK。

書き込み保護の解除。
```
i2cset -y -f 1 0x46 0xf3 1
```

書き込み。
```
sudo ./eepflash.sh -f=sense.eep -t=24c32 -w
```

書き込み保護の設定　.
```
i2cset -y -f 1 0x46 0xf3 0
```

あとはSenseHATの再起動をしなければならないのでシャットダウン後再び起動で動いた。

### 教訓

安いのには理由がある。
SenseHAT買うなら[I-O DATA製](https://www.amazon.co.jp/dp/B01N3AARYT)の方が楽。
