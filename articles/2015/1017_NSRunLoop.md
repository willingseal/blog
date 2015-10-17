###NSRunLoop学习笔记

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


![](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/Art/runloop.jpg)


 Predefined run loop modes:
 
 NSDefaultRunLoopMode
 
 NSConnectionReplyMode

NSModalPanelRunLoopMode

NSEventTrackingRunLoopMode :Cocoa uses this mode to restrict incoming events during mouse-dragging loops and other sorts of user interface tracking loops.

NSRunLoopCommonModes： this set includes the default, modal, and event tracking modes by default.


A run loop receives events from two different types of sources. Input sources deliver asynchronous events ，Timer sources deliver synchronous events。

除 了 处 理 输 入 源 ， run loops 也 会 生 成 关 于 run loop 行为的 通 知
(notifications)。注册的 run loop 观察者(run-loop Observers)可以收到这些通知，
并在线程上面使用它们来做额外的处理。











