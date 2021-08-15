
# Ubuntu下用Lean源码编译openwrt

源码地址：https://github.com/coolsnowwolf/lede  感谢Lean大神

1：首先装好 Ubuntu 64bit，推荐 Ubuntu 20.04 LTS x64

2：命令行输入

```
sudo apt-get update
```

然后输入

```
sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync
```

3： 使用

```
git clone https://github.com/coolsnowwolf/lede 
```

命令下载好源代码

然后

```
cd lede 
```

进入目录

 4：缝合一些其他插件

原地址:https://github.com/kenzok8/openwrt-packages

添加下面代码复制到 lede源码根目录 `feeds.conf.default` 文件 

```
src-git kenzo https://github.com/kenzok8/openwrt-packages

src-git small https://github.com/kenzok8/small
```

5:

```
./scripts/feeds update -a

./scripts/feeds install -a

make menuconfig
```

6：

```
make -j8 download V=s
```

下载dl库（国内请尽量全局科学上网）

7：输入

```
make -j1 V=s
```

（-j1 后面是线程数。第一次编译推荐用单线程）即可开始编译你要的固件了。

### 再次编译 （适用于不更改配置功能和插件，仅升级）

```
cd lede

git pull

./scripts/feeds update -a && ./scripts/feeds install -a

make defconfig

make -j8 download

make -j$(($(nproc) + 1)) V=s
```

更改配置编译

```
cd lede

rm -rf ./tmp && rm -rf .config

make menuconfig

make -j$(($(nproc) + 1)) V=s
```

开启IPV6

选上extra packages——ipv6helper

在 Network – Firewall – ip6tables 下启用 ip6tables-extra 和 ip6tables-mod-nat 项。

更改LAN口的默认IP地址

```
cd lede

vim package/base-files/files/bin/config_generate
```



大概在99行找到我们默认的原IP地址（192.168.1.1），按“i”把对应的IP更改即可

然后按shift+: 输入wq回车保存退出



编译丰富插件时，建议修改下面两项默认大小，留足插件空间。（ x86/64 ）！！！

Target Images ---> (16) Kernel partition size (in MB)       #默认是 (16) 建议修改 (256)

Target Images ---> (160) Root filesystem partition size (in MB) #默认是 (160) 建议修改 (512)



如果需要 Cloudflare DDNS 组件

默认情况下 Open­Wrt 中并没有 Cloud­flare DDNS 功能，就算勾选了DDNS也不包含cloudflare运营商。所以需要在编译时选择相应的组件，其位置在 `Network`→`IP Addresses and Names` →ddns-scripets_cloudflare.com-v4