# View的工作原理
## ViewRoot和DecorView
 ViewRoot：对应ViewRootImpl类，它是连接WindowManager和DecorView的纽带，View的三大流程均是通过ViewRoot来完成的。在ActivityThread中，当Activity对象被创建完毕后，会将DecorView添加到Window中，同时会创建ViewRootImpl对象，并将ViewRootImpl对象和DecorView建立关联。
View的绘制流程是从ViewRoot的performTraversals方法开始的。

performTraversals方法的调用流程：
 ![image.png](https://upload-images.jianshu.io/upload_images/11142016-5a441086d95cad16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图，performTraversals会依次调用performMesaure、performLayout、performDraw来完成View绘制的流程。当调用onMeasure方法对子元素进行测量时，测量的流程就传递到子元素中了，这时就完成了一次测量过程，子元素再重复父元素的measure过程，这样就完成了View树的遍历。layout、draw流程也是一样的，区别就是performDraw的传递过程在draw方法是通过dispatchDraw来实现的，不过本质是一样的。

measure过程决定了View的宽高；layout过程决定了View四个顶点的坐标和实际的宽高；draw过程决定了View的显示，只有draw过程完成后，View的内容才会呈现在屏幕上；

DecorView: 顶级View，其内部包含一个竖直的LinearLayout，LinearLayout包含Title和Content两部分，我们设置的View就时被添加到了Content中，Content的Id为content，是一个FrameLayout。DecorView也是一个FrameLayout,View的事件都是先经过DecorView，然后才传递给我们的View

## MeasureSpec
MeasureSpec很大程度上决定了一个View的尺寸规格，之所以说很大程度上，是因为这个过程还受父容器的影响，因为父容器影响View的MeasureSpec的创建过程。系统会将View的LayoutParams根据父容器所施加的规则转换成对应的MeasureSpec，然后再根据这个measureSpec来测量View的宽高。这里的宽高是测量宽高，不一定是最终宽高

MeasureSpec代表一个32位int值，高2位代表SpecMode，低30位代表SpecSize，SpecMode代表测量模式，SpecSize代表某种测量模式下的规格大小。

MeasureMode有三种：
1. UNSPECIFIED： 父容器不对View有任何限制，要多大给多大，这种情况一般用于系统内部，表示一种测量状态
2. EXACTLY：父容器以检测出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它对应于LayoutParams中的match_parent和具体的数值这两种模式
3. AT_MOST：父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值。它对应于LayoutParams种的wrap_content。

### MeasureSpec和LayoutParams的关系
1. View测量过程中，系统将LayoutParams在父容器的约束下转换成对应MeasureSpec,然后再根据这个MeasureSpec来确定View测量后的宽高。LayoutParams需要和父容器一起才能决定MeasureSpec，从而决定View 的宽高。
2. 对于顶级View（DecorView），其MeasureSpec是由窗口的尺寸和自身的LayoutParams来共同决定
3. 对于普通View，由父容器的MeasureSpec和自身的LayoutParams来共同决定，MeasureSpec一旦确定后，onMeasure方法中就可以确定View的宽高。 
   - 当View采用固定宽高时，不管父容器的MeasureSpec是什么，View都是精确模式（EXACTLY），并且大小遵从LayoutParams中的大小 
   - View宽高是match_parent时，如果父容器是EXACTLY，那么View也是EXACTLY，大小为父容器剩余空间； 如果父容器是AT_MOST，那么View也是AT_MOST，大小为父容器剩余空间
   - View宽高是wrap_content时，不管父容器是什么模式，View都是AT_MOST，大小为父容器剩余空间


## View的工作流程
#### measure过程
   1. View的measure过程
View的测量过程由measure方法完成，measure方法是final类型方法，子类不能重写此方法，所以只看onMeasure方法的实现就行：

  ```
   protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(),  heightMeasureSpec));
    }

  public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }

   protected int getSuggestedMinimumHeight() {
        return (mBackground == null) ? mMinHeight : max(mMinHeight,     mBackground.getMinimumHeight());

    }

    protected int getSuggestedMinimumWidth() {
        return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
    }

  ```
   EXACTLY返回的MeasureSize是View测量后的大小。（这个是我们所关注的）

  （UNSPECIFIED用于系统内部的测量过程，我们一般不需要管它）
  UNSPECIFIED返回的是size 这个size就是getSuggestedMinimumHeight()和  getSuggestedMinimumWidth()
  分析一下getSuggestedMinimumHeight()方法，  getSuggestedMinimumWidth（）方法同理
   如果View没有设置背景，则返回mMinHeight等于android:minHeight所对应的值，这个值可以为0；如果设置了背景则返回android:minHeight和背景的最小宽度（mBackground.getMinimumHeight）这两者中的最大值

  注意：
直接继承View的自定义控件需要重写onMeasure方法并设置wrap_content时的自身大小，否则在布局中使用wrap_content就和使用math_parent的效果相同。因为根据源码getChildMeasureSpec()可以得出AT_MOST模式返回的大小为父容器目前可用最大大小

  2. ViewGroup的measure流程
  ViewGroup除了完成自己的measure外，还会遍历去调用所有子元素的measure方法，各个子元素再去递归去执行整个过程，和View不同的是，ViewGroup是一个抽象类，因此它没有重写View的onMeasure方法，但是它提供了一个叫measureChildren方法
 ```
     protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
        final int size = mChildrenCount;
        final View[] children = mChildren;
        for (int i = 0; i < size; ++i) {
            final View child = children[i];
            if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
                measureChild(child, widthMeasureSpec, heightMeasureSpec);
            }
        }
    }

  protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();

        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);

        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```
measureChild方法就是提取出子元素的LayoutParams，然后再通过getChildMeasureSpec创建子元素的MeasureSpec,接着把MeasureSpec传递给子元素的measure方法来进行测量

  在Activity的onCreate、onStart、onResume方法中获取View宽高的方法
  ```
     //第一种 此方法会被多次调用
     @Override
      public void onWindowFocusChanged(boolean hasFocus) {
        super.onWindowFocusChanged(hasFocus);
        if(hasFocus){
            int width = view.getMeasuredWidth();
            int height = view.getMeasuredHeight();
          }
         }

  //第二种将Runnable放到消息队列的尾部，等Looper调用此Runnable时， View也初始化好了
    @Override
    protected void onStart() {
        super.onStart();
        view.post(new Runnable() {
            @Override
            public void run() {
                int width = view.getMeasuredWidth();
                int height = view.getMeasuredHeight();
            }
        });
    }
      //第三种，当View树状态或者View树内部View可见行发生改变时，被调用。此方法会被多次调用
     view.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
         @Override
         public void onGlobalLayout() {
             int width = view.getMeasuredWidth();
             int height = view.getMeasuredHeight();
         }
     });

        //第四种，手动去调用measure测量 分情况
        //match_parent 不知道父容器的剩余空间，无法测量
        //宽高固定，都是100
        int widthMeasureSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
        int heightMeasureSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
        view.measure(widthMeasureSpec, heightMeasureSpec);

        //宽高是wrap_content
      int widthMeasureSpec1 = View.MeasureSpec.makeMeasureSpec(1<<30 -1, View.MeasureSpec.AT_MOST);
        int heightMeasureSpec1 = View.MeasureSpec.makeMeasureSpec(1<<30 -1, View.MeasureSpec.AT_MOST);
        view.measure(widthMeasureSpec1,heightMeasureSpec1);
  ```


#### layout过程  
layout的作用是ViewGroup用来确定子元素的位置，当ViewGroup的位置被确定后，它在onLayout中会遍历所有的子元素并调用其layout方法，在layout方法中onLayout方法又会被调用。
先看下layout方法的实现：
```
 public void layout(int l, int t, int r, int b) {
        if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
            onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }

        int oldL = mLeft;
        int oldT = mTop;
        int oldB = mBottom;
        int oldR = mRight;
      //通过setFrame确定四个顶点位置 此时在父容器中的位置已经确定
        boolean changed = isLayoutModeOptical(mParent) ?
                setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

        if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
            //调用onLayout方法确定子元素的位置
            onLayout(changed, l, t, r, b);
            if (shouldDrawRoundScrollbar()) {
                if(mRoundScrollbarRenderer == null) {
                    mRoundScrollbarRenderer = new RoundScrollbarRenderer(this);
                }
            } else {
                mRoundScrollbarRenderer = null;
            }
            mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnLayoutChangeListeners != null) {
                ArrayList<OnLayoutChangeListener> listenersCopy =
                        (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
                int numListeners = listenersCopy.size();
                for (int i = 0; i < numListeners; ++i) {
                    listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
                }
            }
        }

        final boolean wasLayoutValid = isLayoutValid();

        mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
        mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;

       下方代码省略。。
    }
```
onLayout方法是一个空实现，需要子元素自己实现

##### View的测量宽高和最终宽高
在View的默认实现中，测量宽高和最终宽高是相等的，只不过测量宽高形成于measure过程，最终宽高形成于layout过程，两者赋值的时机不同，测量宽高赋值的时机稍早些。重写layout方法并修改l、t、r、b数值会导致测量宽高和最终宽高不一致

#### draw过程
作用是将View绘制到屏幕上，步骤为：
1. 绘制背景background.draw(canvas)
2. 绘制自己（onDraw）
3. 绘制children（dispatchDraw）
4. 绘制装饰(onDrawScrollBars)
这些在源码上可以明显的看到
```
 public void draw(Canvas canvas) {
      上方代码省略。。
        /*
         * Draw traversal performs several drawing steps which must be executed
         * in the appropriate order:
         *
         *      1. Draw the background
         *      2. If necessary, save the canvas' layers to prepare for fading
         *      3. Draw view's content
         *      4. Draw children
         *      5. If necessary, draw the fading edges and restore layers
         *      6. Draw decorations (scrollbars for instance)
         */

        // Step 1, draw the background, if needed
        int saveCount;

        if (!dirtyOpaque) {
            drawBackground(canvas);
        }

        // skip step 2 & 5 if possible (common case)
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);

            // Step 4, draw the children
            dispatchDraw(canvas);

              代码省略。。。

            // Step 6, draw decorations (foreground, scrollbars)
            onDrawForeground(canvas);

            // Step 7, draw the default focus highlight
            drawDefaultFocusHighlight(canvas);

            if (debugDraw()) {
                debugDrawFocus(canvas);
            }

            // we're done...
            return;
        }

      代码省略。。。
    }
```


View绘制过程的传递是通过dispatchDraw来实现的，dispatchDraw会遍历调用所有子元素的draw方法，如此draw事件就一层层传递了下去
