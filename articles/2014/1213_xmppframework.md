#iOS的xmppframework简介

2014-12-13

1登录和好友上下线

1.1XMPP中常用对象们

XMPPStream：xmpp基础服务类

XMPPRoster：好友列表类

XMPPRosterCoreDataStorage：好友列表（用户账号）在core data中的操作类

XMPPvCardCoreDataStorage：好友名片（昵称，签名，性别，年龄等信息）在core data中的操作类

XMPPvCardTemp：好友名片实体类，从数据库里取出来的都是它

xmppvCardAvatarModule：好友头像

XMPPReconnect：如果失去连接,自动重连

XMPPRoom：提供多用户聊天支持

XMPPPubSub：发布订阅



1.2登录操作，也就是连接xmpp服务器
<pre lang="objc" line="1">- (void)connect {
    if (self.xmppStream == nil) {
        self.xmppStream = [[XMPPStream alloc] init];
        [self.xmppStream addDelegate:self delegateQueue:dispatch_get_main_queue()];
    }
    if (![self.xmppStream isConnected]) {
        NSString *username = [[NSUserDefaults standardUserDefaults] objectForKey:@"username"];
        XMPPJID *jid = [XMPPJID jidWithUser:username domain:@"lizhen" resource:@"Ework"];
        [self.xmppStream setMyJID:jid];
        [self.xmppStream setHostName:@"10.4.125.113"];
        NSError *error = nil;
        if (![self.xmppStream connect:&error]) {
            NSLog(@"Connect Error: %@", [[error userInfo] description]);
        }
    }
}</pre>
connect成功之后

会依次调用XMPPStreamDelegate的方法，

首先调用
<pre>- (void)xmppStream:(XMPPStream *)sender socketDidConnect:(GCDAsyncSocket *)socket</pre>

然后
<pre>
- (void)xmppStreamDidConnect:(XMPPStream *)sender</pre>

在该方法下面需要使用xmppStream 的authenticateWithPassword方法进行密码验证，成功的话会响应delegate的方法，就是下面这个

<pre>- (void)xmppStreamDidAuthenticate:(XMPPStream *)sender</pre>

1.3上线

实现 - (void)xmppStreamDidAuthenticate:(XMPPStream *)sender 委托方法
<pre lang="objc" line="1">- (void)xmppStreamDidAuthenticate:(XMPPStream *)sender {
    XMPPPresence *presence = [XMPPPresence presenceWithType:@"available"];
    [self.xmppStream sendElement:presence];
}</pre>
1.4退出并断开连接
<pre lang="objc" line="1">- (void)disconnect {
    XMPPPresence *presence = [XMPPPresence presenceWithType:@"unavailable"];
    [self.xmppStream sendElement:presence];

    [self.xmppStream disconnect];
}</pre>

1.5好友状态

获取好友状态，通过实现
<pre>- (void)xmppStream:(XMPPStream *)sender didReceivePresence:(XMPPPresence *)presence</pre>
方法

当接收到 <presence /> 标签的内容时，XMPPFramework 框架回调该方法

一个 <presence /> 标签的格式一般如下：
<pre lang="xml" line="1">
&lt;presence from=""&gt;
　　&lt;show&gt;这里是显示的内容&lt;show /&gt;
　　&lt;status&gt;这里是显示的状态&lt;status /&gt;
&lt;presence /&gt;</pre>
presence 的状态：

available 上线

away 离开

do not disturb 忙碌

unavailable 下线
<pre lang="objc" line="1">- (void)xmppStream:(XMPPStream *)sender didReceivePresence:(XMPPPresence *)presence {
    NSString *presenceType = [presence type];
    NSString *presenceFromUser = [[presence from] user];
    if (![presenceFromUser isEqualToString:[[sender myJID] user]]) {
        if ([presenceType isEqualToString:@"available"]) {
            //
        } else if ([presenceType isEqualToString:@"unavailable"]) {
            //
        }
    }
}</pre>
2接收消息和发送消息
2.1接收消息

通过实现
- (void)xmppStream:(XMPPStream *)sender didReceiveMessage:(XMPPMessage *)message;
方法

当接收到 <message /> 标签的内容时，XMPPFramework 框架回调该方法

