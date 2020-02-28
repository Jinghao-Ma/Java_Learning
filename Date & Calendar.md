## Date & Calendar

------

[TOC]

------

### Date

**JDK API**

- java.util
  - Date
  - Calendar
- java.time (JDK >= 1.8)
  - LocalDate
  - LocalTime
  - ZonedDateTime
  - Instant



**获取当前时间**

- `new Date()`
- `toString()`
- `long getTime()`

**把`java.util.Date`转化为`String`**

- `toString()`
- `toGMTString()`
- `toLocaleString()`
- `SimpleDateFormat`



#### Code Demo

epoch time

```java
import java.util.Date;

/**
 * 获取long类型的epoch time
 * long epoch time和Date之间的相互转化
 */
public class Main {

    public static void main(String[] args) {
        // 获取epoch time
        // 即从1970年1月1日零点（GMT时区）到现在经历的秒数
        System.out.println(System.currentTimeMillis()); // 1573198105154

        Date now = new Date();
        System.out.println(now); // Fri Nov 08 15:28:25 CST 2019
    
        // .getTime()可获取任意时间的Long epoch time
        System.out.println(now.getTime()); // 1573198105162
        // 将Long epoch time转化为Date
        System.out.println(new Date(now.getTime())); // Fri Nov 08 15:28:25 CST 2019
    }
}
```

date to string

```java
import java.util.Date;

/**
 * Date2String
 * 将Date转化为String
 */
public class Date2String {

    public static void main(String[] args) {
        
        Date now  = new Date();

        System.out.println(now.toString()); // Fri Nov 08 15:35:00 CST 2019
        
        // 以下两种函数已被舍弃
        System.out.println(now.toGMTString()); // 8 Nov 2019 07:35:00 GMT
        System.out.println(now.toLocaleString()); // 08-Nov-2019 15:35:00
    }
}
```

date format

```JAVA
import java.text.SimpleDateFormat;
import java.util.Date;

/**
 * Format
 * 通过SimpleDateFormat将Date转化为不同的格式
 */
public class Format {

    public static void main(String[] args) {
        
        Date now = new Date();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        System.out.println(sdf.format(now)); // 2019-11-08 15:36:38

        SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss z");
        System.out.println(sdf1.format(now)); // 2019-11-08 15:36:38 CST

        SimpleDateFormat sdf2 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss Z");
        System.out.println(sdf2.format(now)); // 2019-11-08 15:36:38 +0800
    }
}
```



```java
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.Locale;

/**
 * Parse -- String 转化为 Date
 * 
 * 需根据String的格式来通过SimpleDateFormat来进行转换
 */
public class Parse {

    public static void main(String[] args) throws Exception{
        
        String s1 = "2019-11-08 16:11:11";
        Date date1 = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").parse(s1);
        System.out.println(date1); // Fri Nov 08 16:11:11 CST 2019

        String s2 = "Nov/08/2019 16:22:22";
        Date date2 = new SimpleDateFormat("MMM/dd/yyyy hh:mm:ss", Locale.US).parse(s2);
        System.out.println(date2); //  Fri Nov 08 16:22:22 CST 2019
    }
}
```

------



### Calendar

`java.util.calendar`

`Calender.getInstance()`  获取当前时间

 

**获取年、月、日、时、分、秒**：

- `Date getTime()`
- `long getTimeInMillis()`
- `get(int field)`

 

**设置指定时间：**

- `setTime(Date)`
- `setTimeInMillis(long)`
- `set(int field, int value)`

 

**转换时区：**

- `setTimeZone(TimeZone)`
- `get(int field, int value)`

 

**加减时间：**

- `add(int field, int value)`



#### Code Demo

