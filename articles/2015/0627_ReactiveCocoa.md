#（译）ReactiveCocoa的自述

2015-06-27

本文翻译自GitHub上的开源框架ReactiveCocoa的readme,
英文原文链接<a href="https://github.com/ReactiveCocoa/ReactiveCocoa">https://github.com/ReactiveCocoa/ReactiveCocoa</a>.

ReactiveCocoa (RAC)是一个Objective-C的框架,它的灵感来自<a href="https://en.wikipedia.org/wiki/Functional_reactive_programming">函数式响应式编程</a>.
如果你已经很熟悉函数式响应式编程编程或者了解ReactiveCocoa的一些基本前提,check out<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/tree/swift-development/Documentation">Documentation</a>文件夹作为框架的概述,这里面有一些关于它怎么工作的深层次的信息.

感谢<a href="http://www.rheinfabrik.de">Rheinfabrik</a>对ReactiveCocoa 3的开发慷慨地赞助.


<strong><span style="color: #3d82c6;">什么是ReactiveCocoa?</span></strong>

ReactiveCocoa文档写得很厉害,并且详细地介绍了RAC是什么以及它是怎么工作的?

如果你多学一点，我们推荐下面这些资源:

1.<a href="https://github.com/ReactiveCocoa/ReactiveCocoa#introduction">Introduction</a>

2.<a href="https://github.com/ReactiveCocoa/ReactiveCocoa#when-to-use-reactivecocoa">When to use ReactiveCocoa</a>

3.<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/blob/swift-development/Documentation/FrameworkOverview.md">Framework Overview</a>

4.<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/blob/swift-development/Documentation/BasicOperators.md">Basic Operators</a>

5.<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/tree/swift-development/ReactiveCocoa">Header documentation</a>

6.Previously answered <a href="https://github.com/ReactiveCocoa/ReactiveCocoa">Stack Overflow</a> questions and <a href="https://github.com/ReactiveCocoa/ReactiveCocoa/issues?q=is%3Aclosed+label%3Aquestion">GitHub issues</a>

7.The rest of the <a href="https://github.com/ReactiveCocoa/ReactiveCocoa/tree/swift-development/Documentation">Documentation</a> folder

8.<a href="https://leanpub.com/iosfrp/">Functional Reactive Programming on iOS</a>(eBook)

如果你有任何其他的问题,请随意提交issue,

<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/issues/new">file an issue</a>.

<strong><span style="color: #3d82c6;">介绍</span></strong>

ReactiveCocoa的灵感来自<a href="https://en.wikipedia.org/wiki/Functional_reactive_programming">函数式响应式编程</a>.Rather than using mutable variables which are replaced and modified in-place,RAC提供signals（表现为RACSignal）来捕捉当前以及将来的值.

通过对signals进行连接,绑定和响应,不需要连续地观察和更新值,软件就能写了.

举个例子,一个text field能够绑定到最新状态,即使它在变,而不需要用额外的代码去更新text field每一秒的状态.它有点像KVO,但它用blocks代替了重写-observeValueForKeyPath:ofObject:change:context:.
Signals也能够呈现异步的操作,有点像<a href="https://en.wikipedia.org/wiki/Futures_and_promises">futures and
promises</a>.这极大地简化了异步软件,包括了网络处理的代码.

RAC有一个主要的优点,就是提供了一个单一的，统一的方法去处理异步的行为,包括delegate方法,blocks回调,target-action机制,notifications和KVO.

这里有一个简单的例子:
<pre lang="objc" style="background: #E8F2FB;">
// When self.username changes, logs the new name to the console.
//
// RACObserve(self, username) creates a new RACSignal that sends the current
// value of self.username, then the new value whenever it changes.
// -subscribeNext: will execute the block whenever the signal sends a value.
[RACObserve(self, username) subscribeNext:^(NSString *newName) {
    NSLog(@"%@", newName);
}];
</pre>
这不像KVO notifications,signals能够连接在一起并且能够同时进行操作:
<pre lang="objc" style="background: #E8F2FB;">
// Only logs names that starts with "j".
//
// -filter returns a new RACSignal that only sends a new value when its block
// returns YES.
[[RACObserve(self, username)
    filter:^(NSString *newName) {
        return [newName hasPrefix:@"j"];
    }]
    subscribeNext:^(NSString *newName) {
        NSLog(@"%@", newName);
    }];
