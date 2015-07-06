#iOS开发概述UIkit动力学

2014-10-18

UIkit动力学是UIkit框架中模拟真实世界的一些特性。

UIDynamicAnimator

主要有UIDynamicAnimator类，通过这个类中的不同行为来实现一些动态特性。



它一般有两种初始化方法，先讲常见的第一种
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">animator= [[UIDynamicAnimator alloc] initWithReferenceView:self.view];</pre>
动态特性的实现主要依靠它所添加的行为，通过以下方法进行添加和移除，
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">[animator addBehavior:attachmentBehavior];
[animator removeAllBehaviors];</pre>
接下来介绍五个不同的行为，UIAttachmentBehavior（吸附），UICollisionBehavior（碰撞），UIGravityBehavior（重力），UIPushBehavior（推动），UISnapBehavior（捕捉）。另外还有一个辅助的行为UIDynamicItemBehavior，用来在item层级设定一些参数，比如item的摩擦，阻力，角阻力，弹性密度和可允许的旋转等等。

UIAttachmentBehavior（吸附）
先讲吸附行为，

它的初始化方法
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">attachmentBehavior = [[UIAttachmentBehavior alloc] initWithItem:iv
                                                                offsetFromCenter:centerOffset
                                                                attachedToAnchor:location];</pre>

item是实现UIDynamicItem协议的id类型，这里设置吸附一个UIImageView的实例iv。offset可以设置吸附的偏移，anchor是设置锚点。

 UIAttachmentBehavior有几个属性，例如damping，frequency。damping是阻尼数值，frequency是震动频率

直接上代码，实现一个pan手势，让一个image跟着手势跑
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">-(void)gesture:(UIPanGestureRecognizer *)gesture{
    CGPoint location = [gesture locationInView:self.view];
    CGPoint boxLocation = [gesture locationInView:iv];

    switch (gesture.state) {
        case UIGestureRecognizerStateBegan:{
            NSLog(@"you touch started position %@",NSStringFromCGPoint(location));
            NSLog(@"location in image started is %@",NSStringFromCGPoint(boxLocation));

            [animator removeAllBehaviors];

            // Create an attachment binding the anchor point (the finger's current location)
            // to a certain position on the view (the offset)

            UIOffset centerOffset = UIOffsetMake(boxLocation.x - CGRectGetMidX(iv.bounds),
                                                 boxLocation.y - CGRectGetMidY(iv.bounds));
            attachmentBehavior = [[UIAttachmentBehavior alloc] initWithItem:iv
                                                                offsetFromCenter:centerOffset
                                                                attachedToAnchor:location];

            attachmentBehavior.damping=0.5;
            attachmentBehavior.frequency=0.8;


            // Tell the animator to use this attachment behavior
            [animator addBehavior:attachmentBehavior];
            break;
        }
        case UIGestureRecognizerStateEnded: {
               [animator removeBehavior:attachmentBehavior];

            break;
        }
        default:
              [attachmentBehavior setAnchorPoint:[gesture locationInView:self.view]];
            break;
    }
}</pre>
UIPushBehavior（推动）
 UIPushBehavior 可以为一个UIView施加一个力的作用，这个力可以是持续的，也可以只是一个冲量。我们可以指定力的大小，方向和作用点等等信息。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">pushBehavior = [[UIPushBehavior alloc]
                                                initWithItems:@[iv]
                                                mode:UIPushBehaviorModeInstantaneous];</pre>

UIPushBehavior 有pushDirection、magnitude等属性，
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">
//1
            CGPoint velocity = [gesture velocityInView:self.view];
            CGFloat magnitude = sqrtf((velocity.x * velocity.x) + (velocity.y * velocity.y));

            if (magnitude > ThrowingThreshold) {
                //2
                pushBehavior = [[UIPushBehavior alloc]
                                                initWithItems:@[iv]
                                                mode:UIPushBehaviorModeInstantaneous];
                pushBehavior.pushDirection = CGVectorMake((velocity.x / 10) , (velocity.y / 10));
                pushBehavior.magnitude = magnitude / ThrowingvelocityPadding;


                [animator addBehavior:pushBehavior];

                //3
//                UIDynamicItemBehavior 其实是一个辅助的行为，用来在item层级设定一些参数，比如item的摩擦，阻力，角阻力，弹性密度和可允许的旋转等等
                NSInteger angle = arc4random_uniform(20) - 10;

                itemBehavior = [[UIDynamicItemBehavior alloc] initWithItems:@[iv]];
                itemBehavior.friction = 0.2;
                itemBehavior.allowsRotation = YES;
                [itemBehavior addAngularVelocity:angle forItem:iv];
                [animator addBehavior:itemBehavior];

                //4
                [self performSelector:@selector(resetDemo) withObject:nil afterDelay:0.4];
            }</pre>

