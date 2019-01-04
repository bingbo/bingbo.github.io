### 当前日期时间,LocalDate,LocalDateTime,LocalTime表示

```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
DateTimeFormatter formatter1 = DateTimeFormatter.ofPattern("yyyy-MM-dd");
//当前时间
LocalTime time=LocalTime.now();
//当前日期时间
LocalDateTime date=LocalDateTime.now();
```

### 日期时间格式化及解析

> 注意这里的格式化中，如果字符串时间是xxxx-xx-xx xx:xx:xx，则解析时指定的fomatter格式也必须是yyyy-MM-dd HH:mm:ss，否则解析失败

```java
//格式化日期或时间为字符串
String d = date.format(formatter);
System.out.println("格式化时间：" + d);
//2017-03-10T16:30:19.528
System.out.println("格式化时间1：" +DateTimeFormatter.ISO_LOCAL_DATE_TIME.format(LocalDateTime.now()));
//2017-3-10 16:33:12
System.out.println("格式化时间2：" +DateTimeFormatter.ofLocalizedDateTime(FormatStyle.MEDIUM).format(LocalDateTime.now()));
//解析时间字符串为LocalDateTime
LocalDateTime dateTime = LocalDateTime.parse(d, formatter);
System.out.println(dateTime);
```

### 时间缀Instant表示

```java
//Instant表示某一个时间点的时间戳，可以类比于java.uti.Date
Instant instant=Instant.now();
//当前时间秒数，即时间缀
System.out.println(instant.getEpochSecond());
System.out.println(instant.getNano());
```

### LocalDate,LocalDateTime与Date等新旧时间日期转换

```java
//LocalDateTime转Date
Date date1 = Date.from(date.atZone(ZoneId.systemDefault()).toInstant());
System.out.println(date1);
System.out.println(ZoneId.systemDefault());

//Date转LocalDateTime
Date date2 = new Date();
LocalDateTime dateTime1 = LocalDateTime.ofInstant(date2.toInstant(), ZoneId.systemDefault());
System.out.println(formatter.format(dateTime1));
```

### 前后加减日期时间操作

```java
//获取当前日期
System.out.println(LocalDate.now().format(formatter1));
//获取当前日期的前一天
System.out.println(LocalDate.now().minusDays(1).format(formatter1));
//获取当前日期的前一月
System.out.println(LocalDate.now().minusMonths(1).format(formatter1));
//获取当前日期的前一周
System.out.println(LocalDate.now().minusWeeks(1).format(formatter1));

//当前日期时间
System.out.println(LocalDateTime.now().format(formatter));
//当前日期时间的前一天
System.out.println(LocalDateTime.now().minusDays(1).format(formatter));
//当前日期时间的前一月
System.out.println(LocalDateTime.now().minusMonths(1).format(formatter));
//当前日期时间的前一周
System.out.println(LocalDateTime.now().minusWeeks(1).format(formatter));
//当前日期时间的前一小时
System.out.println(LocalDateTime.now().minusHours(1).format(formatter));
//当前日期时间的前一秒
System.out.println(LocalDateTime.now().minusSeconds(1).format(formatter));
```

### 获取指定时间的日期

```java
LocalDateTime midnight = LocalDateTime.now().withHour(0).withMinute(0).withSecond(0);
System.out.println(midnight.format(formatter));

//获取当天午夜00-00-00的日期时间
LocalTime midTime = LocalTime.MIDNIGHT;
LocalDate today = LocalDate.now();
LocalDateTime todayMidnight = LocalDateTime.of(today, midTime);
System.out.println(todayMidnight);
//等价于
LocalDateTime todayMidnight1 = today.atTime(midTime);
System.out.println(todayMidnight1);
//等价于
LocalDateTime atStartOfDay = LocalDate.now().atStartOfDay();
System.out.println("一天的开始：" + atStartOfDay.format(formatter));


//明天午夜00-00-00的日期时间
LocalDateTime tomorrowMidnight = todayMidnight.plusDays(1);
System.out.println(tomorrowMidnight);
```

### Duration表示Instant之间的时间差，可以用来统计任务的执行时间，也支持各种运算操作

