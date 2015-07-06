#（译）WebViewJavascriptBridge－Obj-C和JavaScript互通消息的桥梁

2015-06-23

本文翻译自Marcus Westin的开源框架WebViewJavascriptBridge的readme,英文原文链接<a title="https://github.com/marcuswestin/WebViewJavascriptBridge" href="https://github.com/marcuswestin/WebViewJavascriptBridge">https://github.com/marcuswestin/WebViewJavascriptBridge</a>.

WebViewJavascriptBridge是Obj-C和JavaScript通过UIWebViews/WebViews互通消息的一个iOS/OSX的桥梁. 如果你喜欢WebViewJavascriptBridge,你可能也想check out<a title="WebViewProxy" href="https://github.com/marcuswestin/WebViewProxy">WebViewProxy</a>.

Obj-C和JavaScript原理简单说一下，Obj-C调用JavaScript很简单，可以通过webview的stringByEvaluatingJavaScriptFromString:方法调用JavaScript代码；JavaScript调用Obj-C，则是通过web view的代理方法shouldStartLoadWithRequest：来接收JavaScript的网络请求从而实现调用。
 <!--more-->


<strong><span style="color: #3d82c6;">正在使用WebViewJavascriptBridge的项目</span></strong>

有很多的公司和项目在使用WebViewJavascriptBridge.这个列表不完整,你可以随意添加.

<a title="Facebook Messenger" href="https://www.facebook.com/mobile/messenger">Facebook Messenger</a>

<a title="Facebook Paper" href="https://facebook.com/paper">Facebook Paper</a>

<a title="Yardsale" href="https://www.getyardsale.com/">Yardsale</a>

<a title="EverTrue" href="http://www.evertrue.com/">EverTrue</a>

<a title="Game Insight" href="http://www.game-insight.com/">Game Insight</a>

<a title="Altralogica" href="http://www.altralogica.it">Altralogica</a>

<a title="Sush.io" href="http://www.sush.io">Sush.io</a>

Flutterby Labs

JD Media's <a title="PLAudio" href="https://itunes.apple.com/us/app/ding-sheng-zhong-hua/id537273940?mt=8">鼎盛中华</a>

Dojo4's <a title="PLAudio" href="http://dojo4.github.io/imbed/">Imbed</a>

<a title="CareZone" href="https://carezone.com">CareZone</a>

<a title="Hemlig" href="http://www.hemlig.co">Hemlig</a>


<strong><span style="color: #3d82c6;">安装& 例子 (iOS & OSX)</span></strong>

首先打开Example Apps文件夹.打开iOS或者OSX的工程然后点击运行.

在你自己的工程中用WebViewJavascriptBridge:

1) 拖动 WebViewJavascriptBridge 文件夹到你的工程中.

在出现的对话框中, 取消 (我觉得应该是选中)"Copy items into destination group's folder" 并且 选择 "Create groups for any folders"

2) 引入头文件并且申明一个属性:

<pre lang="objc" style="background: #E8F2FB;">
#import "WebViewJavascriptBridge.h"
...
@property WebViewJavascriptBridge* bridge;
</pre>
3) 实例化WebViewJavascriptBridge并且带上一个UIWebView (iOS)或者WebView (OSX):
<pre lang="objc" style="background: #E8F2FB;">
self.bridge = [WebViewJavascriptBridge bridgeForWebView:webView handler:^(id data, WVJBResponseCallback responseCallback) {
	NSLog(@"Received message from javascript: %@", data);
	responseCallback(@"Right back atcha");
}];
</pre>
4) 首先从ObjC到javascript发送一些消息:
<pre lang="objc" style="background: #E8F2FB;">
[self.bridge send:@"Well hello there"];
[self.bridge send:[NSDictionary dictionaryWithObject:@"Foo" forKey:@"Bar"]];
[self.bridge send:@"Give me a response, will you?" responseCallback:^(id responseData) {
	NSLog(@"ObjC got its response! %@", responseData);
}];
</pre>
5) 然后,看看javascript这边:
<pre lang="objc" style="background: #E8F2FB;">
	function connectWebViewJavascriptBridge(callback) {
		if (window.WebViewJavascriptBridge) {
			callback(WebViewJavascriptBridge)
		} else {
			document.addEventListener('WebViewJavascriptBridgeReady', function() {
				callback(WebViewJavascriptBridge)
			}, false)
		}
	}

	connectWebViewJavascriptBridge(function(bridge) {

		/* Init your app here */

		bridge.init(function(message, responseCallback) {
			alert('Received message: ' + message)
			if (responseCallback) {
				responseCallback("Right back atcha")
			}
		})
		bridge.send('Hello from the javascript')
		bridge.send('Please respond to this', function responseCallback(responseData) {
			console.log("Javascript got its response", responseData)
		})
	})
