# Android.View-notes
Android之View的一些理解和整理。结合了入门的‘’第一行代码‘’和进阶的‘’Android开发艺术探索‘’两本书和一个实例Demo~  

## 开发艺术探索之View的事件体系
> 主要介绍View的事件分发和滑动冲突问题的解决方案

### 1. view的基础知识
> View的位置参数、MotionEvent和TouchSlop对象、VelocityTracker、GestureDetector和Scroller对象

**1.1 什么是view**
> View是Android中所有控件的基类，View的本身可以是单个空间，也可以是多个控件组成的一组控件，即ViewGroup，ViewGroup继承自View，其内部可以有子View，这样就形成了View树的结构   
* 简单的说ViewGroup内部可以有View 每个View还可以是ViewGroup 以此类推

**1.2 View的位置参数**
> View的位置主要由它的四个顶点来决定，即它的四个属性：top、left、right、bottom，分别表示View左上角的坐标点（ top，left） 以及右下角的坐标点（ right，bottom）    
* 同时，我们可以得到View的大小：
```
width = right - left
height = bottom - top
```   
* 而这四个参数可以由以下方式获取:
```
Left = getLeft();
Right = getRight();
Top = getTop();
Bottom = getBottom();
```   
* Android3.0后，View增加了x、y、translationX和translationY这几个参数。其中x和y是View左上角的坐标，而translationX和translationY是View左上角相对于容器的偏移量。他们之间的换算关系如下：     
```
x = left + translationX;
y = top + translationY;
```    
> top,left表示原始左上角坐标，而x,y表示变化后的左上角坐标  在View没有平移时，x=left,y=top  View平移的过程中，top和left不会改变，改变的是x、y、translationX和translationY     

**1.3 MotionEvent和TouchSlop**   
* MotionEvent   
> 事件类型   
> * ACTION_DOWN 手指刚接触屏幕
> * ACTION_MOVE 手指在屏幕上移动    
> * ACTION_UP 手指从屏幕上松开    
> 通过MotionEven对象我们可以得到事件发生的x和y坐标，我们可以通过getX/getY和getRawX/getRawY得到。它们的区别是：getX/getY返回的是相对于当前View左上角的x和y坐标，getRawX/getRawY返回的是相对于手机屏幕左上角的x和y坐标    

* TouchSloup    
> TouchSloup是系统所能识别出的被认为是滑动的最小距离最小距离最小距离，这是一个常量，与设备有关，可通过以下方法获得：
`ViewConfiguration.get(getContext()).getScaledTouchSloup()`    
> 当我们处理滑动时，比如滑动距离小于这个值，我们就可以过滤这个事件（系统会默认过滤），从而有更好的用户体验    

**1.4 VelocityTracker、GestureDetector和Scroller**    
* VelocityTracker    
> 速度追踪，用于追踪手指在滑动过程中的速度，包括水平放向速度和竖直方向速度。使用方法:    
> * 在View的onTouchEvent方法中追踪当前单击事件的速度
```
VelocityRracker velocityTracker = VelocityTracker.obtain();
velocityTracker.addMovement(event);
```    
> * 计算速度，获得水平速度和竖直速度
```
velocityTracker.computeCurrentVelocity(1000);
int xVelocity = (int)velocityTracker.getXVelocity();
int yVelocity = (int)velocityTracker.getYVelocity();
```   
> 注意，获取速度之前必须先计算速度，即调用computeCurrentVelocity方法，这里指的速度是指一段时间内手指滑过的像素数，1000指的是1000毫秒，得到的是1000毫秒内滑过的像素数。速度可正可负：速度 = （ 终点位置 - 起点位置） / 时间段   
> * 最后，当不需要使用的时候，需要调用clear()方法重置并回收内存：
```
velocityTracker.clear();
velocityTracker.recycle();
```     

