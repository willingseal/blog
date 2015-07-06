#iOS网络请求框架：MKNetWorkKit的使用

2014-07-11

MKNetWorkKit是由一个印度小伙子写的，是用于网络请求的库，支持ARC，代码的网址这里给出。

作者源码地址（MugunthKumar）：<a href="https://github.com/MugunthKumar/MKNetworkKit" title="MKNetworkKit">MKNetworkKit</a>

作者关于类库介绍的地址（MugunthKumar）：<a href="http://blog.mugunthkumar.com/products/ios-framework-introducing-mknetworkkit/" title="ios-framework-introducing-mknetworkkit">ios-framework-introducing-mknetworkkit</a>

作者类库介绍中文翻译地址（翻译作者，csdn博主kmyhy，杨宏焱）：<a href="http://blog.csdn.net/kmyhy/article/details/12276287" title="csdn博主kmyhy">csdn博主kmyhy</a>

这里面大多数时候会引用到它里面的demo，MKNetWorkKit－iOS－Demo，本文主要是根据作者原文以及作者的demo还有自己的使用写的文章。





在整个程序中只有一个全局队列
MKNetWorkKit中主要有两个类，MKNetworkEngine和MKNetworkOperation，MKNetworkOperation就是一个操作，是NSOperation的子类，每个HTTP操作通过MKNetworkEngine入队，队列是一个NSOperationQueue，在程序是一个单例，在MKNetworkEngine的initialize里创建。

GET请求
一般自己会复写MKNetworkEngine，在这里提供一个每一个网络请求的接口，可以传入参数，请求完成和失败的block。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
self.yahooEngine = [[YahooEngine alloc] initWithHostName:@"download.finance.yahoo.com"];</pre>
YahooEngine的GET请求方法的申明
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
typedef void (^CurrencyResponseBlock)(double rate);
-(MKNetworkOperation*) currencyRateFor:(NSString*) sourceCurrency
                   inCurrency:(NSString*) targetCurrency
                 completionHandler:(CurrencyResponseBlock) completion</pre>
GET请求方法的实现
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
-(MKNetworkOperation*) currencyRateFor:(NSString*) sourceCurrency
                            inCurrency:(NSString*) targetCurrency
                          completionHandler:(CurrencyResponseBlock) completionBlock
                               errorHandler:(MKNKErrorBlock) errorBlock {

    MKNetworkOperation *op = [self operationWithPath:YAHOO_URL(sourceCurrency, targetCurrency)
                                                   params:nil
                                             httpMethod:@"GET"];

    [op addCompletionHandler:^(MKNetworkOperation *completedOperation)
     {
         // the completionBlock will be called twice.
         // if you are interested only in new values, move that code within the else block

         NSString *valueString = [[completedOperation responseString] componentsSeparatedByString:@","][1];
         DLog(@"%@", valueString);
         if([completedOperation isCachedResponse]) {
             DLog(@"Data from cache %@", [completedOperation responseString]);
         }
         else {
             DLog(@"Data from server %@", [completedOperation responseString]);
         }

         completionBlock([valueString doubleValue]);

     }errorHandler:^(MKNetworkOperation *errorOp, NSError* error) {

         errorBlock(error);
     }];

    [self enqueueOperation:op];

    return op;
}</pre>
GET请求方法的调用
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
self.currencyOperation = [ApplicationDelegate.yahooEngine currencyRateFor:@"SGD"
                                                                   inCurrency:@"USD"
                                                                 completionHandler:^(double rate) {

    [[[UIAlertView alloc] initWithTitle:@"Today's Singapore Dollar Rates"
                                                                                                                     message:[NSString stringWithFormat:@"%.2f", rate]
                                                                                                                    delegate:nil
                                                                                                           cancelButtonTitle:NSLocalizedString(@"Dismiss", @"")
                                                                                                           otherButtonTitles:nil] show];
                                                                 }
                                                                      errorHandler:^(NSError* error) {
    DLog(@"%@\t%@\t%@\t%@", [error localizedDescription], [error localizedFailureReason],
                                                                               [error localizedRecoveryOptions], [error localizedRecoverySuggestion]);
                                                                      }];</pre>

