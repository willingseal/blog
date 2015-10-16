##NSObject分析笔记
https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/

NSObject主要分布在<objc/NSObject.h>和Foundation的NSObject.h以及一些其他的类中，<objc/NSObject.h>定义了NSObject类和NSObject协议，其他文件有很多的category，关于usr/include的objc实现文件苹果开源在
https://opensource.apple.com/tarballs/objc4/

objc/NSObject.h文件可以见
https://opensource.apple.com/source/objc4/objc4-532.2/runtime/NSObject.mm

###NSObject类
######Class类型的变量isa

代码来自https://opensource.apple.com/tarballs/objc4/

<pre>
typedef struct objc_class *Class;

struct objc_class : objc_object {
}

struct objc_object {
private:
    isa_t isa;
public:

    // ISA() assumes this is NOT a tagged pointer object
    Class ISA();

    // getIsa() allows this to be a tagged pointer object
    Class getIsa();
}


union isa_t 
{
    Class cls;
}

objc_object::ISA() 
{
    assert(!isTaggedPointer()); 
    return isa.cls;
}

objc_object::getIsa() 
{

    return ISA();
}
</pre>
######Initializing a Class
<pre>	
+ initialize

+ load
</pre>

作用：在Objective-C的类被加载和初始化的时候，也可以收到方法回调，可以在适当的情况下做一些定制处理。而这正是load和initialize方法可以帮我们做到的。

http://www.molotang.com/articles/1929.html

initialize和load的区别在于：load是只要类所在文件被引用就会被调用，而initialize是在类或者其子类的第一个方法被调用前调用。所以如果类没有被引用进项目，就不会有load调用；但即使类文件被引用进来，但是没有使用，那么initialize也不会被调用。

http://www.cnblogs.com/ider/archive/2012/09/29/objective_c_load_vs_initialize.html


######Creating, Copying, and Deallocating Objects
<pre>
+ alloc
For historical reasons, alloc invokes allocWithZone:.

+ (instancetype)allocWithZone:(struct _NSZone *)zone;


+ (instancetype)allocWithZone:(struct _NSZone *)zone

- init       //Designated Initializer
</pre>

源码：
其实alloc和allocWithZone的源码，最后都是返回下面这个函数
<pre>
	return _class_createInstanceFromZone (cls, extraBytes, nil);
</pre>

new doesn't support custom initializers (like initWithString);

alloc-init is more explicit than new.

http://stackoverflow.com/questions/719877/use-of-alloc-init-instead-of-new

