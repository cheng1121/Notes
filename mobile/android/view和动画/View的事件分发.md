图片参考：[【源码解析】View的事件分发](https://www.jianshu.com/p/5ec375057127)

![image.png](https://upload-images.jianshu.io/upload_images/11142016-6f57843bea6e92cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


总结：
1. 同一个事件序列是指从手指触摸屏幕到离开屏幕的过程中所产生的一系列事件，以down开始，中间一系列的move事件，最终以up事件结束
2. 正常情况下一个事件序列只能被一个View拦截且消耗。通过特殊手段可以做到，如View将自己处理的事件通过onTouchEvent强行传递给其他View处理
3. 一个View一旦决定拦截，那么这一个事件序列都会由它来处理，并且它的onInterceptTouchEvent方法不会再被调用
4. 一个View决定处理事件，如果事件不被消耗（onTouchEvent返回false），那么整个事件序列将返回给父View的onTouchEvent方法来处理
5. 如果View不消耗除了ACTION_DOWN以外的其他事件，那么这个事件将消失，父View的onTouchEvent不会被调用，后续事件继续传递给当前View，最终这些事件会传递给Activity来处理
6. View没有onInterceptTouchEvent,一旦有点击事件传递给它，那么它的onT ouchEvent就会被触发
7. View的onTouchEvent默认会消耗事件，除非它是不可点击状态。View的longClickable属性默认为false，clickable属性分情况，比如Button就默认为true，TextView默认为false
8. View的enable属性不影响onTouchEvent的默认返回值。
9. onClick会发生的前提是当前View是可点击的，并且它收到了down和up事件
10. 事件传递过程是由外向内的，即事件总是先传递给父元素，然后再由父元素分发给子View，通过requestDisallowInterceptTouchEvent方法可以在子元素中干预父元素的事件分发过程，但是ACTION_DOWN事件除外