根据 XMPP 协议，消息体的内容存储在标签 <body /> 内
<pre lang="objc" line="1">- (void)xmppStream:(XMPPStream *)sender didReceiveMessage:(XMPPMessage *)message {
    NSString *messageBody = [[message elementForName:@"body"] stringValue];
}</pre>
2.2发送消息

发送消息，我们需要根据 XMPP 协议，将数据放到 <message /> 标签内，例如：
<pre lang="objc" line="1">&lt;message type="chat" to="xiaoming@example.com"&gt;
　　&lt;body&gt;Hello World!&lt;body /&gt;
&lt;message /&gt;</pre>
<pre lang="objc" line="1">- (void)sendMessage:(NSString *) message toUser:(NSString *) user {
    NSXMLElement *body = [NSXMLElement elementWithName:@"body"];
    [body setStringValue:message];
    NSXMLElement *message = [NSXMLElement elementWithName:@"message"];
    [message addAttributeWithName:@"type" stringValue:@"chat"];
    NSString *to = [NSString stringWithFormat:@"%@@example.com", user];
    [message addAttributeWithName:@"to" stringValue:to];
    [message addChild:body];
    [self.xmppStream sendElement:message];
}</pre>
3获取好友信息和删除好友
3.1好友列表和好友名片
<pre lang="objc" line="1">[_xmppRoster fetchRoster];//获取好友列表
//获取到一个好友节点
- (void)xmppRoster:(XMPPRoster *)sender didRecieveRosterItem:(NSXMLElement *)item
//获取完好友列表
- (void)xmppRosterDidEndPopulating:(XMPPRoster *)sender
//到服务器上请求联系人名片信息
- (void)fetchvCardTempForJID:(XMPPJID *)jid;
//请求联系人的名片，如果数据库有就不请求，没有就发送名片请求
- (void)fetchvCardTempForJID:(XMPPJID *)jid ignoreStorage:(BOOL)ignoreStorage;
//获取联系人的名片，如果数据库有就返回，没有返回空，并到服务器上抓取
- (XMPPvCardTemp *)vCardTempForJID:(XMPPJID *)jid shouldFetch:(BOOL)shouldFetch;
//更新自己的名片信息
- (void)updateMyvCardTemp:(XMPPvCardTemp *)vCardTemp;
//获取到一盒联系人的名片信息的回调
- (void)xmppvCardTempModule:(XMPPvCardTempModule *)vCardTempModule
        didReceivevCardTemp:(XMPPvCardTemp *)vCardTemp
                     forJID:(XMPPJID *)jid</pre>
3.2添加好友
<pre lang="objc" line="1">//name为用户账号
    - (void)XMPPAddFriendSubscribe:(NSString *)name
    {
        //XMPPHOST 就是服务器名，  主机名
        XMPPJID *jid = [XMPPJID jidWithString:[NSString stringWithFormat:@"%@@%@",name,XMPPHOST]];
        //[presence addAttributeWithName:@"subscription" stringValue:@"好友"];
        [xmppRoster subscribePresenceToUser:jid];

    }</pre>

3.3收到添加好友的请求
 <pre lang="objc" line="1">- (void)xmppRoster:(XMPPRoster *)sender didReceivePresenceSubscriptionRequest:(XMPPPresence *)presence
    {
        //取得好友状态
        NSString *presenceType = [NSString stringWithFormat:@"%@", [presence type]]; //online/offline
        //请求的用户
        NSString *presenceFromUser =[NSString stringWithFormat:@"%@", [[presence from] user]];
        NSLog(@"presenceType:%@",presenceType);
        NSLog(@"presence2:%@  sender2:%@",presence,sender);
        XMPPJID *jid = [XMPPJID jidWithString:presenceFromUser];
        //接收添加好友请求
        [xmppRoster acceptPresenceSubscriptionRequestFrom:jid andAddToRoster:YES];
    }</pre>
3.4删除好友

//删除好友，name为好友账号
<pre lang="objc" line="1">- (void)removeBuddy:(NSString *)name
{
    XMPPJID *jid = [XMPPJID jidWithString:[NSString stringWithFormat:@"%@@%@",name,XMPPHOST]];

    [self xmppRoster] removeUser:jid];
}</pre>
4聊天室
    //初始化聊天室
