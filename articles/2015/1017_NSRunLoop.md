###NSRunLoop学习笔记
https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSRunLoop_Class/
#####Accessing Run Loops and Modes

<pre>
+ (NSRunLoop *)currentRunLoop
</pre>

Returns the NSRunLoop object for the current thread.

If a run loop does not yet exist for the thread, one is created and returned.

<pre>
@property(readonly, copy) NSString *currentMode
</pre>
The receiver's current input mode. (read-only)

This method returns the current input mode only while the receiver is running; otherwise, it returns nil.

The current mode is set by the methods that run the run loop, such as acceptInputForMode:beforeDate: and runMode:beforeDate:.


<pre>
- (NSDate *)limitDateForMode:(NSString *)mode
</pre>
Performs one pass through the run loop in the specified mode and returns the date at which the next timer is scheduled to fire.

<pre>
+ (NSRunLoop *)mainRunLoop
</pre>

Returns the run loop of the main thread.

<pre>
- (CFRunLoopRef)getCFRunLoop
</pre>

Returns the receiver's underlying CFRunLoop Reference object.

#####Managing Timers

<pre>
- (void)addTimer:(NSTimer *)aTimer
         forMode:(NSString *)mode
</pre>
Registers a given timer with a given input mode.

Cocoa中可以使用以下NSTimer类方法来创建并调配一个定时器：
<pre>
scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:
 
scheduledTimerWithTimeInterval:invocation:repeats:
</pre>
上述方法创建了定时器并以默认模式把它们添加到当前线程的run loop。你可以手工的创建NSTimer对象，并通过NSRunLoop的addTimer:forMode:把它添加到run loop。
第一个定时器在初始化后1秒开始运行，此后每隔0.1秒运行。第二个定时器则在初始化后0.2秒开始运行，此后每隔0.2秒运行。
<pre>
NSRunLoop* myRunLoop = [NSRunLoop currentRunLoop];
 
// Create and schedule the first timer.
NSDate* futureDate = [NSDate dateWithTimeIntervalSinceNow:1.0];
NSTimer* myTimer = [[NSTimer alloc] initWithFireDate:futureDate
                        interval:0.1
                        target:self
                        selector:@selector(myDoFireTimer1:)
                        userInfo:nil
                        repeats:YES];
[myRunLoop addTimer:myTimer forMode:NSDefaultRunLoopMode];
 
// Create and schedule the second timer.
[NSTimer scheduledTimerWithTimeInterval:0.2
                        target:self
                        selector:@selector(myDoFireTimer2:)
                        userInfo:nil
                        repeats:YES];
</pre>
http://www.dreamingwish.com/article/ios-multithread-program-runloop-the.html

#####Managing Ports

<pre>
- (void)addPort:(NSPort *)aPort
        forMode:(NSString *)mode
</pre>
Adds a port as an input source to the specified mode of the run loop.
<pre>
- (void)removePort:(NSPort *)aPort
           forMode:(NSString *)mode
</pre>
Removes a port from the specified input mode of the run loop.


#####Running a Loop
<pre>

- (void)run

</pre>
Puts the receiver into a permanent loop, during which time it processes data from all attached input sources.

If no input sources or timers are attached to the run loop, this method exits immediately; otherwise, it runs the receiver in the NSDefaultRunLoopMode by repeatedly invoking runMode:beforeDate:. In other words, this method effectively begins an infinite loop that processes data from the run loop’s input sources and timers.

If you want the run loop to terminate, you shouldn't use this method. Instead, use one of the other run methods and also check other arbitrary conditions of your own, in a loop. A simple example would be:

<pre>
BOOL shouldKeepRunning = YES;        // global
NSRunLoop *theRL = [NSRunLoop currentRunLoop];
while (shouldKeepRunning && [theRL runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]]);
</pre>

where shouldKeepRunning is set to NO somewhere else in the program.


<pre>
- (BOOL)runMode:(NSString *)mode
     beforeDate:(NSDate *)limitDate

</pre>


Runs the loop once, blocking for input in the specified mode until a given date.

NSRunloop并不真的是一个loop，的apple的文档中 也提到了需要自己写while或者for语句来实现,类似下面：
<pre>
while(running){ 
    [NSRunLoop currentRunLoop]     runMode:NSDefaultRunLoopMode beforeDate:[NSDate distantFuture]];
}
</pre>