</pre>
Signals也能够用来导出状态.而不是observing properties或者设置其他的 properties去反应新的值,RAC通过signals and operations让表示属性变得有可能:
<pre lang="objc" style="background: #E8F2FB;">
// Creates a one-way binding so that self.createEnabled will be
// true whenever self.password and self.passwordConfirmation
// are equal.
//
// RAC() is a macro that makes the binding look nicer.
//
// +combineLatest:reduce: takes an array of signals, executes the block with the
// latest value from each signal whenever any of them changes, and returns a new
// RACSignal that sends the return value of that block as values.
RAC(self, createEnabled) = [RACSignal
    combineLatest:@[ RACObserve(self, password), RACObserve(self, passwordConfirmation) ]
    reduce:^(NSString *password, NSString *passwordConfirm) {
        return @([passwordConfirm isEqualToString:password]);
    }];
</pre>
Signals不仅仅能够用在KVO,还可以用在很多的地方.比如说,它们也能够展示button presses:
<pre lang="objc" style="background: #E8F2FB;">
// Logs a message whenever the button is pressed.
//
// RACCommand creates signals to represent UI actions. Each signal can
// represent a button press, for example, and have additional work associated
// with it.
//
// -rac_command is an addition to NSButton. The button will send itself on that
// command whenever it's pressed.
self.button.rac_command = [[RACCommand alloc] initWithSignalBlock:^(id _) {
    NSLog(@"button was pressed!");
    return [RACSignal empty];
}];
</pre>
或者异步的网络操作:
<pre lang="objc" style="background: #E8F2FB;">
// Hooks up a "Log in" button to log in over the network.
//
// This block will be run whenever the login command is executed, starting
// the login process.
self.loginCommand = [[RACCommand alloc] initWithSignalBlock:^(id sender) {
    // The hypothetical -logIn method returns a signal that sends a value when
    // the network request finishes.
    return [client logIn];
}];

// -executionSignals returns a signal that includes the signals returned from
// the above block, one for each time the command is executed.
[self.loginCommand.executionSignals subscribeNext:^(RACSignal *loginSignal) {
    // Log a message whenever we log in successfully.
    [loginSignal subscribeCompleted:^{
        NSLog(@"Logged in successfully!");
    }];
}];

// Executes the login command when the button is pressed.
self.loginButton.rac_command = self.loginCommand;
</pre>
Signals能够展示timers,其他的UI事件,或者其他跟时间改变有关的东西.

对于用signals来进行异步操作,通过连接和改变这些signals能够进行更加复杂的行为.在一组操作完成时,工作能够很简单触发:
<pre lang="objc" style="background: #E8F2FB;">
// Performs 2 network operations and logs a message to the console when they are
// both completed.
//
// +merge: takes an array of signals and returns a new RACSignal that passes
// through the values of all of the signals and completes when all of the
// signals complete.
//
// -subscribeCompleted: will execute the block when the signal completes.
[[RACSignal
    merge:@[ [client fetchUserRepos], [client fetchOrgRepos] ]]
    subscribeCompleted:^{
        NSLog(@"They're both done!");
    }];
</pre>
Signals能够顺序地执行异步操作,而不是嵌套block回调.这个和<a href="https://en.wikipedia.org/wiki/Futures_and_promises">futures and promises</a>很相似:
<pre lang="objc" style="background: #E8F2FB;">
// Logs in the user, then loads any cached messages, then fetches the remaining
// messages from the server. After that's all done, logs a message to the
// console.
//
// The hypothetical -logInUser methods returns a signal that completes after
// logging in.
//
// -flattenMap: will execute its block whenever the signal sends a value, and
// returns a new RACSignal that merges all of the signals returned from the block
// into a single signal.
[[[[client
    logInUser]
    flattenMap:^(User *user) {
        // Return a signal that loads cached messages for the user.
        return [client loadCachedMessagesForUser:user];
    }]
    flattenMap:^(NSArray *messages) {
        // Return a signal that fetches any remaining messages.
        return [client fetchMessagesAfterMessage:messages.lastObject];
    }]
    subscribeNext:^(NSArray *newMessages) {
        NSLog(@"New messages: %@", newMessages);
    } completed:^{
        NSLog(@"Fetched all messages.");
    }];
</pre>
RAC也能够简单地绑定异步操作的结果:
<pre lang="objc" style="background: #E8F2FB;">
// Creates a one-way binding so that self.imageView.image will be set as the user's
// avatar as soon as it's downloaded.
//
// The hypothetical -fetchUserWithUsername: method returns a signal which sends
// the user.
//
// -deliverOn: creates new signals that will do their work on other queues. In
// this example, it's used to move work to a background queue and then back to the main thread.
//
// -map: calls its block with each user that's fetched and returns a new
// RACSignal that sends values returned from the block.
RAC(self.imageView, image) = [[[[client
    fetchUserWithUsername:@"joshaber"]
    deliverOn:[RACScheduler scheduler]]
    map:^(User *user) {
        // Download the avatar (this is done on a background queue).
        return [[NSImage alloc] initWithContentsOfURL:user.avatarURL];
    }]
    // Now the assignment will be done on the main thread.
    deliverOn:RACScheduler.mainThreadScheduler];