```java
//计算两个时间相差的秒数
long secs = ChronoUnit.SECONDS.between(midnight, LocalDateTime.now());
System.out.println(secs);
long nanos = Duration.between(midnight, LocalDateTime.now()).toNanos();
long secs1 = TimeUnit.NANOSECONDS.toSeconds(nanos);
System.out.println(secs1);

//计算两个时间的相差的分钟数
long minits = ChronoUnit.MINUTES.between(midnight, LocalDateTime.now());
System.out.println(minits);
long minits1 = Duration.between(midnight, LocalDateTime.now()).toMinutes();
System.out.println(minits1);

//计算两个时间的相差的小时数
long hours = ChronoUnit.HOURS.between(midnight, LocalDateTime.now());
System.out.println(hours);
long hours1 = Duration.between(midnight,LocalDateTime.now()).toHours();
System.out.println(hours1);
```

### Period用来表示两个LocalDate之间的时间差

> 注：Period.between()是计算两个时间在一个周期内相差天数，如一个月内，一周内，即30天，7天为一个周期内相差的天数，如2017-03-10与2017-04-10这之间的时间能正常计算出相差的天数，但如果相差大于一个周期（月，周）等则会出现问题，并不是实际相差的天数，通过`eforeDate.until(nowDate, ChronoUnit.DAYS)`可获取实际相差天数

```java
LocalDate today1 = LocalDate.now();
LocalDate yesterday = LocalDate.of(today1.getYear(), today1.getMonth(), today1.getDayOfMonth() - 1);
Period period = yesterday.until(today1);
System.out.println("period: " + period);
//等价于
period = Period.between(yesterday, today1);
System.out.println("period: " + period);
System.out.println(period.isNegative());
System.out.println(period.getDays());

//util与period.between的区别
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDate nowDate= LocalDate.parse("2017-04-06 14:30:47", formatter);
LocalDate beforeDate =LocalDate.parse("2017-03-06 14:30:47", formatter);
int diffDays = Period.between(beforeDate, nowDate).getDays();

System.out.println(diffDays);
System.out.println(beforeDate.until(nowDate, ChronoUnit.DAYS));
System.out.println(beforeDate.until(nowDate).getDays());

//结果
0
31
0
```


### TemporalAdjusters时间调节器，用于表示某个月第一天、下个周一等日期

```java
LocalDate firstDay = LocalDate.now().with(TemporalAdjusters.firstDayOfMonth());
System.out.println("每月的第一天：" + firstDay);
LocalDate lastDay = LocalDate.now().minusMonths(1).with(TemporalAdjusters.firstDayOfMonth());
System.out.println("上个月的第一天：" + lastDay);
LocalDate ld = LocalDate.now().with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
System.out.println("每月的最后一个星期日：" + ld);
LocalDate ld1 = LocalDate.now().with(TemporalAdjusters.nextOrSame(DayOfWeek.MONDAY));
System.out.println("下一个星期一：" + ld1);
LocalDate ld2 = LocalDate.now().with(TemporalAdjusters.previousOrSame(DayOfWeek.MONDAY));
System.out.println("上一个星期一：" + ld2);
LocalDate ld3 = LocalDate.now().with(TemporalAdjusters.lastDayOfMonth());
System.out.println("一个月的最后一天：" + ld3);
LocalDate ld4 = LocalDate.now().minusDays(1);
System.out.println("前一天：" + ld4);
```

### 日期格式

```
年       yy: 16      yyyy: 2016
月       M: 1        MM: 01
日       d: 3        dd: 03
周       e: 3        E:    Web
时       H: 9        HH: 09
钟       mm: 02
秒       ss: 00
纳秒      nnnnnn:000000
时区偏移    x: -04     xx:-0400
```

### 日历系统-Chronology

java8提供了4套日历：

* ThaiBuddhistDate：泰国佛教历
* MinguoDate：中华民国历
* JapaneseDate：日本历
* HijrahDate：伊斯兰历

```java
//公历转换
LocalDate dd = LocalDate.now();
//民国日期
MinguoDate minguoDate = MinguoDate.from(dd);
System.out.println(dd);
//日本日期 Japanese Heisei 29-03-10
System.out.println(JapaneseDate.from(dd));
//等价于当前日期 Japanese Heisei 29-03-10
ChronoLocalDate chronoLocalDate = JapaneseChronology.INSTANCE.dateNow();
System.out.println(chronoLocalDate);
```
