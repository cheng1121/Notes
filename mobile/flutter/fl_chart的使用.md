# fl_chart的使用

`fl_chart`是一个使用`dart`语言编写的图表控件，其中包括常用的**条形图**、**饼状图**、**折线图**等，不仅可以添加基本统计数据，还可以使用动画让其动起来。

项目内使用的是空安全版本`0.20.1`

[pub地址](https://pub.dev/packages/fl_chart)

[github地址](https://github.com/imaNNeoFighT/fl_chart)

## 添加依赖

```yaml
dependencies:
  fl_chart: ^0.20.1
```

## 柱状图

### BarChart

柱状图的主体，其构造函数中有两个参数`data`和`duration`,其中`data`是必传参数用来构建柱状图的样式，`duration`是动画执行时长；

`BarChartData`是决定柱状图的样式，其参数包括：

#### 1. barGroups

柱状图的每个柱的样式和数据

```dart
BarChartGroupData(
          x: x,
          showingTooltipIndicators: <int>[0],
          barsSpace: 10,
          barRods: <BarChartRodData>[
            BarChartRodData(
                y: y, borderRadius: const BorderRadius.all(Radius.zero)),
          ]);
```

参数及作用：

1. `x`: 在x轴中的位置，正负都可
2. `barRods`:  柱形每段的样式集合；每个柱形都可以由多段组成，并使用`BarChartRodData`设置每段的数据和样式，`BarChartRodData`的参数
   - `y`：这一段数据是多少
   - `borderRadius`: 柱形的边框圆角
   - `colors`: 渐变色
   - `gradientFrom`：渐变色起始位置
   - `gradientTo`: 渐变色终点
   - `gradientColorStops`：渐变色终止点
   - `width`：柱形宽度
   - `backDrawRodData`: 在柱形下方绘制其他颜色（需要把`BarChartRodData`的`colors`参数设置为透明或者半透明，才能看到效果）
   - `rodStackItems`:堆积图表
3. `showingTooltipIndicators`：显示柱形上方的提示；当传`<int>[0]`时才显示提示，集合中0换成其他值就不显示
4. `barsSpace`：柱形之间的间距(实际设置无效)

#### 2. groupsSpace

当`alignment`是`BarChartAlignment.center`时，每个柱形之间的间距

#### 3. alignment

柱形的排列位置，如：居左`BarChartAlignment.start`,居中`BarChartAlignment.center`,环绕`BarChartAlignment.spaceAround`等等

#### 4. titlesData

柱的标题数据：可以设置旋转角度，控制哪个方向的内容显示，显示的间隔，距离柱形的间距等等；使用`FlTitlesData`,具体样式使用`SideTitles`设置,

`SideTitles`的参数：

- `showTitles`：是否显示
- `margin`: 与四周的边距
- `rotateAngle`: 文字的旋转角度
- `getTitles`:根据返回值设置显示的文字
- `getTextStyles`：文字样式
- `interval`：title显示的间隔
- `reservedSize`：title需要的空间
- `CheckToShowTitle`:动态确认title是否显示

#### 5. barTouchData

是否可点击柱形并决定提示的样式

- `enabled`：是否开启触摸
- `touchTooltipData`：提示框的样式
- `touchExtraThreshold`：触摸距离阈值
- `allowTouchBarBackDraw`:允许触摸柱形下方的绘制的内容
- `handleBuiltInTouches`: 触摸时是否显示提示
- `touchCallback`:触摸回调

#### 6. axisTitleData

柱形图每个轴添加标题数据(如：x轴表示：人数; y轴表示：分类)

#### 7. maxY

Y轴最大值

#### 8. minY

Y轴最小值

#### 9. gridData

柱形图添加背景表格，设置表格线的样式

#### 10. borderData

轴线的样式,是否显示等

#### 11. rangeAnnotations

动画

#### 11. backgroundColor

柱状图背景颜色

示例：

```dart

///柱状图
///bar chart
class AppBarChart extends StatelessWidget {
  const AppBarChart({Key key, @required this.data, @required this.titles})
      : assert(data.length == titles.length),
        super(key: key);

  ///每个类型的数量 [data.length]表示分类总数
  final List<int> data;

  ///与分类 对应的标题,既：类型名称
  ///[titles.length] == [data.length]
  final List<String> titles;

  @override
  Widget build(BuildContext context) {
    /// build column and set column style
    BarChartGroupData buildData(int x, double y) {
      return BarChartGroupData(x: x, showingTooltipIndicators: <int>[
        0
      ], barRods: <BarChartRodData>[
        BarChartRodData(
          y: y,
          colors: <Color>[Colors.blue],
          borderRadius: const BorderRadius.all(Radius.zero),
        ),
      ]);
    }

    ///column list
    final List<BarChartGroupData> barGroupData =
        List<BarChartGroupData>.generate(data.length, (int index) {
      final int count = data[index];
      return buildData(index, count.toDouble());
    });

    ///column tooltip and styles,can not rotate
    final BarTouchTooltipData touchTooltipData = BarTouchTooltipData(
      tooltipBgColor: Colors.transparent,
      tooltipPadding: const EdgeInsets.all(0),
      tooltipBottomMargin: 8,
      getTooltipItem: (
        BarChartGroupData group,
        int groupIndex,
        BarChartRodData rod,
        int rodIndex,
      ) {
        return BarTooltipItem(
          rod.y.round().toString(),
          TextStyle(
            color: Colors.blue,
            fontWeight: FontWeight.bold,
          ),
        );
      },
    );

    ///title and style
    final FlTitlesData flTitlesData = FlTitlesData(
      show: true,
      rightTitles: SideTitles(
          showTitles: true,
          margin: 5,
          interval: 20,
          rotateAngle: -90,
          getTitles: (double value) {
            return value.toInt().toString();
          }),
      leftTitles: SideTitles(showTitles: false),
      bottomTitles: SideTitles(
        showTitles: true,
        margin: 50,
        rotateAngle: -90,
        getTitles: (double value) {
          return titles[value.abs().toInt()];
        },
      ),
    );

    /// bar chart body
    Widget barChart = BarChart(
      BarChartData(
        ///column maximum
        ///列最大值
        maxY: 180,
        alignment: BarChartAlignment.spaceAround,

        barTouchData: BarTouchData(
          enabled: false,
          touchTooltipData: touchTooltipData,
        ),
        borderData: FlBorderData(
            show: true, border: Border(bottom: BorderSide(color: Colors.grey))),
        titlesData: flTitlesData,
        barGroups: barGroupData,
      ),
    );

    ///rotate pi/2
    ///todo 旋转90度，让条形图倒下（PS:可能会去掉,需修改[flTitlesData]）
    barChart = RotatedBox(
      quarterTurns: 1,
      child: barChart,
    );

    return barChart;
  }
}

```



## 饼状图

饼状图的主体是`PieChart`类，通过参数`data`,传入`PieChartData`控制饼状图的样式;

需要给一个固定的大小，或者使用`AspectRatio`确定大小比例

### 一、`PieChartData`的参数

#### 1. `sections`

是`PieChartSectionData`类型的集合，`PieChartSectionData`是构建圆弧样式，比例等数据的，具体参数在下方

#### 2. `centerSpaceRadius`

是饼状图中心部分圆形的半径，如果为`0`则，没有中心部分的圆形

#### 3. `centerSpaceColor`

中新部分圆形的颜色

#### 4. `sectionsSpace`

每段圆弧之间的间隔

#### 5. `startDegreeOffset`

饼状图的旋转角度，范围是`0~360`默认是0

#### 6. `pieTouchData`

圆弧的点击事件，由`PieTouchData`设置，`PieTouchData`的回调会返回`PieTouchResponse`,其内部封装了，点击的圆弧、拖动的距离、拖动的角度和速度等等数据

#### 7. `borderData`

饼状图的外边框，使用`FlBorderData`设置



### 二、`PieChartSectionData`的参数

#### 1. `value`

本段圆弧所占百分比

#### 2. `color`

本段圆弧的颜色

#### 3. `radius`

本段圆弧的宽度

#### 4. `showTitle`

是否显示圆弧的标题，位于对应圆弧上。可使用`title`参数设置标题展示的内容

#### 5. `title`

设置标题内容

#### 6. `badgeWidget`

设置圆弧上的提示内容，需传入`Widget`自定义展示的内容和样式

#### 7. `titlePositionPercentageOffset`

标题相对于圆弧中心的偏移量。`1`表示靠近外部,可以大于1

#### 8. `badgePositionPercentageOffset`

`badge widget`相对于圆弧中心的偏移量，`0`靠近内部，`1`靠近外部可以大于`1`



## 折线图

