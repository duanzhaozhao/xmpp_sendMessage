# xmpp_sendMessage
xmpp发送即时消息

XMPP即时通讯
Java领域的即时通信的解决方案可以考虑openfire+spark+smack。
XMPP的基本网络结构
XMPP中定义了三个角色，客户端，服务器，网关。通信能够在这三者的任意两个之间双向发生。服务器同时承担了客户端信息记录，连接管理和信息的路由功能。网关承担着与异构即时通信系统的互联互通，异构系统可以包括SMS（短信），MSN，ICQ等。基本的网络形式是单客户端通过TCP/IP连接到单服务器，然后在之上传输XML。
 
XMPP工作原理说明
1.	所有从一个client到另一个client的jabber消息和数据都要通过XMPP server
2.	client连接到server
3.	server利用本地目录系统的证书对其认证
4.	client制定目标地址，让server告知目标状态
5.	server查找，连接并进行相互认证
6.	client间进行交互
 
搭建Openfire服务器
首先，去官网下载一份服务器的代码或可执行文件。（直接下载exe，运行即可）
下载地址：http://www.igniterealtime.org/downloads/index.jsp  windows可以下载exe
一步一步安装到结束，安装完看到页面
 

恭喜，安装已经成功了。接下来我们对服务器进行一下配置：
 
当然选择中文了

 
这里的域名直接输入你电脑IP号就可以了，端口使用默认的

 
这里选择嵌入数据库，这样不需要自己配置
 
选择初始设置，下一步
 
这里输入你的邮箱和密码。切记这个就是你admin帐号的密码，一定要记住不要忘记了。这一步结束了，配置也已经完成了，接下来我们登录到服务器去看一下：输入http://127.0.0.1:9090就回车进入管理页面，也可以从服务器端的 Launch Admin   按钮进入，在里面填写你刚刚  admin的用户名和密码就OK了，进去以后可以对你的服务器进行管理了。
测试客户端Spark
可以直接用官网上的Spark来测试我们的实时信息，下载安装后登陆即可。
客户端（Smack）详解
在android上开发用的其实是移植版的类库asmack，需要先下载asmack.jar。
Smack是一个用于和XMPP服务器通信的类库，由此可以实现即时通讯和聊天。
1.	ConnectionConfiguration
作为用于与XMPP服务建立连接的配置。它能配置；连接是否使用TLS，SASL加密。
   包含内嵌类：ConnectionConfiguration.SecurityMode
2.XMPPConnection.
   XMPPConnection这个类用来连接XMPP服务。
   可以使用connect()方法建立与服务器的连接。disconnect()方法断开与服务器的连接.
   在创建连接前可以使用XMPPConnection.DEBUG_ENABLED = true; 使开发过程中可以弹出一个GUI窗口，用于显示我们的连接与发送Packet的信息。
3.ChatManager
   用于监控当前所有chat。可以使用createChat(String userJID, MessageListener listener)创建一个聊天。
4.Chat
   Chat用于监控两个用户间的一系列message。使用addMessageListener(MessageListener listener)当有任何消息到达时将会触发listener的processMessage(Chat chat, Message message)方法。
我们可以使用sendMessage()发送消息，这个方法有两个重载方法，一种类类型的参数时String类型，另一种则是传入Message对象。
5.Message
  Message用于表示一个消息包(可以用调试工具看到发送包和接收包的具体内容)。它有以下多种类型：
   Message.Type.NORMAL -- （默认）文本消息（比如邮件）。区别于chat，是在两个用户实体进行聊天外的消息内容，可以回复。可以做需要“回执”动作之类的消息。
   Message.Type.CHAT -- 典型的短消息，消息类型用于两个实体用户之间聊天。如QQ聊天的一行一行显示的消息。
   Message.Type.GROUP_CHAT -- 群聊消息类型
   Message.Type.HEADLINE -- 滚动显示的消息，即不可回复的消息类型（新闻, 体育, 市场信息, RSS feeds, 等等）
   Message.TYPE.ERROR -- 错误的消息
下面给出headline的消息类型的使用样例：
XMPPConnection connection = new XMPPConnection(jabber.org);
connection.login(user, password);
Message message = new Message();
message.setTo("tom@jabber.org");
message.setSubject("Server down");
message.setBody("server has just gone down");
message.setType(Message.Type.HEADLINE);
connection.sendPacket(message);
connection.close();
6.Roster
1）表示存储了很多RosterEntry的一个花名册。为了易于管理，花名册的项被分贝到了各个group中。
2）当建立与XMPP服务的连接后可以使用connection.getRoster()获取Roster对象。
3）别的用户可以使用一个订阅请求(相当于QQ加好友)尝试订阅目的用户。
可以使用枚举类型Roster.SubscriptionMode的值处理这些请求：
accept_all: 接收所有订阅请求
reject_all：拒绝所有订阅请求
manual：手工处理订阅请求
4）创建组：RosterGroup group = roster.createGroup("大学");
 向组中添加RosterEntry对象: group.addEntry(entry);
7.RosterEntry
   表示Roster（花名册）中的每条记录.它包含了用户的JID，用户名，或用户分配的昵称.
 
8.RosterGroup
   表示RosterEntry的组。可以使用addEntry(RosterEntry entry)添加。
 contains(String user) 判断某用户是否在组中。
 当然removeEntry(RosterEntry entry)就是从组中移除了。
 getEntries()获取所有RosterEntry。
 
9.Presence
   表示XMPP状态的packet。每个presence packet都有一个状态。用枚举类型Presence.Type的值表示：
available -- （默认）用户空闲状态
unavailable -- 用户没空看消息
subscribe -- 请求订阅别人，即请求加对方为好友
subscribed -- 统一被别人订阅，也就是确认被对方加为好友
unsubscribe -- 他取消订阅别人，请求删除某好友
unsubscribed -- 拒绝被别人订阅，即拒绝对放的添加请求
error -- 当前状态packet有错误
内嵌两个枚举类型:Presence.Mode和Presence.Type.
可以使用setStatus自定义用户当前的状态（像QQ一样的)。
可以用if(presence.isAvailable() == true) 来判断好友是否在线。



