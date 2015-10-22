#NSOperation & NSOperationQueue
- [NSOperation](#NSOperation)
- [NSOperationQueue](#NSOperationQueue)
- [NSBlockOperation](#NSBlockOperation)
- [NSInvocationOperation](#NSInvocationOperation)


##NSOperation
https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/NSOperation_class/

######Executing the Operation
<pre>
- (void)start

</pre>
Begins the execution of the operation.

An operation is not considered ready to execute if it is still dependent on other operations that have not yet finished.

首先说明，一个NSOperation并没有创建一个新的NSThread，操作一直在[NSThread currentThread]中进行，可以从GNU的源码中得知。
下面是start的简化版源码
<pre>
- (void) start
{

  [internal->lock lock];
    {
      if (YES == [self isConcurrent])
	{
	  [NSException raise: NSInvalidArgumentException
		      format: @"[%@-%@] called on concurrent operation",
	    NSStringFromClass([self class]), NSStringFromSelector(_cmd)];
	}
      if (YES == [self isExecuting])
	{
	  [NSException raise: NSInvalidArgumentException
		      format: @"[%@-%@] called on executing operation",
	    NSStringFromClass([self class]), NSStringFromSelector(_cmd)];
	}
      if (YES == [self isFinished])
	{
	  [NSException raise: NSInvalidArgumentException
		      format: @"[%@-%@] called on finished operation",
	    NSStringFromClass([self class]), NSStringFromSelector(_cmd)];
	}
      if (NO == [self isReady])
	{
	  [NSException raise: NSInvalidArgumentException
		      format: @"[%@-%@] called on operation which is not ready",
	    NSStringFromClass([self class]), NSStringFromSelector(_cmd)];
	}
      if (NO == internal->executing)
	{
	  [self willChangeValueForKey: @"isExecuting"];
	  internal->executing = YES;
	  [self didChangeValueForKey: @"isExecuting"];
	}
    }

  [internal->lock unlock];

    {
      if (NO == [self isCancelled])
	{
	  [NSThread setThreadPriority: internal->threadPriority];
	  [self main];
	}
    }


  [self _finish];
}
</pre>
<pre>
- (void) _finish
{
 
  if (NO == internal->finished)
    {
      if (YES == internal->executing)
        {
	  [self willChangeValueForKey: @"isExecuting"];
	  [self willChangeValueForKey: @"isFinished"];
	  internal->executing = NO;
	  internal->finished = YES;
	  [self didChangeValueForKey: @"isFinished"];
	  [self didChangeValueForKey: @"isExecuting"];
	}
      
      if (NULL != internal->completionBlock)
	{
	  CALL_BLOCK_NO_ARGS(internal->completionBlock);
	}
    }
}
</pre>
<pre>
- (void)main

</pre>
Performs the receiver’s non-concurrent task.

The default implementation of this method does nothing. You should override this method to perform the desired task. In your implementation, do not invoke super.

If you are implementing a concurrent operation, you are not required to override this method but may do so if you plan to call it from your custom start method.


main方法的源码就是一个空方法，需要子类实现。
<pre>
- (void) main;
{
  return;	// OSX default implementation does nothing
}
</pre>
<pre>

@property(copy) void (^completionBlock)(void)

</pre>
The block to execute after the operation’s main task is completed.


######Canceling Operations
<pre>
- (void)cancel

</pre>

大致源码如下
<pre>
- (void) cancel
{
  if (NO == internal->cancelled && NO == [self isFinished])
    {
      [internal->lock lock];
      if (NO == internal->cancelled && NO == [self isFinished])
	{
	    {
	      [self willChangeValueForKey: @"isCancelled"];
	      internal->cancelled = YES;
	      if (NO == internal->ready)
		{
	          [self willChangeValueForKey: @"isReady"];
		  internal->ready = YES;
	          [self didChangeValueForKey: @"isReady"];
		}
	      [self didChangeValueForKey: @"isCancelled"];
	    }

	}
      [internal->lock unlock];
    }
}
</pre>

######Getting the Operation Status

<pre>
cancelled
 Property
 
executing
 Property
 
finished
 Property
 
concurrent
 Property
 
asynchronous
 Property
 
ready
 Property
 
name
 Property
</pre>

<pre>
@property (readonly, getter=isConcurrent) BOOL concurrent; // To be deprecated; use and override 'asynchronous' below

</pre>
The value of this property is YES for operations that run asynchronously with respect to the current thread or NO for operations that run synchronously on the current thread. The default value of this property is NO.

In OS X v10.6 and later, operation queues ignore the value in this property and always start operations on a separate thread.


<pre>
@property (readonly, getter=isAsynchronous) BOOL asynchronous NS_AVAILABLE(10_8, 7_0);

</pre>
The value of this property is YES for operations that run asynchronously with respect to the current thread or NO for operations that run synchronously on the current thread. The default value of this property is NO.

When implementing an asynchronous operation object, you must implement this property and return YES.


######Managing Dependencies

<pre>
- (void)addDependency:(NSOperation *)operation


</pre>
addDependency主要是把这个未完成的operation加入到self的依赖数组里面。
<pre>
- (void) addDependency: (NSOperation *)op
{

  [internal->lock lock];


    {
      if (NSNotFound == [internal->dependencies indexOfObjectIdenticalTo: op])
	{
	  [self willChangeValueForKey: @"dependencies"];
          [internal->dependencies addObject: op];
	  /* We only need to watch for changes if it's possible for them to
	   * happen and make a difference.
	   */
	  if (NO == [op isFinished]
	    && NO == [self isCancelled]
	    && NO == [self isExecuting]
	    && NO == [self isFinished])
	    {
	      /* Can change readiness if we are neither cancelled nor
	       * executing nor finished.  So we need to observe for the
	       * finish of the dependency.
	       */
	      [op addObserver: self
		   forKeyPath: @"isFinished"
		      options: NSKeyValueObservingOptionNew
		      context: NULL];
	      if (internal->ready == YES)
		{
		  /* The new dependency stops us being ready ...
		   * change state.
		   */
		  [self willChangeValueForKey: @"isReady"];
		  internal->ready = NO;
		  [self didChangeValueForKey: @"isReady"];
		}
	    }
	  [self didChangeValueForKey: @"dependencies"];
	}
    }

  [internal->lock unlock];
}

</pre>
<pre>
- (void)removeDependency:(NSOperation *)operation
@property(readonly, copy) NSArray <NSOperation *> *dependencies
</pre>

######Configuring the Execution Priority
<pre>
@property NSQualityOfService qualityOfService
</pre>
The relative amount of importance for granting system resources to the operation.

Service levels affect the priority with which operation objects are given access to system resources such as CPU time, network resources, disk resources, and so on.


<pre>
@property NSOperationQueuePriority queuePriority

</pre>
The execution priority of the operation in an operation queue.





######Waiting on an Operation Object

<pre>
- (void)waitUntilFinished

</pre>
waitUntilFinished就是给加了一个条件锁。
<pre>
- (void) waitUntilFinished
{
  if (NO == [self isFinished])
    {
      [internal->lock lock];
      if (nil == internal->cond)
	{
	  /* Set up condition to wait on and observer to unblock.
	   */
	  internal->cond = [[NSConditionLock alloc] initWithCondition: 0];
	  [self addObserver: self
		 forKeyPath: @"isFinished"
		    options: NSKeyValueObservingOptionNew
		    context: NULL];
	  /* Some other thread could have marked us as finished while we
	   * were setting up ... so we can fake the observation if needed.
	   */
          if (YES == [self isFinished])
	    {
	      [self observeValueForKeyPath: @"isFinished"
				  ofObject: self
				    change: nil
				   context: nil];
	    }
	}
      [internal->lock unlock];
      [internal->cond lockWhenCondition: 1];	// Wait for finish
      [internal->cond unlockWithCondition: 1];	// Signal any other watchers
    }
}
</pre>

##NSOperationQueue

https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/NSOperationQueue_class/


######Getting Specific Operation Queues
<pre>
+ (NSOperationQueue *)currentQueue
</pre>
Returns the operation queue that launched the current operation.

GNUStep source code:
<pre>
+ (id) currentQueue
{
  if ([NSThread isMainThread])
    {
      return mainQueue;
    }
  return [[[NSThread currentThread] threadDictionary] objectForKey: @"NSOperationQueue"];
}
</pre>
<pre>
+ (NSOperationQueue *)mainQueue
</pre>
Returns the operation queue associated with the main thread.

######Managing Operations in the Queue

<pre>
- (void)addOperation:(NSOperation *)operation
</pre>

添加未完成的NSOperation对象到operations，source code 如下
<pre>
- (void) addOperation: (NSOperation *)op
{

  [internal->lock lock];
  if (NSNotFound == [internal->operations indexOfObjectIdenticalTo: op]
    && NO == [op isFinished])
    {
      [op addObserver: self
	   forKeyPath: @"isReady"
	      options: NSKeyValueObservingOptionNew
	      context: NULL];
      [self willChangeValueForKey: @"operations"];
      [self willChangeValueForKey: @"operationCount"];
      [internal->operations addObject: op];
      [self didChangeValueForKey: @"operationCount"];
      [self didChangeValueForKey: @"operations"];
      if (YES == [op isReady])
	{
	  [self observeValueForKeyPath: @"isReady"
			      ofObject: op
				change: nil
			       context: nil];
	}
    }
  [internal->lock unlock];
}
</pre>

<pre>
- (void)addOperations:(NSArray<NSOperation *> *)ops
    waitUntilFinished:(BOOL)wait
</pre>

把ops数组加入operations内，当wait为YES时会等到所有的ops全部执行才可以。

<pre>
- (void)addOperationWithBlock:(void (^)(void))block

</pre>
The block to execute from the operation object. The block should take no parameters and have no return value.

<pre>
@property(readonly, copy) NSArray <__kindof NSOperation *> *operations

</pre>

__kindof主要作用还是编译器层面的类型检查,表明operations数组中的对象类型是NSOperation类型或NSOperation子类型

[iOS开发~Objective-C新特性](http://blog.csdn.net/lizhongfu2013/article/details/47375649)

 the array may contain operations that are executing or waiting to be executed. The list may also contain operations that were executing when the array was initially retrieved but have subsequently finished.
 
 
 <pre>
 @property(readonly) NSUInteger operationCount

 </pre>
Because the number of operations in the queue changes as those operations finish executing, the value returned by this property reflects the instantaneous number of operations at the time the property was accessed. By the time you use the value, the actual number of operations may be different. As a result, do not use this value for object enumerations or other precise calculations.

<pre>
- (void)cancelAllOperations

</pre>
Cancels all queued and executing operations.

<pre>
- (void)waitUntilAllOperationsAreFinished

</pre>
Blocks the current thread until all of the receiver’s queued and executing operations finish executing.

######Managing the Execution of Operations
<pre>
@property NSQualityOfService qualityOfService
</pre>
The default service level to apply to operations executed using the queue.

Service levels affect the priority with which operation objects are given access to system resources such as CPU time, network resources, disk resources, and so on.
<pre>
@property NSInteger maxConcurrentOperationCount

</pre>

The maximum number of queued operations that can execute at the same time.

Reducing the number of concurrent operations does not affect any operations that are currently executing. 


######Suspending Operations

<pre>
@property(getter=isSuspended) BOOL suspended
</pre>
A Boolean value indicating whether the queue is actively scheduling operations for execution.

######Configuring the Queue
<pre>
@property (nullable, assign /* actually retain */) dispatch_queue_t underlyingQueue NS_AVAILABLE(10_10, 8_0);
</pre>
The value of this property must not be the value returned by dispatch_get_main_queue. 

It can be used as follows to assign to NSOperationQueue:
<pre>
NSOperationQueue *concurrentQueueForServerCommunication = [[NSOperationQueue alloc] init];
    dispatch_queue_t concurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    concurrentQueueForServerCommunication.underlyingQueue = concurrentQueue;
</pre>
Or assign from NSOperationQueue:
<pre>
NSOperationQueue *concurrentQueueForServerCommunication = [[NSOperationQueue alloc] init];
dispatch_queue_t concurrentQueue = concurrentQueueForServerCommunication.underlyingQueue;
</pre>
http://stackoverflow.com/questions/16563410/get-underlying-dispatch-queue-t-from-nsoperationqueue

<pre>
@property(copy) NSString *name
</pre>
The name of the operation queue.

######私有方法"- (void) _execute"
首先在maxConcurrentOperationCount和suspended的setter方法里面都会调用_execute方法,以及在其他属性如operationCount、operations、值发生变化的时候都会调用它。
<pre>


- (void) setMaxConcurrentOperationCount: (NSInteger)cnt
{
  if (cnt != internal->count)
    {
      internal->count = cnt;
      [self didChangeValueForKey: @"maxConcurrentOperationCount"];
    }
  [self _execute];
}


- (void) setSuspended: (BOOL)flag
{

  internal->suspended = flag;
  [self didChangeValueForKey: @"suspended"];
  [self _execute];
}

- (void) observeValueForKeyPath: (NSString *)keyPath
		       ofObject: (id)object
                         change: (NSDictionary *)change
                        context: (void *)context
{
  if (YES == [object isFinished])
    {
      internal->executing--;
      [object removeObserver: self
		  forKeyPath: @"isFinished"];
      [internal->operations removeObjectIdenticalTo: object];
      [self didChangeValueForKey: @"operationCount"];
      [self didChangeValueForKey: @"operations"];
    }
  else if (YES == [object isReady])
    {
      [object removeObserver: self
		  forKeyPath: @"isReady"];
      [internal->waiting addObject: object];
    }
  [self _execute];
}
</pre>


<pre>

static NSInteger	maxConcurrent = 200;	// Thread pool size

#define	POOL	8// The pool of threads for 'non-concurrent' operations in a queue.


/* Check for operations which can be executed and start them.
 */



- (void) _execute
{
  NSInteger	max;


  max = [self maxConcurrentOperationCount];
  if (NSOperationQueueDefaultMaxConcurrentOperationCount == max)
    {
      max = maxConcurrent;
    }

  while (NO == [self isSuspended]
    && max > internal->executing
    && [internal->waiting count] > 0)
    {
      NSOperation	*op;

      /* Make sure we have a sorted queue of operations waiting to execute.
       */
      [internal->waiting sortUsingFunction: sortFunc context: 0];

      /* Take the first operation from the queue and start it executing.
       * We set ourselves up as an observer for the operating finishing
       * and we keep track of the count of operations we have started,
       * but the actual startup is left to the NSOperation -start method.
       */
      op = [internal->waiting objectAtIndex: 0];
      [internal->waiting removeObjectAtIndex: 0];
      [op addObserver: self
	   forKeyPath: @"isFinished"
	      options: NSKeyValueObservingOptionNew
	      context: NULL];
      internal->executing++;
      if (YES == [op isConcurrent])
	{
          [op start];
	}
      else
	{
	  NSUInteger	pending;

	  [internal->cond lock];
	  pending = [internal->starting count];
	  [internal->starting addObject: op];

	  /* Create a new thread if all existing threads are busy and
	   * we haven't reached the pool limit.
	   */
	  if (0 == internal->threadCount
	    || (pending > 0 && internal->threadCount < POOL))
	    {
	      internal->threadCount++;
	      [NSThread detachNewThreadSelector: @selector(_thread)
				       toTarget: self
				     withObject: nil];
	    }
	  /* Tell the thread pool that there is an operation to start.
	   */
	  [internal->cond unlockWithCondition: 1];
	}
    }
}

</pre>

从上面的源码可以得知

* 线程池最大支持的并发operation是200个

* 在一个队列里对于非并发操作最多允许8个线程

* 只有在所有存在的线程都很忙并且没有达到线程池限制的时候（比如这个定义的就是8个线程），才会创建新的线程。所以并不是一个queue就是一个线程，可能一个也可能几个。

* queue中增加operation，暂停queue等操作都会执行_execute，_execute方法其实就是一个队列，依次从等待队列里面start每一个operation。


##NSBlockOperation
######Managing the Blocks in the Operation
<pre>
+ (instancetype)blockOperationWithBlock:(void (^)(void))block

</pre>
Creates and returns an NSBlockOperation object and adds the specified block to it.
<pre>
- (void)addExecutionBlock:(void (^)(void))block
</pre>
Adds the specified block to the receiver’s list of blocks to perform.
<pre>
@property(readonly, copy) NSArray <void (^executionBlocks)(void)> *
</pre>
The blocks in this array are copies of those originally added using the addExecutionBlock: method.


##NSInvocationOperation

######Initialization
<pre>
- (instancetype)initWithTarget:(id)target
                      selector:(SEL)sel
                        object:(id)arg
</pre>
Returns an NSInvocationOperation object initialized with the specified target and selector.


<pre>
- (instancetype)initWithInvocation:(NSInvocation *)inv

</pre>
Returns an NSInvocationOperation object initialized with the specified invocation object.

######Getting Attributes
<pre>
@property(readonly, retain) NSInvocation *invocation
</pre>
The invocation object identifying the target object, selector, and parameters to use to execute the operation’s task.

<pre>
@property(readonly, retain) id result
</pre>
The result of the invocation or method. (read-only)



关于NSInvocationOperation的实现，可以从源码中看到它子类化了main方法
<pre>
- (void) main
{
  if (![self isCancelled])
    {
      NS_DURING
	[_invocation invoke];
      NS_HANDLER
	_exception = [localException copy];
      NS_ENDHANDLER
    }
}

</pre>