* GestureDetector        
> 手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为。使用方法：
> * 1.创建一个GestureDetector对象并实现OnGestureListener接口，根据需要，也可实现OnDoubleTapListener接口从而监听双击行为：
```
GestureDetector mGestureDetector = new GestureDetector(this);
//解决长按屏幕后无法拖动的现象
mGestureDetector.setIsLongpressEnabled(false);
```    
> * 2.接着在目标View的OnTouchEvent方法中添加以下实现：
```
boolean consume = mGestureDetector.onTouchEvent(event);
return consume;
```   
> 完成上面2步就可以有选择的实现OnGestureListener和OnDoubleTapListener的方法了
> * OnGestureListener和OnDoubleTapListener接口中具体的方法
![图片](https://img-blog.csdn.net/20160322205838021)     
* 其中常用的方法有：onSingleTapUp(单击)、onFling(快速滑动)、onScroll(拖动)、onLongPress(长按)和onDoubleTap（ 双击）。建议：如果只是监听滑动相关的，可以自己在onTouchEvent中实现，如果要监听双击这种行为，那么就使用GestureDetector     

* Scroller      
> 因为View的scrollTo、scrollBy进行滑动的时候、过程是瞬间完成的 如果没有过渡效果明显不舒服的 所以这个时候就可以弹性滑动对象，用于实现View的弹性滑动。其本身无法让View弹性滑动，需要和View的computeScroll方法配合使用才能完成这个功能。使用方法：   
```
Scroller scroller = new Scroller(mContext);
//缓慢移动到指定位置
private void smoothScrollTo(int destX,int destY){
    int scrollX = getScrollX();
    int delta = destX - scrollX;
    //1000ms内滑向destX,效果就是慢慢滑动
    mScroller.startScroll(scrollX,0,delta,0,1000);
    invalidata();
} 
@Override
public void computeScroll(){
    if(mScroller.computeScrollOffset()){
    scrollTo(mScroller.getCurrX,mScroller.getCurrY());
    postInvalidate();
    }
}
```    

### 2. View的滑动   
> 三种方式实现View滑动    

**2.1 使用scrollTo/scrollBy**    
> scrollBy实际调用了scrollTo，它实现了基于当前位置的相对滑动，而scrollTo则实现了绝对滑动     
* scrollTo和scrollBy只能改变View的内容位置而不能改变View在布局中的位置==就是说文字飘了 但是布局还在那不会跟着适应。滑动偏移量mScrollX和mScrollY的正负与实际滑动方向相反，即从左向右滑动，mScrollX为负值，从上往下滑动mScrollY为负值 

**2.2 使用动画**     
> 使用动画移动View，主要是操作View的translationX和translationY属性，既可以采用传统的View动画，也可以采用属性动画，如果使用属性动画，为了能够兼容3.0以下的版本，需要采用开源动画库nineolddandroids。 如使用属性动画：(View在100ms内向右移动100像素)
`ObjectAnimator.ofFloat(targetView,"translationX",0,100).setDuration(100).start();   //属性动画100ms内向右移动100像素 `   

**2.3 改变布局属性**     
> 通过改变布局属性来移动View，即改变LayoutParams   
```
MarginLayoutParams params = (MarginLayoutParams)Btn.getLayoutParams();
params.width += 100;
params.leftMargin += 100;
Btn.setLayoutParams(params);
```

**2.4 各种滑动方式的对比**    
> * scrollTo/scrollBy：操作简单，适合对View内容的滑动--内容      
  * 动画：操作简单，主要适用于没有交互的View和实现复杂的动画效果--没有交互     
  * 改变布局参数：操作稍微复杂，适用于有交互的View--有交互      
  
### 3. 弹性滑动    

### 4. View的事件分发机制    

**4.1 点击事件的传递规则**   
> 点击事件是MotionEvent。首先我们先看看下面一段伪代码，通过它我们可以理解到点击事件的传递规则：    
```
public boolean dispatchTouchEvent (MotionEvent ev){
boolean consume = false;
if (onInterceptTouchEvnet(ev){
    consume = onTouchEvent(ev);
} else {
    consume = child.dispatchTouchEnvet(ev);
} 
return consume;
}
```

* 上面代码主要涉及到以下三个方法：  

` public boolean dispatchTouchEvent(MotionEvent ev); `
> 这个方法用来进行事件的分发。如果事件传递给当前view，则调用此方法。返回结果表示是否消耗此事件，受内部的onTouchEvent和下级View的dispatchTouchEvent方法影响               

` public boolean onInterceptTouchEvent(MotionEvent ev); `
> 这个方法用来判断是否拦截事件。在dispatchTouchEvent方法中方法中方法中调用。返回结果表示是否拦截 会影响到dispatchTouchEvent    

`public boolean onTouchEvent(MotionEvent ev);`   
> 这个方法用来是否处理点击事件。在dispatchTouchEvent方法中方法中方法中调用，返回结果表示是否消耗事件。如果不消耗，则在同一个事件序列中，当前View无法再次接收到事件  

* 点击事件的传递规则：外dispatchTouchEvent-->内1onInterceptTouchEvent-->内2onTouchEvent
> 对于一个根ViewGroup，点击事件产生后，首先会传递给他自己，这时候就会调用他的第一个方法dispatchTouchEvent，如果Viewgroup的内部也就是第二个方法onInterceptTouchEvent返回true表示他要拦截事件，接下来事件就会交给ViewGroup处理，调用ViewGroup的内部也就是第三个方法onTouchEvent；如果ViewGroup的第二个方法onInteceptTouchEvent返回值为false，表示ViewGroup不拦截该事件，这时这跳出来啦，这样事件就传递给他的子View，接下来子View的dispatchTouchEvent方法，如此反复直到事件被最终处理循环循环循环~    

* 处理事件时的优先级关系： onTouchListener > onTouchEvent >onClickListener：
> 当一个View需要处理事件时，如果它设置了OnTouchListener，那么onTouch方法会被调用，如果onTouch返回false，则当前View的onTouchEvent方法会被调用，返回true则不会被调用，同时，在onTouchEvent方法中如果设置了OnClickListener，那么他的onClick方法会被调用

* 关于事件传递的机制，这里给出一些结论：   
> * 一个事件系列以down事件开始，中间包含数量不定的move事件，最终以up事件结束     
> * 正常情况下，一个事件序列只能由一个View拦截并消耗     
> * 某个View拦截了事件后，该事件序列只能由它去处理，并且它的onInterceptTouchEvent不会再被调用   都拦截了还调用个啥哦      
> * 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（ onTouchEvnet返回false） ，那么同一事件序列中的其他事件都不会交给他处理，并且事件将重新交由他的父元素去处理，即父元素的onTouchEvent被调用   因为自己还没搞好没空别的了     
> * 如果View不消耗ACTION_DOWN以外的其他事件，那么这个事件将会消失，此时父元素的onTouchEvent并不会被调用，并且当前View可以持续收到后续的事件，最终消失的点击事件会传递给Activity去处理     
> * ViewGroup默认不拦截任何事件     
> * View没有onInterceptTouchEvent方法，一旦事件传递给它， 只能干活 毕竟最底层啦  它的onTouchEvent方法会被调用    
> * onClick会发生的前提是当前View是可点击的，并且收到了down和up事   
> * 事件传递过程总是由外向内的，即事件总是先传递给父元素，然后由父元素分发给子View，不过也可以反过来哦~  通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的分发过程，但是ACTION_DOWN事件除外   

### 5 滑动冲突    
> 在界面中，只要内外两层同时可以滑动，这个时候就会产生滑动冲突。滑动冲突的解决有固定的方法

**5.1 常见的滑动冲突场景**    
![图片](https://img-blog.csdn.net/20170214140820945?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhbmdfZnJlZWRvbQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)   
* 外部滑动和内部滑动方向不一致  如图比如外viewpager和内listview嵌套，但这种情况下viewpager自身已经对滑动冲突进行了处理    
* 外部滑动方向和内部滑动方向一致  那就很稳  
* 上面两种情况的嵌套。只要解决1和2即可   

**5.2 滑动冲突的处理规则**    
* 对于场景一，处理的规则是：如图当用户左右滑动时，需要让外部的View拦截点击事件，当用户上下滑动的时候，需要让内部的View拦截点击事件。根据滑动的方向判断谁来拦截事件   毕竟这里的外ViewPager就是用来横向拦截的！        
* 对于场景二，由于滑动方向一致，这时候只能在业务上找到突破点，根据业务需求，规定什么时候让外部View拦截事件，什么时候由内部View拦截事件  简单的说就是根据夹角  比如水平和竖直方向的夹角  哪个大就哪个咯   
* 场景三的情况相对比较复杂，同样根据需求在业务上找到突破点    