[Objectivev-C中的alloc和init问题](http://www.daxueit.com/article/4371.html)

[谈ObjC对象的两段构造模式](http://blog.devtang.com/blog/2013/01/13/two-stage-creation-on-cocoa/)

######Testing Class Functionality
<pre>
+ instancesRespondToSelector:
</pre>

Under the hood, -[NSObject respondsToSelector:] is implemented like this:
<pre>
- (BOOL)respondsToSelector:(SEL)aSelector {
    return class_respondsToSelector([self class], aSelector);
}</pre>
and +[Class instancesRespondToSelector:] is implemented like this:
<pre>
+ (BOOL)instancesRespondToSelector:(SEL)aSelector {
    return class_respondsToSelector(self, aSelector);
}</pre>

I used [HopperMAC下的反编译软件](http://www.hopperapp.com/) on CoreFoundation to figure this out.


So, there's basically no difference. However, you can override respondsToSelector in your own class to return YES or NO on a per-instance basis (NSProxy does this). You can't do that with instancesRespondToSelector.



[http://stackoverflow.com/questions/11574478/what-is-the-difference-between-instancesrespondtoselector-and-respondstoselector](http://stackoverflow.com/questions/11574478/what-is-the-difference-between-instancesrespondtoselector-and-respondstoselector)


######Testing Protocol Conformance
<pre>
+ conformsToProtocol:
</pre>
http://stackoverflow.com/questions/15350503/id-myobjmyprotocol-vs-if-obj-class-conformstoprotocolprotocolmyprotocol

######Obtaining Information About Methods
<pre>
- (IMP)methodForSelector:(SEL)aSelector
 
</pre> 

http://www.cnblogs.com/tangbinblog/archive/2012/11/08/2760060.html


<pre>
  Person *person=[[Person alloc] initWithWeight:68];
  person.name=@"Jetta";

  SEL print_sel=NSSelectorFromString(@"print:");
  IMP imp=[person methodForSelector: print_sel];
  imp(person,print_sel,@"*********");
</pre>



http://stackoverflow.com/questions/2650190/objective-c-and-use-of-sel-imp

Here's a good [tutorial](http://www.cocoawithlove.com/2008/02/imp-of-current-method.html) for getting the current IMP (with an overview of IMPs). A very basic example of IMPs and SELs is:
<pre>
- (void)methodWithInt:(int)firstInt andInt:(int)secondInt { NSLog(@"%d", firstInt + secondInt); }

SEL theSelector = @selector(methodWithInt:andInt:);
IMP theImplementation = [self methodForSelector:theSelector]; 
//note that if the method doesn't return void, you have to explicitly typecast the IMP, e.g. int(* foo)(id, SEL, int, int) = ...
</pre>
You could then invoke the IMP like so:
<pre>
theImplementation(self, theSelector, 3, 5);
</pre>

http://www.cocoawithlove.com/2008/02/imp-of-current-method.html

In Objective-C, every method of every class in your program has a data structure constructed for it at runtime. In Objective-C 2.0, this structure is officially "opaque" (it's still there but you access its contents through functions instead of directly).

Let's look at what a Method structure contains...
<pre>
struct objc_method
{
  SEL method_name;
  char * method_types;
  IMP method_imp;
};
typedef objc_method Method;
</pre>
So the method has 3 properties:

* method_name describes the signature of the method; its value is shared by all methods with the same name and parameter names

* method_types describes the types of the parameters to the method.

* method_imp is the function pointer, i.e. the address of the code invoked when this method is selected for invocation.

It's this "method_imp" that we are challenged to find. The IMP of a method is the address of its code, also known as its function pointer.








######Forwarding Messages
<pre>
- (id)forwardingTargetForSelector:(SEL)aSelector

Returns the object to which unrecognized messages should first be directed.
</pre>


<pre>
- (void)forwardInvocation:(NSInvocation *)anInvocation

Overridden by subclasses to forward messages to other objects.


- (void)forwardInvocation:(NSInvocation *)invocation
{
    SEL aSelector = [invocation selector];
 
    if ([friend respondsToSelector:aSelector])
        [invocation invokeWithTarget:friend];
    else
        [super forwardInvocation:invocation];
}
</pre>

http://segmentfault.com/a/1190000000685432

在oc里面，发送消息给一个并不响应这个方法的对象，是合法的。apple设计这种消息转发机制的原因是用来模拟多重继承（oc原生是不支持多重继承的）。消息转发的大致过程是：
<pre>
1、查找该类及其父类的cache和方法分发表，找不到的情况下执行第2步。
2、执行+(BOOL)resolveInstanceMethod:(SEL)aSEL方法。
3、接下来 Runtime 会调用 – (id)forwardingTargetForSelector:(SEL)aSelector 方法。
这就给了程序员第二次机会，如果你没办法在自己的类里面找到替代方法，你就重载这个方法，然后把消息转给其他的Object。
4、最后，Runtime 会调用 – (void)forwardInvocation:(NSInvocation *)anInvocation 这个方法。NSInvocation 其实就是一条消息的封装。如果你能拿到 NSInvocation，那你就能修改这条消息的 target, selector 和 arguments。

void fooMethod(id obj, SEL _cmd)
{
 NSLog(@"Doing Foo");
}
//重载resolveInstanceMethod
+(BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if(aSEL == @selector(doFoo:)){
        class_addMethod([self class],aSEL,(IMP)fooMethod,"v@:");
        return YES;
    }
    return [super resolveInstanceMethod];
}

- (id)forwardingTargetForSelector:(SEL)aSelector
{
    if(aSelector == @selector(mysteriousMethod:)){
        return alternateObject;
    }
    return [super forwardingTargetForSelector:aSelector];
}

-(void)forwardInvocation:(NSInvocation *)invocation
{
    SEL invSEL = invocation.selector;

    if([altObject respondsToSelector:invSEL]) {
        [invocation invokeWithTarget:altObject];
    } else {
        [self doesNotRecognizeSelector:invSEL];
    }
}

</pre>




######Dynamically Resolving Methods
<pre>
+ (BOOL)resolveInstanceMethod:(SEL)name
Dynamically provides an implementation for a given selector for an instance method.


void dynamicMethodIMP(id self, SEL _cmd)
{
    // implementation ....
}
+ (BOOL) resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically))
    {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSel];
}
</pre>

######Error Handling

http://www.jianshu.com/p/4e9f875a2797
<pre>
- (void)doesNotRecognizeSelector:(SEL)aSelector
@implementation Person
- (void)sayHi {
    NSLog(@"if the subClass of Person not override this method, you will get a exception");
    [self doesNotRecognizeSelector:_cmd];
}
@end
</pre>

######Discardable Content Proxy Support
<pre>
@property(readonly, retain) id autoContentAccessingProxy
//定义在Foundation的NSObject (NSDiscardableContentProxy)
</pre>
http://stackoverflow.com/questions/8015940/does-nsobject-autocontentaccessingproxy-work-at-all


######Sending Messages
<pre>
- performSelector:withObject:afterDelay:
//定义在objc的NSObject protocol
+ cancelPreviousPerformRequestsWithTarget:selector:object:
//定义在NSRunLoop的NSObject (NSDelayedPerforming)
</pre>


http://www.cnblogs.com/ygm900/p/3169706.html

"- performSelector:withObject:afterDelay:"这个方法是单线程的，也就是说只有当前调用此方法的函数执行完毕后，selector方法才会被调用。

我们可以利用cancelPreviousPerformRequestsWithTarget直接取消一个对象在当前Run Loop中的所有未执行的performSelector:withObject:afterDelay:方法所产生的Selector Sources

"- performSelector:withObject:afterDelay:"如果是带参数，那"+ cancelPreviousPerformRequestsWithTarget:selector:object:"取消时的参数也要一致，否则不能取消成功。

http://blog.csdn.net/samuelltk/article/details/8994313

https://www.mgenware.com/blog/?p=463

http://blog.csdn.net/wangqiuyun/article/details/7587929



###Warning


every object is a subclass of NSObject

That is an incorrect statement. You can create objects that do not inherit from NSObject. it's not really recommended, but it is possible.

NSProxy is an example - it does not inherit from NSObject.