<pre>
- (void)runUntilDate:(NSDate *)limitDate
</pre>
Runs the loop until the specified date, during which time it processes data from all attached input sources.


<pre>
- (void)acceptInputForMode:(NSString *)mode
                beforeDate:(NSDate *)limitDate
</pre>

Runs the loop once or until the specified date, accepting input only for the specified mode.

#####Scheduling and Canceling Messages
<pre>
- (void)performSelector:(SEL)aSelector
                 target:(id)target
               argument:(id)anArgument
                  order:(NSUInteger)order
                  modes:(NSArray<NSString *> *)modes
</pre>
Schedules the sending of a message on the current run loop.

<pre>
- (void)cancelPerformSelector:(SEL)aSelector
                       target:(id)target
                     argument:(id)anArgument
</pre>
Cancels the sending of a previously scheduled message.





<pre>
- (void)cancelPerformSelectorsWithTarget:(id)target
</pre>

Cancels all outstanding ordered performs scheduled with a given target.

[Apple-Threading Programming Guide-Run Loops](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)

[Apple-Threading Programming Guide-Run Loops-中文](http://www.dreamingwish.com/article/ios-multithread-program-runloop-the.html)

Structure of a run loop and its sources



 Predefined run loop modes:
 
 NSDefaultRunLoopMode
 
 NSConnectionReplyMode

NSModalPanelRunLoopMode

NSEventTrackingRunLoopMode :Cocoa uses this mode to restrict incoming events during mouse-dragging loops and other sorts of user interface tracking loops.

NSRunLoopCommonModes： this set includes the default, modal, and event tracking modes by default.


A run loop receives events from two different types of sources. Input sources deliver asynchronous events, usually messages from another thread or from a different application. Timer sources deliver synchronous events, occurring at a scheduled time or repeating interval. 

除 了 处 理 输 入 源 ， run loops 也 会 生 成 关 于 run loop 行为的 通 知
(notifications)。注册的 run loop 观察者(run-loop Observers)可以收到这些通知，
并在线程上面使用它们来做额外的处理。


![](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/Art/runloop.jpg)

![](http://images.cnitblog.com/blog/241349/201301/06011637-2e0975908bac4b44bee988074cda1d17.gif)



[iOS中Run Loop的那些坑](http://www.hrchen.com/2013/07/tricky-runloop-on-ios/):深度好文，指出了NSRunLoop的一些常见疑难问题。
特别引用坑四
<pre>
在使用NSURLConnection或者NSStream时，需要考虑到Run Loop的问题，默认情况下这两个对象生成后都是运行在当前线程的NSDefaultRunLoopMode模式的，如果是在主线程，那么在滚动ScrollView或者TableView时，主线程的Run Loop会运行在UITrackingRunLoopMode模式，那么NSURLConnection或者NSStream的回调就无法运行。因此最好是指定NSURLConnection或NSStream在Run Loop中的运行模式，两者有相同的接口- (void)scheduleInRunLoop:(NSRunLoop *)aRunLoop forMode:(NSString *)mode;来设置NSURLConnection和NSStream的Run Loop以及模式，而且设置的Mode要设置为NSRunLoopCommonModes，因为NSRunLoopCommonModes默认会包含NSDefaultRunLoopMode和UITrackingRunLoopMode，这样无论Run Loop运行在哪个模式，都可以保证NSURLConnection或者NSStream的回调可以被调用。另外如果是在子线程中你设置了自定义的Run Loop模式，还可以用接口CFRunLoopAddCommonMode(runLoopRef, mode)添加到NSRunLoopCommonModes。

不过NSURLConnection的使用有点特殊，必须使用它的designated initializer - (id)initWithRequest:(NSURLRequest *)request delegate:(id)delegate startImmediately:(BOOL)startImmediately生成NSURLConnection对象，而且第三个参数是否立刻启动NSURLConnection要设置为NO，之后再用- (void)scheduleInRunLoop:(NSRunLoop *)aRunLoop forMode:(NSString *)mode;设置Run Loop与模式，再调用[NSURLConnectionObject start]启动。如果是其他接口生成NSURLConnection，用- (void)scheduleInRunLoop:(NSRunLoop *)aRunLoop forMode:(NSString *)mode;设置Run Loop和mode都不会起作用。
</pre>
具体可以查看AFNetworking的实现

http://www.hrchen.com/2013/06/multi-threading-programming-of-ios-part-1/