UIGravityBehavior（重力）
 直接上代码，实现随机掉落一张图片的代码
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">// Set up
    self.animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];

    self.gravityBeahvior = [[UIGravityBehavior alloc] initWithItems:nil];




    [self.animator addBehavior:self.gravityBeahvior];


- (void)tapped:(UITapGestureRecognizer *)gesture {

    NSUInteger num = arc4random() % 40 + 1;
    NSString *filename = [NSString stringWithFormat:@"m%lu", (unsigned long)num];
    UIImage *image = [UIImage imageNamed:filename];
    UIImageView *imageView = [[UIImageView alloc] initWithImage:image];
    [self.view addSubview:imageView];

    CGPoint tappedPos = [gesture locationInView:gesture.view];
    imageView.center = tappedPos;

    [self.gravityBeahvior addItem:imageView];

}</pre>
UICollisionBehavior（碰撞）
继续上面的代码，当图片快掉落出边界的时候有 碰撞效果，这个就是UICollisionBehavior实现的。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">// Set up
    self.animator = [[UIDynamicAnimator alloc] initWithReferenceView:self.view];

    self.gravityBeahvior = [[UIGravityBehavior alloc] initWithItems:nil];

    self.collisionBehavior = [[UICollisionBehavior alloc] initWithItems:nil];
    self.collisionBehavior.translatesReferenceBoundsIntoBoundary = YES;

    self.itemBehavior = [[UIDynamicItemBehavior alloc] initWithItems:nil];
    self.itemBehavior.elasticity = 0.6;
    self.itemBehavior.friction = 0.5;
    self.itemBehavior.resistance = 0.5;


    [self.animator addBehavior:self.gravityBeahvior];
    [self.animator addBehavior:self.collisionBehavior];
    [self.animator addBehavior:self.itemBehavior];



- (void)tapped:(UITapGestureRecognizer *)gesture {

    NSUInteger num = arc4random() % 40 + 1;
    NSString *filename = [NSString stringWithFormat:@"m%lu", (unsigned long)num];
    UIImage *image = [UIImage imageNamed:filename];
    UIImageView *imageView = [[UIImageView alloc] initWithImage:image];
    [self.view addSubview:imageView];

    CGPoint tappedPos = [gesture locationInView:gesture.view];
    imageView.center = tappedPos;

    [self.gravityBeahvior addItem:imageView];
    [self.collisionBehavior addItem:imageView];
    [self.itemBehavior addItem:imageView];
}</pre>
另外，UICollisionBehavior有它的代理，其中列举两个方法，它们表示行为开始和结束的时候的代理。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">- (void)collisionBehavior:(UICollisionBehavior*)behavior beganContactForItem:(id <UIDynamicItem>)item withBoundaryIdentifier:(id <NSCopying>)identifier atPoint:(CGPoint)p;
- (void)collisionBehavior:(UICollisionBehavior*)behavior endedContactForItem:(id <UIDynamicItem>)item withBoundaryIdentifier:(id <NSCopying>)identifier;</pre>

UISnapBehavior（捕捉）
UISnapBehavior 将UIView通过动画吸附到某个点上。
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">- (void) handleTap:(UITapGestureRecognizer *)paramTap{
    CGPoint tapPoint = [paramTap locationInView:self.view];

    if (self.snapBehavior != nil){
        [self.animator removeBehavior:self.snapBehavior];
    }
    self.snapBehavior = [[UISnapBehavior alloc] initWithItem:self.squareView snapToPoint:tapPoint];
    self.snapBehavior.damping = 0.5f;  //剧列程度
    [self.animator addBehavior:self.snapBehavior];
}</pre>
UICollectionView与UIDynamicAnimator
文章开头说到UIDynamicAnimator有两种初始化方法，这里介绍它与UICollectionView的完美结合，让UICollectionView产生各种动态特性的行为。

