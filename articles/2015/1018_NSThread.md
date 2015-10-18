##NSThread学习笔记

https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSThread_Class/


[Threading Programming Guide-中文版](http://www.dreamingwish.com/article/ios-multi-threaded-programming-guide-directory.html)
######Initializing an NSThread Object
<pre>

- init
 Designated Initializer
 
 
- initWithTarget:selector:object:
 
</pre>


######Starting a Thread
<pre>

+ detachNewThreadSelector:toTarget:withObject:


- start


</pre>
"+ detachNewThreadSelector:toTarget:withObject:"的源码可以从GNUstep获得
<pre>
+ (void) detachNewThreadSelector: (SEL)aSelector
		        toTarget: (id)aTarget
                      withObject: (id)anArgument
{
  NSThread	*thread;

  /*
   * Create the new thread.
   */
  thread = [[NSThread alloc] initWithTarget: aTarget
                                   selector: aSelector
                                     object: anArgument];

  [thread start];
  RELEASE(thread);
}

</pre>
<pre>
- (void)main // thread body method
</pre>
If you subclass NSThread, you can override this method and use it to implement the main body of your thread instead. If you do so, you do not need to invoke super.

You should never invoke this method directly. You should always start your thread by invoking the start method.



######Determining the Thread’s Execution State

<pre>
executing
 Property
 
finished
 Property
 
cancelled
 Property
</pre>









######Working with the Main Thread

<pre>

+ (BOOL)isMainThread

@property(readonly) BOOL isMainThread

+ (NSThread *)mainThread
</pre>



######Querying the Environment



<pre>
+ (BOOL)isMultiThreaded

+ (NSThread *)currentThread

+ (NSArray<NSNumber *> *)callStackReturnAddresses
</pre>


<pre>
+ (NSArray<NSString *> *)callStackSymbols
</pre>
This method returns an array of strings describing the call stack backtrace of the current thread at the moment this method was called. 

<pre>
- (void)callStackSymbolsExampleAction{
    NSArray *callStackSymbols=[NSThread callStackSymbols];
    NSLog(@"%@",callStackSymbols);
}
</pre>
<pre>
2015-10-18 16:47:47.961 Example[15463:448063] (
	0   Example                             0x0000000109dd586e -[ViewController callStackSymbolsExampleAction] + 46
	1   Example                             0x0000000109dd5839 -[ViewController viewDidLoad] + 73
	2   UIKit                               0x000000010ad8d931 -[UIViewController loadViewIfRequired] + 1344
	3   UIKit                               0x000000010ad8dc7d -[UIViewController view] + 27
	4   UIKit                               0x000000010ac6b0c0 -[UIWindow addRootViewControllerViewIfPossible] + 61
	5   UIKit                               0x000000010ac6b7bd -[UIWindow _setHidden:forced:] + 302
	6   UIKit                               0x000000010ac7d020 -[UIWindow makeKeyAndVisible] + 43
	7   UIKit                               0x000000010abfa93c -[UIApplication _callInitializationDelegatesForMainScene:transitionContext:] + 4131
	8   UIKit                               0x000000010ac00e15 -[UIApplication _runWithMainScene:transitionContext:completion:] + 1755
	9   UIKit                               0x000000010abfdff0 -[UIApplication workspaceDidEndTransaction:] + 188
	10  FrontBoardServices                  0x000000010d5187ac -[FBSSerialQueue _performNext] + 192
	11  FrontBoardServices                  0x000000010d518b1a -[FBSSerialQueue _performNextFromRunLoopSource] + 45
	12  CoreFoundation                      0x000000010a7820a1 __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 17
	13  CoreFoundation                      0x000000010a777fcc __CFRunLoopDoSources0 + 556
	14  CoreFoundation                      0x000000010a777483 __CFRunLoopRun + 867
	15  CoreFoundation                      0x000000010a776e98 CFRunLoopRunSpecific + 488
	16  UIKit                               0x000000010abfd98d -[UIApplication _run] + 402
	17  UIKit                               0x000000010ac02676 UIApplicationMain + 171
	18  Example                             0x0000000109dd5bcf main + 111
	19  libdyld.dylib                       0x000000010cee192d start + 1
)

</pre>

[让XCode的 Stack Trace信息可读](http://blog.devtang.com/blog/2012/11/14/make-stack-trace-more-readable/#jtss-tsina)


######Working with Thread Properties




<pre>
@property(readonly, retain) NSMutableDictionary *threadDictionary
</pre>

每个线程都维护一个在线程任何地方都能获取的字典。 我们可以使用NSThread的threadDictionary方法获取一个NSMutableDictionary对象，然后添加我们需要的字段和数据。

<pre>
+ (NSDateFormatter *)currentDateFormatter
{
	NSMutableDictionary *threadDictionary = [[NSThread currentThread] threadDictionary] ;
	NSDateFormatter *dateFormatter = [threadDictionary objectForKey: @"DDMyDateFormatter"] ;
	if (dateFormatter == nil)
	{
		dateFormatter = [[NSDateFormatter alloc] init] ;
		[dateFormatter setLocale: [[NSLocale alloc] initWithLocaleIdentifier: @"en_GB"]] ;//en_GB	English (United Kingdom)
		[dateFormatter setDateFormat: @"YYYY-MM-dd HH:mm:ss ZZZ"] ;
		[threadDictionary setObject: dateFormatter forKey: @"DDMyDateFormatter"] ;
	}
	return dateFormatter ;
}
</pre>

[ios locale identifiers list](https://gist.github.com/jacobbubu/1836273)


<pre>

name
 Property
</pre>

<pre>
@property NSUInteger stackSize
</pre>
The stack size of the receiver, in bytes.

需要注意的是，主线程会占用 1M 的栈区空间，新开的线程会占用 512K 的栈区空间，新开的线程可以暂停，可以休眠，但是杀不掉，系统自己会回收这个空间。


######Working with Thread Priorities


<pre>

+ threadPriority

threadPriority
 Property
 
+ setThreadPriority:
</pre>


######Notifications
<pre>

NSDidBecomeSingleThreadedNotification
NSThreadWillExitNotification
NSWillBecomeMultiThreadedNotification
</pre>

NSThreadWillExitNotification会在调用"+ (void) exit"的时候调用
<pre>

+ (void) exit
{
  NSThread	*t;

  t = GSCurrentThread();
  if (t->_active == YES)
    {
      unregisterActiveThread (t);

      if (t == defaultThread || defaultThread == nil)
	{
	  /* For the default thread, we exit the process.
	   */
	  exit(0);
	}
      else
	{
          pthread_exit(NULL);
	}
    }
}
</pre>
<pre>

static void
unregisterActiveThread(NSThread *thread)
{
  if (thread->_active == YES)
    {
      /*
       * Set the thread to be inactive to avoid any possibility of recursion.
       */
      thread->_active = NO;
      thread->_finished = YES;

      /*
       * Let observers know this thread is exiting.
       */
      if (nc == nil)
	{
	  nc = RETAIN([NSNotificationCenter defaultCenter]);
	}
      [nc postNotificationName: NSThreadWillExitNotification
			object: thread
		      userInfo: nil];

      [(GSRunLoopThreadInfo*)thread->_runLoopInfo invalidate];
      [thread  release];

      [[NSGarbageCollector defaultCollector] enableCollectorForPointer: thread];
      pthread_setspecific(thread_object_key, nil);
    }
}


</pre>