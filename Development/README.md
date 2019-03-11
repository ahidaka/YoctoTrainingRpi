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
~/rp3/build/rpi で下記のコマンド実行します。

```sh
$ bitbake core-image-base 
```

環境によっても異なりますが、初回はビルドに3時間～10時間かかります。
2回目以降は1/3～半分程度。時間がかかるのは、インターネット上の情報を参照しながら、Linuxカーネル、ブートローダー、全アプリケーションのビルドを行うためです。


## MicroSDカードのコピー

### SDカードのデバイス名調査
マイクロSDカードをホストPC装着して　下記のコマンド実行します。

```sh
$ lsblk
```

表示結果のパーティーション名と容量から、デバイス名を探します。USBカードリーダーの経由の場合は sdc 等と表示されます。下記は16GBのマイクロSDカードを接続している例で、sde がマイクロSDカードのデバイス名なので、「/dev/sde」が割り当てられています。

```sh
sdb      8:16   0 111.8G  0 disk
+-sdb1   8:17   0   762M  0 part /boot
+-sdb2   8:18   0     1K  0 part
+-sdb5   8:21   0  15.3G  0 part [SWAP]
L-sdb6   8:22   0  95.8G  0 part /
sde      8:64   1  14.8G  0 disk
L-sde1   8:65   1  14.8G  0 part /media/hidaka/0403-0201
```

### アンマウントと書き込み実行

ディレクトリ移動後、管理者権限でアンマウントと書き込み実行します。

```sh
$ cd ~/rp3/build/rpi3/tmp/deploy/images/raspberrypi3
$ sudo umount /dev/sdc*
$ sudo dd if=./core-image-base-raspberrypi3.rpi-sdimg of=/dev/sdc bs=1M
$ sudo umount /dev/sdc*
```

### マイクロSDカードの取り出し

コマンド完了後、マイクロSDカードを取り出します。

## ターゲットの起動
マイクロSDカードをターゲットに装着して、必要に応じてモニター、キーボード、マウス、LANを接続後 Raspberry Pi 3 の電源を入れます。後述のドライバーのクロス開発のテストをする場合は、LAN接続が必須です。

**ユーザー名：root　パスワード無し**

でログインします。今回の設定は openssh Server を有効にしているため、LANを接続している場合には ssh でそのままログインできるので、扱いに注意してください。
 	
### 固定IPアドレス
ターゲットにログイン後、**/etc/network/interface** ファイルを編集して固定IPアドレスを設定可能です。

## SDK

カーネルのコンフィグレーション設定や、ドライバーのクロス開発を行うためにSDKをビルドしてインストールします。

### SDK ビルド

#### bitbake 実行