</pre>
这里仅仅说了RAC能做什么,但很难说清RAC为什么如此强大.虽然通过这个README很难说清RAC,但我尽可能用更少的代码,更少的模版,把更好的代码去表达清楚.

如果想要更多的示例代码,可以check out<a href="https://github.com/AshFurrow/C-41">C-41</a> 或者 <a href="https://github.com/jspahrsummers/GroceryList">GroceryList</a>,这些都是真正用ReactiveCocoa写的iOS apps.更多的RAC信息可以看一下<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/blob/swift-development/Documentation">Documentation</a>文件夹.


<strong><span style="color: #3d82c6;">什么时候用ReactiveCocoa</span></strong>

乍看上去,ReactiveCocoa是很抽象的,它可能很难理解如何将它应用到具体的问题.
这里有一些RAC常用的地方.

<strong>处理异步或者事件驱动数据源</strong>

很多Cocoa编程集中在响应user events或者改变application state.这样写代码很快地会变得很复杂,就像一个意大利面,需要处理大量的回调和状态变量的问题.

这个模式表面上看起来不同,像UI回调,网络响应,和KVO notifications,实际上有很多的共同之处。<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/blob/swift-development/ReactiveCocoa/Objective-C/RACSignal.h">RACSignal</a>统一了这些API,这样他们能够组装在一起然后用相同的方式操作.

举例看一下下面的代码:
<pre lang="objc" style="background: #E8F2FB;">
static void *ObservationContext = &ObservationContext;

- (void)viewDidLoad {
    [super viewDidLoad];

    [LoginManager.sharedManager addObserver:self forKeyPath:@"loggingIn" options:NSKeyValueObservingOptionInitial context:&ObservationContext];
    [NSNotificationCenter.defaultCenter addObserver:self selector:@selector(loggedOut:) name:UserDidLogOutNotification object:LoginManager.sharedManager];

    [self.usernameTextField addTarget:self action:@selector(updateLogInButton) forControlEvents:UIControlEventEditingChanged];
    [self.passwordTextField addTarget:self action:@selector(updateLogInButton) forControlEvents:UIControlEventEditingChanged];
    [self.logInButton addTarget:self action:@selector(logInPressed:) forControlEvents:UIControlEventTouchUpInside];
}

- (void)dealloc {
    [LoginManager.sharedManager removeObserver:self forKeyPath:@"loggingIn" context:ObservationContext];
    [NSNotificationCenter.defaultCenter removeObserver:self];
}

- (void)updateLogInButton {
    BOOL textFieldsNonEmpty = self.usernameTextField.text.length > 0 && self.passwordTextField.text.length > 0;
    BOOL readyToLogIn = !LoginManager.sharedManager.isLoggingIn && !self.loggedIn;
    self.logInButton.enabled = textFieldsNonEmpty && readyToLogIn;
}

- (IBAction)logInPressed:(UIButton *)sender {
    [[LoginManager sharedManager]
        logInWithUsername:self.usernameTextField.text
        password:self.passwordTextField.text
        success:^{
            self.loggedIn = YES;
        } failure:^(NSError *error) {
            [self presentError:error];
        }];
}

