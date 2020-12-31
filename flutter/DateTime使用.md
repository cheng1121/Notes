# DateTime

### 获取当前日期所在周
```
int weekday = DateTime.now().weekday;
///weekday 的范围是0到6

```

### 计算周日
```
DateTime now =DateTime.now();

DateTime sunday = now.subtract(Duration(days: now.weekday));
周日
```

### 计算每月开始的偏移天数(也就是第一周有几天是上一个月的)

```
///系统日历中的工具类
int firstDayOffset(int year, int month, MaterialLocalizations localizations) {
  // 0-based day of week for the month and year, with 0 representing Monday.
  final int weekdayFromMonday = DateTime(year, month).weekday - 1;

  // 0-based start of week depending on the locale, with 0 representing Sunday.
  int firstDayOfWeekIndex = localizations.firstDayOfWeekIndex;

  // firstDayOfWeekIndex recomputed to be Monday-based, in order to compare with
  // weekdayFromMonday.
  firstDayOfWeekIndex = (firstDayOfWeekIndex - 1) % 7;

  // Number of days between the first day of week appearing on the calendar,
  // and the day corresponding to the first of the month.
  return (weekdayFromMonday - firstDayOfWeekIndex) % 7;
}

```

### 计算一个月有多少天数
```
int getDaysInMonth(int year, int month) {
  if (month == DateTime.february) {
    final bool isLeapYear =
        (year % 4 == 0) && (year % 100 != 0) || (year % 400 == 0);
    if (isLeapYear) return 29;
    return 28;
  }
  const List<int> daysInMonth = <int>[
    31,
    -1,
    31,
    30,
    31,
    30,
    31,
    31,
    30,
    31,
    30,
    31
  ];
  return daysInMonth[month - 1];
}

```

### 是否是同一天
```
bool isSameDay(DateTime dateA, DateTime dateB) {
  return
    dateA?.year == dateB?.year &&
        dateA?.month == dateB?.month &&
        dateA?.day == dateB?.day;
}

```