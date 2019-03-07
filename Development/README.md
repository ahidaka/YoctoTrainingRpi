# YoctoTrainingRpi / Development

## 環境

* ホスト

    Ubuntu 18.04 LTS / PC (高速な実マシンを推奨)

* ターゲット

    Raspberry Pi 3

* メディア関連

    Micro SD card + Reader / Writer

## 準備

ビルドする前の準備です。全てホストPCで実行します。

### ツールのインストール

```sh
$ sudo apt install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath socat libsdl1.2-dev xterm
```

### Python 2.7のインストールと設定
bitbakeがPython 2.7 に依存し、Ubuntu 18.04では標準利用不可のため必要な処理。以下を管理者権限で実行します。

```sh
# apt install python2.7
# ln -sf /usr/bin/python2.7 /usr/bin/python
# ln -sf /usr/bin/python2.7 /usr/bin/python2
# dpkg-reconfigure dash  #■注意：質問にNで答えます
```

### ディレクトリの作成
以下、開発用トップディレクトリを「~/rp3」とする例で説明。

```sh
$ cd
$ mkdir rp3
$ cd rp3
$ mkdir build downloads sstate sstate-cache tmp yocto
```

### ダウンロード
Raspberry Pi用 Yocto Projectをダウンロードして、ソースコード展開します。
thrud=Yocto 最新安定版コードを確定 Checkoutしています。

```sh
$ cd yocto
$ git clone -b thud git://git.yoctoproject.org/poky.git poky
$ cd poky
$ git checkout thud
$ cd ..
$ git clone -b thud git://git.openembedded.org/meta-openembedded
$ git clone -b thud git://git.yoctoproject.org/meta-raspberrypi
```

### 環境変数設定

```sh
$ cd ~/rp3
$ source yocto/poky/oe-init-build-env build/rpi3/
```

sourceコマンド実行後、ディレクトリが　~/rp3/build/rpi3　となります。

### confファイル設定

~/rp3/build/rpi3/conf にある bblayers.conf と local.conf をエディタで修正します。以下は nano エディタで編集する場合の操作例。

#### bblayers.conf の編集

```yaml
# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  <strong> ${TOPDIR}/../../yocto/poky/meta \ </strong>
  ${TOPDIR}/../../yocto/poky/meta-poky \
  ${TOPDIR}/../../yocto/poky/meta-yocto-bsp \
  ${TOPDIR}/../../yocto/meta-openembedded/meta-oe \
  ${TOPDIR}/../../yocto/meta-openembedded/meta-networking \
  ${TOPDIR}/../../yocto/meta-openembedded/meta-python \
  ${TOPDIR}/../../yocto/meta-raspberrypi \
  "
```


    # POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
    # changes incompatibly
    POKY_BBLAYERS_CONF_VERSION = "2"

    BBPATH = "${TOPDIR}"
    BBFILES ?= ""

    BBLAYERS ?= " \
      ** ${TOPDIR}/../../yocto/poky/meta \ **
      ${TOPDIR}/../../yocto/poky/meta-poky \
      ${TOPDIR}/../../yocto/poky/meta-yocto-bsp \
      ${TOPDIR}/../../yocto/meta-openembedded/meta-oe \
      ${TOPDIR}/../../yocto/meta-openembedded/meta-networking \
      ${TOPDIR}/../../yocto/meta-openembedded/meta-python \
      ${TOPDIR}/../../yocto/meta-raspberrypi \
      "



```sh

```



## 開発準備



