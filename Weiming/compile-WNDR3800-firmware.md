# 编译 WNDR3800 固件

[CreateTime]: # (2020.12.02)
[ModifyTime]: # (2021.03.02)

WNDR3800 是我的第一台“洋垃圾”路由器，18 年的时候斥巨资 145 元买来的，没想到能用这么久。

曾经编译过一次它的固件，但是已经过去很久，都已经忘记该怎么操作了。后来想升级一下固件，到网上去下载别人编译好的固件，结果怎么用都不好用，总是会有一些小问题，所以又打算自己编译固件。这次把编译的过程记录下来，以后要是想编译的话，也不怕网上找不到资料了。

## 下载源码及安装依赖

源码仓库地址：[coolsnowwolf/lede](https://github.com/coolsnowwolf/lede) （[备份地址](https://github.com/aomnisz/lede)）

其实源码仓库里的 `README.md` 也介绍了编译命令（如下所示），所以这个记录主要是配置方面的。

```markdown
编译命令如下:

1. 首先装好 Ubuntu 64bit，推荐  Ubuntu 18 LTS x64

2. 命令行输入 `sudo apt-get update` ，然后输入 `sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync`

3. 使用 `git clone https://github.com/coolsnowwolf/lede.git` 命令下载好源代码，然后 `cd lede` 进入目录

4. `./scripts/feeds update -a && ./scripts/feeds install -a && make menuconfig`

5. `make -j8 download V=s` 下载dl库（国内请尽量全局科学上网）

6. 输入 `make -j1 V=s`（-j1 后面是线程数。第一次编译推荐用单线程）即可开始编译你要的固件了。

本套代码保证肯定可以编译成功。里面包括了 R20 所有源代码，包括 IPK 的。
```

那么首先当然是安装系统（Ubuntu 18.04 虚拟机），然后换源并将软件更新到最新。

```bash
sudo -s
apt update
apt install open-vm-tools-desktop
cp /etc/apt/sources.list /etc/apt/sources.list.backup
sed -i 's/cn.archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
sed -i 's/security.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
apt update && apt upgrade && apt autoremove
exit
```

然后按照 README 中说的安装好编译需要的软件，还有下载源码。

```bash
sudo apt update
sudo apt install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync
git clone https://github.com/coolsnowwolf/lede --depth 1 ~/lede
cd ~/lede/
```

## 配置编译选项

在配置之前，先装一个主题**（可选）**

```bash
rm -rf package/lean/luci-theme-argon
git clone -b 18.06 https://github.com/jerrykuku/luci-theme-argon.git package/lean/luci-theme-argon
```

然后还要增加一些插件，有些东西被阉掉了：[参考这里](https://mianao.info/2020/05/05/%E7%BC%96%E8%AF%91%E6%9B%B4%E6%96%B0OpenWrt-PassWall%E5%92%8CSSR-plus%E6%8F%92%E4%BB%B6)

```bash
sed -i 's/^#\(.*helloworld\)/\1/' feeds.conf.default
# or
echo "src-git small https://github.com/kenzok8/small" >> feeds.conf.default
echo "src-git kenzo https://github.com/kenzok8/openwrt-packages" >> feeds.conf.default
```

接着就是重头戏的配置了

```bash
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
```

- 选择 CPU 型号 ``Target System`` 选为 `Atheros ARM7xxx/ARM9xxx`
- 选择路由器型号 ``Target Profile`` 选为 `NETGEAR WNDR3800`
- ``Extra packages`` > ``ipv6helper``
- ``LuCI`` > ``3. Applications`` > ``luci-app-accesscontrol`` (cancel)
- ``LuCI`` > ``3. Applications`` > ``luci-app-ddns`` (cancel)
- ``LuCI`` > ``3. Applications`` > ``luci-app-frpc``
- ``LuCI`` > ``3. Applications`` > ``luci-app-nlbwmon`` (cancel)
- ``LuCI`` > ``3. Applications`` > ``luci-app-ssr-plus``
- ``LuCI`` > ``3. Applications`` > ``luci-app-unblockmusic`` (cancel)
- ``LuCI`` > ``3. Applications`` > ``luci-app-vsftpd`` (cancel)
- ``LuCI`` > ``3. Applications`` > ``luci-app-webadmin`` (cancel)
- ``LuCI`` > ``4. Themes`` > ``luci-theme-argon``/...
- ``Network`` > ``Firewall`` > ``ip6tables`` > ``ip6tables-extra`` / ``ip6tables-mod-nat``

## 开始编译

需要先下载再编译，`-j[jobs]` 是指线程数，后面的数字可以根据需要进行更改。

```bash
make -j1 download V=s
make -j1 V=s
```

最终编译好的固件会在 `~/lede/bin/targets/` 中。

## 后记

记一些链接，虽然以后可能会挂掉。

- [LuCI Applications 添加插件应用说明](https://www.codenong.com/js2ea938eff48f/)
- [个性化编译 LEDE 固件](https://qingwubleach.blogspot.com/2019/07/lede.html)（修改WIFI名等）
- [OpenWRT IPv6 三种配置方式](http://blog.kompaz.win/2017/02/22/OpenWRT%20IPv6%20%E9%85%8D%E7%BD%AE/)（使用 IPv6 中继）

---

**PS.** 在折腾过程中，无意间发现有人[用 Github Actions 去编译固件](https://p3terx.com/archives/build-openwrt-with-github-actions.html)。以后有空搞搞嘿嘿嘿。

**PPS.** 已经搞好了：[aomnisz/Actions-OpenWrt-WNDR3800](https://github.com/aomnisz/Actions-OpenWrt-WNDR3800)

---

再顺便记一下 Newifi3 的编译。大体上相同，就配置不太一样。

- 选择 CPU 型号 ``Target System`` 选为 `MediaTek Ralink MIPS`
- 选择 CPU 子型号 ``Subtarget`` 选为 `MT7621 based boards`
- 选择路由器型号 ``Target Profile`` 选为 `Newifi D2`

还是一些链接

- [多设备 OpenWrt Aciton 固件云编译](https://ivansolis1989.github.io/OpenWrt-DIY/)
