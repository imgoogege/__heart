# vps配置科学上网]shadowsocks服务端配置,事实证明centos是linux中最好用的。

标签: 垃圾箱

```
1. Update 2015/03/30 竟然乱码...整个重新整理编辑了一下。非专业行为，如有不当之处，欢迎指出。
2. Update 2015/04/05 linux可参考Fedora21[配置shadowsocks for chrome](http://blog.csdn.net/bit_Line/article/details/44888213)，用于配置客户端，Debian系列/Red Hat系列可通用。
相比Python版本的ss客户端，会轻量级一些。windows用户客户端安装很方便，直接下载后一路“下一步”，下载见：
[shadowsocks-c#版](http://sourceforge.net/projects/shadowsocksgui/files/dist/)win7/XP或者已安装.Net2.0的用户，下载Shadowsocks-win-x.x.x.zip; Win8+/.Net4.0下载Shadowsocks-win-dotnet4.0-x.x.x.zip即可。
最后只需要配置参数（包括linux和windows），请直接参看第3条更新中的链接文章。
3. Update 2015/04/05 关于各处的参数设置，我特地写了一篇博客，参看 【[shadowsocks参数设置说明](http://blog.csdn.net/bit_Line/article/details/44888841)】
4. Update 2015/06/16 添加了被“墙”的网站样例。如果读者遇到任何相关疑问，都可随时联系窝哟~
5. Update 2016/06/10 Migirate to github gist :(
```

## 第0章 什么是“墙”？什么是“翻墙”？为甚么要“翻墙”？如何“翻墙”？
> Pre: 基于很多人对这几个问题了解并不多，这里做一个简单的介绍，希望能让你有一个大概的印象，然后就知道如何自己去查找资料补充更多的姿势了。

