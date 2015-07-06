#iOS的AssetsLibrary框架访问所有相片

2014-10-16

该框架下有几个类，ALAssetsLibrary，ALAssetsGroup，ALAsset，ALAssetsFilter，ALAssetRepresentation。



ALAssetsLibrary类

ALAssetsLibrary类可以实现查看相册列表，增加相册，保存图片到相册等功能。

例如enumerateGroupsWithTypes方法列举所有相册。



ALAssetsGroup

ALAssetsGroup就是相册的类，可以通过valueForProperty方法查看不同属性的值，如：ALAssetsGroupPropertyName，相册名。

ALAssetsGroup类有几个方法，posterImage方法就是相册的封面图片，numberOfAssets方法获取该相册的图片视频数量，可以通过enumerateAssetsUsingBlock方法列举出所有照片。

ALAssetsGroup 可以使用setAssetsFilter:(ALAssetsFilter *)filter过滤照片或者视频等。



首先是获取所有相册，通过ALAssetsLibrary的实例方法得到ALAssetsGroup类数组。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
    ALAssetsLibrary *assetsLibrary;
    NSMutableArray *groupArray;
    assetsLibrary = [[ALAssetsLibrary alloc] init];
    groupArray=[[NSMutableArray alloc] initWithCapacity:1];
    [assetsLibrary enumerateGroupsWithTypes:ALAssetsGroupAll usingBlock:^(ALAssetsGroup *group, BOOL *stop) {
        if (group) {
            [groupArray addObject:group];

            //            通过这个可以知道相册的名字，从而也可以知道安装的部分应用
            //例如 Name:柚子相机, Type:Album, Assets count:1
            NSLog(@"%@",group);
        }
    } failureBlock:^(NSError *error) {
        NSLog(@"Group not found!\n");
    }];</pre>
ALAsset类

ALAsset类也可以通过valueForProperty方法查看不同属性的值，如：ALAssetPropertyType，asset的类型，有三种ALAssetTypePhoto, ALAssetTypeVideo or ALAssetTypeUnknown。

另外还可以通过该方法获取ALAssetPropertyLocation（照片位置），ALAssetPropertyDuration（视频时间），ALAssetPropertyDate（照片拍摄日期）等。

可以通过thumbnail方法就是获取该照片。



根据相册获取该相册下所有图片，通过ALAssetsGroup的实例方法得到ALAsset类数组。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;"> [_group enumerateAssetsUsingBlock:^(ALAsset *result, NSUInteger index, BOOL *stop) {
        if (result) {
            [imageArray addObject:result];
            NSLog(@"%@",result);
            iv.image=[UIImage imageWithCGImage: result.thumbnail];
            NSString *type=[result valueForProperty:ALAssetPropertyType];
        }
    }];</pre>
ALAssetRepresentation类

ALAsset类有一个defaultRepresentation方法，返回值是ALAssetRepresentation类，该类的作用就是获取该资源图片的详细资源信息。

如
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">//
//获取资源图片的详细资源信息
ALAssetRepresentation* representation = [asset defaultRepresentation];
//获取资源图片的长宽
CGSize dimension = [representation dimensions];
 //获取资源图片的高清图
[representation fullResolutionImage];
//获取资源图片的全屏图
[representation fullScreenImage];
//获取资源图片的名字
NSString* filename = [representation filename];
NSLog(@"filename:%@",filename);
//缩放倍数
[representation scale];
//图片资源容量大小
[representation size];
//图片资源原数据
 [representation metadata];
//旋转方向
[representation orientation];
 //资源图片url地址，该地址和ALAsset通过ALAssetPropertyAssetURL获取的url地址是一样的
NSURL* url = [representation url];
NSLog(@"url:%@",url);
//资源图片uti，唯一标示符
NSLog(@"uti:%@",[representation UTI]);</pre>
