# Drawable
### mipmap文件夹
google专门用来放置应用icon的。有两个功能：
1. 把icon专门独立出来放置，方便查找
2. 可以让图标自动拥有跨设备密度展示的能力，比如说一台屏幕密度为xxhdpi的设备可以自动加载mipmap-xxxhdpi中的icon作为应用的图标，这样图标看上去就更加细腻

launcher图标的尺寸：
密度 | 尺寸
---|---
mipmap-mdpi | 48 * 48
mipmap-hdpi | 72 * 72
mipmap-xhdpi | 96 * 96 
mipmap-xxhdpi | 144 * 144
mipmap-xxxhdpi | 192 * 192

### drawable文件夹
##### 获取屏幕密度的方法
```
float xdpi = getResources().getDisplayMetrics().xdpi;
float ydpi = getResources().getDisplayMetrics().ydpi;

我的手机运行结果：
dpi x = 537.882 dpi y = 537.388
```
其中xdpi代表宽度的dpi值，ydpi代表高度的dpi值，取值范围如下：
dpi范围(dpi) | 密度
---|---
0 ~ 120 | ldpi
120 ~ 160 | mdpi 
160 ~ 240 | hdpi
240 ~ 320 | xhdpi
320 ~ 480 | xxhdpi
480 ~ 640 | xxxhdpi

所以我的手机是属于xxxhdpi的

##### drawable使用
系统匹配图片的规则：
1. 系统会根据屏幕密码，去对应密度的drawable文件夹下查找与资源id对应的图片
2. 如果与屏幕密度相对应的文件夹中没有相应图片，则会去更高密度的drawable文件夹下去读取
3. 没有更高密度的drawable文件夹，则去drawable-nodpi文件夹下读取
4. 如果drawable-nodpi没有，则去比屏幕密度低的drawable文件夹下读取
5. 如果在比当前屏幕密度小的drawable文件夹中读取到了图片，则系统会认为此图片是专门为低密度的设备制作的，显示在高密度的设备上会出现像素过低的问题，所以系统会把图片放大
6. 同第五条，如果在高密度的drawable文件夹中读取到图片，会认为图片像素过高，会把图片缩小
7. 放大或者缩小倍率为：当前设备密度所在dpi范围的最大值与图片所在drawable文件夹对应密度范围dpi的最大值的比值

- 以我的手机为例，图片匹配顺序为：xxxhdpi > nodpi > xxhdpi > xhdpi > hdpi > mdpi > ldpi
   1. 图片放大或者缩小的倍率：根据第7条和上方密度表可知，设备的dpi为640，xxxhdpi读取到图片后不做处理
   2. nodpi读取到图片，不做处理（该文件夹主要存放不能被拉伸的图片）
   3. xxhdpi 480dpi 放大 640/480 = 1.3倍
   4. xhdpi 320dpi 放大 640/320 =2倍
   5. hdpi 240dpi 放大 640/240 =2.7倍
   6. mdpi 160dpi 放大 640/160 = 4倍
   7. ldpi 120dpi 放大 640/120 = 5.3倍

 
- 另外一个屏幕密度为 xxhdpi的设备的图片匹配顺序为：xxhdpi > xxxhdpi > nodpi > xhdpi > hdpi > mdpi > ldpi
  1. xxhdpi 480dpi 图片不做处理
  2. xxxhdpi 640dpi 缩小 480/640 = 0.75倍
  3. nodpi 不做处理
  4. xhdpi 320dpi 放大 480/320 = 1.5倍
  5. hdpi 240dpi 放大 480/240 = 2倍
  6. mdpi 160dpi 放大 480/160 = 3倍
  7. ldpi 120dpi 放大 480/120 = 4倍
  

日常开发过程中，可以只做xxhdpi和xxxhdpi两套图，用以节省内存开销，和缩小apk体积




内容参考郭神：

[Android drawable微技巧，你所不知道的drawable的那些细节](https://blog.csdn.net/guolin_blog/article/details/50727753#commentsedit)