</pre>

<strong><span style="color: #3d82c6;">Contributors & Forks</span></strong>

Contributors: <a title="https://github.com/marcuswestin/WebViewJavascriptBridge/graphs/contributors" href="https://github.com/marcuswestin/WebViewJavascriptBridge/graphs/contributors">https://github.com/marcuswestin/WebViewJavascriptBridge/graphs/contributors</a>

Forks:  <a title="https://github.com/marcuswestin/WebViewJavascriptBridge/network/members" href="https://github.com/marcuswestin/WebViewJavascriptBridge/network/members">https://github.com/marcuswestin/WebViewJavascriptBridge/network/members</a>

<strong><span style="color: #3d82c6;">API 参考</span></strong>


<strong>ObjC API</strong>

[WebViewJavascriptBridge bridgeForWebView: (UIWebView/WebView*)webview handler:(WVJBHandler)handler]
[WebViewJavascriptBridge bridgeForWebView:
(UIWebView/WebView*)webview webViewDelegate:
(UIWebViewDelegate*)webViewDelegate handler:(WVJBHandler)handler]

给web view创建一个javascript的桥梁.
假如javascript需要一个反馈,那么 WVJBResponseCallback 不能为 nil .
当然,通过 webViewDelegate:(UIWebViewDelegate*)webViewDelegate 你可以得到<a title="web view的生命周期事件" href="http://developer.apple.com/library/ios/documentation/uikit/reference/UIWebViewDelegate_Protocol/Reference/Reference.html">web view的生命周期事件</a>.

例子:
<pre lang="objc" style="background: #E8F2FB;">
	[WebViewJavascriptBridge bridgeForWebView:webView handler:^(id data, WVJBResponseCallback responseCallback) {
		NSLog(@"Received message from javascript: %@", data);
		if (responseCallback) {
			responseCallback(@"Right back atcha");
		}
	}]

	[WebViewJavascriptBridge bridgeForWebView:webView webViewDelegate:self handler:^(id data, WVJBResponseCallback responseCallback) { /* ... */ }];
</pre>
[bridge send:(id)data]
[bridge send:(id)data responseCallback:(WVJBResponseCallback)responseCallback]

发送一个消息给javascript. 并且在发送消息成功后可以通过 responseCallback block做出一 些反应.

例子:
<pre lang="objc" style="background: #E8F2FB;">
	[self.bridge send:@"Hi"];
	[self.bridge send:[NSDictionary dictionaryWithObject:@"Foo" forKey:@"Bar"]];
	[self.bridge send:@"I expect a response!" responseCallback:^(id responseData) {
		NSLog(@"Got response! %@", responseData);
	}];
</pre>
[bridge registerHandler:(NSString*)handlerName handler: (WVJBHandler)handler]

注册一个叫做 handlerName 的handler. javascript能够通过 WebViewJavascriptBridge.callHandler("handlerName") 调起这个handler.

例子:
<pre lang="objc" style="background: #E8F2FB;">
	[self.bridge registerHandler:@"getScreenHeight" handler:^(id data, WVJBResponseCallback responseCallback) {
		responseCallback([NSNumber numberWithInt:[UIScreen mainScreen].bounds.size.height]);
	}];
</pre>
[bridge callHandler:(NSString*)handlerName data:(id)data]
[bridge callHandler:(NSString*)handlerName data:(id)data responseCallback:(WVJBResponseCallback)callback]

调起javascript叫做 handlerName的handler. 在调用 handler成功后可以通过responseCallback block做出反应.

