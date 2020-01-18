---
title: 时间魔法 Swift篇
date: 2016-05-27
tags: Swift
---
# 简介
* NSDate 。在日期编程中，这个对象描述了日期和时间信息。可以把日期和时间看做是类中的普通属性，它不但用于日期，也用于时间处理。格式化，这个概念在直接处理 NSDate 对象时还用不到，只有在将日期对象转换为字符串对象时，才能用到格式化。
* NSDateComponents 。这个类可以简单的看做是 NSDate 的“姐妹”类，因为它为开发者带来了许多关于日期的便捷操作。其中一项重要内容是：它可以将日期和时间分割成独立的属性，这样就可以直接访问每项属性，这在诸如日期计算之类的任务中非常有用。
<!-- more -->
 除了上面这些功能外， NSDateComponents 类在计算过去或未来的时间上也很有用。只需要简单的对某个子属性（年，月年等）执行加减操作，就可以算出未来或过去的一个时间。另外，NSDateComponents 类还适合查找两个日期之间的间隔。
* NSCalendar 。这个类的功能并不在本文的讨论范围，但是，NSDate 和 NSDateComponents 之间的互相转换，却是由 NSCalendar 类来控制的，因为需要制定某个 NSCalendar 对象，才能完成转换。事实上，系统在进行转换时，需要知道使用的日历（历法）是哪个，然后才能获得正确的转换结果。要知道，世界上有许多不同的日历，其年月日的值是各不相同的。
*  NSDateFormatter 。这个类会帮助我们将 NSDate 对象转换为字符串对象，也可以将字符串对象转换为 NSDate 对象。通过它，可以将 NSDate 对象按照预定义的日期样式直接转换成字符串，也可以按自定义的日期格式进行转换。
 NSDateFormatter 对象也支持本地化功能，只需要提供一个有效的 NSLocale 对象，就可以按照给定的locale设置转换成合适的字符串内容。
* NSDateComponentsFormatter 。它有一个重要目的：输入日期和时间，输出格式化好的可读字符串。它包含了许多方法来完成这个任务。