你是否记得iOS系统中信息应用中的附有弹性的消息列表，他就是加入了UIAttachmentBehavior吸附行为，这里通过一个UICollectionView实现类似效果。

主要是复写UICollectionViewFlowLayout，在layout中为每一个布局属性元素加上吸附行为就可以了。

关于复写layout，可以参考onevcat的博客

<a href="http://www.onevcat.com/2012/08/advanced-collection-view/" title="advanced-collection-view">advanced-collection-view</a>

下面就直接上代码了

首先遍历每个 collection view layout attribute 来创建和添加新的 dynamic animator
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">-(void)prepareLayout {
    [super prepareLayout];

    if (!_animator) {
        _animator = [[UIDynamicAnimator alloc] initWithCollectionViewLayout:self];
        CGSize contentSize = [self collectionViewContentSize];
        NSArray *items = [super layoutAttributesForElementsInRect:CGRectMake(0, 0, contentSize.width, contentSize.height)];

        for (UICollectionViewLayoutAttributes *item in items) {
            UIAttachmentBehavior *attachment = [[UIAttachmentBehavior alloc] initWithItem:item attachedToAnchor:item.center];

            attachment.length = 0;
            attachment.damping = self.damping;
            attachment.frequency = self.frequency;

            [_animator addBehavior:attachment];
        }
    }
}</pre>
接下来我们现在需要实现 layoutAttributesForElementsInRect: 和 layoutAttributesForItemAtIndexPath: 这两个方法，UIKit 会调用它们来询问 collection view 每一个 item 的布局信息。我们写的代码会把这些查询交给专门做这些事的 dynamic animator
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">-(NSArray *)layoutAttributesForElementsInRect:(CGRect)rect {
    return [_animator itemsInRect:rect];
}

- (UICollectionViewLayoutAttributes *)layoutAttributesForItemAtIndexPath:(NSIndexPath *)indexPath {
    return [_animator layoutAttributesForCellAtIndexPath:indexPath];
}</pre>
然后是响应滚动事件的方法

这个方法会在 collection view 的 bound 发生改变的时候被调用，根据最新的 content offset 调整我们的 dynamic animator 中的 behaviors 的参数。在重新调整这些 behavior 的 item 之后，我们在这个方法中返回 NO；因为 dynamic animator 会关心 layout 的无效问题，所以在这种情况下，它不需要去主动使其无效
<pre lang="objc" escaped="true" style="background: #E8F2FB ;">-(BOOL)shouldInvalidateLayoutForBoundsChange:(CGRect)newBounds {
    UIScrollView *scrollView = self.collectionView;
    CGFloat scrollDelta = newBounds.origin.y - scrollView.bounds.origin.y;
    NSLog(@" %f   %f",newBounds.origin.y,scrollView.bounds.origin.y);
    CGPoint touchLocation = [scrollView.panGestureRecognizer locationInView:scrollView];

    for (UIAttachmentBehavior *behavior in _animator.behaviors) {

        CGPoint anchorPoint = behavior.anchorPoint;
        CGFloat distanceFromTouch = fabsf(touchLocation.y - anchorPoint.y);
        CGFloat scrollResistance = distanceFromTouch / self.resistanceFactor;

        UICollectionViewLayoutAttributes *item = [behavior.items firstObject];
        CGPoint center = item.center;
        center.y += (scrollDelta > 0) ? MIN(scrollDelta, scrollDelta * scrollResistance)
        : MAX(scrollDelta, scrollDelta * scrollResistance);
        item.center = center;

        [_animator updateItemUsingCurrentState:item];
    }
    return NO;
}</pre>
让我们仔细查看这个代码的细节。首先我们得到了这个 scroll view（就是我们的 collection view ），然后计算它的 content offset 中 y 的变化（在这个例子中，我们的 collection view 是垂直滑动的）。一旦我们得到这个增量，我们需要得到用户接触的位置。这是非常重要的，因为我们希望离接触位置比较近的那些物体能移动地更迅速些，而离接触位置比较远的那些物体则应该滞后些。

对于 dynamic animator 中的每个 behavior，我们将接触点到该 behavior 物体的  y 的距离除以 500。分母越小，这个 collection view 的的交互就越有弹簧的感觉。
