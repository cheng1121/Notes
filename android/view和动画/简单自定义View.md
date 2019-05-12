# 简单自定义View
自定义View分为 继承View和继承ViewGroup两种

这里使用继承View的方式，实现一个手指在屏幕上移动同时绘制出手指移动的轨迹

主要是通过重写onMeasure、onLayout、onDraw等三个方法，来实现需要的效果

首先来看下这三个方法的作用：
### onMeasure
测量方法，主要是用来测量View的宽高
```
 //获取xml中设置的宽高 和测量模式
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);
```
##### 测量模式
测量模式共有三种：
1. UNSPECIFIED 没有任何限制
   - 父容器对该View没有任何限制，view需要多大的尺寸就给多大的尺寸(一般在系统中使用的多，我们在自定义view时基本上不用考虑)
2. EXACTLY  精确模式
   - xml设置了固定尺寸，或者设置了match_parent则为该模式
3. AT_MOST 当前View能取的最大尺寸
   - xml中设置了wrap_content，为该模式

PS：详细参考
[View工作原理](https://github.com/Freedom12521/Android/blob/master/android/view/View的工作原理.md)

##### 使用
在继承View的自定义控件中，需要重写onMeasure方法来处理AT_MOST测量模式，如果不处理的话，xml中使用wrap_content属性时，该属性不起作用，相当于match_parent属性

根据Mode设置默认的宽高
```
  @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //获取xml中设置的宽高 和测量模式
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);
        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) { //宽高都是 wrap_content
            setMeasuredDimension(DEFAULT_SIZE, DEFAULT_SIZE);
        } else if (widthMode == MeasureSpec.AT_MOST) { //宽为wrap_content
            setMeasuredDimension(DEFAULT_SIZE, heightSize);
        } else if (heightMode == MeasureSpec.AT_MOST) { //高为wrap_content
            setMeasuredDimension(widthSize, DEFAULT_SIZE);
        }
    }
```

### onLayout
确定View所在位置（这里不需要重写，具体可以参考[View工作原理](https://github.com/Freedom12521/Android/blob/master/android/view/View的工作原理.md)）


### onTouchEvent
重写onTouchEvent，拦截手指触摸屏幕，并在屏幕上移动的事件
```
 @Override
    public boolean onTouchEvent(MotionEvent event) {

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:  //手指按下，获取按下时的坐标
                Log.i(TAG, "onTouchEvent: ACTION_DOWN");
                break;
            case MotionEvent.ACTION_MOVE:
                moveX = event.getX();
                moveY = event.getY();
                Log.i(TAG, "onTouchEvent: ACTION_MOVE");
                invalidate();
                break;
            case MotionEvent.ACTION_UP:
                Log.i(TAG, "onTouchEvent: ACTION_UP");
                break;
        }
         //必须返回true，消耗事件，否则move事件不触发
        return true;
    }
```


### onDraw
把图片、圆、点、圆弧等效果绘制到屏幕上，展示出来。
这里我们要把手指移动的路径绘制出来

绘制路径：
```
   if (saveX == 0 && saveY == 0) {
            mPath.moveTo(startX, startY);
            saveX = moveX;
            saveY = moveY;
        } else if (moveX != saveX || moveY != saveY) {
            mPath.lineTo(moveX, moveY);
            saveX = moveX;
            saveY = moveY;
        }

        canvas.drawPath(mPath, mPaint);
```
绘制的绿点：
```
  private void drawAnim(Canvas canvas) {
        if (isStartAnim) { //当手指抬起时绘制在路径上移动的圆点
            //圆点的半径
            canvas.drawCircle(centerPoint[0], centerPoint[1], 25, mCirclePaint);
        }

    }
```

动画：使用属性动画ValueAnimator
```
  /**
     * 初始化动画
     */
    private void initAnim() {
        //计算路径的长度
        //第二个参数为true表示路径闭合 ，false为不闭合
        mPathMeasure = new PathMeasure(mPath, false);
        float length = mPathMeasure.getLength();
        mValueAnimator = ValueAnimator.ofFloat(0, length);
        //让时间跟着路径的长度增长 增长曲线为已2为底的对数  表示为路径越长绿色圆点移动的速度越快
        long duration = (long) (log2(length) * 1000);
        mValueAnimator.setDuration(duration);
        mValueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float value = (float) animation.getAnimatedValue();
                mPathMeasure.getPosTan(value, centerPoint, null);

                invalidate();
            }
        });
        mValueAnimator.start();
    }
```
完整代码：
```
/**
 * 自定义View
 * <p>
 * 根据手指滑动来绘制一个路径
 * <p>
 * 手抬起后，执行动画(一个绿色的圆点沿着路径移动)
 */
public class CustomView extends View {
    private static final String TAG = "CustomView";
    //默认宽高
    private static final int DEFAULT_SIZE = 50;
    //跟随手指移动的画笔
    private Paint mPaint;
    //path的起点(不能改变)
    private float startX, startY;
    //变化的点(saveX Y 为终点，)
    private float moveX, moveY, saveX, saveY;
    private Path mPath;
    //判断是否画移动的圆点
    private boolean isStartAnim = false;
    //测量path的长度
    private PathMeasure mPathMeasure;
    //圆点的中心点
    private float[] centerPoint = new float[2];
    //属性动画
    private ValueAnimator mValueAnimator;
    //动画中移动的小圆点画笔
    private Paint mCirclePaint;


    public CustomView(Context context) {
        super(context);

        init();
    }

    private void init() {
        if (mPaint == null) {
            mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            mPaint.setColor(Color.RED);
            mPaint.setStrokeWidth(15);
            //画线  FILL画的是一个闭合区域
            mPaint.setStyle(Paint.Style.STROKE);
        }
        if (mCirclePaint == null) {
            mCirclePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            // FILL画的是一个闭合区域（实心圆） STROKE 为空心圆
            mCirclePaint.setStyle(Paint.Style.FILL);
            mCirclePaint.setColor(Color.GREEN);
        }

        if (mPath == null) {
            mPath = new Path();
        }

    }

    public CustomView(Context context, @Nullable AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public CustomView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        //获取xml中设置的宽高 和测量模式
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(heightMeasureSpec);
        int heightSize = MeasureSpec.getSize(heightMeasureSpec);
        if (widthMode == MeasureSpec.AT_MOST && heightMode == MeasureSpec.AT_MOST) { //宽高都是 wrap_content
            setMeasuredDimension(DEFAULT_SIZE, DEFAULT_SIZE);
        } else if (widthMode == MeasureSpec.AT_MOST) { //宽为wrap_content
            setMeasuredDimension(DEFAULT_SIZE, heightSize);
        } else if (heightMode == MeasureSpec.AT_MOST) { //高为wrap_content
            setMeasuredDimension(widthSize, DEFAULT_SIZE);
        }


    }

    @Override
    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
        super.onLayout(changed, left, top, right, bottom);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        //Log.i(TAG, "onDraw: ======");
        drawAllPoints(canvas);

        drawAnim(canvas);
    }


    private void drawAnim(Canvas canvas) {
        if (isStartAnim) { //当手指抬起时绘制在路径上移动的圆点
            //圆点的半径
            canvas.drawCircle(centerPoint[0], centerPoint[1], 25, mCirclePaint);
        }

    }

    private void drawAllPoints(Canvas canvas) {
        // Log.i(TAG, "drawAllPoints: saveX = " + saveX + " saveY = " + saveY + " moveX = " + moveX + " moveY = " + moveY);
        if (saveX == 0 && saveY == 0) {
            mPath.moveTo(startX, startY);
            saveX = moveX;
            saveY = moveY;
        } else if (moveX != saveX || moveY != saveY) {
            mPath.lineTo(moveX, moveY);
            saveX = moveX;
            saveY = moveY;
        }

        canvas.drawPath(mPath, mPaint);

    }


    @Override
    public boolean onTouchEvent(MotionEvent event) {

        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:  //手指按下，获取按下时的坐标
                // Log.i(TAG, "onTouchEvent: ACTION_DOWN");
                if (startX == 0 && startY == 0) {
                    startX = event.getX();
                    startY = event.getY();
                }
                if (mValueAnimator != null && mValueAnimator.isRunning()) {
                    mValueAnimator.cancel();
                    isStartAnim = false;
                    invalidate();
                }

                break;
            case MotionEvent.ACTION_MOVE:
                moveX = event.getX();
                moveY = event.getY();
                //Log.i(TAG, "onTouchEvent: ACTION_MOVE");
                invalidate();
                break;
            case MotionEvent.ACTION_UP:
                // Log.i(TAG, "onTouchEvent: ACTION_UP");
                isStartAnim = true;
                initAnim();
                break;
        }

        return true;
    }

    /**
     * 初始化动画
     */
    private void initAnim() {
        //第二个参数为true表示路径闭合 ，false为不闭合
        mPathMeasure = new PathMeasure(mPath, false);
        float length = mPathMeasure.getLength();
        mValueAnimator = ValueAnimator.ofFloat(0, length);
        //让时间跟着路径的长度增长 增长曲线为已2为底的对数  表示为路径越长绿色圆点移动的速度越快
        long duration = (long) (log2(length) * 1000);
        mValueAnimator.setDuration(duration);
        mValueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float value = (float) animation.getAnimatedValue();
                mPathMeasure.getPosTan(value, centerPoint, null);

                invalidate();
            }
        });
        mValueAnimator.start();
    }

    //对数运算
    private double log2(double value) {
        return Math.log(value) / Math.log(2);
    }

}
```

效果图：

![自定义View效果图](https://github.com/Freedom12521/Android/blob/master/android/images/自定义View效果.gif)


