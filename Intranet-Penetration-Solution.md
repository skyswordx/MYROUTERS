
# 内网穿透方案

ZeroTier 让我伤透了心
-  [关于zerotier免费版部署在路由器上面的问题请教](https://www.chiphell.com/thread-2639319-1-1.html)
- [关于zerotier免费版-OPENWRT专版-恩山无线论坛 (right.com.cn)](https://www.right.com.cn/forum/thread-8398208-1-1.html)
- [免费内网穿透方案——ZeroTier+OpenWRT_openwrt zerotier-CSDN博客](https://blog.csdn.net/qq_39300041/article/details/126645375)

找到其他解决方案：WireGuard + tailscalse
-  [比zerotier更好的内网穿透方案——Tailscale | 瓦解的生活记事 (hin.cool)](https://hin.cool/posts/tailscale.html)

安装 tailscale 时的 trouble-shooting
- 先直接在 luci 页面进行过滤查出 tailscale 软件包的名字
- 然后直接在 luci 页面进行安装 or 更新软件源均发现失败
	- `SyntaxError: Unexpected end of JSON input`
- 之后在终端 ssh 中试验 `apk update` 和 `apk add` 发现成功

安装对应适配的 Luci GUI 界面
- [luci-app-tailscale插件编写的爬坑记录 - 等风起 (asvow.com)](https://asvow.com/tailscale/)
- [asvow/luci-app-tailscale: LuCI support for tailscale (github.com)](https://github.com/asvow/luci-app-tailscale)

