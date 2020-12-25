# 目录

* [关于 openwrt-raspberry](#关于-openwrt-raspberry)
* [起步](#起步)
* [准备](#准备)
* [新手上路](#新手上路)
* [进价](#进价)
* [行动](#行动)
* [效果](#效果)
* [具体方案](#具体方案)

# 关于 openwrt-raspberry

**[跳过说明直接看方案，点这里！点这里！点这里！](./guide.md)**

对于普通百兆网络，一般单台中等性能的路由器基本可以满足日常使用需要。但是随着千兆网络逐渐普及，高速访问网络资源的需求就日渐突出，坊间流行叫跑满带宽。对于带宽满速受到很多因素的影响，其中比较重要的一点就是路由器。如果之前关注过笔者应该对此有所了解，[**简单版**](https://github.com/felix-fly/v2ray-openwrt) 可以应对一般使用需要，[**优化版**](https://github.com/felix-fly/v2ray-dnsmasq-dnscrypt) 通过分流对于普通网络流量性能提升明显，但是对于特殊流量，提升效果不是很显著，根本的原因就是路由器本身的硬件性能（主要为CPU）遇到了瓶颈。特殊流量目前的方式都是会使用到tls的加密解密，而对于一般路由器使用的嵌入式CPU，这个操作就成为了要害。

# 起步

鉴于嵌入式CPU性能方面的不足，笔者也曾经考虑过被风起流行的软路由，但是东宝坛一圈之后，还是有些犹豫，如果选择软路由方案，有几个需要考虑的方面：

* 成本：几百~上千
* 功耗：10w起
* 尺寸：不算小
* 功能：强大，是否需要
* 稳定性：未知，就算机器是新的，但是内部的cpu、内存、硬盘也未必是新的
* 型号：太多了，选择起来很困难，个人感觉里面水有点深

纠结了些许天，终还是没能定下来。偶然看到有人搞树莓派，于是调转马头奔向这边来。一搜之后，顿觉豁然开朗。最早还是树莓派刚出的时候关注过一段时间，但是也没有开发板方面的需求，就逐渐淡忘了。想不到2019年它已经迎来了第四代产品，最重要的是相比上代性能上有了大幅提升，这不正是我所需要的吗！

# 准备

东搜未果，转而向宝，宝弄了个什么红包省钱卡，每天只能领一券，各种套路啊。瞄了瞄我的准树莓派4b，很认真的计算一下，此处省略一万字。。。

嫌麻烦的小伙伴就买套餐好了，到手万事俱备。笔者比较爱折腾，不喜被定制，采用了手动拼接方案：

* 树莓派4b裸板
* 全被动无风扇外壳
* tf卡
* 18w快充头
* type-c电源线
* tf读卡器
* micro HDMI转接头（非必须）

如果手头有闲置的手机快充头和type-c电源线，这个也可以省了，tf卡估计都曾经有过，找不到再买个。我这里除了读卡器其它都是买新，总共下来也就花了不足400块。

附上几个购买链接，扫码即可

* 树莓派4b

  ![raspberry4](images/raspberry4.png)

* 无风扇全被动散热外壳

  ![box](images/box.png)

# 新手上路

心心念念的树莓派4b终于拿到手了，真心够小巧，轻松掌握。赶紧官方教程走起，平时都用ubuntu，树莓派刚好也有，赶紧刷起来。不知道是micro HDMI转接头还是html到dvi的转接头有兼容性问题，反正显示器是没有收到信号，不管它了，文档里也有提无显示安装的一些操作步骤，照着来。话说这个ubuntu server还真不小，写卡很久，估计是我的读卡器还是老的usb 2.0
版。

插卡到树莓派，点亮，等了一会，然后尝试通过ssh去连接不成功，虽然刚才安装的时候按照文档配置了wifi，但是似乎没有成功。没关系，反正wifi又不重要，于是把树莓派接入路由器，查询得到树莓派ip然后ssh登录上去，成功，首次需要设置密码，然后就是各种安装配置。

经过不懈的努力终于将ubuntu的环境配置好了，谁知重启了之后ssh无法登录了，OMG。后来才知道是配置zsh为默认shell时修改/etc/passwd把zsh路径搞错了。

搞不定了只好重新又装了一遍。配好了环境，安装好了v2ray，剩下的就是在路由器上把特殊流量导向树莓派了。心想着改一下iptables规则就可以的事情远没有想象的那么简单，其实还是对网络这一块不了解啊。中间曲折的过程就不一一表述了，谁折腾谁知道。

# 进价

v2ray在树莓派中运行的中规中矩，这颗cpu也确实表现出了它的强大（与路由器相比）。但是考虑到v2ray是go语言性能方面与如火如荼的trojan相比可能存在一些劣势，于是又考虑用trojan取代v2ray的工作。

trojan之路也并非一帆风顺。树莓派上安装了trojan之后发现似乎并不支持端口复用，也就是说可能无法享受多线程的好处。查下来内核应该是已经支持了的，不知为何trojan会报warning，不完美。

此路不通，另辟蹊径。看了看树莓派支持的其它系统，不太感冒。想起油管上曾经看到过树莓派刷openwrt的，要不来个这个？官网看了已经支持树莓派4b了，不错，走起。不过笔者这里并没有用官方openwrt而是用的[lede](https://github.com/coolsnowwolf/lede)，也是支持树莓派4b的，集成了大家普遍需要的资源。我的主路由k2p也是从这里自编译的，编译的部分教程很多，利用好github。

# 行动

历尽千辛万苦终于将路由器和树莓派这对冤家成功的紧密联系在了一起。有人会说可以用树莓派做旁路由（也叫单臂路由）。是的，旁路由的确也是一种解决方案，笔者这里并没有采用是基于以下考虑：

* 目前openwrt对于树莓派4b的支持是snapshot，也就是说并不是很稳定，如果让树莓派做主路由一旦出现问题就是家里整个网络都受影响
* 旁路由的配置同样也不轻松
* 流量全部经过树莓派对于树莓派应该也会有一定压力，毕竟它不是为路由而生
* 对于笔者用的k2p支持硬件nat不用太可惜了

综合考虑之后，还是采取这种分而治之的策略，专业的人做专业的事。

# 效果

树莓派4b目前被笔者拿来当mini电脑了，播个视频、网页刷刷，简单家用比台式机省电多了～～～

与此同时又入手了一个(友善)NanoPi R2S来接替树莓派原本的工作，目前跑的是xray的xtls协议，再配合xtls-rprx-splice性能还是杠杠的！！！

目前家中为移动千兆宽带，在wifi连接下speedtest.net测速最高到300多mbps，此时主路由k2p的cpu负载不高（得益于路由表，无需socks转发），r2s负载50%～60%左右。不方便接有线，没法测试有线连接速度，个人推测还会有提升。基于此，对于千兆宽带的日常应用此方案已经完全满足，性价比超软路由。

# 具体方案

[看这里！ 看这里！ 看这里！](./guide.md)
