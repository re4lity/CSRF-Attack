# CSRF-Attack

## 配置

软件：Vmware Workstation 12 Player

系统：SEEDUbuntu12.04

网络：NAT

其他：

1. Firefox浏览器(安装好LiveHTTPHeaders插件，用于检查HTTP请求和响应)；
2. Apache网络服务器；
3. Elgg网页应用

### Starting the Apache Server

开启Apache服务器很简单，因为已经预装在SEEDUbuntu中了，只需要使用一下命令开启就行：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image2.png)

### The Elgg Web Application

这是一个开源基于网页的社交应用，同样已经预装在SEEDUbuntu里，并且创建了一些用户，信息如下：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image3.png)

###配置DNS

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image4.png)

这个项目需要用到以上两个URL，启动Apache之后就可以在虚拟机内部访问到了，因为在hosts文件中已经配置了这两个URL是指向localhost的：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image5.png)

###配置Apache服务器

项目中使用Apache服务器作为网站的host，借助name-based virtual hosting技术，可以在同一台主机上host多个网站。配置文件default在/etc/apache2/sites-available/路径下：

![配置](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image6.png)

可以看到Apache用的是80端口，每个网站都有一个VirtualHost块，指定网站的URL和对应的资源目录。比如上面这个block，就绑定了[www.CSRFLabElgg.com](www.CSRFLabElgg.com)和/var/www/CSRF/elgg路径，修改该路径下的文件就能改变网站的配置。

## 什么是CSRF攻击

CSRF全称Cross-Site Request Forgery，是跨站请求伪造攻击的意思，这里面有两个关键词，一个是跨站，一个是伪造。前者说明CSRF攻击发生时所伴随的请求的来源，后者说明了该请求的产生方式。所谓伪造即该请求并不是用户本身的意愿，而是由攻击者构造，由受害者被动发出的。流程如下：

![CSRF](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image7.png)

用户首先登录了一个普通网站A，这时新的session就会被创建，用户的浏览器会自动存储cookie信息以及session id，这样短期内再打开网站或者网站的其他链接，用户就不需要重复登陆了。如果此时，用户访问了攻击者制作的恶意网站B，攻击者设置的对A的跨站请求就会被触发，并且由于浏览器的机制，cookie信息和session id会被附加到这个请求中(即使它是跨站的)。于是，A就会以为这个请求是用户发出的从而执行请求内容。只要稍加设计，攻击者就能利用该漏洞进行盗取用户隐私信息，篡改用户数据等行为。

具体来说，恶意网站可以伪造HTTP的GET和POST请求，在HTML的img,iframe,frame等标签中可以发起GET请求，而form标签可以发起POST请求。前者相对简单，后者需要用到JavaScript的技术。因为Elgg只使用POST，所以实验中会涉及到HTTP POST请求和JavaScript的使用。

## CSRF Ataack using GET Request

这个任务要求通过CSRF手段伪造GET请求，通过引诱对方打开恶意网站来实现GET请求的发送，并借助受害者浏览器保存的Cookie进行伪装。

假设矮穷矬Boby想和白富美Alice成为好友，但Alice不同意，于是Boby灵机一动，决定通过CSRF的手段强制让Alice加自己好友。具体来说Boby给Alice发了一个URL，Alice对此表示好奇，就点开了这个URL，此时她就会在不知情的情况下把Boby添加为好友了。

要实现这样的功能首先要了解Elgg中添加好友的机制：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image8.png)

打开两个浏览器应用，分别登录Alice和Boby的账户。要添加对方为好友可以先在导航栏找到More，然后点击Members选项来到Members页面，这里可以看到全部用户：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image9.png)

找到Boby后，点击进入Boby的Profile页面，可以看到头像下方有一个Add friend按钮，点击后就会自动添加Boby为好友了，我们需要观察这个动作是如何实现的。

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image10.png)

要了解Add friend触发什么请求我们可以使用LiveHttpHeader这个插件，SEEDUbuntu已经预装好，在Firefox菜单栏的Tools选项卡中选择LiveHttpHeader就可以打开，然后勾选Capture，即可进行抓包：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image11.png)

点击Add friend后，查看Live HTTP headers的抓包结果，可以清晰地看到这是一个GET请求，高亮的部分就是发出的链接。于是，我们需要做的就是把这个GET请求放在自己做的恶意网站中，并且把网站URL发给Alice，Alice点击后，这个跨站点的GET请求会被触发，浏览器识别出这是向Elgg发送的GET请求，于是自动把Cookie和Session ID附加上去，这时Elgg的服务器就会认为这个GET请求是Alice自己发出的，把Boby添加为Alice的朋友，Boby龌龊的计划就成功了。

### 具体实现

在/var/www/CSRF/Attacker/路径下创建task1.html：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image12.png)

编写一个简单的HTML网页，借助img标签的src属性来发起跨站请求，把刚刚抓包得到的链接填进去：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image13.png)

然后把这个网站的URL(http://www.csrflabattacker.com/task1.html)发给Alice，Alice点击后img标签自动触发GET请求，于是刷新一下就会发现Boby已经变成Alice的friend了：

![CSRF1](https://raw.githubusercontent.com/familyld/CSRF-Attack/master/graph/image14.png)