POST请求，上传一张图片
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
-(MKNetworkOperation*) uploadImageFromFile:(NSString*) file
                         completionHandler:(IDBlock) completionBlock
                              errorHandler:(MKNKErrorBlock) errorBlock {

  MKNetworkOperation *op = [self operationWithPath:@"upload.php"
                                            params:@{@"Submit": @"YES"}
                                        httpMethod:@"POST"];

  [op addFile:file forKey:@"image"];

  // setFreezable uploads your images after connection is restored!
  [op setFreezable:YES];

  [op addCompletionHandler:^(MKNetworkOperation* completedOperation) {

    NSString *xmlString = [completedOperation responseString];

    DLog(@"%@", xmlString);
    completionBlock(xmlString);
  }
              errorHandler:^(MKNetworkOperation *errorOp, NSError* error) {

                errorBlock(error);
              }];

  [self enqueueOperation:op];


  return op;
}</pre>
上传的时候可以增加一个进度条，所以在调用这个方法的时候可以有以下代码。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
NSString *uploadPath = [[[NSBundle mainBundle] resourcePath] stringByAppendingFormat:@"/SampleImage.jpg"];
    self.uploadOperation = [ApplicationDelegate.testsEngine uploadImageFromFile:uploadPath
                                                                       completionHandler:^(id twitPicURL) {

                                                                           UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Uploaded to"
                                                                                                                           message:(NSString*) twitPicURL
                                                                                                                          delegate:nil
                                                                                                                 cancelButtonTitle:NSLocalizedString(@"Dismiss", @"")
                                                                                                                 otherButtonTitles:nil];
                                                                           [alert show];
                                                                           self.uploadProgessBar.progress = 0.0;
                                                                       }
                                                                            errorHandler:^(NSError* error) {

                                                                                [UIAlertView showWithError:error];
                                                                            }];

    [self.uploadOperation onUploadProgressChanged:^(double progress) {

        DLog(@"%.2f", progress*100.0);
        self.uploadProgessBar.progress = progress;
    }];</pre>
正确显示网络状态指示


正确显示网络状态指示的判断条件就是操作operation数大于0的时候。


自动改变队列大小
在wifi以及3G网络是会调用setMaxConcurrentOperationCount：方法切换最大操作数的大小。


自动缓存
MKNetWorkKit能够缓存你所有的GET请求，当请求相同时会调用本地缓存内容。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
[self.yahooEngine useCache];</pre>
清空缓存使用如下方法
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
[ApplicationDelegate.yahooEngine emptyCache];</pre>
冻结网络操作
网络断开时的网络请求会被冻结，通过NSKeyedArchiver序列化在本地，当有网时会再次请求。MKNetworkOperation的setFreezable:方法就是冻结操作。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
[op setFreezable:YES];</pre>
类似的请求只请求一个操作

就是同一个URL只请求一次。


图片缓存
MKNetworkEngine的imageAtURL:completionHandler:errorHandler:方法能够缓存图片。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
[ApplicationDelegate.apiEngine imageAtURL:[NSURL URLWithString:(_strategyModel
.titlepic)] completionHandler:^(UIImage* fetchedImage, NSURL* url, BOOL isInCache){
        [iv setImage:fetchedImage];

    }errorHandler:nil];</pre>
当然MKNetworkEngine的operationWithURLString: params: httpMethod:方法也可以请求图片，并且缓存，它其实就是根据一个URL地址来创建一个网络线程，然后把它加入到队列就可以了。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
MKNetworkOperation *op1 =[ApplicationDelegate.apiEngine operationWithURLString:_strategyModel.titlepic params:nil httpMethod:nil];

    [op1 addCompletionHandler:^(MKNetworkOperation *completedOperation) {

//        [op1 responseImage];
          [iv setImage:[completedOperation responseImage]];
    } errorHandler:^(MKNetworkOperation *errorOp, NSError* error) {


    }];

    [ApplicationDelegate.apiEngine enqueueOperation:op1];</pre>
其实MKNetworkOperation有各种response数据，这里是responseImage，还有responseString、responseJSON、responseXML。

不过请求一个图片还可以使用MK提供的category，UIImageView+MKNetworkKitAdditions里面的方法，如下，不过我试了之后好像没有缓存，不过这个可以用于tableview上每个cell上显示的图片，比上面的要方便许多。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
[self.thumbnailImage setImageFromURL:[NSURL URLWithString:self.loadingImageURLString]
                      placeHolderImage:nil];</pre>
缓存operation
其实每一个get请求都有缓存，要想知道你返回的数据是否缓存，可以用以下方法。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
[op addCompletionHandler:^(MKNetworkOperation *completedOperation)
     {
         // the completionBlock will be called twice.
         // if you are interested only in new values, move that code within the else block


         if([completedOperation isCachedResponse]) {
             DLog(@"Data from cache %@", [completedOperation responseString]);
         }
         else {
             DLog(@"Data from server %@", [completedOperation responseString]);
         }

         completionBlock([valueString doubleValue]);

     }errorHandler:^(MKNetworkOperation *errorOp, NSError* error) {

         errorBlock(error);
     }];</pre>
下载文件
下载文件的时候也可以增加进度条，方法类似上面上传文件的使用，下载文件痛哟在操作上增加一个一个输出流，下载文件的实现可以如下。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
-(MKNetworkOperation*) downloadFatAssFileFrom:(NSString*) remoteURL toFile:(NSString*) filePath {

  MKNetworkOperation *op = [self operationWithURLString:remoteURL];

  [op addDownloadStream:[NSOutputStream outputStreamToFileAtPath:filePath
                                                          append:YES]];

  [self enqueueOperation:op];
  return op;
}</pre>
