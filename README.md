# 从零开始的 openWRT 之旅

## 拿到路由器的第一步

拿到路由器首先要确认目标平台和架构是什么，该目标平台及其架构对应的 OpenWRT 版本有没有稳定版本、还是只有 Snapshot 版本
> 这是因为官网的下载页面是按照稳定版和 Snapshot 版本区分两个并行的下载资源导航页，然后在这两个导航页分布着各个目标平台及架构的文件夹、在各自目标平台的文件夹下才分布着对应的 SDK、toolchain 等资源、在各自架构的文件嘉下才分布着对应的软件包和源列表


## 从源码编译插件需要构建 SDK 编译链

这里首先根据自己之前查到的路由器信息，到这个到官网的[下载页面](https://openwrt.org/zh/downloads)中先下载对应的 SDK 源码压缩包
> 这里以京东云亚瑟 JDC-AX 1800 PRO 为例，它的目标平台和架构是 `qualcommax/ipq60xx`，所以在对应目标平台的架构处进行下载即可

之后将这个压 SDK 源码缩包转移到 WSL 环境下进行解压
```shell
tar -I zstd -xvf <openwrt-SDK.tar.zst> # 解压对应的 SDK 源码即可

cd openWrt-SDK && make menuconfig # 切换到 SDK 根目录
# 然后第一次运行 make menuconfig 生成 .config 文件

# 更新 SDK 中官方 feeds 源索引并安装
./scripts/feeds update -a
./scripts/feeds install -a
```

然后要明白，利用这个 SDK 工程编译新的插件安装包的方法，本质上是在 SDK 工程里进行 `make` 时测得到位于 ` SDK/package ` 文件夹下面的对应软件包的 ` makefile ` 文件
```shell
. # SDK 源码根目录
├── LICENSES
├── bin # 输出编译之后的插件安装包
├── build_dir 
├── dl
├── feeds # 官方推荐的自己编译软件包的工作流
├── include 
├── package # make menuconfig 和 make 时检测软件包是否准备充分的地方
├── scripts # 官方提供的一组为自己编译软件包而准备的工作流
├── staging_dir
├── target # 该 SDK 的目标平台
└── tmp
```

从这个 SDK 的工程文件目录和之前的操作可以知道，官方其实提供了一套利用 `feeds` 的机制自己编译软件包的工具。这里的 `feeds` 其实就像某种包管理器，在一个 `feeds.conf.default` 文件中就收录了各个软件包的 `feeds` 索引。但是单有文件的话还是没有对软件源进行更新、也并没有真正在 SDK 中配置好编译规则，所以之前还要执行的两个命令就是根据 `feeds` 进行更新和安装编译规则，最终在把 `SDK/package/` 路径下面的软件包文件夹及其 `makefile` 配置好

然后要添加自己的软件包的话，其实按照官方的意思也是要配置包管理器的软件镜像源索引，
> 这是官方的流程指引 [OpenWrt Wiki 使用​SDK开发工具包](https://openwrt.org/zh/docs/guide-developer/toolchain/using_the_sdk#%E4%BD%BF%E7%94%A8%E2%80%8Bsdk%E5%BC%80%E5%8F%91%E5%B7%A5%E5%85%B7%E5%8C%85)


但在自己实践的时候发现，直接在 `feeds` 文件里添加 git 仓库之后安装是失败的，可能是 `feeds` 对 git 仓库的文件有相应的要求所以安装失败。安装失败的后果就是没有在 `SDK/package/` 这个目录下配置好对应插件的文件夹及其文件夹下的 `makefile` 文件


所以在使用 `feeds` 的 `update` 和 `install` 官方 `feeds` 软件源之后，为了省事可以自己在 ` SDK/package/ ` 下面新建一个插件文件夹，然后 clone 插件的 git 源码仓库之后手动搞一个 ` makefile ` 进行配置

在有了对应的 `makefile` 文件之后，就可以回到 SDK 的根目录，进行下面的操作
```shell
# 多线程编译
make defconfig

# 根据对应软件包的 makefile 编译
make package/path/to/pack_or_mod/compile -j$(grep processor /proc/cpuinfo | wc -l)
```

## 为编译插件编写 makefile 以及完整的编译流程


## 传送文件给路由器

1. 推荐使用 `ssh`
	1. scp + [WinSCP: Free SFTP and FTP client for Windows](https://winscp.net/eng/index.php)
2. 使用 U 盘，进行挂载（~~当然也可以分区，不过无关~~）
	1. NTFS 需要 `ntfs-3g` 这个软件包，并且依赖 `kmod-fuse` 模块
		- [OpenWrt Wiki package: kmod-fuse](https://openwrt.org/packages/pkgdata/kmod-fuse)
	2. EXT4 + WSL

## 在路由器终端中使用 `apk` 包管理器

把路由器想象成一个主机就行了，这里面的包管理器就是 `apk`，然后 `apk add <package>` 就相当于 ubuntu 下面的 `apt install <package>`，使用 `--help` 选项查看其他选项和参数的使用

## 路由器固件的内核模块缺失导致在路由器使用 `apk` 安装软件包无失败

这是编译固件时的错误，最好的解决方案是编译一个新的有这方面的固件

临时解决方案是先在 SDK 补全这个模块，然后以此构建需要的软件包

在 SDK 编译时，由于之前已经更新和安装了官方的软件源，这时候应该先在 `menuconfig` 里面，按下 `/` 进行搜索看看有没有已经在官方软件源里面的软件包，如果有的话就可以直接编译了

## 绕过用 `apk` 安装第三方软件包时的签名安全限制

得要把文件传输到路由器下面手动使用下面命令安装，直接用 Luci 安装会提示限制错误
```shell
apk add <custom_package> --allow-untrusted
# apk add  --allow-untrusted 空出软件包位置以便直接复制
```

## 配置 minieap 以便通过校园网锐捷认证

符合 SYSU 锐捷认证体质的配置思路来源于群友[Aether Chen](https://github.com/chenjunyu19)，在此感谢群友的经验，DHCP 改成无，然后重试次数和超时设为 0（无限）下面是这一思路的最简配置
```shell
# 调整好用户名、密码和网卡
username =
password =
nic =                                                       

module = rjv3
fake-dns2 = 0.0.0.0
fake-serial = 0

# 关键点在于下面的配置
stage-timeout = 0
max-retries = 0

# 核心保证不能让 minieap 退出，要让它响应用户名请求
# 锐捷客户端大概 5 分钟没响应的话就会切断链路
# minieap 是个状态机，认证成功以后服务器又持续询问用户名
# 会让状态机跳回到认证过程状态，会触发状态超时，然后退出
# 说白了就是 minieap 程序没适配，但是改改配置也能用
# 可以当这个是心跳，锐捷这个是魔改 802.1x，比如这个 “心跳” 就比较奇葩

# 正常运行以后应该每 30 或 60 秒就会显示一次回应用户名
```

配合 `luci-app-minieap` 插件在 web 端修改参数，通过 `uci` 文件，软链接到位于 `/etc/minieap.conf` 的配置文件的效果如下
```shell
# root@LibWrt:/etc: cat minieap.conf
auth-round=1
daemonize=3
dhcp-type=3
eap-bcast-addr=0
heartbeat=60
if-impl=sockraw
max-dhcp-count=3
max-fail=3
module = rjv3

#### 核心 ####
max-retries=3
#############

nic=eth0
no-auto-reauth=1
password=
pid-file=/var/run/minieap.pid
pingcommand=minieap -k 1
service=internet

##### 核心 #####
stage-timeout=5
###############

username=
version-str=RG-SU For Linux V1.0
wait-after-fail=30
```
## 参考链接

`qualcommax/ipq60xx` 目标平台的官方软件源及其镜像站
-  [南京大学镜像站：NJU OpenWRT Mirror](https://mirror.nju.edu.cn/openwrt/snapshots/targets/qualcommax/ipq60xx/)
-  [Index of /snapshots/targets/qualcommax/ipq60xx/ (openwrt.org)](http://downloads.openwrt.org/snapshots/targets/qualcommax/ipq60xx/)

其他人分享的京东云亚瑟 AX 1800 PRO 路由器固件编译链的相关仓库，这里主要是在不确定其目标平台和架构时，阅读其中的编译配置文件以便确认相关的信息
- [iGocsen/OpenWrt-IPQ60xx: OpenWrt 固件——适配Redmi AX5、Xiaomi AX1800、ZN M2、CMIOT AX18、Qihoo V6、JDC AX5、JDC AX1800 PRO (github.com)](https://github.com/iGocsen/OpenWrt-IPQ60xx)
- [lazyoop/openwrt-ipq60xx: JD AX1800PRo路由OpenWrt kernel6.x nss加速 (github.com)](https://github.com/lazyoop/openwrt-ipq60xx)
- [Hashcoel/jdc_ax1800-pro: OpenWrt 固件——适配Redmi AX5、Xiaomi AX1800、ZN M2、CMIOT AX18、Qihoo V6、JDC AX5、JDC AX1800 PRO (github.com)](https://github.com/Hashcoel/jdc_ax1800-pro)
- [laipeng668/openwrt-ci-roc: 京东云（亚瑟&雅典娜&太乙）路由器满血NSS固件云编译 (github.com)](https://github.com/laipeng668/openwrt-ci-roc)

网上和京东云亚瑟 AX 1800 PRO 路由器相关的 blog
- [京东云路由器AX1800Pro(亚瑟)刷QWRT | 顾师傅的网络备忘录 (hellogu.github.io)](https://hellogu.github.io/post/jing-dong-yun-lu-you-qi-shua-qwrt/)


安装第三方 apk 时会出现签名认证失败的报错，需要手动设置
- [快照版离线安装apk提示UNTRUSTED signature · Issue #1602 · immortalwrt/immortalwrt (github.com)](https://github.com/immortalwrt/immortalwrt/issues/1602)

OpenWRT 相关的官方页面
- [OpenWrt 固件资源下载导航](https://openwrt.org/zh/downloads)

有关交叉编译软件源的指引
- 有用
	-  [个人开发记录 -- openwrt编译,添加自制二进制文件/添加包 (getce.cn)](https://www.getce.cn/show/104.html)
	- [BoringCat/minieap-openwrt: minieap的openwrt Makefile (github.com)](https://github.com/BoringCat/minieap-openwrt)
	- [BoringCat/luci-app-minieap (github.com)](https://github.com/BoringCat/luci-app-minieap)
- 可能有用
	- [OpenWRT下载第三方软件包时 ‘ ./feeds install -a’ 存在的问题 - 勤劳小虾米 - 博客园 (cnblogs.com)](https://www.cnblogs.com/heng-xing/p/13396192.html)
	- [openwrt-minieap-gdufs/docs/how-to-configure-minieap-in-openwrt.md at master · jimlee2048/openwrt-minieap-gdufs (github.com)](https://github.com/jimlee2048/openwrt-minieap-gdufs/blob/master/docs/how-to-configure-minieap-in-openwrt.md)
	- [OpenWrt官方（OpenWrt23.05）SDK编译PassWall教程 · xiaorouji/openwrt-passwall · Discussion #1603 (github.com)](https://github.com/xiaorouji/openwrt-passwall/discussions/1603)
	- [动手编译适合自己路由器的 ipk | 雪山深处 (talaxy.site)](https://www.talaxy.site/mentohust-minieap/)
	- [为OpenWRT开发配置交叉编译环境 | 水波粼粼 (xuzhe.tj.cn)](https://www.xuzhe.tj.cn/index.php/2023/11/06/%e4%b8%baopenwrt%e5%bc%80%e5%8f%91%e9%85%8d%e7%bd%ae%e4%ba%a4%e5%8f%89%e7%bc%96%e8%af%91%e7%8e%af%e5%a2%83/)
	- [Openwrt 交叉编译(Crosscompile)及使用SDK生成ipk安装包 – YJ's Blog (yjblog.net)](https://www.yjblog.net/post/123.html)
	- [超详细！手把手演示编译OpenWrt内核驱动模块_openwrt编译内核-CSDN博客](https://blog.csdn.net/qq_41453285/article/details/102760270)
	- [muink/luci-app-tinyfilemanager: LuCI Tiny File Manager: Web based File Manager in PHP (github.com)](https://github.com/muink/luci-app-tinyfilemanager)

配置 minieap 的部分参考
- https://github.com/updateing/minieap/blob/master/README.md
- [BoringCat/minieap-openwrt: minieap的openwrt Makefile (github.com)](https://github.com/BoringCat/minieap-openwrt)
- [BoringCat/luci-app-minieap (github.com)](https://github.com/BoringCat/luci-app-minieap)