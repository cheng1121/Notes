# View基础知识
View是Android中所有控件的基类，不管是简单的Button和TextView还是复杂的RelativeLayout和ListView，它们的共同基类都是View。View是一种界面层的控件的一种抽象，它代表了一个控件。还有ViewGroup，翻译为控件组，它内部包含了许多个控件，即一组View。
## View的位置参数
View的位置主要由它的四个顶点来决定，分别对应于View的四个属性：top、left、right、bottom，其中top是左上角纵坐标，left是左上角横坐标，right是右下角横坐标，bottom是右下角纵坐标，这些坐标都是相对于View的父容器来说的，因此是相对坐标。

获得方式：
left = getLeft();
top = getTop();
right = getRight();
bottom = getBottom();

android 3.0后新增了几个参数:x、y、translationX和translationY
x,y是View左上角的坐标
translationX和translationY是View左上角相对于父容器的偏移量，默认值为0
这几个参数的换算关系：
  x = left + translationsX
  y = top + translationsY
view在平移的过程中，left和top表示的是原始左上角的位置信息，数值不会改变，此时放生改变的是x、y、translationsX、translationsY

## MotionEvent和TouchSlop
1. MotionEvent

通过MotionEvent方法可以得到屏幕点击事件的x和y坐标。

有两组方法：
- getX/getY：返回的是相对于View左上角的x和y坐标

- getRawX/getRawY：返回的是相对于手机屏幕左上角的x和y坐标

2. TouchSlop

是系统所能识别出的被认为是滑动的最小距离，如果手指滑动的距离小于这个值，那么系统就认为不是在进行滑动操作。它是一个常量，在不同设备上这个值可能是不同的。通过ViewConfiguration.get(getContext()).getScaledTouchSlop()来获得这个常量

## VelocityTracker、GestureDetector和Scroller

1. VelocityTracker：速度追踪，用于追踪手指在滑动过程中的速度，包括水平和竖直方向的速度。
   ```
       //追踪手指移动的速度
        VelocityTracker velocityTracker = VelocityTracker.obtain();
        velocityTracker.addMovement(event);
        //计算1s内手指在屏幕上的移动速度
        velocityTracker.computeCurrentVelocity(1000);
        //水平方向上的移动速度
        xVelocity = (int) velocityTracker.getXVelocity();
        //竖直方向上的移动速度
         yVelocity = (int) velocityTracker.getYVelocity();

        mTv.setText("1s内的移动速度  水平：" + xVelocity + "  竖直："+ yVelocity);

        //不使用的时候 回收内存
        velocityTracker.clear();
        velocityTracker.recycle();
   ```
2. GestureDetector：手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为；
```
   //监听屏幕点击事件
   private GestureDetector.OnGestureListener gestureDetector = new GestureDetector.OnGestureListener() {
   
        //手指按下
        @Override
        public boolean onDown(MotionEvent e) {
            return false;
        }

        //手指轻按屏幕
        @Override
        public void onShowPress(MotionEvent e) {

        }

        //轻触屏幕后松开
        @Override
        public boolean onSingleTapUp(MotionEvent e) {
            return false;
        }

        //手指按下屏幕并拖动
        @Override
        public boolean onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY) {
            return false;
        }

        //长按屏幕
        @Override
        public void onLongPress(MotionEvent e) {

        }
        //按下屏幕后快速滑动后松开
        @Override
        public boolean onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY) {
            return false;
        }
    };


    @Override
    public boolean onTouchEvent(MotionEvent event) {
         //传入event来触发对应事件的监听
         boolean consume =  gestureDetector.onDown(event);
         
        return consume;
    }
```
3. Scroller：弹性滑动对象，用于实现View的弹性滑动

    scroller.startScroll开始滑动

