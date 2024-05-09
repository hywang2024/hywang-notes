## 计算日期距今
 1. 百度搜索日历，选之前或者之后一天，会计算距离今天有多少天，是过去多少天，还是还有多少天
 2. 生日一般是农历，又是每年都过，所以是一个重复的日期，所以设计函数的时候都考虑到
 3. 大致思路就是使用Java工具将日期转换成农历

## 实现代码
```java

public class IntervalDayUtils {
    public static void main(String[] args) { 
        int days = days(2, 1, 2023, 12, 11);
        System.out.println(days);
    }

    public static final String DATE_FMT_3 = "yyyy-MM-dd";


    /**
     * @param repeat 是否每年重复 1:是 生日每年都过
     * @param solar  1是阳历 2农历
     * @param year   年
     * @param month  月
     * @param day    日
     * @return
     */
    public static int days(int repeat, int solar, int year, int month, int day) {
        int status;
        int intervalDays;
        String monthStr = month > 9 ? month + "" : "0" + month;
        String dayStr = day > 9 ? day + "" : "0" + day;
        LocalDate localDate = stringToLocalDate(year + "-" + monthStr + "-" + dayStr, DATE_FMT_3);
        if (2 == solar) {
            //农历转 阳历
            localDate = lunarToSolar(year, month, day);
        }
        LocalDate nowLocalDate = LocalDate.now();
        //判断输入日期是否在当前时间之后
        if (localDate.isAfter(nowLocalDate)) {
            status = 0;
            intervalDays = (getIntervalDays(nowLocalDate, localDate));
        } else {
            //判断是不是当天
            if (nowLocalDate.equals(localDate)) {
                status = 0;
                intervalDays = (getIntervalDays(nowLocalDate, localDate));
            } else {
                //输入日期是在之前的日期
                //1.是今年的  2，不是今年的
                int nowYear = nowLocalDate.getYear();
                int monthValue = localDate.getMonthValue();
                int dayOfMonth = localDate.getDayOfMonth();
                //如果是闰年 考虑2.29的问题
                if (!DateUtils.isLeapYear(nowYear) && 2 == monthValue && 29 == dayOfMonth) {
                    monthValue = 3;
                    dayOfMonth = 1;
                }
                //把输入日期换成今年的年份， 判断是在之前 还是之后
                LocalDate newLocalDate = LocalDate.of(nowYear, monthValue, dayOfMonth);
                if (2 == solar) {
                    //换成农历
                    LocalDate nowSolarDate = lunarToSolar(nowLocalDate.getYear(), nowLocalDate.getMonthValue(), nowLocalDate.getDayOfMonth());
                    if (nowSolarDate.getMonthValue() <= month || nowSolarDate.getDayOfMonth() <= day) {
                        nowYear = nowYear - 1;
                    }
                    //先换成今年农历的那一天，再转成阳历 计算
                    LocalDate la = stringToLocalDate(nowYear + "-" + month + "-" + day, DATE_FMT_3);
                    newLocalDate = lunarToSolar(la.getYear(), la.getMonthValue(), la.getDayOfMonth());
                }
                if (newLocalDate.isAfter(nowLocalDate)) {
                    if (1 == repeat) {
                        status = 1;
                        intervalDays = getIntervalDays(nowLocalDate, newLocalDate);
                    } else {
                        status = 2;
                        intervalDays = getIntervalDays(nowLocalDate, localDate);
                    }
                } else {
                    //当天
                    if (newLocalDate.equals(nowLocalDate)) {
                        if (1 == repeat) {
                            status = 0;
                            intervalDays = 0;
                        } else {
                            status = 2;
                            intervalDays = getIntervalDays(nowLocalDate, localDate);
                        }
                    } else {
                        //之前
                        if (1 == repeat) {
                            status = 1;
                            int nextYear = (nowYear + 1);
                            int newMonthValue = newLocalDate.getMonthValue();
                            int dayOfMonth2 = newLocalDate.getDayOfMonth();
                            if (!DateUtils.isLeapYear(nextYear) && 2 == newMonthValue && 29 == dayOfMonth2) {
                                newMonthValue = 3;
                                dayOfMonth2 = 1;
                            }
                            LocalDate nextLocalDate = LocalDate.of(nextYear, newMonthValue, dayOfMonth2);
                            if (2 == solar) {
                                //先换成今年农历的那一天，再转成阳历 计算
                                LocalDate la = stringToLocalDate(nextYear + "-" + month + "-" + day, DateUtils.DATE_FMT_3);
                                nextLocalDate = lunarToSolar(la.getYear(), la.getMonthValue(), la.getDayOfMonth());
                            }
                            intervalDays = getIntervalDays(nowLocalDate, nextLocalDate);
                        } else {
                            status = 2;
                            intervalDays = getIntervalDays(nowLocalDate, localDate);
                        }
                    }
                }

            }
        }

        if (2 == status) {
            return -intervalDays;
        }
        return intervalDays;
    }

    /**
     * 阳历转农历
     * @param year
     * @param intMonth
     * @param intday
     * @return
     */
    private static LocalDate lunarToSolar(int year, int intMonth, int intday) {
        int[] ints = LunarCalendar.lunarToSolar(year, intMonth, intday, DateUtils.isLeapYear(year));
        if (ints.length >= 3) {
            String lunarYear = ints[0] + "";
            String lunarMonth = ints[1] > 9 ? ints[1] + "" : "0" + ints[1];
            String lunarDay = ints[2] > 9 ? ints[2] + "" : "0" + ints[2];
            return DateUtils.stringToLocalDate(lunarYear + "-" + lunarMonth + "-" + lunarDay, DateUtils.DATE_FMT_3);
        }
        return LocalDate.now();
    }

    /**
     * 获取两个时间间隔 天数
     * @param before
     * @param after
     * @return
     */
    public static int getIntervalDays(LocalDate before, LocalDate after) {
        return (int) Math.abs((before.toEpochDay() - after.toEpochDay()));
    }

    /**
     * string 转 LocalDate
     *
     * @param dateStr 例："2017-08-11"
     * @param format  例："yyyy-MM-dd"
     * @return
     */
    public static LocalDate stringToLocalDate(String dateStr, String format) {
        try {
            DateTimeFormatter formatter = DateTimeFormatter.ofPattern(format);
            return LocalDate.parse(dateStr, formatter);
        } catch (DateTimeParseException e) {
            e.printStackTrace();
        }
        return null;
    }
}

```
