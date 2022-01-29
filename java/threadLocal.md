## SimpleDateFormat线程不安全及解决办法

### 1、为什么SimpleDateFormat不安全？

代码演示：

```java
//日期工具包
public class DateUtil {
    private static final SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static String formatDate(Date date) {
        return sdf.format(date);
    }

    public static Date parse(String strDate) throws ParseException {
        return sdf.parse(strDate);
    }
}
```

测试format方法（将Date对象转换为String对象）：

```java
public class Main{
	public static void main(String[] args) {
        ExecutorService executorService  = Executors.newFixedThreadPool(100);

        for (; ; ) {
            executorService.execute(() -> {
                try {
                    String str = DateUtil.formatDate(new Date());
                    System.out.println(str);
                } catch (Exception e) {
                    e.printStackTrace();
                    System.exit(1);
                }
            });
        }
    }
}
```

测试parse方法（将Date对象转换为String对象）

```java
public class Main{
	public static void main(String[] args) {
        ExecutorService executorService  = Executors.newFixedThreadPool(100);

        for (; ; ) {
            executorService.execute(() -> {
                try {
                    System.out.println(DateUtil.parse("2019-06-15 16:35:20"));
                } catch (Exception e) {
                    e.printStackTrace();
                    System.exit(1);
                }
            });
        }
    }
}
```

两种方法会出现各种报错，有时还有可能结果不正确。

### 2、原理分析

通过查看源码发现，原来SimpleDateFormat类内部有一个Calendar对象引用，这个类的概念比较复杂，牵扯到时区与本地化的问题等等，所以JDK使用了成员变量的方式来传递参数，Calendar用来储存和这个SimpleDateFormat相关的日期信息，例如sdf.parse(dateStr)，sdf.format(date) 诸如此类的方法参数传入的日期相关String，Date等等, 都是交由Calendar引用来储存的。

这样就会导致一个问题，如果你的SimpleDateFormat是个static的, 那么多个线程之间就会共享这个SimpleDateFormat, 同时也是共享这个Calendar引用。单例、多线程、又有成员变量（这个变量在方法中是可以修改的），这个场景很像servlet，在高并发的情况下，容易出现幻读成员变量的现象，故说SimpleDateFormat是线程不安全的对象。

servlet因是线程不安全的，所以我们使用servlet的原则是不设置成员变量。

#### parse方法：

![img](threadLocal.assets/20170610144713565)

调用SimpleDataFormat的parse()方法会先调用Calendar.clear（），然后调用Calendar.add()，如果一个线程先调用了add()然后另一个线程又调用了clear()，这时候parse()方法解析的时间就不对了。

#### format方法

![img](threadLocal.assets/20170610144740003)

如果有两个线程，线程1调用format方法，改变了calendar这个字段。

这时，线程1发生中断，线程2开始执行，它也改变了calendar。

这时，线程2也发生了中断，线程1重新执行，此时，calendar已然不是它所设的值。如果多个线程同时争抢calendar对象，则会出现各种问题，时间不对，线程挂死等等。

### 3、解决方法

#### 方法一

将SimpleDateFormat定义成局部变量。

缺点：调用一次方法就会创建一个SimpleDateFormat对象，方法结束又要作为垃圾回收。

#### 方法二：

方法加同步锁synchronized，在同一时刻，只有一个线程可以执行类中的某个方法。

缺点：性能较差，每次都要等待锁释放后其他线程才可以执行。

#### 方法三：

使用第三方库joda-time，由第三方考虑线程不安全的问题。

#### 方法四：

使用ThreadLocal本地变量来解决：

```java
class DateUtil2 {
    private static final String DATE_FORMAT = "yyyy-MM-dd HH:mm:ss";
    private static ThreadLocal<DateFormat> threadLocal = new ThreadLocal<>();

    public static DateFormat getDateForm() {
        DateFormat df = threadLocal.get();
        if (df == null) {
            df = new SimpleDateFormat(DATE_FORMAT);
            threadLocal.set(df);
        }
        return df;
    }

    public static String formatDate(Date date) {
        return getDateForm().format(date);
    }

    public static Date parse(String strDate) throws Exception {
        return getDateForm().parse(strDate);
    }

}
```

