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

```sh
$ cd conf
$ nano bblayers.conf 
$ nano local.conf 
```

#### bblayers.conf の編集

編集後の内容

```yaml
# POKY_BBLAYERS_CONF_VERSION is increased each time build/conf/bblayers.conf
# changes incompatibly
POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  ${TOPDIR}/../../yocto/poky/meta \
  ${TOPDIR}/../../yocto/poky/meta-poky \
  ${TOPDIR}/../../yocto/poky/meta-yocto-bsp \
  ${TOPDIR}/../../yocto/meta-openembedded/meta-oe \
  ${TOPDIR}/../../yocto/meta-openembedded/meta-networking \
  ${TOPDIR}/../../yocto/meta-openembedded/meta-python \
  ${TOPDIR}/../../yocto/meta-raspberrypi \
  "
```
#### local.conf の編集

編集後の内容

```yaml
MACHINE = "raspberrypi3"
DISTRO ?= "poky"
PACKAGE_CLASSES ?= "package_rpm"
EXTRA_IMAGE_FEATURES ?= "debug-tweaks ssh-server-openssh"
USER_CLASSES ?= "buildstats image-mklibs image-prelink"
PATCHRESOLVE = "noop"
CONF_VERSION = "1"

SSTATE_DIR ?= "${TOPDIR}/../../sstate-cache"
TMPDIR ?= "${TOPDIR}/../../tmp"
DL_DIR ?= "${TOPDIR}/../../downloads"

PACKAGECONFIG_append_pn-qemu-native = " sdl"
PACKAGECONFIG_append_pn-nativesdk-qemu = " sdl"
BB_DISKMON_DIRS ??= "\
    STOPTASKS,${TMPDIR},1G,100K \
    STOPTASKS,${DL_DIR},1G,100K \
    STOPTASKS,${SSTATE_DIR},1G,100K \
    STOPTASKS,/tmp,100M,100K \
    ABORT,${TMPDIR},100M,1K \
    ABORT,${DL_DIR},100M,1K \
    ABORT,${SSTATE_DIR},100M,1K \
    ABORT,/tmp,10M,1K"

#SDKMACHINE ?= "i686"
#ASSUME_PROVIDED += "libsdl-native"
```

## ビルド

### bitbake実行
	■~/rp3/build/rpi　で下記のコマンド実行
bitbake core-image-base

```sh
$ cd conf
$ nano bblayers.conf 
$ nano local.conf 
```

初回は環境により3時間～10時間かかります
2回目以降は1/3～半分程度
インターネット上の情報を参照しながら、Linuxカーネル、ブートローダー、全アプリケーションのビルドを行います。