***
<!-- more -->
![time](http://o7ttfnm00.bkt.clouddn.com/5.jpg)

* [NSDate](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSDate_Class/)
* [NSDateFormatter](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSDateFormatter_Class/index.html)
* [NSDateComponentFormatter](https://developer.apple.com/library/watchos/documentation/Foundation/Reference/NSDateComponentsFormatter_class/index.html)
* [NSCalendar](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSCalendar_Class/)
***

# NSDate
```
//获取当前时间
let date1 = NSDate()     //"May 27, 2016, 2:36 PM"
let str1 = String(date1) //"2016-05-27 06:36:17 +0000"

//获取从1970年1月1日00:00到当前时间的秒数
var interval : NSTimeInterval = date1.timeIntervalSince1970 //1464330977.49057 

let date2 = NSDate()
//计算时间差
interval = date2.timeIntervalSinceDate(date1) //0.2234339714050293
//date1 距现在的时间差
interval = date1.timeIntervalSinceNow         //-0.2241280078887939

//得到date2后一天的时间对象
let date3 = date2 .dateByAddingTimeInterval(24*3600)             // 实例方法
let date4 = NSDate.init(timeInterval: 24*3600, sinceDate: date2) // 类方法
print("date3 == \(date3) ; date4 == \(date4)")                   
//"date3 == 2016-05-28 06:36:17 +0000 ; date4 == 2016-05-28 06:36:17 +0000"

//得到距现在多少秒后一个日期时间对象
let date5 = NSDate.init(timeIntervalSinceNow: 3*24*3600) //"May 30, 2016, 2:36 PM"
print(date5)                                             //"2016-05-30 06:36:17 +0000\n"

//未来
let date6 = NSDate.distantFuture() //"Jan 1, 4001, 8:00 AM"
//亘古
let date7 = NSDate.distantPast()   //"4001-01-01 00:00:00 +0000\n"
print("future == \(date6); past == \(date7)")
//"future == 4001-01-01 00:00:00 +0000; past == 0000-12-30 00:00:00 +0000"
```
***
# NSDateFormatter

## 初始化
```
let dateFormatter = NSDateFormatter()
```
有两种方式可以设置格式:一种是通过一些预定义的日期格式化样式（dateStyle）；另外一种是通过某些说明符来手动设置日期格式。stringFromDate 方法的使用也很重要，它是真正执行转换的代码。当谈到日期、字符串转换时，就指的是这个方法，而其他步骤只是起到定制结果的辅助作用。如果你在项目里要用到日期转换，这个方法会非常方便。
```
dateFormatter.dateStyle = NSDateFormatterStyle.FullStyle   //完整样式（FullStyle）
var convertedDate = dateFormatter.stringFromDate(date1)    //"Friday, May 27, 2016"

dateFormatter.dateStyle = NSDateFormatterStyle.LongStyle   //长样式（Long Style）
convertedDate = dateFormatter.stringFromDate(date1)        //"May 27, 2016"

dateFormatter.dateStyle = NSDateFormatterStyle.MediumStyle //中等样式(Medium Style)
convertedDate = dateFormatter.stringFromDate(date1)        //"May 27, 2016"

dateFormatter.dateStyle = NSDateFormatterStyle.ShortStyle  //短样式（Short Style）
convertedDate = dateFormatter.stringFromDate(date1)        //"5/27/16"
```
## 改变时区
```
dateFormatter.dateStyle = NSDateFormatterStyle.FullStyle
// 希腊
dateFormatter.locale = NSLocale(localeIdentifier: "el_GR")
convertedDate = dateFormatter.stringFromDate(date1)        //"Παρασκευή, 27 Μαΐου 2016"
// 法国
dateFormatter.locale = NSLocale(localeIdentifier: "fr_FR")
convertedDate = dateFormatter.stringFromDate(date1)        //"vendredi 27 mai 2016"
// 本地
dateFormatter.locale = NSLocale.currentLocale()
convertedDate = dateFormatter.stringFromDate(date1)        //"Friday, May 27, 2016"
```
## 自定义的日期格式

[设置自定义日期格式](http://unicode.org/reports/tr35/tr35-6.html#Date_Format_Patterns)在两种场景中很有用：          
1.当预定义的日期样式不能满足我们的需求；2.当我们需要把一个复杂的日期字符串（比如Thu, 08 Oct 2015 09:22:33 GMT）转换成日期对象。要想设置合适的日期格式（对象），必须搭配使用一系列说明符。说明符也是简单的字符，但是对于date formatter来说有特定的含义。
```
 EEEE：“星期”的全名（比如Monday）。如需缩写，指定1-3个字符（如E，EE，EEE代表Mon）。
 MMMM：“月份”的全名（比如October）。如需缩写，指定1-3个字符（如M，MM，MMM代表Oct）。
 dd：某月的第几天（例如，09或15）
 yyyy：四位字符串表示“年”（例如2015）
 HH：两位字符串表示“小时”（例如08或19）
 mm：两位字符串表示“分钟”（例如05或54）
 ss：两位字符串表示“秒”
 zzz：三位字符串表示“时区”（例如GMT）
 GGG：公元前BC或公元后AD
 ```
 
```
dateFormatter.dateFormat = "EEEE, MMMM dd, yyyy"
convertedDate = dateFormatter.stringFromDate(date1)        //"Friday, May 27, 2016"

dateFormatter.dateFormat = "HH:mm:ss"
convertedDate = dateFormatter.stringFromDate(date1)        //"14:36:17"

var dateAsString = "27-05-2016 23:59"
dateFormatter.dateFormat = "dd-MM-yyyy HH:mm"
var newDate = dateFormatter.dateFromString(dateAsString)   //"May 27, 2016, 11:59 PM"

// 包含时区信息的复杂字符串：
dateAsString = "Thu, 27 May 2016 09:22:33 GMT"
dateFormatter.dateFormat = "EEE, dd MMM yyyy HH:mm:ss zzz"
newDate = dateFormatter.dateFromString(dateAsString)       //"May 27, 2016, 5:22 PM"
```
***
# NSDateComponents

## NSDate到NSDateComponents

NSCalendar 的 components(_:fromDate:)，这个方法接受两个参数：第二个是日期对象；第一个参数比较有意思，它接收若干个 NSCalendarUnit 类型值，NSCAlendarUnit 用来说明需要的日期部分。NSCalendarUnit 是一个结构体，你可以在 这个[文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSCalendar_Class/#//apple_ref/swift/struct/c:@E@NSCalendarUnit) 中看到所有属性。这里需要注意：若某个组件没有在第一个参数中指定，就无法访问它。如：在这个例子中，我们没有指定 NSCalendarUnit.TimeZone，这样就无法访问时区的组件，比如print(dateComponents.timezone)调用会造成一个运行时错误。如果你需要额外的日期组件，只能重新调用一次calendar.Components方法，把你需要的Calendar Unit添加进去。

```
let calendar = NSCalendar.currentCalendar()
let dateComponents = calendar.components([NSCalendarUnit.Day, NSCalendarUnit.Month, NSCalendarUnit.Year, NSCalendarUnit.WeekOfYear, NSCalendarUnit.Hour, NSCalendarUnit.Minute, NSCalendarUnit.Second, NSCalendarUnit.Nanosecond], fromDate: date1)

print("day = \(dateComponents.day)", "month = \(dateComponents.month)", "year = \(dateComponents.year)", "week of year = \(dateComponents.weekOfYear)", "hour = \(dateComponents.hour)", "minute = \(dateComponents.minute)", "second = \(dateComponents.second)", "nanosecond = \(dateComponents.nanosecond)" , separator: ", ", terminator: "")

//"day = 27, month = 5, year = 2016, week of year = 22, hour = 14, minute = 36, second = 17, nanosecond = 490570008"

let year = dateComponents.year             //年
let mondth = dateComponents.month          //月
let day = dateComponents.day               //日
let weekOfYear = dateComponents.weekOfYear //第几周
let hour = dateComponents.hour             //时
let minute = dateComponents.minute         //分
let second = dateComponents.second         //秒
let nanosecond = dateComponents.nanosecond //毫微秒
```
## NSDateComponents到NSDate

### 初始化

这个过程中不需要使用calendar unit。只用初始化一个新的 NSDateComponents 对象，然后显式的设置你需要的组件的值，然后调用 NSCalendar 的 dateFromComponents 方法即可
```
let components = NSDateComponents()
components.day = 5
components.month = 01
components.year = 2016
components.hour = 19
components.minute = 30
newDate = calendar.dateFromComponents(components)
```
### 改动时区对转换日期对象的影响
```
GMT = Greenwich Mean Time（格林尼治标准时间）
CST = China Standard Time（中国标准时间）
CET = Central European Time(欧洲中部时间）
时区缩写的列表（http://www.timeanddate.com/time/zones/）
```
```
components.timeZone = NSTimeZone(abbreviation: "GMT")
newDate = calendar.dateFromComponents(components)     //"Jan 6, 2016, 3:30 AM"

components.timeZone = NSTimeZone(abbreviation: "CST")
newDate = calendar.dateFromComponents(components)     //"Jan 6, 2016, 9:30 AM"

components.timeZone = NSTimeZone(abbreviation: "CET")
newDate = calendar.dateFromComponents(components)     //"Jan 6, 2016, 2:30 AM"
```
***
# 比较日期和时间
earlierDate:, 它用于判断一个日期是否早于另外一个日期。对应的还有一个是 laterDate:
```
earlierDate:
如果 date1 早于 date2，该方法返回date1
如过 date2 早于 date1，该方法返回date2
如果 date1 和 date2 相同，返回date1

dateFormatter.dateFormat = "MMM dd, yyyy zzz"
dateAsString = "May 27, 2016 GMT"
var date8 = dateFormatter.dateFromString(dateAsString)! //"May 27, 2016, 8:00 AM"

var date10 = date1.earlierDate(date8)                   //"May 27, 2016, 8:00 AM"
date10 = date1.laterDate(date8)                         //"May 27, 2016, 3:12 PM"
```
NSDate 的 compare: 方法，它需要搭配使用 NSComparisonResult 枚举体。
```
if date1.compare(date2) == NSComparisonResult.OrderedDescending{
    print("Date1 is later than date2")
}else if date1.compare(date2) == NSComparisonResult.OrderedAscending{
    print("Date1 is Earlier than Date2") //"Date1 is Earlier than Date2\n"
}else if date1.compare(date2) == NSComparisonResult.OrderedSame {
    print("Same date")
}
```
时间间隔（time interval)，查找到每个日期（到现在）的时间间隔，进行比对。
```
if date1.timeIntervalSinceReferenceDate > date2.timeIntervalSinceReferenceDate {
    print("Date1 is Later than Date2")
}else if date1.timeIntervalSinceReferenceDate <  date2.timeIntervalSinceReferenceDate {
    print("Date1 is Earlier than Date2") //"Date1 is Earlier than Date2\n"
}else {
    print("Same dates")
}
```
在下面的方法里，会看到“2000-01-01”的日期，这是因为若 NSDate 对象没有指定日期，只指定时间的话，会自动添加默认的日期属性。
```
dateFormatter.dateFormat = "HH:mm:ss zzz"
dateAsString = "14:28:16 GMT"
let date11 = dateFormatter.dateFromString(dateAsString)!
dateAsString = "19:53:12 GMT"
let date12 = dateFormatter.dateFromString(dateAsString)!

if date1.earlierDate(date2) == date1 {
    if date1.isEqualToDate(date2) {
        print("Same time")
    }else {
        print("\(date1) is earlier than \(date2)")
        //2016-05-27 07:12:44 +0000 is earlier than 2016-05-27 07:12:44 +0000
    }
}else {
    print("\(date2) is earlier than \(date1)")
}
```
***
# 计算未来和过去的日期
两种不同的方法：第一种使用 NSCalendar 类和 NSCalendarUnit 结构体；第二种使用 NSDateComponents 类。

假定我们需要为这个日期往后推两个月又5天。
```
let monthsToAdd = 2
let daysToAdd = 5

```
这里用到的方法是 NSCAlendar 类的 dateByAddingUnit:value:toDate:options: 方法。它的作用是添加某个日历单元值（如年月日时分秒等）到现有的日期对象上，然后返回新的日期对象。我们需要添加两个日历单元到当前日期，直接用这个方法是不可能的（它每次只能设置一个calendar unit）。关键是调用两次这个方法，设置不同的日历单元，就能得到最终结果。
```
var calculatedDate = NSCalendar.currentCalendar().dateByAddingUnit(NSCalendarUnit.Month, value: monthsToAdd, toDate: date1, options: NSCalendarOptions.init(rawValue: 0))
calculatedDate = NSCalendar.currentCalendar().dateByAddingUnit(NSCalendarUnit.Day, value: daysToAdd, toDate: calculatedDate!, options: NSCalendarOptions.init(rawValue: 0))
```
当日历单元多的时候，你就需要多次调用这个方法。在日历单元比较多的时候，更好的方法是使用 NSDateComponents 类。初始化一个 NSDateComponents 对象，并设置月份和天的信息。然后我们调用 NSCalendar 的另一个方法dateByAddingComponents:toDate:options:，并最终获得我们需要的日期对象。
```
let newDateComponents = NSDateComponents()
newDateComponents.month = monthsToAdd
newDateComponents.day = daysToAdd

calculatedDate = NSCalendar.currentCalendar().dateByAddingComponents(newDateComponents, toDate: date1, options: NSCalendarOptions.init(rawValue: 0))

```
// 注意：在以上调用 NSCalendar 方法的地方，最后一个参数options都没有被设置。如果你需要具体设置options的值，请参考完整的 [官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSCalendar_Class/)。

***
# 计算日期间隔
通过date components来计算日期对象间隔。这个新方法叫 components:fromDate:toDate:options:,第一个参数是 NSCalendarUnit 值的数组。这里要注意，如果第一个日期如果晚于第二个日期，则结果会返回负值。
```
var diffFateComponents = NSCalendar.currentCalendar().components([NSCalendarUnit.Year, NSCalendarUnit.Month, NSCalendarUnit.Day, NSCalendarUnit.Hour, NSCalendarUnit.Minute, NSCalendarUnit.Second], fromDate: date1, toDate: date14, options: NSCalendarOptions.init(rawValue: 0))

print("The difference between dates is: \(diffFateComponents.year) years, \(diffFateComponents.month) months, \(diffFateComponents.day) days, \(diffFateComponents.hour) hours, \(diffFateComponents.minute) minutes, \(diffFateComponents.second) seconds")
//"The difference between dates is: 1 years, 9 months, 5 days, 16 hours, 50 minutes, 42 seconds"

let diffYear = diffFateComponents.year
let diffMonth = diffFateComponents.month
let diffDay = diffFateComponents.day
let diffHour = diffFateComponents.hour
let diffMinute = diffFateComponents.minute
let diffSecond = diffFateComponents.second
```
NSDateComponentsFormatter 类，它提供了多种用于自动计算日期间隔的方法，并可以返回格式化字符串结果。unitsStyle 属性指定我们使用的 dateComponentsFormatter 以何种格式打印日期的间隔。这里我们使用 完整 样式。
```
let dateComponentsFormatter = NSDateComponentsFormatter()
dateComponentsFormatter.unitsStyle = NSDateComponentsFormatterUnitsStyle.Full

let interval2 = date14.timeIntervalSinceDate(date1)
dateComponentsFormatter.stringFromTimeInterval(interval2)
//"1 years, 9 months, 5 days, 16 hours, 50 minutes, 42 seconds"
```
最后，在第三种计算的方法中，我们将两个日期传递给 NSDateComponentsFormatter 对象的一个叫 stringFromDate:toDate: 的方法。但是这个方法需要有个前置的条件：NSDateComponentsFormatter 的 allowedUnits 属性必须被提前设置，这个属性接受数组类型的值，这里至少要设置一个日历单元的值。否则这个方法会返回nil值。所以，在这个方法的使用中，我们“告诉”它需要获取哪些日历单元，它会按照对应的日历单元返回结果：
```
dateComponentsFormatter.allowedUnits = [NSCalendarUnit.Year, NSCalendarUnit.Month, NSCalendarUnit.Day, NSCalendarUnit.Hour, NSCalendarUnit.Minute, NSCalendarUnit.Second]
let autoFormattedDifference = dateComponentsFormatter.stringFromDate(date1, toDate: date14)
//"1 years, 9 months, 5 days, 16 hours, 50 minutes, 42 seconds"
```
