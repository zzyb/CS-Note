# Java的Calendar与Date的总结

## Date

### 继承层次：

- [java.lang.Object](itss://chm/java/lang/Object.html)
- - java.util.Date

### 解释：

**Date类表示特定的时间瞬间，精度为毫秒。**

`毫秒值`表示<u>1970年1月1日00：00：00.000 GMT</u>之后的毫秒数。

### 代码：

```java
Date date1 = new Date();
//创建一个当前时间的Date对象。
Date date2 = new Date(946656000000L);
//传入毫秒值，创建一个Date对象。
```

- 946656000000L 代表了2000年1月1日0时的毫秒值。
- 对Date对象直接print得到的是默认格式`Thu Nov 07 11:05:09 CST 2019`

## Calendar

### 继承层次：

- [java.lang.Object](itss://chm/java/lang/Object.html)
- - java.util.Calendar

### 解释：

Calendar类是一个<u>抽象类</u>，**提供用于在特定时间点和一组日历字段（例如YEAR，MONTH，DAY_OF_MONTH，HOUR等）之间进行转换的方法，以及用于处理日历字段（例如获取日期的下一周）的方法**。时间的瞬间可以用毫秒值表示，该值是从格林尼治标准时间1970年1月1日00：00：00.000到纪元的偏移量。

### 代码：

```java
//创建日历类型对象（使用静态方法getInstance）
        Calendar c = Calendar.getInstance();
        //直接输出日历对象（不是想要的结果！！！）！！！
        System.out.println(c);
        //通过Calendar的getTime方法，转化为Date类型对象。（底层:getTimeInMillis「获得当前时间的毫秒表示」）
        Date date3 = c.getTime();
        System.out.println(date3);

				//Calendar的get方法，可以获得年、月、日、周、月的第几天等
        int year = c.get(Calendar.YEAR);
        int month = c.get(Calendar.MONTH)+1;//月是从0开始的，需要加1
        int day = c.get(Calendar.DAY_OF_MONTH);
        System.out.println(year+">"+month+">"+day);

				//Calendar的add方法，可以对年、月、日等 +或-去指定数值
        c.add(Calendar.YEAR,1);//对年加1
        year = c.get(Calendar.YEAR);
        month = c.get(Calendar.MONTH)+1;
        day = c.get(Calendar.DAY_OF_MONTH);
        System.out.println(year+">"+month+">"+day);

				//将Date转化为Calendar
        Date date4 = new Date();
        Calendar c4 = Calendar.getInstance();
        c4.setTime(date4);//！！！使用setTime方法，传入Date对象。
        year = c4.get(Calendar.YEAR);
        month = c4.get(Calendar.MONTH)+1;
        day = c4.get(Calendar.DAY_OF_MONTH);
        System.out.println(year+">"+month+">"+day);
```



## SimpleDateFromat

### 继承层次：

- [java.lang.Object](itss://chm/java/lang/Object.html)
- - [java.text.Format](itss://chm/java/text/Format.html)
  - - [java.text.DateFormat](itss://chm/java/text/DateFormat.html)
    - - java.text.SimpleDateFormat

### 解释：

`SimpleDateFormat`是一个具体的类，用于以区域设置敏感的方式<u>格式化和解析</u>**日期**。

### 代码：

```java
				//定一个一个Date对象date1
				Date date1 = new Date(); 

				//设置特定的日期格式
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        //使用SimpleDateFromat的format(new Date())方法，将Date默认日期类型转化为sdf指定日期格式
        String date1_str = sdf.format(date1);
        System.out.println(date1_str);//输出得到"yyyy-MM-dd HH:mm:ss"格式的日期。

        String date2_str = "2015-9-12 10:45:10";
        //需要处理异常！
        //SimpleDateFormat的parse(String)方法可以将规定格式的String转化为Date类型
        Date date2 = sdf.parse(date2_str);//得到"2015-9-12 10:45:10"转化的Date对象
        System.out.println(date2);//输出
```

- DateFormat.format(Date date)方法；
  - 传入Date类型，返回String 类型（规定格式）。
- DateFormat.parse(String source)方法；
  - 传入String类型（规定格式），返回Date类型。