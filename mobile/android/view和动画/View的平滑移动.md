# View的平滑移动
## Srcoller

```
public class ScrollerView extends AppCompatTextView {
    private static final String TAG = "ScrollerView";

    private Scroller mScroller;
    private int direction =-1;
    private Paint mTextPaint;

    public ScrollerView(Context context) {
        this(context,null);
    }

    public ScrollerView(Context context, AttributeSet attrs) {
        this(context, attrs,0);
    }

    public ScrollerView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mScroller = new Scroller(getContext());
        mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mTextPaint.setColor(Color.RED);
        mTextPaint.setTextSize(40);
    }



    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(1000,1000);
    }


    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        smoothScrollTo(500,500);
    }

    private void smoothScrollTo(int destX, int destY){
        int scrollX = getScrollX();
        int scrollY = getScrollY();
        int deltaX = destX  - scrollX;
        int deltaY = destY - scrollY;
        //1000ms内滑向destX,效果就是慢慢滑动
        mScroller.startScroll(scrollX,scrollY,deltaX,deltaY,5000);
        invalidate();
    }



    @Override
    public void computeScroll() {

       if(mScroller.computeScrollOffset()){

            scrollTo(-mScroller.getCurrX(),-mScroller.getCurrY());
            postInvalidate();
       }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        //super.onDraw(canvas);
        canvas.drawColor(Color.BLUE);

        canvas.drawText("哈是对方很舒服",0,0,mTextPaint);
    }

```
Scroller只会移动控件里边的内容，不会移动控件本身


## 动画实现平滑移动

```
public class AnimMoveView extends View {
    private Paint mTextPaint;
    private static final String TAG = "AnimMoveView";

    public AnimMoveView(Context context) {
        this(context,null);
    }

    public AnimMoveView(Context context, AttributeSet attrs) {
        this(context, attrs,0);
    }

    public AnimMoveView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        mTextPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mTextPaint.setTextSize(15* getContext().getResources().getDisplayMetrics().density);
        mTextPaint.setColor(Color.GREEN);
    }


    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        //使用属性动画模拟Scroller
        ValueAnimator valueAnimator = ValueAnimator.ofInt(0,100);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                //动画完成度，取值为 0到1
                 float fraction = animation.getAnimatedFraction();
                 float value =100 * fraction;
                Log.i(TAG, "onAnimationUpdate: " + value);
                //计算要x,y移动的距离
                 scrollTo((int) getTranslationX() +(int) value,(int)getTranslationY()+(int) value);
            }
        });
        valueAnimator.setDuration(5000);
        valueAnimator.start();

    }


    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        setMeasuredDimension(800,800);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        canvas.drawColor(Color.RED);
        String text = "属性动画实现平滑移动";
        canvas.drawText(text,getWidth()/2-mTextPaint.measureText(text)/2,getHeight()/2,mTextPaint);
    }
}

```

```
 //使用属性动画完成View的平滑移动
 ObjectAnimator objectAnimator = ObjectAnimator.ofFloat(view,"translationX",0,300);
 objectAnimator.setDuration(5000);
 objectAnimator.start();
 //注意： translationX方法的参数类型为float
 //所以 要选择ObjectAnimator.ofFloat()；来完成动画
        

```

## 使用延时策略实现
1. 使用Handler发送延时消息
2. 使用view的postDelayed方法
3. 使用线程的sleep方法

都是延时调用scrollTo方法来实现具体代码就不贴了，比较简单