(译)GCDWebServer概述－基于GCD的轻量级的HTTP服务器
========

11.12

[![Build Status](https://travis-ci.org/swisspol/GCDWebServer.svg?branch=master)](https://travis-ci.org/swisspol/GCDWebServer)
[![Version](http://cocoapod-badges.herokuapp.com/v/GCDWebServer/badge.png)](http://cocoadocs.org/docsets/GCDWebServer)
[![Platform](http://cocoapod-badges.herokuapp.com/p/GCDWebServer/badge.png)](https://github.com/swisspol/GCDWebServer)
[![License](http://img.shields.io/cocoapods/l/GCDWebServer.svg)](LICENSE)

GCDWebServer是一个现代化的轻量级的基于HTTP 1.1的GCD server，它主要用于嵌入OS X & iOS apps。它开始编写时考虑下面几个目标：

* 优雅，易于使用的架构，只有4个核心类: server, connection, request and response (可以看下面的 "了解GCDWebServer的架构" )
* 精心设计的API以及完整的文档让你方便集成和自定义
* 为了更好的性能和并发通过[Grand Central Dispatch](http://en.wikipedia.org/wiki/Grand_Central_Dispatch)完全建立在事件驱动设计之上
* 没有依赖第三方库
* 在 [New BSD License](LICENSE)下是可用的

额外的内置功能:

* 允许执行传入的HTTP请求完全异步处理程序
* 尽量减少磁盘流大的HTTP请求或响应主体的内存使用
* [web forms](http://www.w3.org/TR/html401/interact/forms.html#h-17.13.4)提交的解析器用"application/x-www-form-urlencoded" or "multipart/form-data"编码（包括文件上传）
* [JSON](http://www.json.org/) 解析和序列化，主要是给 request and response HTTP bodies
* [Chunked transfer encoding](https://en.wikipedia.org/wiki/Chunked_transfer_encoding) for request and response HTTP bodies
* [HTTP compression](https://en.wikipedia.org/wiki/HTTP_compression) with gzip for request and response HTTP bodies
* [HTTP range](https://en.wikipedia.org/wiki/Byte_serving) support for requests of local files
* [Basic](https://en.wikipedia.org/wiki/Basic_access_authentication) and [Digest Access](https://en.wikipedia.org/wiki/Digest_access_authentication) authentications for password protection
* 自动处理iOS apps中前台，后台，挂起模式之间的转换
* 支持 IPv4 和 IPv6
* NAT端口映射 (只有IPv4)

包括的扩展:

* [GCDWebUploader](GCDWebUploader/GCDWebUploader.h):  ```GCDWebServer```的子类 ，that implements an interface for uploading and downloading files using a web browser 用web浏览器实现了上传和下载文件的接口
* [GCDWebDAVServer](GCDWebDAVServer/GCDWebDAVServer.h): ```GCDWebServer```的子类， 它实现了一个1级[WebDAV](https://en.wikipedia.org/wiki/WebDAV)服务器（与OS X的Finder部分2级支持）

不支持什么（但不是真正从一个嵌入式HTTP服务器所需的）：

* 保持连接
* HTTPS

要求:

* OS X 10.7 or later (x86_64)
* iOS 5.0 or later (armv7, armv7s or arm64)
* ARC memory management only (if you need MRC support use GCDWebServer 3.1 and earlier)

入门
===============
下载或者check out[最新发布版](https://github.com/swisspol/GCDWebServer/releases)的GCDWebServer，然后添加整个“GCDWebServer”子文件夹到你的Xcode项目。如果您打算使用像GCDWebDAVServer或GCDWebUploader扩展之一，添加这些子文件夹。

另外，您也可以使用[CocoaPods](http://cocoapods.org/)通过简单地添加这一行到您的Podfile GCDWebServer安装：

```
pod "GCDWebServer", "~> 3.0"
```
如果你想使用GCDWebUploader，使用下面的替代：
```
pod "GCDWebServer/WebUploader", "~> 3.0"
```
使用GCDWebDAVServer则用下面的:
```
pod "GCDWebServer/WebDAV", "~> 3.0"
```

最后运行 `$ pod install`.

您还可以使用[Carthage](https://github.com/Carthage/Carthage)加入这一行到你的Cartfile（3.2.5第一次正式发布支持Carthage）：
```
github "swisspol/GCDWebServer" ~> 3.2.5
```

然后运行 `$ carthage update` 并且增加生成的框架到您的Xcode项目中 (see [Carthage instructions](https://github.com/Carthage/Carthage#adding-frameworks-to-an-application)).

帮助 & 支持
==============
为了帮助使用GCDWebServer，最好要问你的问题在Stack Overflow的[`gcdwebserver`](http://stackoverflow.com/questions/tagged/gcdwebserver)标签上。但请务必先阅读本自述！


For bug reports or enhancement requests, please use [GitHub issues](https://github.com/swisspol/GCDWebServer/issues) instead.对于bug报告或改进要求，请使用GitHub的[GitHub issues](https://github.com/swisspol/GCDWebServer/issues)替代。

Hello World
===========
这些代码片段展示了如何实现运行在8080端口上，并对任何请求返回一个“Hello World”HTML页面的自定义HTTP服务器。由于GCDWebServer使用GCD块来处理请求，不需要子类或代表，这会让你的代码非常干净。


**重要信息:** 假如你没有用CocoaPods, 确保你增加了 `libz` 系统库到你的 Xcode target.

**OS X version (command line tool):**

```objectivec
#import "GCDWebServer.h"
#import "GCDWebServerDataResponse.h"

int main(int argc, const char* argv[]) {
  @autoreleasepool {
    
    // Create server
    GCDWebServer* webServer = [[GCDWebServer alloc] init];
    
    // Add a handler to respond to GET requests on any URL
    [webServer addDefaultHandlerForMethod:@"GET"
                             requestClass:[GCDWebServerRequest class]
                             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
      
      return [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
      
    }];
    
    // Use convenience method that runs server on port 8080
    // until SIGINT (Ctrl-C in Terminal) or SIGTERM is received
    [webServer runWithPort:8080 bonjourName:nil];
    NSLog(@"Visit %@ in your web browser", webServer.serverURL);
    
  }
  return 0;
}
```

**iOS version:**
```objectivec
#import "GCDWebServer.h"
#import "GCDWebServerDataResponse.h"

@interface AppDelegate : NSObject <UIApplicationDelegate> {
  GCDWebServer* _webServer;
}
@end

@implementation AppDelegate

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
  
  // Create server
  _webServer = [[GCDWebServer alloc] init];
  
  // Add a handler to respond to GET requests on any URL
  [_webServer addDefaultHandlerForMethod:@"GET"
                            requestClass:[GCDWebServerRequest class]
                            processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
    
    return [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
    
  }];
  
  // Start server on port 8080
  [_webServer startWithPort:8080 bonjourName:nil];
  NSLog(@"Visit %@ in your web browser", _webServer.serverURL);
  
  return YES;
}

@end
```

**OS X Swift version (command line tool):**

***webServer.swift***
```swift
import Foundation
import GCDWebServers

func initWebServer() {

    let webServer = GCDWebServer()

    webServer.addDefaultHandlerForMethod("GET", requestClass: GCDWebServerRequest.self, processBlock: {request in
    return GCDWebServerDataResponse(HTML:"<html><body><p>Hello World</p></body></html>")
        
    })
    
    webServer.runWithPort(8080, bonjourName: "GCD Web Server")
    
    print("Visit \(webServer.serverURL) in your web browser")
}
```

***WebServer-Bridging-Header.h***
```objectivec
#import <GCDWebServers/GCDWebServer.h>
#import <GCDWebServers/GCDWebServerDataResponse.h>
```

Web Based Uploads in iOS Apps
=============================


GCDWebUploader是 ```GCDWebServer``` 的子类，提供了一个现成使用的HTML5文件上传和下载。用户使用一个干净的用户界面的网络浏览器就可以上传，下载，删除文件，以及你的iOS应用程序的沙箱中的一个目录创建目录，了。

简单地实例和运行一个```GCDWebUploader```实例，然后通过web浏览器访问```http://{YOUR-IOS-DEVICE-IP-ADDRESS}/``` 就可以了


```objectivec
#import "GCDWebUploader.h"

@interface AppDelegate : NSObject <UIApplicationDelegate> {
  GCDWebUploader* _webUploader;
}
@end

@implementation AppDelegate

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
  NSString* documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
  _webUploader = [[GCDWebUploader alloc] initWithUploadDirectory:documentsPath];
  [_webUploader start];
  NSLog(@"Visit %@ in your web browser", _webUploader.serverURL);
  return YES;
}

@end
```

WebDAV Server in iOS Apps
=========================


GCDWebDAVServer是 ```GCDWebServer```的子类 ，提供了1级标准的WebDAV服务器。用户使用WebDAV客户端就可以上传，下载，删除文件，以及你的iOS应用程序的沙箱中的一个目录创建目录，这很像 [Transmit](https://panic.com/transmit/) (Mac), [ForkLift](http://binarynights.com/forklift/) (Mac) or [CyberDuck](http://cyberduck.io/) (Mac / Windows).

GCDWebDAVServer也应与[OS X Finder](http://support.apple.com/kb/PH13859)一起工具，因为它是2级标准（但只有WebDAV实现的是OS X的客户端时才需要）。


简单地实例和运行一个```GCDWebDAVServer```实例，然后通过web浏览器访问```http://{YOUR-IOS-DEVICE-IP-ADDRESS}/``` 就可以了

```objectivec
#import "GCDWebDAVServer.h"

@interface AppDelegate : NSObject <UIApplicationDelegate> {
  GCDWebDAVServer* _davServer;
}
@end

@implementation AppDelegate

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
  NSString* documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
  _davServer = [[GCDWebDAVServer alloc] initWithUploadDirectory:documentsPath];
  [_davServer start];
  NSLog(@"Visit %@ in your WebDAV client", _davServer.serverURL);
  return YES;
}

@end
```

服务静态网站
========================


GCDWebServer包括一个内置的处理程序，可以递归服务目录（它也可以让你控制如何在["Cache-Control"](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)头应设置）：

**OS X version (command line tool):**
```objectivec
#import "GCDWebServer.h"

int main(int argc, const char* argv[]) {
  @autoreleasepool {
    
    GCDWebServer* webServer = [[GCDWebServer alloc] init];
    [webServer addGETHandlerForBasePath:@"/" directoryPath:NSHomeDirectory() indexFilename:nil cacheAge:3600 allowRangeRequests:YES];
    [webServer runWithPort:8080];
    
  }
  return 0;
}
```

使用 GCDWebServer
==================



首先创建的 ```GCDWebServer```类的一个实例。请注意，您可以在相同的应用程序运行多个Web服务器，只要它们监听在听不同的端口上。

然后添加一个或多个“handlers”的服务器：每个handler都有机会处理传入的Web请求，并提供响应。handlers被称为一个后进先出队列，因此最新添加的handler覆盖任何以前添加的。

最后你开始一个指定端口的服务器。



了解 GCDWebServer的架构
=========================================

GCDWebServer的体系结构包括只有4个核心类：

* [GCDWebServer](GCDWebServer/Core/GCDWebServer.h) 管理socket包括监听新的HTTP连接以及用于服务器的handlers列表
* [GCDWebServerConnection](GCDWebServer/Core/GCDWebServerConnection.h) 由 ```GCDWebServer``` 实例来处理每一个新的HTTP连接。每个实例都保持活着直到连接关闭。您不能直接使用这个类，但它公开的，所以你可以继承它。
* [GCDWebServerRequest](GCDWebServer/Core/GCDWebServerRequest.h)由```GCDWebServerConnection``` 实例创建在接收到HTTP headers后。它包括的request和处理HTTP body。 GCDWebServer带有```GCDWebServerRequest```的[几个子类](GCDWebServer/Requests)来处理常见的情况下，像存储机身内存，或者传输到磁盘上的文件。
* [GCDWebServerResponse](GCDWebServer/Core/GCDWebServerResponse.h)在收到response HTTP headers以及可选的body的时候创建。 GCDWebServer带有```GCDWebServerResponse```的[几个子类](GCDWebServer/Responses)来处理常见的情况下，像在内存中的HTML文本或流从磁盘上的文件。

实现 Handlers
=====================

GCDWebServer凭借“handlers”处理传入的Web请求，并生成响应。Handlers通过GCD块来实现，这使得它很容易使用。然而， __他们在任意线程GCD内执行，所以特别要注意线程安全和重入__.

Handlers需要2个GCD块：

* 该 ```GCDWebServerMatchBlock``` 被称为在每个handler添加到```GCDWebServer```实例每当一个Web请求开始（即HTTP头已收）。它是通过对Web请求（HTTP方法，URL，标题......）的基本信息，而且必须决​​定是否要处理与否。如果是，它必须返回一个新的 ```GCDWebServerRequest```实例（见上文）与此信息创建的。否则，它只是返回零。



* 该```GCDWebServerProcessBlock```或```GCDWebServerAsyncProcessBlock``` 调用在web请求已经被完全接收并传递在上一步中创建```GCDWebServerRequest```实例后。它必须返回同步（如果使用```GCDWebServerProcessBlock```）或异步（如果使用```GCDWebServerAsyncProcessBlock```）一个```GCDWebServerResponse```实例（见上文）或nil，否则这将导致在500的HTTP状态代码返回给客户端。建议返回到客户端上的错误可以返回[GCDWebServerErrorResponse](GCDWebServer/Responses/GCDWebServerErrorResponse.h) 的一个实例，以便更多有用的信息。






注意在 ```GCDWebServer```大多数方法来添加handlers只需要```GCDWebServerProcessBlock```或 ```GCDWebServerAsyncProcessBlock``` ，因为他们已经提供了一个内置的```GCDWebServerMatchBlock``` 如匹配一个正则表达式的URL路径。


异步HTTP响应
===========================

新的GCDWebServer3.0是有能力异步处理HTTP请求即添加handlers到服务器异步地生成自己```GCDWebServerResponse``` 。这是通过添加handlers使用一个```GCDWebServerAsyncProcessBlock```来代替```GCDWebServerProcessBlock```。下面是一个例子：


**（同步版本）**，产生的HTTP响应的handler块：


```objectivec
[webServer addDefaultHandlerForMethod:@"GET"
                         requestClass:[GCDWebServerRequest class]
                         processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
  
  GCDWebServerDataResponse* response = [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
  return response;
  
}];
```


**（异步版本）**handler直接返回，生产HTTP response后回调GCDWebServer：



```objectivec
[webServer addDefaultHandlerForMethod:@"GET"
                         requestClass:[GCDWebServerRequest class]
                    asyncProcessBlock:^(GCDWebServerRequest* request, GCDWebServerCompletionBlock completionBlock) {
  
  // Do some async operation like network access or file I/O (simulated here using dispatch_after())
  dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(3 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    GCDWebServerDataResponse* response = [GCDWebServerDataResponse responseWithHTML:@"<html><body><p>Hello World</p></body></html>"];
    completionBlock(response);
  });

}];
```


**（先进的异步版本）**handler立即返回流式HTTP响应，这本身异步产生其内容：



```objectivec
[webServer addDefaultHandlerForMethod:@"GET"
                         requestClass:[GCDWebServerRequest class]
                         processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
  
  NSMutableArray* contents = [NSMutableArray arrayWithObjects:@"<html><body><p>\n", @"Hello World!\n", @"</p></body></html>\n", nil];  // Fake data source we are reading from
  GCDWebServerStreamedResponse* response = [GCDWebServerStreamedResponse responseWithContentType:@"text/html" asyncStreamBlock:^(GCDWebServerBodyReaderCompletionBlock completionBlock) {
    
    // Simulate a delay reading from the fake data source
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
      NSString* string = contents.firstObject;
      if (string) {
        [contents removeObjectAtIndex:0];
        completionBlock([string dataUsingEncoding:NSUTF8StringEncoding], nil);  // Generate the 2nd part of the stream data
      } else {
        completionBlock([NSData data], nil);  // Must pass an empty NSData to signal the end of the stream
      }
    });
    
  }];
  return response;
  
}];
```

*需要注意的是，你甚至可以结合这两种异步和先进的异步版本，以异步返回异步HTTP响应！*

GCDWebServer和后台模式的iOS应用
===========================================




在做网络操作的iOS应用程序，你必须小心处理时的[iOS放入后台应用程序会发生什么](https://developer.apple.com/library/ios/technotes/tn2277/_index.html). 。通常情况下，你必须停止所有的网络服务器，应用程序在后台运行，并在回来到前台时重新启动这些应用程序。这可能会变得非常复杂的考虑服务器可能有持续的连接，他们需要停止的时候。


幸运的是，GCDWebServer做这一切自动为您：

- GCDWebServer开始[后台任务](https://developer.apple.com/library/ios/documentation/iphone/conceptual/iphoneosprogrammingguide/ManagingYourApplicationsFlow/ManagingYourApplicationsFlow.html)时的第一个HTTP连接打开，并且只有当最后一个被关闭时才结束它。这防止iOS的从暂停该应用在进入后台后，这会立即杀死到客户端的HTTP连接。
 - 当应用程序在后台，只要新的HTTP连接不断被启动，后台任务将继续存在和iOS不会暂停的应用程序（除非在突发和意外的内存压力）。
 - 当最后一个HTTP连接关闭如果应用程序仍然在后台，GCDWebServer将暂停本身并停止接受新的连接，如果你曾要求```-stop```（这种行为可以用```GCDWebServerOption_AutomaticallySuspendInBackground``` 选项被禁用）。
- 如果应用程序在后台但没有HTTP连接打开，GCDWebServer将立即暂停本身并停止接受新的连接，如果你曾要求```-stop```（这种行为可以用```GCDWebServerOption_AutomaticallySuspendInBackground```选项被禁用）。
- 如果应用程序回到前台和GCDWebServer已经暂停，它会自动恢复本身，并开始接受再新HTTP连接，如果您曾呼吁```-start```。


HTTP连接经常发起batches (or bursts)，例如，在装载具有多个资源的网页时。这使得难以精确地检测当最后的HTTP连接已经关闭：这是可能的2个连续的HTTP连接同批的部分将由一个小的延迟分离，而不是重叠。这将是坏的对于客户端，如果GCDWebServer暂停自己的权利没有掌控。该```GCDWebServerOption_ConnectedStateCoalescingInterval```选项解决这个问题是优雅的，它迫使GCDWebServer等待一段额外的延迟执行任何动作后，最后一个HTTP连接已关闭，以防万一这个延迟​​时间内启动一个新的HTTP连接之前。













登录GCDWebServer
=======================


无论是对调试和信息的目的，GCDWebServer记录消息的广泛每当有事情发生。此外，在“Debug”模式与“Release”模式建设GCDWebServer的时候，它会记录更多的信息，而且还进行了一些内部的一致性检查。要启用此行为，编译GCDWebServer当定义预处理器常量```DEBUG=1```。在Xcode target 设置，这可以通过增加```DEBUG=1```到内置的“Debug”配置构建时设置```GCC_PREPROCESSOR_DEFINITIONS```完成。最后，您还可以通过调用 ```+[GCDWebServer setLogLevel:]```控制日志记录的在运行时。


默认情况下，记录的GCDWebServer的所有邮件都发送到其内置的日志记录功能，它只是输出到```stderr```（假设终端类型设备连接）。为了更好地与因记录的信息量您的应用程序的其他部分或整合，你可能想使用另一种记录工具。


GCDWebServer自动支持[XLFacility](https://github.com/swisspol/XLFacility)（由同一作者的GCDWebServer也开源）和[CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack)。如果其中任何一个是在同一个Xcode项目，GCDWebServer应自动使用，替代内置的日志记录功能（见[GCDWebServerPrivate.h](GCDWebServer/Core/GCDWebServerPrivate.h)的实施细则）它。

它也可以使用自定义日志记录功能 - 看[GCDWebServer.h](GCDWebServer/Core/GCDWebServer.h)以获取更多信息。










高级示例1：实现HTTP重定向
===============================================

Here's an example handler that redirects "/" to "/index.html" using the convenience method on ```GCDWebServerResponse``` (it sets the HTTP status code and "Location" header automatically):

下面是一个例子handler重定向“/”到“/index.html”使用上```GCDWebServerResponse``` 的便捷方法（它会自动设置HTTP状态代码和“Location”header）：



```objectivec
[self addHandlerForMethod:@"GET"
                     path:@"/"
             requestClass:[GCDWebServerRequest class]
             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
    
  return [GCDWebServerResponse responseWithRedirect:[NSURL URLWithString:@"index.html" relativeToURL:request.URL]
                                          permanent:NO];
    
}];
```

高级示例2：实现Forms
======================================


要实现HTTP表单，你需要一对handlers：

* GET handler不期望在HTTP请求的任何body，因此使用 ```GCDWebServerRequest```类。handler生成包含一个简单的HTML表单的响应。

* 在POST handler期望的form值是在HTTP请求的body和percent-encoded。幸运的是，GCDWebServer提供请求类```GCDWebServerURLEncodedFormRequest``` 能够自动解析这样的body。handler简单地回显从用户提交的表单值。




```objectivec
[webServer addHandlerForMethod:@"GET"
                          path:@"/"
                  requestClass:[GCDWebServerRequest class]
                  processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
  
  NSString* html = @" \
    <html><body> \
      <form name=\"input\" action=\"/\" method=\"post\" enctype=\"application/x-www-form-urlencoded\"> \
      Value: <input type=\"text\" name=\"value\"> \
      <input type=\"submit\" value=\"Submit\"> \
      </form> \
    </body></html> \
  ";
  return [GCDWebServerDataResponse responseWithHTML:html];
  
}];

[webServer addHandlerForMethod:@"POST"
                          path:@"/"
                  requestClass:[GCDWebServerURLEncodedFormRequest class]
                  processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
  
  NSString* value = [[(GCDWebServerURLEncodedFormRequest*)request arguments] objectForKey:@"value"];
  NSString* html = [NSString stringWithFormat:@"<html><body><p>%@</p></body></html>", value];
  return [GCDWebServerDataResponse responseWithHTML:html];
  
}];
```

高级示例3：服务动态网站
=============================================



GCDWebServer提供的扩充类```GCDWebServerDataResponse```，可以返回从模板和一组变量（使用格式 ```%variable%```）生成的HTML内容。这是一个非常基本的模板系统，并且真的打算为出发点，通过继承 ```GCDWebServerResponse```建设更先进的模板系统。

假设你有一个网站目录中您的应用程序包含HTML模板文件以及相应的CSS，脚本和图像，这是很容易把它变成一个动态的网站：


```objectivec
// Get the path to the website directory
NSString* websitePath = [[NSBundle mainBundle] pathForResource:@"Website" ofType:nil];

// Add a default handler to serve static files (i.e. anything other than HTML files)
[self addGETHandlerForBasePath:@"/" directoryPath:websitePath indexFilename:nil cacheAge:3600 allowRangeRequests:YES];

// Add an override handler for all requests to "*.html" URLs to do the special HTML templatization
[self addHandlerForMethod:@"GET"
                pathRegex:@"/.*\.html"
             requestClass:[GCDWebServerRequest class]
             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
    
    NSDictionary* variables = [NSDictionary dictionaryWithObjectsAndKeys:@"value", @"variable", nil];
    return [GCDWebServerDataResponse responseWithHTMLTemplate:[websitePath stringByAppendingPathComponent:request.path]
                                                    variables:variables];
    
}];

// Add an override handler to redirect "/" URL to "/index.html"
[self addHandlerForMethod:@"GET"
                     path:@"/"
             requestClass:[GCDWebServerRequest class]
             processBlock:^GCDWebServerResponse *(GCDWebServerRequest* request) {
    
    return [GCDWebServerResponse responseWithRedirect:[NSURL URLWithString:@"index.html" relativeToURL:request.URL]
                                            permanent:NO];
    
];

```

最后一个例子：文件的下载和上传从iOS应用
======================================================


GCDWebServer最初是为一个漫画阅读器应用程序[ComicFlow](http://itunes.apple.com/us/app/comicflow/id409290355?mt=8) iPad版编写的。它允许用户连接到他们的iPad与他们的网络浏览器通过WiFi然后上传，下载和整理漫画文件里面的应用程序。

ComicFlow是[完全开源](https://github.com/swisspol/ComicFlow) 的，你可以看到它如何使用GCDWebServer在 [WebServer.h](https://github.com/swisspol/ComicFlow/blob/master/Classes/WebServer.h)和[WebServer.m](https://github.com/swisspol/ComicFlow/blob/master/Classes/WebServer.m)文件。




转载请附原文链接[https://github.com/coderyi/blog/blob/master/articles/2015/1112_GCDWebServer_README.md](https://github.com/coderyi/blog/blob/master/articles/2015/1112_GCDWebServer_README.md)





