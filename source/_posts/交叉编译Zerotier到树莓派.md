---
title: 交叉编译Zerotier到树莓派
categories: 瞎折腾记录
toc: true
tags:
  - 嵌入式
abbrlink: e9442809
date: 2026-02-13 17:16:03
---


## 背景

在给树莓派上刷写了最新的Rocky 10之后，发现Zerotier官方没有针对其发布相应的软件包，要是在树莓派上编译Zerotier则更是地狱，1G版本的rpi不仅慢还有可能爆内存，如此看来只能选择交叉编译的方式了。而Zerotier官方的编译流程并没有交叉编译这一步，所以需要改一些参数才行。

<!-- more -->

{% raw %}
<style type="text/css">
.heimu { color: #000; background-color: #000; }
.heimu:hover { color: #fff; }
</style>
{% endraw %}

## 准备

### 获取源码

要交叉编译肯定需要Zerotier的代码，先将其克隆到本地仓库：

```shell
git clone https://github.com/zerotier/ZeroTierOne.git -b 1.16.1 # 只需要1.16.1分支下的内容
```

### 配置编译环境

还需要编译所用的工具和交叉编译工具链。一般编译所用工具直接安装`build-essential`和`cmake`就行，

```shell
sudo apt install build-essential cmake --install-suggests
```

交叉编译工具链可以直接使用系统提供的版本，写一个正则在`apt`里搜一下：

```shell
$ apt search ^[a-z+]+-[0-9]+-aarch64-linux-gnu$
Sorting... Done
Full Text Search... Done
cpp-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU C preprocessor

cpp-11-aarch64-linux-gnu/jammy-updates,jammy-security,now 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU C preprocessor

cpp-12-aarch64-linux-gnu/jammy-updates,jammy-security,now 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU C preprocessor

cpp-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU C preprocessor

g++-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU C++ compiler (cross compiler for arm64 architecture)

g++-11-aarch64-linux-gnu/jammy-updates,jammy-security,now 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU C++ compiler (cross compiler for arm64 architecture)

g++-12-aarch64-linux-gnu/jammy-updates,jammy-security,now 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU C++ compiler (cross compiler for arm64 architecture)

g++-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU C++ compiler (cross compiler for arm64 architecture)

gcc-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU C compiler (cross compiler for arm64 architecture)

gcc-11-aarch64-linux-gnu/jammy-updates,jammy-security,now 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU C compiler (cross compiler for arm64 architecture)

gcc-12-aarch64-linux-gnu/jammy-updates,jammy-security,now 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU C compiler (cross compiler for arm64 architecture)

gcc-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU C compiler (cross compiler for arm64 architecture)

gccgo-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU Go compiler

gccgo-11-aarch64-linux-gnu/jammy-updates,jammy-security 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU Go compiler

gccgo-12-aarch64-linux-gnu/jammy-updates,jammy-security 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU Go compiler

gccgo-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU Go compiler

gdc-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU D compiler (version 2) (cross compiler for arm64 architecture)

gdc-11-aarch64-linux-gnu/jammy-updates,jammy-security 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU D compiler (version 2) (cross compiler for arm64 architecture)

gdc-12-aarch64-linux-gnu/jammy-updates,jammy-security 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU D compiler (version 2) (cross compiler for arm64 architecture)

gdc-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU D compiler (version 2) (cross compiler for arm64 architecture)

gfortran-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU Fortran compiler

gfortran-11-aarch64-linux-gnu/jammy-updates,jammy-security 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU Fortran compiler

gfortran-12-aarch64-linux-gnu/jammy-updates,jammy-security 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU Fortran compiler

gfortran-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU Fortran compiler

gnat-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU Ada compiler

gnat-11-aarch64-linux-gnu/jammy-updates,jammy-security 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU Ada compiler

gnat-12-aarch64-linux-gnu/jammy-updates,jammy-security 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU Ada compiler

gnat-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU Ada compiler

gobjc++-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C++ compiler

gobjc++-11-aarch64-linux-gnu/jammy-updates,jammy-security 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C++ compiler

gobjc++-12-aarch64-linux-gnu/jammy-updates,jammy-security 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C++ compiler

gobjc++-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C++ compiler

gobjc-10-aarch64-linux-gnu/jammy-updates,jammy-security 10.5.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C compiler

gobjc-11-aarch64-linux-gnu/jammy-updates,jammy-security 11.4.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C compiler

gobjc-12-aarch64-linux-gnu/jammy-updates,jammy-security 12.3.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C compiler

gobjc-9-aarch64-linux-gnu/jammy-updates,jammy-security 9.5.0-1ubuntu1~22.04cross1 amd64
  GNU Objective-C compiler
```

这也不是很老的软件，显然安装`gcc-12-aarch64-linux-gnu`和`g++-12-aarch64-linux-gnu`两个包就行。

```shell
sudo apt install gcc-12-aarch64-linux-gnu g++-12-aarch64-linux-gnu
```

装好之后验证一下`gcc`和`g++`能不能用：

```cosole
$ aarch64-linux-gnu-gcc --version
aarch64-linux-gnu-gcc (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

$ aarch64-linux-gnu-g++ --version
aarch64-linux-gnu-g++ (Ubuntu 11.4.0-1ubuntu1~22.04) 11.4.0
Copyright (C) 2021 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

```

## 交叉编译

### 阅读编译配置

官方的编译说明给的很简短：

* To build on Mac and Linux just type `make`. On FreeBSD and OpenBSD `gmake` (GNU make) is required and can be installed from packages or ports. For Windows there is a Visual Studio solution in `windows/`.
* The project supports various build targets and platforms:
  * Standard build: `make`
  * Self-test build: `make selftest`
  * Platform-specific requirements:
    * FreeBSD/OpenBSD: Use `gmake` (GNU make)
    * Windows: Visual Studio solution in `windows/`


观察一下`./Makefile`，看看从哪里下手进行交叉编译：

```Makefile
# Common makefile -- loads make rules for each platform

OSTYPE=$(shell uname -s)

ifeq ($(OSTYPE),Darwin)
  include make-mac.mk
endif

ifeq ($(OSTYPE),Linux)
  include make-linux.mk
endif

ifeq ($(OSTYPE),FreeBSD)
  CC=clang
  CXX=clang++
  ZT_BUILD_PLATFORM=7
  include make-bsd.mk
endif
ifeq ($(OSTYPE),OpenBSD)
  CC=clang
  CXX=clang++
  ZT_BUILD_PLATFORM=9
  include make-bsd.mk
endif

ifeq ($(OSTYPE),NetBSD)
  include make-netbsd.mk
endif

drone:
  @echo "rendering .drone.yaml from .drone.jsonnet"
  drone jsonnet --format --stream
  drone sign zerotier/ZeroTierOne --save

clang-format:
  find node osdep service tcp-proxy nonfree/controller -iname '*.cpp' -o -iname '*.hpp' | xargs clang-format -i
```

根据提示看一下`./make-linux.mk`，这里给出一些关键片段：

```Makefile
# Determine system build architecture from compiler target. This is hairy due to "ARM wrestling."
CC_MACH=$(shell $(CC) -dumpmachine | cut -d '-' -f 1)
ZT_ARCHITECTURE=999
ifeq ($(CC_MACH),x86_64)
  ZT_ARCHITECTURE=2
  ZT_USE_X64_ASM_SALSA=1
  ZT_USE_X64_ASM_ED25519=1
  override CFLAGS+=-msse -msse2
  override CXXFLAGS+=-msse -msse2
  ZT_SSO_SUPPORTED=1
  ifeq ($(ZT_CONTROLLER),1)
    EXT_ARCH=amd64
  endif
endif
```

上文这一段负责获取编译器的`target`，

```Makefile
ifeq ($(CC_MACH),arm64)
  ZT_ARCHITECTURE=4
  ZT_SSO_SUPPORTED=1
  ZT_USE_X64_ASM_ED25519=0
  override DEFS+=-DZT_NO_TYPE_PUNNING -DZT_ARCH_ARM_HAS_NEON -march=armv8-a+crypto -mtune=generic -mstrict-align
endif
ifeq ($(CC_MACH),aarch64)
  ZT_ARCHITECTURE=4
  ZT_SSO_SUPPORTED=1
  ZT_USE_X64_ASM_ED25519=0
  override DEFS+=-DZT_NO_TYPE_PUNNING -DZT_ARCH_ARM_HAS_NEON -march=armv8-a+crypto -mtune=generic -mstrict-align
  ifeq ($(ZT_CONTROLLER),1)
    EXT_ARCH=arm64
  endif
endif
```

上面这一段是当`target`是`aarch64`时配置的编译选项。

```Makefile
ifeq ($(ZT_SSO_SUPPORTED), 1)
  ifeq ($(ZT_EMBEDDED),)
    override DEFS+=-DZT_SSO_SUPPORTED=1
    ifeq ($(ZT_DEBUG),1)
      LDLIBS+=rustybits/target/debug/libzeroidc.a -ldl -lssl -lcrypto
    else
      LDLIBS+=rustybits/target/release/libzeroidc.a -ldl -lssl -lcrypto
    endif
  endif
endif
```

```Makefile
ifeq ($(ZT_SSO_SUPPORTED), 1)
ifeq ($(ZT_EMBEDDED),)
zeroidc:  FORCE
  export PATH=/${HOME}/.cargo/bin:$$PATH; cd rustybits && cargo build $(ZT_CARGO_FLAGS) -p zeroidc
endif
else
zeroidc:
endif

ifeq ($(ZT_CONTROLLER), 1)
smeeclient:  FORCE
  export PATH=/${HOME}/.cargo/bin:$$PATH; cd rustybits && cargo build $(ZT_CARGO_FLAGS) -p smeeclient
else
smeeclient:
endif
```

上面这两段分别描述了当`ZT_SSO_SUPPORTED`参数开启时增加的链接库和编译过程。由于另外链接了rust库，而cargo的交叉编译配置需要对`ZT_CARGO_FLAGS`做额外的修改，还要改链接位置，十分不优雅{% raw %}<span class="heimu">（而且编译完跑起来碰到了段错误）</span>{% endraw %}，索性直接不编译，之后`make`的时候加上`ZT_SSO_SUPPORTED=0`就好了。

```Makefile
# Note: keep the symlinks in /var/lib/zerotier-one to the binaries since these
# provide backward compatibility with old releases where the binaries actually
# lived here. Folks got scripts.

install:  FORCE
  mkdir -p $(DESTDIR)/usr/sbin
  rm -f $(DESTDIR)/usr/sbin/zerotier-one
  cp -f zerotier-one $(DESTDIR)/usr/sbin/zerotier-one
  rm -f $(DESTDIR)/usr/sbin/zerotier-cli
  rm -f $(DESTDIR)/usr/sbin/zerotier-idtool
  ln -s zerotier-one $(DESTDIR)/usr/sbin/zerotier-cli
  ln -s zerotier-one $(DESTDIR)/usr/sbin/zerotier-idtool
  mkdir -p $(DESTDIR)/var/lib/zerotier-one
  rm -f $(DESTDIR)/var/lib/zerotier-one/zerotier-one
  rm -f $(DESTDIR)/var/lib/zerotier-one/zerotier-cli
  rm -f $(DESTDIR)/var/lib/zerotier-one/zerotier-idtool
  ln -s ../../../usr/sbin/zerotier-one $(DESTDIR)/var/lib/zerotier-one/zerotier-one
  ln -s ../../../usr/sbin/zerotier-one $(DESTDIR)/var/lib/zerotier-one/zerotier-cli
  ln -s ../../../usr/sbin/zerotier-one $(DESTDIR)/var/lib/zerotier-one/zerotier-idtool
  mkdir -p $(DESTDIR)/usr/share/man/man8
  rm -f $(DESTDIR)/usr/share/man/man8/zerotier-one.8.gz
  cat doc/zerotier-one.8 | gzip -9 >$(DESTDIR)/usr/share/man/man8/zerotier-one.8.gz
  mkdir -p $(DESTDIR)/usr/share/man/man1
  rm -f $(DESTDIR)/usr/share/man/man1/zerotier-idtool.1.gz
  rm -f $(DESTDIR)/usr/share/man/man1/zerotier-cli.1.gz
  cat doc/zerotier-cli.1 | gzip -9 >$(DESTDIR)/usr/share/man/man1/zerotier-cli.1.gz
  cat doc/zerotier-idtool.1 | gzip -9 >$(DESTDIR)/usr/share/man/man1/zerotier-idtool.1.gz
  cp ext/installfiles/linux/zerotier-one.te $(DESTDIR)/var/lib/zerotier-one/zerotier-one.te
```

`install`明显就是复制一下文件，因为需要把编译成果拷贝走，编译的时候记得指定`DESTDIR`安装目录，这里就选`./install`。

```shell
mkdir -p ./install
```

### 执行交叉编译

明确了编译参数之后即可进行交叉编译，据前文，我们的编译参数有：

* `CC=aarch64-linux-gnu-gcc`
* `CXX=aarch64-linux-gnu-g++`
* `ZT_SSO_SUPPORTED=0`

安装时需要的参数有：

* `DESTDIR=$(pwd)/install`

直接带参数用全部线程`make`：

```shell
make -j$(nproc) CC=aarch64-linux-gnu-gcc CXX=aarch64-linux-gnu-g++ ZT_SSO_SUPPORTED=0
```

如果编译过程报错，别用多线程编译，单线程跑一下看是哪的问题。

编译通过后可以执行安装了，记得带参数：

```cosole
make install DESTDIR=$(pwd)/install
```

看一下`./install`里的文件：

```shell
$ tree
.
├── usr
│   ├── sbin
│   │   ├── zerotier-cli -> zerotier-one
│   │   ├── zerotier-idtool -> zerotier-one
│   │   └── zerotier-one
│   └── share
│       └── man
│           ├── man1
│           │   ├── zerotier-cli.1.gz
│           │   └── zerotier-idtool.1.gz
│           └── man8
│               └── zerotier-one.8.gz
└── var
    └── lib
        └── zerotier-one
            ├── zerotier-cli -> ../../../usr/sbin/zerotier-one
            ├── zerotier-idtool -> ../../../usr/sbin/zerotier-one
            ├── zerotier-one -> ../../../usr/sbin/zerotier-one
            └── zerotier-one.te

9 directories, 10 files
```

可以再看一下编译出来的ELF文件头：

```shell
$ readelf -h ./install/usr/sbin/zerotier-one
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 03 00 00 00 00 00 00 00 00
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - GNU
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           AArch64
  Version:                           0x1
  Entry point address:               0x407dc0
  Start of program headers:          64 (bytes into file)
  Start of section headers:          4629184 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           56 (bytes)
  Number of program headers:         6
  Size of section headers:           64 (bytes)
  Number of section headers:         28
  Section header string table index: 27
```

没啥毛病，把对应文件改一下权限，复制到树莓派对应目录就行，完美。
