###NSArray学习笔记

https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSArray_Class/

######Sending Messages to Elements

<pre>
- (void)makeObjectsPerformSelector:(SEL)aSelector NS_SWIFT_UNAVAILABLE("Use enumerateObjectsUsingBlock: or a for loop instead");


- (void)makeObjectsPerformSelector:(SEL)aSelector
                        withObject:(id)anObject

</pre>
这是 NSArray和NSSet的两个方法，相信大家很少用，它类似于 for循环，但是效率高于for循环。

makeObjectsPerformSelector:类似于NSNotifation机制，并发的执行同一件事，不能像for循环那样区别对待
所以参数 argument 必须是非基本类型 ，如果要是用基本类型 请转换程 NSNumber 或者NSValue。

用法：如果一个数组objArr中存储了一组有hide属性的对象，需要将数组里所有对象的hide全部赋值为真，就可以这么写：
<pre>
[objArr makeObjectsPerformSelector:@selector(setHidden:) withObject:@YES];  
</pre>

######Deriving New Arrays
<pre>
- (NSArray<ObjectType> *)arrayByAddingObject:(ObjectType)anObject;
</pre>
在数组末尾添加对象，返回新数组。

######Sorting
<pre>
typedef NS_OPTIONS(NSUInteger, NSBinarySearchingOptions) {
	NSBinarySearchingFirstEqual = (1UL << 8),
	NSBinarySearchingLastEqual = (1UL << 9),
	NSBinarySearchingInsertionIndex = (1UL << 10),
};

- (NSUInteger)indexOfObject:(ObjectType)obj inSortedRange:(NSRange)r options:(NSBinarySearchingOptions)opts usingComparator:(NSComparator)cmp NS_AVAILABLE(10_6, 4_0); // binary search
</pre>
可以用二分查找在指定区域返回通过代码块方法的索引
######Working with String Elements
<pre>
- (NSString *)componentsJoinedByString:(NSString *)separator
</pre>

Constructs and returns an NSString object that is the result of interposing a given separator between the elements of the array.


<pre>
NSArray *a = [[NSArray alloc] initWithObjects:@"冬瓜",@"西瓜",@"火龙果",@"大头",@"小狗",nil ];
NSString *b = [a componentsJoinedByString:@","];
//b is @"冬瓜,西瓜,火龙果,大头,小狗"    
</pre>


######Creating a Description
<pre>
- (BOOL)writeToFile:(NSString *)path
         atomically:(BOOL)flag
</pre>

NSArray想写入属性列表的对象是限制的，为NSString，NSData，NSDate，NSNumber，NSArray，NSDictionary。特别是NSNull不能写入。

为了解决这个问题可以给NSArray的category增加两个方法，在写入和读取的时候，用NSKeyedArchiver和NSKeyedUnarchiver进行对NSArray接归档。具体参考下面链接

http://stackoverflow.com/questions/3010363/nsarray-writetofile-fails

######Collecting Paths
<pre>
- (NSArray<NSString *> *)pathsMatchingExtensions:(NSArray<NSString *> *)filterTypes
//来自NSPathUtilities.h的NSArray<ObjectType> (NSArrayPathExtensions)  
</pre>


Returns an array containing all the pathname elements in the receiving array that have filename extensions from a given array.

<pre>
NSArray *htmlFiles = [manager contentsOfDirectoryAtPath:documentsDirectoryPath error:nil]
                                                        pathsMatchingExtensions:[NSArray arrayWithObjects:@"html", nil]];
</pre>



###