<pre lang="objc" line="1">
    XMPPJID *roomJID = [XMPPJID jidWithString:ROOM_JID];

    xmppRoom = [[XMPPRoom alloc] initWithRoomStorage:self jid:roomJID];

    [xmppRoom activate:xmppStream];
    [xmppRoom addDelegate:self delegateQueue:dispatch_get_main_queue()];</pre>
 //创建聊天室成功
<pre lang="objc" line="1">
 -(void)xmppRoomDidCreate:(XMPPRoom *)sender
    {
        DDLogInfo(@"%@: %@", THIS_FILE, THIS_METHOD);
    }</pre>
//加入聊天室，使用昵称
   <pre lang="objc" line="1"> [xmppRoom joinRoomUsingNickname:@"quack" history:nil];</pre>
//获取聊天室信息
<pre lang="objc" line="1">
 - (void)xmppRoomDidJoin:(XMPPRoom *)sender
    {
        [xmppRoom fetchConfigurationForm];
        [xmppRoom fetchBanList];
        [xmppRoom fetchMembersList];
        [xmppRoom fetchModeratorsList];
    }</pre>

如果房间存在，会调用委托
<pre lang="objc" line="1">
    // 收到禁止名单列表
    - (void)xmppRoom:(XMPPRoom *)sender didFetchBanList:(NSArray *)items;
    // 收到好友名单列表
    - (void)xmppRoom:(XMPPRoom *)sender didFetchMembersList:(NSArray *)items;
    // 收到主持人名单列表
    - (void)xmppRoom:(XMPPRoom *)sender didFetchModeratorsList:(NSArray *)items;</pre>
房间不存在，调用委托
<pre lang="objc" line="1">
    - (void)xmppRoom:(XMPPRoom *)sender didNotFetchBanList:(XMPPIQ *)iqError;
    - (void)xmppRoom:(XMPPRoom *)sender didNotFetchMembersList:(XMPPIQ *)iqError;
    - (void)xmppRoom:(XMPPRoom *)sender didNotFetchModeratorsList:(XMPPIQ *)iqError;</pre>
离开房间
<pre lang="objc" line="1">
[xmppRoom deactivate:xmppStream];</pre>
XMPPRoomDelegate的其他代理方法

    //离开聊天室
<pre lang="objc" line="1">- (void)xmppRoomDidLeave:(XMPPRoom *)sender
    {
        DDLogVerbose(@"%@: %@", THIS_FILE, THIS_METHOD);
    }</pre>
//新人加入群聊
<pre lang="objc" line="1">- (void)xmppRoom:(XMPPRoom *)sender occupantDidJoin:(XMPPJID *)occupantJID
    {
        DDLogVerbose(@"%@: %@", THIS_FILE, THIS_METHOD);
    }
</pre>
//有人退出群聊
<pre lang="objc" line="1"> - (void)xmppRoom:(XMPPRoom *)sender occupantDidLeave:(XMPPJID *)occupantJID
    {
        DDLogVerbose(@"%@: %@", THIS_FILE, THIS_METHOD);
    }</pre>

//有人在群里发言
<pre lang="objc" line="1"> - (void)xmppRoom:(XMPPRoom *)sender didReceiveMessage:(XMPPMessage *)message fromOccupant:(XMPPJID *)occupantJID
    {
        DDLogVerbose(@"%@: %@", THIS_FILE, THIS_METHOD);
    }</pre>
5消息回执
这个是XEP－0184协议的内容

协议内容：

发送消息时附加回执请求
<pre lang="objc" line="1">
    &lt;message
    from='northumberland@shakespeare.lit/westminster'
    id='richard2-4.1.247'
    to='kingrichard@royalty.england.lit/throne'&gt;
        &lt;body&gt;My lord, dispatch; read o'er these articles.&lt;/body&gt;
        &lt;request xmlns='urn:xmpp:receipts'/&gt;
    &lt;/message&gt;</pre>