例子:
<pre lang="objc" style="background: #E8F2FB;">
	[self.bridge callHandler:@"showAlert" data:@"Hi from ObjC to JS!"];
	[self.bridge callHandler:@"getCurrentPageUrl" data:nil responseCallback:^(id responseData) {
		NSLog(@"Current UIWebView page URL is: %@", responseData);
	}];
</pre>

定义 bundle

WebViewJavascriptBridge 要求 WebViewJavascriptBridge.js.txt 文件嵌入到web view来创建一个在JS这边的桥梁.标准的实现是用 mainBundle 找到这个文件.如果你建立一个 静态库并且你把这个文件放在其他地方,你可以用下面的方法找 WebViewJavascriptBridge.js.txt 文件:

[WebViewJavascriptBridge bridgeForWebView:
(UIWebView/WebView*)webView webViewDelegate:
(UIWebViewDelegate*)webViewDelegate handler:(WVJBHandler)handler
resourceBundle:(NSBundle*)bundle

例子:
<pre lang="objc" style="background: #E8F2FB;">
[WebViewJavascriptBridge bridgeForWebView:_webView
                          webViewDelegate:self
                                  handler:^(id data, WVJBResponseCallback responseCallback) {
	NSLog(@"Received message from javascript: %@", data);
                                  }
                           resourceBundle:[NSBundle bundleWithURL:[[NSBundle mainBundle] URLForResource:@"ResourcesBundle" withExtension:@"bundle"]]
];
</pre>

<strong>Javascript API</strong>

document.addEventListener('WebViewJavascriptBridgeReady',
function onBridgeReady(event) { ... }, false)

 一直等待 WebViewJavascriptBridgeReady DOM 事件.
 
例子:


<pre lang="objc" style="background: #E8F2FB;">

	document.addEventListener('WebViewJavascriptBridgeReady', function(event) {
		var bridge = event.bridge
		// Start using the bridge
	}, false)
</pre>
bridge.init(function messageHandler(data, response) { ... })


初始化这个桥. 这会调起 'WebViewJavascriptBridgeReady' 的事件handler.
这个 messageHandler 函数会接收通过ObjC的 [bridge send:(id)data] 和 [bridge send:(id)data responseCallback:(WVJBResponseCallback)responseCallback] 方法 发送的所有消息.

如果ObjC发送消息时有WVJBResponseCallback block,那么可以通过response对象发送消 息.

例子:
<pre lang="objc" style="background: #E8F2FB;">
	bridge.init(function(data, responseCallback) {
		alert("Got data " + JSON.stringify(data))
		if (responseCallback) {
			responseCallback("Right back atcha!")
		}
	})
</pre>
bridge.send("Hi there!")
bridge.send({ Foo:"Bar" })
bridge.send(data, function responseCallback(responseData) { ...
})

给ObjC发送消息. 在发送成功后可以通过 responseCallback 函数做出反应.

 例子:
<pre lang="objc" style="background: #E8F2FB;">

	bridge.send("Hi there!")
	bridge.send("Hi there!", function(responseData) {
		alert("I got a response! "+JSON.stringify(responseData))
	})
</pre>
bridge.registerHandler("handlerName", function(responseData) { ... })

注册一个叫做 handlerName 的handler. ObjC能够通过 [bridge callHandler:"handlerName" data:@"Foo"] 和 [bridge callHandler:"handlerName" data:@"Foo" responseCallback:^(id responseData) { ... }] 两个方法调起这个handler.

例子:
<pre lang="objc" style="background: #E8F2FB;">

	bridge.registerHandler("showAlert", function(data) { alert(data) })
	bridge.registerHandler("getCurrentPageUrl", function(data, responseCallback) {
		responseCallback(document.location.toString())
	})
 </pre>
 
<strong><span style="color: #3d82c6;">iOS4 支持 (包括 JSONKit)</span></strong>

注:iOS4支持尚未在V2 +测试.

WebViewJavascriptBridge 默认使用 NSJSONSerialization .如果你需要iOS 4的支持,你可以 用<a titl2="JSONKit" href="https://github.com/johnezang/JSONKit/">JSONKit</a>,并且增加 USE_JSONKIT 的预处理宏到你的工程中.

转载请注翻译原文链接:<a title="http://www.coderyi.com/archives/751" href="http://www.coderyi.com/archives/751">http://www.coderyi.com/archives/751</a>
