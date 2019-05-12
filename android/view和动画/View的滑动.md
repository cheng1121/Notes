# View的滑动

## 使用scrollTo/scrollBy
使用scrollTo和scrollBy来实现View的滑动，只能将View的内容进行移动，并不能将View本身进行移动，也就是说，不管怎么滑动，也不可能将当前View滑动到附近View所在的区域。
## 使用动画
使用平移动画来操作View的translationX和translationY属性，来让View平移，可以使用属性动画、补间动画等。

View动画是对View的影像做操作，它并不能真正改变View的位置参数，包括宽高，并且使用补间动画，如果希望动画后的状态得以保留还得将fillAfter属性设置为true，否则动画执行完成后的效果会消失。而属性动画不会。

## 改变布局参数
通过在View左边放一个空View宽度为0，当需要平移右侧View的时候，可以改变左侧空View的宽度来让右侧的View达到平移效果，改变宽度的代码如下：

```
  ViewGroup.LayoutParams layoutParams = mTv.getLayoutParams();
        layoutParams.width +=10;
        mTv.requestLayout();
        //或者使用
        //mTv.setLayoutParams(layoutParams);
```

## 几种滑动方式的对比
1. scrollTo/scrollBy:操作简单，适合对View内容的滑动
2. 动画：操作简单，主要适用于没有交互的View和实现复杂的动画效果
3. 改变布局参数：操作稍显复杂，使用于有交互的View

## 实现一个跟随手势移动的View

```
public class MoveView extends View {

    private static final String TAG = "MoveView";
    private int mLastX;
    private int mLastY;

    public MoveView(Context context) {
        super(context);
    }

    public MoveView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public MoveView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);

        setMeasuredDimension(800,400);

    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getRawX();
        int y = (int) event.getRawY();

        switch (event.getAction()) {

            case MotionEvent.ACTION_DOWN:
                break;
            case MotionEvent.ACTION_MOVE:
                //计算出手指移动的距离
                int deltaX = x - mLastX;
                int deltaY = y - mLastY;
                //计算出view移动的距离
                int translationX = (int) (getTranslationX() + deltaX);
                int translationY = (int) (getTranslationY() + deltaY);
                Log.i(TAG, "onTouchEvent: "+ translationX +" ========== " + translationY);
                //设置View移动的距离
                setTranslationX(translationX);
                setTranslationY(translationY);
                break;
            case MotionEvent.ACTION_UP:
                break;
            default:
                break;
        }
        //把手指最终的坐标保存
        mLastX = x;
        mLastY = y;

        //返回值为true，如果返回false表示事件并未被消耗，事件会被父View接收处理，ACTION_MOVE事件不会触发
        return true;
    }




    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        canvas.drawColor(Color.RED);
    }
}

```