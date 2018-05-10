#弹力拉伸效果
自定义一个更好用的SwipeRefreshLayout

熟悉SwipeRefreshLayout的同学一定知道，SwipeRefreshLayout是android里面专为RecyclerView，NestedScrollView提供下拉刷新动画的一个控件。可是在使用过程中有些局限性，例如只支持上述控件，不支持ListView，GridView等，另外下拉的动画效果很难更改，而且不支持上拉加载……在很多场景的情况下往往不符合我们的需求。
效果如下：
[效果]:https://github.com/guodongxiaren/ImageCache/raw/master/Logo/foryou.gif
##原理

其实是一个ViewGroup,通过对手势的处理，使子控件实现拉动的动画效果，并再加上两个子控件，上拉的loading和下拉的loading（把loading用控件来封装可以很方便的更改动画，真是贴心~），在处理手势拉动的时候，通知他们显示出对应的效果。代码很长，有很多小细节需要注意，在这里我只介绍几个关键的位置，源代码会发在文章的最后。

##拖拽弹力效果

大家可以看到，拖拽的时候，是有个弹力效果的，也就是说当拖拽的距离大于某个值，拖动的位移就会慢慢减小，最后会变得拖不动
看上去有点酷炫，其实实现起来就是高中数学知识啦，看下关键代码

final float scrollTop = yDiff * DRAG_RATE;
float originalDragPercent = scrollTop / mTotalDragDistance;
mDragPercent = Math.min(1f, Math.abs(originalDragPercent));//拖动的百分比
float extraOS = Math.abs(scrollTop) - mTotalDragDistance;//弹簧效果的位移
float slingshotDist = mSpinnerFinalOffset;
//当弹簧效果位移小余0时，tensionSlingshotPercent为0，否则取弹簧位移于总高度的比值，最大为2
float tensionSlingshotPercent = Math.max(0, Math.min(extraOS, slingshotDist * 2) / slingshotDist);
//对称轴为tensionSlingshotPercent = 2的二次函数，0到2递增
float tensionPercent = (float) ((tensionSlingshotPercent / 4) - Math.pow((tensionSlingshotPercent / 4), 2)) * 2f;
float extraMove = (slingshotDist) * tensionPercent * 2;
targetY = (int) ((slingshotDist * mDragPercent) + extraMove);

解释下几个参数
yDiff —— 根据手势算出的滑动位移
scrollTop —— yDiff乘上一个固定的比率（现在是0.5），可以用来调节“弹簧”的弹性系数
mTotalDragDistance —— 当进度显示100%时的位移
originalDragPercent —— 根据scrollTop与mTotalDragDistance的比值
mDragPercent —— 由于originalDragPercent可能大于1，所以mDragPercent才是拖动的百分比
slingshotDist —— 超过100%后可以被允许拖动的最大距离的二分之一，也是一个常数（现在值 = mSpinnerFinalOffset = mTotalDragDistance）
extraMove —— 弹力距离
targetY —— 想要移动到的目标位置

tensionSlingshotPercent和tensionPercent，就是弹力效果的关键啦

extraOS 在scrollTop>=0时，是从-mTotalDragDistance开始线性递增的，在scrollTop = mTotalDragDistance时，extraOS = 0

tensionSlingshotPercent 在scrollTop从0到mTotalDragDistance阶段，始终为0，在smTotalDragDistance到3*mTotalDragDistance阶段，线性递增，之后一直为2

extraMove 的变化同tensionSlingshotPercent

tensionPercent 是个二次函数，同样映射到scrollTop的变化，在scrollTop从0到mTotalDragDistance阶段，始终为0，在mTotalDragDistance到3mTotalDragDistance阶段，二次函数递增，在3mTotalDragDistance之后恒为0.5

而targetY，在scrollTop从0到mTotalDragDistance阶段，也就是mDragPercent从0到1，extramMove始终为0，然后二次函数递增，在scrollTop > 3*mTotalDragDistance 变为恒值