source コマンドで環境変数を読み込み済状態 (ディレクトリ ~/rp3/build/rpi） で下記のコマンドを実行します。

```sh
$ bitbake core-image-base -c populate_sdk
```

初回は環境により1時間～2時間かかります。2回目以降は1/3～半分程度です。インターネット上の情報を参照しながら、SDKのツールとライブラリのビルドを行います。完了後、SDKのインストーラーが **~/rp3/build/rpi3/tmp/deploy/sdk/** 以下に作成されます。

### SDK インストール

次の様な名前で *.sh ファイルが出来ていることを確認します。 

```sh
~/rp3/build/rpi3/tmp/deploy/sdk/poky-glibc-x86_64-core-image-base-cortexa7t2hf-neon-vfpv4-toolchain-2.6.1.sh
```

これを実行します。

```sh
$ cd ~/rp3/build/rpi3/
$ ./tmp/deploy/sdk/poky-glibc-x86_64-core-image-base-cortexa7t2hf-neon-vfpv4-toolchain-2.6.1.sh
```

SDKディレクトリの作成場所を確認して実行します。

```sh
Poky (Yocto Project Reference Distro) SDK installer version 2.6.1
=================================================================
Enter target directory for SDK (default: /opt/poky/2.6.1):
You are about to install the SDK to "/opt/poky/2.6.1". Proceed[Y/n]?
```

完了後、SDK用の環境設定ファイルパスが次の様に表示されます。

```sh
SDK has been successfully set up and is ready to be used.
Each time you wish to use the SDK in a new shell session, you need to source the environment setup script e.g.
 $ . /opt/poky/2.6.1/environment-setup-cortexa7t2hf-neon-vfpv4-poky-linux-gnueabi    
```
この **/opt/poky/2.6.1/** 以下の長いファイル名がクロス開発用の環境設定ファイルなので、メモしておきます。

## ドライバーのクロス開発ビルド

### ローダブルモジュール・ドライバーをビルドする環境構築

**~/rp3/build/rpi** 以下に、開発用ディレクトリ **/cross-compile/** を作成します。

```sh
$ cd ~/rp3/build/rpi
$ mkdir cross-compile
```

前項で表示されたクロス開発用環境設定ファイルを読み込みます。

```sh
$ source /opt/poky/2.6.1/environment-setup-cortexa7t2hf-neon-vfpv4-poky-linux-gnueabi 
```

### テスト用ローダブル・モジュールソースの入手

GitHub からサンプル・ソースコードを入手します。

```sh
$ cd cross-compile
$ git clone https://github.com/ahidaka/HelloLinuxDriver.git
```

### ローダブル・モジュールソースのビルド

#### Makefileの修正

**Makefile** を編集して、環境に合わせて **KDIR** 変数を設定します。

```sh
$ cd  cleanHelloLinuxDriver/4.15/hello
$ nano Makefile
```

編集後の内容

```makefile
#
EXTRA_CFLAGS := -DDEBUG=1

#
#if yocto for raspberrypi3
KDIR = /home/hidaka/rp3/build/rpi3/tmp/work/raspberrypi3-poky-linux-gnueabi/linux-raspberrypi/1_4.14.79+gitAUTOINC+9ca74c\
53cb-r0/linux-raspberrypi3-standard-build/

#KDIR = /lib/modules/`uname -r`/build

MODS = hello.ko hello_m.ko params.ko
OBJS = hello.o  hello_m.o  params.o
TARGETS = ${MODS}

all: ${TARGETS}

hello.ko: hello.c
        make -C $(KDIR) M=`pwd` KBUILD_VERBOSE=1 modules

hello_m.ko: hello_multi.c hello_extern.c
        make -C $(KDIR) M=`pwd` KBUILD_VERBOSE=1 modules

params.ko: params.c
        make -C $(KDIR) M=`pwd` KBUILD_VERBOSE=1 modules

clean:
        make -C $(KDIR) M=`pwd` clean

obj-m := ${OBJS}

hello_m-objs := hello_multi.o hello_extern.o

clean-files := *~ *.o *.ko *.mod.[co] ${TARGETS} Module.*
```

#### ビルド実行

```sh
make clean
make
```

エラー、警告ともになければビルドが完了しています。

## ドライバーのテスト

前項でビルドしたドライバーを転送して動作テストします。以降は **params.ko** のローダブルモジュールの例です。

### ドライバーの転送

ターゲットにログインして、IPアドレスを調べておきます。以下はターゲットのアドレスが **192.168.51.197** の表示例です。
```sh
# ifconfig
eth0      Link encap:Ethernet  HWaddr B8:27:EB:37:23:4F
          inet addr:192.168.51.197  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: fe80::ba27:ebff:fe37:234f/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:290 errors:0 dropped:87 overruns:0 frame:0
          TX packets:63 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:16589 (16.2 KiB)  TX bytes:11385 (11.1 KiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

ホスト側で操作、**scp** コマンドを使用してビルドしたドライバーを転送します。

```sh
$ scp params.ko root@192.168.51.197:
```

## ドライバーの実行

ホスト側で操作、sshでターゲットにログインします。
```sh
$ ssh -l root 192.168.51.197
```

ログイン後に次の操作をして  **params.ko** が動作していることを確認します。

```sh
$ insmod params.ko
$ echo 1 > /sys/modules/params/sw
$ cat /sys/modules/params/sw
$ dmesg
```

## カーネル・ビルド

### 補足情報

#### カーネルソースの場所

~/rp3/build/rpi3/tmp/work-shared/raspberrypi3/kernel-source/

#### クロスビルド用カーネル作業ディレクトリの場所

各種オブジェクト・モジュール, vmlinux, Image, zImage, System.map 等が配置されます。

~/rp3/build/rpi3/tmp/work/raspberrypi3-poky-linux-gnueabi/linux-raspberrypi/1_4.14.79+gitAUTOINC+9ca74c53cb-r0/

### 準備

ターゲットのカーネルのコンフィグレーション（設定変更）に必要なツールをインストールします。

```sh
$ sudo apt install -y git ccache fakeroot libncurses5-dev
$ sudo apt install -y kernel-package libssl-dev bison flex
```

### コンフィグレーション

```sh
$ cd ~/rp3/build/rpi3
$ bitbake linux-raspberrypi -c menuconfig
```

### カーネルのビルド

```sh
$ bitbake linux-raspberrypi -c compile -f 
```

### カーネルのコンフィグレーション出力
Yocto のカーネルビルドでは一般Linuxの様な **.config** ファイルを
使用しない。現在の CONFIG 設定を **.config** 形式で出力するコマンドは次の通りです。

```sh
$ bitbake linux-raspberrypi -c kernel_configme -f 
```

コマンド完了後、次のパスにコンフィグファイルが作成されます。

~/rp3/build/rpi3/tmp/work/raspberrypi3-poky-linux-gnueabi/linux-raspberrypi/1_4.14.79+gitAUTOINC+9ca74c53cb-r0/linux-raspberrypi3-standard-build/.config

以上