代码实现
<pre lang="objc" line="1">
    NSString *siID = [XMPPStream generateUUID];
    //发送消息
    XMPPMessage *message = [XMPPMessage messageWithType:@"chat" to:jid elementID:siID];
    NSXMLElement *receipt = [NSXMLElement elementWithName:@"request" xmlns:@"urn:xmpp:receipts"];
    [message addChild:receipt];
    [message addBody:@"测试"];
    [self.xmppStream sendElement:message];</pre>
收到回执请求的消息，发送回执
<pre lang="objc" line="1">
    &lt;message
    from='kingrichard@royalty.england.lit/throne'
    id='bi29sg183b4v'
    to='northumberland@shakespeare.lit/westminster'&gt;
        &lt;received xmlns='urn:xmpp:receipts' id='richard2-4.1.247'/&gt;
    &lt;/message&gt;</pre>


代码实现
<pre lang="objc" line="1">- (void)xmppStream:(XMPPStream *)sender didReceiveMessage:(XMPPMessage *)message
    {
        //回执判断
        NSXMLElement *request = [message elementForName:@"request"];
        if (request)
        {
            if ([request.xmlns isEqualToString:@"urn:xmpp:receipts"])//消息回执
            {
                //组装消息回执
                XMPPMessage *msg = [XMPPMessage messageWithType:[message attributeStringValueForName:@"type"] to:message.from elementID:[message attributeStringValueForName:@"id"]];
                NSXMLElement *recieved = [NSXMLElement elementWithName:@"received" xmlns:@"urn:xmpp:receipts"];
                [msg addChild:recieved];

                //发送回执
                [self.xmppStream sendElement:msg];
            }
        }else
        {
            NSXMLElement *received = [message elementForName:@"received"];
            if (received)
            {
                if ([received.xmlns isEqualToString:@"urn:xmpp:receipts"])//消息回执
                {
                    //发送成功
                    NSLog(@"message send success!");
                }
            }
        }

        //消息处理
        //...  </pre>
6添加AutoPing
为了监听服务器是否有效，增加心跳监听。用XEP-0199协议，在XMPPFrameWork框架下，封装了 XMPPAutoPing 和 XMPPPing两个类都可以使用，因为XMPPAutoPing已经组合进了XMPPPing类，所以XMPPAutoPing使用起来更方便。
<pre lang="objc" line="1">//初始化并启动ping
-(void)autoPingProxyServer:(NSString*)strProxyServer
{
    _xmppAutoPing = [[XMPPAutoPingalloc] init];
    [_xmppAutoPingactivate:_xmppStream];
    [_xmppAutoPingaddDelegate:selfdelegateQueue:  dispatch_get_main_queue()];
    _xmppAutoPing.respondsToQueries = YES;
    _xmppAutoPing.pingInterval=2;//ping 间隔时间
    if (nil != strProxyServer)
    {
       _xmppAutoPing.targetJID = [XMPPJID jidWithString: strProxyServer ];//设置ping目标服务器，如果为nil,则监听socketstream当前连接上的那个服务器
    }
}
//卸载监听
 [_xmppAutoPing   deactivate];
 [_xmppAutoPing   removeDelegate:self];
 _xmppAutoPing = nil;
//ping XMPPAutoPingDelegate的委托方法:
- (void)xmppAutoPingDidSendPing:(XMPPAutoPing *)sender
{
    NSLog(@"- (void)xmppAutoPingDidSendPing:(XMPPAutoPing *)sender");
}
- (void)xmppAutoPingDidReceivePong:(XMPPAutoPing *)sender
{
    NSLog(@"- (void)xmppAutoPingDidReceivePong:(XMPPAutoPing *)sender");
}

- (void)xmppAutoPingDidTimeout:(XMPPAutoPing *)sender
{
    NSLog(@"- (void)xmppAutoPingDidTimeout:(XMPPAutoPing *)sender");
}</pre>


如果想要学习更多XMPPFramework的东西，我以前看过一个比较全面的demo，地址：<a href="http://code4app.com/ios/%E5%8D%B3%E6%97%B6%E9%80%9A%E8%AE%AF%E7%BE%A4%E8%81%8A%E7%B3%BB%E7%BB%9F1.2/535651f2933bf0647d8b570f" title="即时通讯群聊系统">即时通讯群聊系统</a><!--more-->