- (void)loggedOut:(NSNotification *)notification {
    self.loggedIn = NO;
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if (context == ObservationContext) {
        [self updateLogInButton];
    } else {
        [super observeValueForKeyPath:keyPath ofObject:object change:change context:context];
    }
}
</pre>
… 用RAC表达的话就像下面这样:
<pre lang="objc" style="background: #E8F2FB;">
- (void)viewDidLoad {
    [super viewDidLoad];

    @weakify(self);

    RAC(self.logInButton, enabled) = [RACSignal
        combineLatest:@[
            self.usernameTextField.rac_textSignal,
            self.passwordTextField.rac_textSignal,
            RACObserve(LoginManager.sharedManager, loggingIn),
            RACObserve(self, loggedIn)
        ] reduce:^(NSString *username, NSString *password, NSNumber *loggingIn, NSNumber *loggedIn) {
            return @(username.length > 0 && password.length > 0 && !loggingIn.boolValue && !loggedIn.boolValue);
        }];

    [[self.logInButton rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(UIButton *sender) {
        @strongify(self);

        RACSignal *loginSignal = [LoginManager.sharedManager
            logInWithUsername:self.usernameTextField.text
            password:self.passwordTextField.text];

            [loginSignal subscribeError:^(NSError *error) {
                @strongify(self);
                [self presentError:error];
            } completed:^{
                @strongify(self);
                self.loggedIn = YES;
            }];
    }];

    RAC(self, loggedIn) = [[NSNotificationCenter.defaultCenter
        rac_addObserverForName:UserDidLogOutNotification object:nil]
        mapReplace:@NO];
}
</pre>
<strong>连接依赖的操作</strong>

依赖经常用在网络请求,当下一个对服务器网络请求需要构建在前一个完成时,可以看一下下面的代码:
<pre lang="objc" style="background: #E8F2FB;">
[client logInWithSuccess:^{
    [client loadCachedMessagesWithSuccess:^(NSArray *messages) {
        [client fetchMessagesAfterMessage:messages.lastObject success:^(NSArray *nextMessages) {
            NSLog(@"Fetched all messages.");
        } failure:^(NSError *error) {
            [self presentError:error];
        }];
    } failure:^(NSError *error) {
        [self presentError:error];
    }];
} failure:^(NSError *error) {
    [self presentError:error];
}];
</pre>
ReactiveCocoa 则让这种模式特别简单:
<pre lang="objc" style="background: #E8F2FB;">
[[[[client logIn]
    then:^{
        return [client loadCachedMessages];
    }]
    flattenMap:^(NSArray *messages) {
        return [client fetchMessagesAfterMessage:messages.lastObject];
    }]
    subscribeError:^(NSError *error) {
        [self presentError:error];
    } completed:^{
        NSLog(@"Fetched all messages.");
    }];
</pre>

<strong>并行地独立地工作</strong>

与独立的数据集并行,然后将它们合并成一个最终的结果在Cocoa中是相当不简单的,并且还经常涉及大量的同步:
<pre lang="objc" style="background: #E8F2FB;">
__block NSArray *databaseObjects;
__block NSArray *fileContents;

NSOperationQueue *backgroundQueue = [[NSOperationQueue alloc] init];
NSBlockOperation *databaseOperation = [NSBlockOperation blockOperationWithBlock:^{
    databaseObjects = [databaseClient fetchObjectsMatchingPredicate:predicate];
}];

NSBlockOperation *filesOperation = [NSBlockOperation blockOperationWithBlock:^{
    NSMutableArray *filesInProgress = [NSMutableArray array];
    for (NSString *path in files) {
        [filesInProgress addObject:[NSData dataWithContentsOfFile:path]];
    }

    fileContents = [filesInProgress copy];
}];

NSBlockOperation *finishOperation = [NSBlockOperation blockOperationWithBlock:^{
    [self finishProcessingDatabaseObjects:databaseObjects fileContents:fileContents];
    NSLog(@"Done processing");
}];

[finishOperation addDependency:databaseOperation];
[finishOperation addDependency:filesOperation];
[backgroundQueue addOperation:databaseOperation];
[backgroundQueue addOperation:filesOperation];
[backgroundQueue addOperation:finishOperation];
</pre>
上面的代码能够简单地用合成signals来清理和优化:
<pre lang="objc" style="background: #E8F2FB;">
RACSignal *databaseSignal = [[databaseClient
    fetchObjectsMatchingPredicate:predicate]
    subscribeOn:[RACScheduler scheduler]];

RACSignal *fileSignal = [RACSignal startEagerlyWithScheduler:[RACScheduler scheduler] block:^(id<RACSubscriber> subscriber) {
    NSMutableArray *filesInProgress = [NSMutableArray array];
    for (NSString *path in files) {
        [filesInProgress addObject:[NSData dataWithContentsOfFile:path]];
    }

    [subscriber sendNext:[filesInProgress copy]];
    [subscriber sendCompleted];
}];

[[RACSignal
    combineLatest:@[ databaseSignal, fileSignal ]
    reduce:^ id (NSArray *databaseObjects, NSArray *fileContents) {
        [self finishProcessingDatabaseObjects:databaseObjects fileContents:fileContents];
        return nil;
    }]
    subscribeCompleted:^{
        NSLog(@"Done processing");
    }];
</pre>
<strong>简化集合转换</strong>

像map, filter, fold/reduce 这些高级功能在Foundation中是极度缺少的m导致了一些像下面这样循环集中的代码:
<pre lang="objc" style="background: #E8F2FB;">
NSMutableArray *results = [NSMutableArray array];
for (NSString *str in strings) {
    if (str.length < 2) {
        continue;
    }

    NSString *newString = [str stringByAppendingString:@"foobar"];
    [results addObject:newString];
}
</pre>
<a href="https://github.com/ReactiveCocoa/ReactiveCocoa/blob/swift-development/ReactiveCocoa/Objective-C/RACSequence.h">RACSequence</a>能够允许Cocoa集合用统一的方式操作:
<pre lang="objc" style="background: #E8F2FB;">
RACSequence *results = [[strings.rac_sequence
    filter:^ BOOL (NSString *str) {
        return str.length >= 2;
    }]
    map:^(NSString *str) {
        return [str stringByAppendingString:@"foobar"];
    }];
</pre>

<strong><span style="color: #3d82c6;">系统要求</span></strong>
ReactiveCocoa 要求 OS X 10.8+ 以及 iOS 8.0+.

<strong><span style="color: #3d82c6;">引入 ReactiveCocoa</span></strong>

增加 RAC 到你的应用中:

 1. 增加 ReactiveCocoa 仓库 作为你应用仓库的一个子模块.
 
 2. 从ReactiveCocoa文件夹中运行 script/bootstrap .
 3. 拖拽 ReactiveCocoa.xcodeproj 到你应用的 Xcode project 或者 workspace中.
 4. 在你应用target的"Build Phases"的选项卡,增加  RAC到 "Link Binary With Libraries"
    On iOS, 增加 libReactiveCocoa-iOS.a.
    On OS X, 增加 ReactiveCocoa.framework.
    RAC 必须选择"Copy Frameworks" . 假如你没有的话, 需要选择"Copy Files"和"Frameworks" .
 5. 增加 "$(BUILD_ROOT)/../IntermediateBuildFilesPath/UninstalledProducts/include"
    $(inherited)到 "Header Search Paths"  (这需要archive builds, 但也没什么影响).
 6. For iOS targets, 增加 -ObjC 到 "Other Linker Flags" .
 7. 假如你增加 RAC到一个project (不是一个workspace), 你需要适当的添加RAC target到你应用的"Target Dependencies".

假如你喜欢用<a href="http://cocoapods.org">CocoaPods</a>,这里有一些慷慨地第三方贡献<a href="https://github.com/CocoaPods/Specs/tree/master/Specs/ReactiveCocoa">ReactiveCocoa podspecs</a> .

想看一个用了RAC的工程,check out<a href="https://github.com/AshFurrow/C-41">C-41</a> 或者 <a href="https://github.com/jspahrsummers/GroceryList">GroceryList</a>,这些是真实的用ReactiveCocoa写的iOS apps.

<strong><span style="color: #3d82c6;">独立开发</span></strong>
假如你的工作用RAC是隔离的而不是将其集成到另一个项目,你会想打开ReactiveCocoa.xcworkspace 而不是.xcodeproj.

<strong><span style="color: #3d82c6;">更多信息</span></strong>

ReactiveCocoa灵感来自.NET的<a href="http://msdn.microsoft.com/en-us/data/gg577609">Reactive
Extensions</a> (Rx).Rx的一些原则也能够很好的用在RAC.这里有些好的Rx资源:

<a href="http://msdn.microsoft.com/en-us/library/hh242985.aspx">Reactive Extensions MSDN entry</a>

<a href="http://leecampbell.blogspot.com/2010/08/reactive-extensions-for-net.html">Reactive Extensions for .NET Introduction</a>

<a href="http://channel9.msdn.com/tags/Rx/">Rx - Channel 9 videos</a>

<a href="http://rxwiki.wikidot.com/">Reactive Extensions wiki</a>

<a href="http://rxwiki.wikidot.com/101samples">101 Rx Samples</a>

<a href="http://www.amazon.com/Programming-Reactive-Extensions-Jesse-Liberty/dp/1430237473">Programming Reactive Extensions and LINQ</a>

RAC和Rx灵感都是来自函数式响应式编程.这里有些关于FRP(functional reactive programming)相关的资源:

<a href="http://elm-lang.org/learn/What-is-FRP.elm">What is FRP? - Elm Language</a>

<a href="http://stackoverflow.com/questions/1028250/what-is-functional-reactive-programming/1030631#1030631">What is Functional Reactive Programming - Stack Overflow</a>

<a href="http://stackoverflow.com/questions/5875929/specification-for-a-functional-reactive-programming-language#5878525">Specification for a Functional Reactive Language - Stack Overflow</a>

<a href="http://elm-lang.org/learn/Escape-from-Callback-Hell.elm">Escape from Callback Hell</a>

<a href="https://www.coursera.org/course/reactive">Principles of Reactive Programming on Coursera</a>


转载请注翻译原文链接:http://www.coderyi.com/archives/765
