---
title: Java 获取 自定义时间格式时间、上周(周一、周日)、上个月(第一天、最后一天)
toc: true
layout: blog
categories:
  - blog
  - Java
tags:
  - Java
  - Calendar
  - Date
date: 2019-02-28 11:00:13
---
{{title}}
<!-- more -->

# 获取时间格式工具类

## 获取当前日期/指定日期的 上周(周一、周日) 时间	 List<String>

```java
/**
 * 
 * @Title: getLastWeekMondayAndSunday
 * @Description: 获取指定日期的 上周:周一、周日 时间
 * @param date 指定日期
 * @return
 * List<String>
 */
public static List<String> getLastWeekMondayAndSunday(Date date) {
	List<String> list = new ArrayList<>(2);
	SimpleDateFormat df = new SimpleDateFormat("yyyyMMdd");
	Calendar calendar = Calendar.getInstance();

	// 获取指定日期的 上个星期周一、周日时间
	if (date != null) {
		calendar.setTime(date);
	}

	// 一周七天
	calendar.add(Calendar.DAY_OF_WEEK, -7);
	// 设置每周第一天为 周一
	calendar.setFirstDayOfWeek(Calendar.MONDAY);

	calendar.set(Calendar.DAY_OF_WEEK, Calendar.MONDAY);
	String monday = df.format(calendar.getTime());
	list.add(monday);

	calendar.set(Calendar.DAY_OF_WEEK, Calendar.SUNDAY);
	String sunday = df.format(calendar.getTime());
	list.add(sunday);

	return list;
}

/**
 * 
 * @Title: getLastWeekMondayAndSunday
 * @Description: 获取当前时间,上周：周一、周日时间
 * @return
 * List<String>
 */
public static List<String> getLastWeekMondayAndSunday() {
	List<String> list = new ArrayList<>(2);
	SimpleDateFormat df = new SimpleDateFormat("yyyyMMdd");
	Calendar calendar = Calendar.getInstance();
	
	// 一周七天
	calendar.add(Calendar.DAY_OF_WEEK, -7);
	// 设置每周第一天为 周一
	calendar.setFirstDayOfWeek(Calendar.MONDAY);

	calendar.set(Calendar.DAY_OF_WEEK, Calendar.MONDAY);
	String monday = df.format(calendar.getTime());
	list.add(monday);

	calendar.set(Calendar.DAY_OF_WEEK, Calendar.SUNDAY);
	String sunday = df.format(calendar.getTime());
	list.add(sunday);

	return list;
}
```

## 获取当前/指定日期的 上个月(第一天、最后一天) 时间 List<String>	

```java
/**
 * 
 * @Title: getLastMonthFirstDayAndLastDay
 * @Description: 获取当前时间,上个月:第一天、最后一天
 * @return
 * List<String>
 */
public static List<String> getLastMonthFirstDayAndLastDay() {
	List<String> list = new ArrayList<>(2);
	// 时间格式
	SimpleDateFormat df = new SimpleDateFormat("yyyyMMdd");
	Calendar calendar = Calendar.getInstance();

	// 上个月的最后一天
	calendar.set(Calendar.DAY_OF_MONTH, 0);
	String lastDay = df.format(calendar.getTime());

	// 上个月的第一天
	calendar.set(Calendar.DAY_OF_MONTH, 1);
	String firstDay = df.format(calendar.getTime());

	list.add(firstDay);
	list.add(lastDay);
	return list;
}

/**
 * 
 * @Title: getLastMonthFirstDayAndLastDay
 * @Description: 获取指定日期,上个月:第一天、最后一天
 * @param date	
 * 	指定日期
 * @return
 * List<String>
 */
public static List<String> getLastMonthFirstDayAndLastDay(Date date) {
	List<String> list = new ArrayList<>(2);
	// 时间格式
	SimpleDateFormat df = new SimpleDateFormat("yyyyMMdd");
	Calendar calendar = Calendar.getInstance();
	calendar.setTime(date);
	// 上个月的最后一天
	calendar.set(Calendar.DAY_OF_MONTH, 0);
	String lastDay = df.format(calendar.getTime());

	// 上个月的第一天
	calendar.set(Calendar.DAY_OF_MONTH, 1);
	String firstDay = df.format(calendar.getTime());

	list.add(firstDay);
	list.add(lastDay);
	return list;
}
```

## 获取当前日期 0时0分0秒 时间	

```java
/**
 * 
 * @Title: todayZero
 * @Description: 获取当前日期0时0分0秒时间
 * @param formartStr(时间格式:yyyyMMdd等)
 * @return
 * String
 */
public static String getTodayZero(String formartStr) {
	Calendar calendar = Calendar.getInstance();
	calendar.setTime(new Date());
	calendar.set(Calendar.HOUR_OF_DAY, 0);
	calendar.set(Calendar.MINUTE, 0);
	calendar.set(Calendar.SECOND, 0);

	SimpleDateFormat format = new SimpleDateFormat(formartStr);
	return format.format(calendar.getTime());
}
```

## 当前日期多少天前的时间	

```java
/**
 * 
 * @Title: getBeforeDay
 * @Description: 获取自定义格式的 ; 多天前时间
 * @param before ：多少天之前,例如：3
 * @param formatStr:时间格式
 * 	 例如： yyyyMMdd | yyyyMMddHH | yyyyMMddHHmm
 * @return
 * String
 */
public static String getBeforeDay(int before, String formatStr) {

	// 24 * 60 * 60 * 1000 = 86400000
	/**
	Date day = new Date(System.currentTimeMillis() - (before * 86400000));
	SimpleDateFormat format = new SimpleDateFormat(formatStr);
	return format.format(day);
	*/

	Calendar calendar = Calendar.getInstance();
	calendar.add(Calendar.DATE, -before);
	SimpleDateFormat format = new SimpleDateFormat(formatStr);
	return format.format(calendar.getTime());
}
```

## 指定日期的N天之后的日期

```java
/**
 * 
 * @Title: getAfterMonth
 * @Description: 获取N天之后日期
 * @param number
 * @return
 * Long
 */
public static Date getAfterDay(Date date, int number) {

	Calendar calendar = Calendar.getInstance();
	calendar.setTime(date);
	calendar.add(Calendar.DATE, number);

	return calendar.getTime();
}
```