- Q1:什么是“墙”？
 - 【摘自维基百科-防火长城条目】
  防火长城（英语：Great Firewall of China，常用简称：GFW，中文也称中国国家防火墙或防火长城[1]，中国大陆民众俗称防火墙[2]），是对中国政府在其互联网边界审查系统（包括相关行政审查系统）的统称。此系统起步于1998年[3]，其英文名称得自于2002年5月17日Charles R. Smith所写的一篇关于中国网络审查的文章《The Great Firewall of China》[4]，取与Great Wall（长城）相谐的效果，简写为Great Firewall，缩写GFW[5]。随着使用的拓广，中文“墙”和英文“GFW”有时也被用作动词，网友所说的“被墙”即指被防火长城所屏蔽。
 - 【另有：中国大陆会被屏蔽的网站】
 - 【附-一份pac列表，内有截至2015-06-16的几乎所有被屏蔽域名(查看密码为pac)：[pacList](http://pastebin.centos.org/46591/54782614/)】
- Q2: 什么是“翻墙”？
 - 【摘自维基百科-突破网络审查条目】
  突破网络审查或突破网络封锁，俗称翻墙（穿墙）或破网[1]，是指针对互联网审查封锁的限制，绕过相应的IP封锁、端口封锁、内容过滤、域名劫持等，实现对网络内容的访问。相应突破网络审查软件的叫法有：翻墙软件、破网软件、破墙软件和穿墙软件。
- Q3: 为什么要“翻墙”？
 - 承上，绝大多数人翻墙是因为很多曾经很常用的网站(如google、youtube[简称U2B]、twitter[推特]、国外众多新闻媒体的网站等)的被墙，为了取得相应的服务，只能翻墙。又如有段时间GMail被全面封禁，导致很多需要使用GMail的用户(比如正在申请出国留学的孩纸、有国际业务需求的公司等)完全无法使用GMail。还可导致大多使用了google字体的网站加载速度变慢。实际上即使只使用国内的搜索引擎等，依然会被过滤掉相当一部分信息。再如有安卓应用开发需求的盆友定然需要连接google来获取相应的API等，亦需翻墙才可。
- Q4: 如何翻墙？
 - 翻墙分为全局模式和局部模式。一般局部模式只影响浏览器或者用户指定的软件，灵活性较好，有SSH、socks代理(socks4/socks5)、IP代理等。全局模式则主要有VPN(比如应用广泛的OpenVPN，另外还有L2TP等)。但是，不管何种翻墙方式，其根本行为都是将要访问的数据加密传输，网络审查无法检查，得到数据包后本地解密，从而达到翻墙的目的。

 - P.S. 本文只介绍局部代理中的socks5.

## 第1章 利用shadowsocks搭建/配置服务端&本地代理
- shadowsocks是一个可穿透防火墙的快速代理。简称ss.
ss由曾经主要（还是就他一人？）由@clowwindy开发维护的一个基于socks5协议的开源项目，官网为shadowsocks.org(当然，没翻墙泥上不去的啦)，托管在github上。注意，另有一个收费服务shadowsocks.com，使用其作为客户端，但是和ss项目是两回事，很多人甚至把它们混淆，shadowsocks.com提供的服务似乎并不太好，窝在网上看到过很多负面评价，但是大部分却直接混淆了二者，怪罪于ss项目本身？！！感觉...真是...蛋碎碎的....
- 国内vps主要面向有搭建网站需求的用户，配置中规中矩，价格上也不便宜。如果你需要搭建自己的ss服务器，建议买一个国外的vps（虚拟主机，自己科普哦）。
 - 窝买的是bandwagonhost的一台128M RAM/5G硬盘空间/每月300G流量/年费7美刀，提供的操作系统是linux(ubuntu,fedora,centos等发行版均可, 自己选择安装)
- 以下关于配置的内容可以从ss的github页面获得----→[shadowsocks使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)
### 1.1 服务端配置
- 直接贴原文了。。。当然如果读者有任何与之有关的问题，都可以在下面提出来，这个博客我经常会来逛逛~

```
管理面板 => https://kiwivm.it7.net/main.php  
  
VPS物理  
**********************  
  
VPS主机  
root密码*************  
ssh端口*****  
  
IP地址**************  
开放端口***********  
密码***************  
Sock5代理端口1080  
加密方式AES-256-CFB  
[plain] view plain copy print?
/*********************************************************************************/  
[plain] view plain copy print?
可能有时候会发现CPU占有率超高，甚至被主机系统限制，并给出警告！可以：  
top命令查看CPU占有率  
解决rsyslogd占有率高:  
sed -i  's/^\$ModLoad imklog/#\$ModLoad imklog/g' /etc/rsyslog.conf  
  
shadowsocks server配置步骤：  
vim /etc/shadowsocks.json  
{  
    "server":"my ip",  
    "server_port":myport,  
    "local_port":1080,  
    "password":"my password",  
    "timeout":600,  
    "method":"aes-256-cfb"  
}
```

### 安装

Debian / Ubuntu:

> apt-get install python-pip
> && pip install shadowsocks

CentOS:

> yum install python-setuptools && easy_install && pip
> pip install shadowsocks

### 使用

> ssserver -p 443 -k password -m rc4-md5

如果要后台运行：

> sudo ssserver -p 443 -k password -m rc4-md5 --user nobody -d start

如果要停止：

> sudo ssserver -d stop

如果要检查日志：

> sudo less /var/log/shadowsocks.log

用 -h 查看所有参数。你也可以使用 [配置文件](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File) 进行配置。

- 服务器搭建: 建议选择 Ubuntu 14.04 LTS 作为服务器以便使用 [TCP Fast](https://github.com/clowwindy/shadowsocks/wiki/TCP-Fast-Open) Open。~~除非有明确理由，不建议用对新手不友好的 CentOS。~~

- 推荐使用以下 VPS：
  [Digital Ocean](https://www.digitalocean.com/?refcode=b1cddd149721) 自带的内核无需自己编译模块即可使用 [hybla](https://github.com/shadowsocks/shadowsocks/wiki/Optimizing-Shadowsocks) 算法
  [Linode](https://www.linode.com/?r=e7932c8b03f9abc8aab71663b90b689a676402d1) 功能强大，机房较多
  [Bandwagon Host](https://bandwagonhost.com/aff.php?pid=19) 提供廉价小内存vps

- 客户端
 - [Windows](https://github.com/shadowsocks/shadowsocks-csharp)
 - [OSX](https://github.com/shadowsocks/shadowsocks-iOS/wiki/Shadowsocks-for-OSX-Help)
 - [Android](https://github.com/shadowsocks/shadowsocks-android)
 - [iOS](https://github.com/shadowsocks/shadowsocks-iOS/wiki/Help)
 - [OpenWRT](https://github.com/shadowsocks/openwrt-shadowsocks)
 - 在你本地的 PC 或手机上使用图形客户端。具体使用参见它们的使用说明。
