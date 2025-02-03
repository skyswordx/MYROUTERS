# 京东云亚瑟 AX1800 PRO 路由器配置记录

## 目标平台、架构和软件源

它的目标平台是 `qualcommax/ipq60xx`

在编译配置文件中的架构是 `aarch64_cortex-a53`
> 显示时是显示这个 `ARMv8 Processor rev 4 (v8l) x 4 (1800MHz)` 

该目标平台对应的 OpenWRT 固件是 Snapshot 的，也就是在研发变动中，没有固定的发行版的
> 这就导致了后面的 `apk` 软件源仓库结构可能会随时变动，当使用 `apk update` 时出现错误提示输出时要注意可能是软件源仓库对应不上
> 并且在设置软件源镜像站的时候，也要注意镜像站是否有同步镜像 Snapshot 相关的软件源，目前在镜像站导航中只找到南京大学镜像站有在镜像 Snapshot 相关的软件源


截止至 2025.1.31 日的官方 `apk` 软件源和镜像源的配置文件分别如下
官方软件源的配置文件，地址是 `/etc/apk/repositories.d/distfeeds.list`
```shell
# target
http://downloads.openwrt.org/snapshots/targets/qualcommax/ipq60xx/packages/packages.adb

# base
http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/base/packages.adb

# luci
http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/luci/packages.adb

# packages
http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/packages/packages.adb

# routing
http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/routing/packages.adb

# telephony
http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/telephony/packages.adb

# video
http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/video/packages.adb

# outOFdate
# http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/mihomo/packages.adb
# http://downloads.openwrt.org/openwrt/snapshots/packages/aarch64_cortex-a53/nss_packages/packages.adb
# http://downloads.openwrt.org/snapshots/packages/aarch64_cortex-a53/sqm_scripts_nss/packages.adb
```

自定义的配置文件，地址是 `/etc/apk/repositories.d/customfeeds.list`
```shell
# add your custom package feeds here
# http://www.example.com/path/to/files/packages.adb

# target
http://mirror.nju.edu.cn/openwrt/snapshots/targets/qualcommax/ipq60xx/packages/packages.adb

# base
http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/base/packages.adb

# luci
http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/luci/packages.adb

# packages
http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/packages/packages.adb

# routing
http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/routing/packages.adb

# telephony
http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/telephony/packages.adb

# video
http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/video/packages.adb

# outOFdate
# http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/mihomo/packages.adb
# http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/nss_packages/packages.adb
# http://mirror.nju.edu.cn/openwrt/snapshots/packages/aarch64_cortex-a53/sqm_scripts_nss/packages.adb
```