```java
import java.util.Calendar;

/**
 * 
 * @ c -- the time code runing
 * .getTime()可转换为Date
 * .getTimeInMillis()可转换为Long epoch time
 * .get()可获取Calenadr中的各项数据
 * .getTimeZone() 获取对应的时区
 */
public class Main {

    public static void main(String[] args) {
        
        Calendar c = Calendar.getInstance();
        System.out.println(c); 
        // java.util.GregorianCalendar[time=1573201988651,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null],firstDayOfWeek=2,minimalDaysInFirstWeek=4,ERA=1,YEAR=2019,MONTH=10,WEEK_OF_YEAR=45,WEEK_OF_MONTH=1,DAY_OF_MONTH=8,DAY_OF_YEAR=312,DAY_OF_WEEK=6,DAY_OF_WEEK_IN_MONTH=2,AM_PM=1,HOUR=4,HOUR_OF_DAY=16,MINUTE=33,SECOND=8,MILLISECOND=651,ZONE_OFFSET=28800000,DST_OFFSET=0]

        System.out.println(c.getTime()); // Fri Nov 08 16:33:08 CST 2019
        System.out.println(c.getTimeInMillis()); // 1573201988651

        System.out.println("Year = " + c.get(Calendar.YEAR)); // Year = 2019
        // Month初始为0，即一月的值为0
        System.out.println("Month = " + c.get(Calendar.MONTH)); // Month = 10
        System.out.println("Day = " + c.get(Calendar.DAY_OF_MONTH)); //  Day = 8
        // Day_of_week初始值为1，表示星期天
        System.out.println("WeekDay = " + c.get(Calendar.DAY_OF_WEEK)); 
        // WeekDay = 6
        System.out.println("Hour = " + c.get(Calendar.HOUR_OF_DAY));
        // Hour = 16
        System.out.println("Minute = " + c.get(Calendar.MINUTE) );
        // Minute = 33
        System.out.println("Second = " + c.get(Calendar.SECOND));
        // Second = 8
        System.out.println("Millis = " + c.get(Calendar.MILLISECOND));
        // Millis = 651
        System.out.println("TimeZone = " + c.getTimeZone());
        //TimeZone = sun.util.calendar.ZoneInfo[id="Asia/Shanghai",offset=28800000,dstSavings=0,useDaylight=false,transitions=19,lastRule=null]
    }
}
```

set

```java
import java.util.Date;
import java.util.Calendar;

/**
 * SetTime
 * .set()可设置对应项的值
 */
public class SetTime {

    public static void main(String[] args) {
        
        Calendar c = Calendar.getInstance();
        Date date = c.getTime();

        System.out.println(date); // Fri Nov 08 16:34:39 CST 2019

        c.clear();
        c.set(Calendar.YEAR, 1992);
        c.set(Calendar.MONTH, 6);
        c.set(Calendar.DAY_OF_MONTH, 24);

        System.out.println(date); // Fri Nov 08 16:34:39 CST 2019
        System.out.println(c.getTime()); // Fri Jul 24 00:00:00 CST 1992
    }
}
```



```java
import java.util.Calendar;
import java.util.TimeZone;

/**
 * TimeZone
 * .setTimeZone() 可设置指定的时区
 */
public class Zone {

    public static void main(String[] args) {
        
        Calendar c = Calendar.getInstance();

        System.out.println(c.getTime());
        System.out.println(c.get(Calendar.HOUR_OF_DAY));

        c.setTimeZone(TimeZone.getTimeZone("America/New York"));

        System.out.println(c.get(Calendar.HOUR_OF_DAY));
        System.out.println(c.get(Calendar.DAY_OF_MONTH));
    }
}
```



```java
import java.util.Calendar;

/**
 *  Calculate
 * .add(Calendar.iterm, value) 可设置指定的日期和时间 
 */
public class Calculate {

    public static void main(String[] args) {
        
        Calendar c = Calendar.getInstance();

        System.out.println(c.getTime()); // Fri Nov 08 16:36:11 CST 2019

        c.add(Calendar.HOUR_OF_DAY, -5);
        c.add(Calendar.DAY_OF_MONTH, 25);

        System.out.println(c.getTime()); // Tue Dec 03 11:36:11 CST 2019
    }
}
```

