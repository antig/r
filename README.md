target OS: isoft 4.0

attack OS: kali

attack tools: arpspoof/ettercap/driftnet/tcpdump/ferret/hamster/sslstrip/


**`文中所涉及工具和技术，只用于理论研究，务必不要用于非法用途`**
**`务必不要在生产环境中进行模拟，文中演示的被攻击主机和网站账号，均为自己所有`**

## 理论部分
#### arp攻击原理
    In computer networking, ARP spoofing, ARP cache poisoning, or ARP poison routing, is a technique by which an attacker sends (spoofed) Address Resolution Protocol (ARP) messages onto a local area network. Generally, the aim is to associate the attacker's MAC address with the IP address of another host, such as the default gateway, causing any traffic meant for that IP address to be sent to the attacker instead.
    ARP spoofing may allow an attacker to intercept data frames on a network, modify the traffic, or stop all traffic. Often the attack is used as an opening for other attacks, such as denial of service, man in the middle, or session hijacking attacks.

以上这段摘自维基百科，指出了arp攻击的原理，以及合理运用arp攻击能达到什么目的（网络中断，中间人，和session劫持）。
arp针对的是OSI网络模型的下三层，这就使得它的攻击范围局限在了以太网之中 ( 如何在交换机环境下进行arp毒化？可以参考[这篇文章][1] search "There are two methods to sniff traffic in a switched environment using ARP Poisoning" )。

#### 实验工具及介绍

    target OS: isoft 4.0
    attack OS: kali
    attack tools: arpspoof/ettercap/driftnet/tcpdump/ferret/hamster/sslstrip/

这些工具的各自用途

    其中isoft4.0是攻击承受者（网关设备同样是攻击承受者，但不在准备范畴之列），
    kali是攻击发起者，
    arpspoof和ettercap用来发起arp毒化（二者均可单独使用），
    driftnet用来嗅探isoft浏览过的图片，
    tcpdump用来抓包，
    ferret和hamster联合起来用来还原isoft上存活的session（会话劫持），
    sslstrip用来对付加密的https流量

如果愿意，当然也可以选择嗅探的cookie然后用插件注入到浏览器实现会话劫持。但是相对来说比较麻烦。
调研发现，tcpdump+ferret+hamster组合是目前能发现的最简单的方案（其中ferret需要单独安装，单独添加apt源）。 


----------


## 实践部分
#### 确认发起者，承受者，网关的ip
![kali的网关和ip][2]
![isoft的网关和ip][3]
**`务必再三确认网关，isoft和kali系统的ip地址，因为发起者要对这些异常流量造成的后果负责 ！`**

#### 最简单的断网攻击
断网攻击的原理和操作都是最简单的，就是利用arpspoof把isoft系统本要发往网关的流量引流到kali（事实上可以引流到任意主机，包括随意虚构的服务器），而kali丢弃这些流量，导致isoft断网。本质原因在于isoft找不到正确的网关
操作办法：
首先确认isoft可以正常访问网页
![正常访问][4]
发起攻击：
![dengdai6207_1484721957388_67.png][5]
再次检查，isoft已经无法正常上网
![已经断网][6]

#### 嗅探图片和密码
和断网的区别是，嗅探首先要确保网络通畅，应该把劫持到的流量发往正确的方向，所以首先打开ip转发
![ip转发][7]
再次用arpspoof发起arp毒化，这次isoft可以正常上网了
现在开始利用driftnet嗅探图片：
![driftnet][8]
好了，现在已经嗅探到了：
![pic][9]
利用-a参数指定后台模式，利用-d指定目录，本次把嗅探到的图片保存在/root/pic目录中，检查效果：
![嗅探到的图片][10]
看来运气不坏！这家伙在浏览美女图片 ~
下一步开始嗅探密码，这次用到的工具是老牌的ettercap，已经很老了，但如今锋利依旧：
![dengdai6207_1484722582368_11.png][11]
ok，密码到手，用户名admin，密码passw0rd

#### 会话劫持
开启毒化，确认ip转发已经打开
这次用到的工具是tcpdump，依然是一个很古老的工具了
![流量获取][12]
利用ferret工具开始分析流量，生成hanster.txt文件到当前目录
利用hamster架设代理服务器
![复原session][13]
设置浏览器代理：
![浏览器代理设置][14]
到此为止，已经可以在攻击机上复原isoft和服务器之间的session通信了
![复原session2][15]
利用会话劫持进入攻击目标的qq邮箱：
![进入邮箱][16]
发送，并在gmail中验证
![gmail][17]
干的漂亮！会话劫持成功，无须对方的密码，即可登录对方的工作邮箱，并任意查看，发送邮件给其他人。
经试验，此功能对于腾讯，百度等众多大公司服务器均适用


  [1]: http://www.harmonysecurity.com/files/HS-P004_ARPPoisoning.pdf
  [2]: http://antig.pub/usr/uploads/2017/01/1785050957.png
  [3]: http://antig.pub/usr/uploads/2017/01/792015734.png
  [4]: http://antig.pub/usr/uploads/2017/01/100449439.png
  [5]: http://antig.pub/usr/uploads/2017/01/1627904599.png
  [6]: http://antig.pub/usr/uploads/2017/01/1730173403.png
  [7]: http://antig.pub/usr/uploads/2017/01/847074702.png
  [8]: http://antig.pub/usr/uploads/2017/01/2655793905.png
  [9]: http://antig.pub/usr/uploads/2017/01/2457736441.png
  [10]: http://antig.pub/usr/uploads/2017/01/660852100.png
  [11]: http://antig.pub/usr/uploads/2017/01/281625370.png
  [12]: http://antig.pub/usr/uploads/2017/01/278554021.png
  [13]: http://antig.pub/usr/uploads/2017/01/3305130381.png
  [14]: http://antig.pub/usr/uploads/2017/01/1283469177.png
  [15]: http://antig.pub/usr/uploads/2017/01/969820934.png
  [16]: http://antig.pub/usr/uploads/2017/01/3115405916.png
  [17]: http://antig.pub/usr/uploads/2017/01/23697667.png
