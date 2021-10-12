* 一个java文件只能有一个public class，public class的名字必须和文件名字一样；



* 每个子类的构造函数的第一句话，都默认调用父类的无参构造函数super()，super语句必须放在第一条；



* 派生类继承抽象类，若没有实现基类中的所有abstract方法，那么派生类也必须被定义为抽象类；实现接口，就必须实现所有未实现的方法，如果没有全部实现，那么只能成为一个抽象类；接口没有构造函数和main函数，抽象类可以有main函数且能运行；



* 静态方法需要调用才会执行，执行顺序：static块>匿名块>构造函数；



* 常量池：相同的值只存取一份，节省内存，共享访问；


> 1. 常量与常量的拼接结果在常量池。且常量池中不会存在相同内容的常量
>
> 2. 只要其中有一个是变量，结果就在堆中，如果变量是final类型的final就是常量，结果拼加后在常量池
>
> 3. 如果拼接的结果调用intern()方法，返回值就在常量池中，str1=str2.intern()
>
> 4. String:代表不可变的字符序列。简称：不可变性。

![](1.png)

![](2.png)

![](3.png)

![](4.png)

![](5.png)



* 不可变对象一旦创建，这个对象(状态/值)不能被更改了，如八个基本型别的包装类的对象，String,BigInteger和BigDecimal等的对象；

> 1. 当对字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值。
>
> 2. 当对现有的字符串进行连接操作时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。
>
> 3. 当调用String的replace()方法修改指定字符或字符串时，也需要重新指定内存区域赋值，不能使用原有的value进行赋值。

![](6.png)



##### 访问规则 #####

![](7.png)



##### 异常规则 #####

![](8.png)

![](9.png)

![](10.png)



* 使用try-catch-finally处理编译时异常，是得程序在编译时就不再报错，但是运行时仍可能报错，相当于我们使用try-catch-finally将一个编译时可能出现的异常，延迟到运行时出现；也即是说try必须有，catch和finally至少要有一个，finally必执行，catch块的异常匹配是从上而下执行的，并且只能进入一个catch块；finally中声明的是一定会被执行的代码，即使catch中又出现异常了；针对于编译时异常，我们说一定要考虑异常的处理。

> 异常打印写法：
>
> e.printStackTrace();
>
> System.out.println(e.getMessage());

![](11.png)

![](12.png)



* 在方法内部程序中，用throw抛出异常；在方法头部声明中，用throws。

````java
public class ReturnExceptionDemo {
	static void methodA() {
		try {
			System.out.println("进入方法A");
			throw new RuntimeException("制造异常");
		} finally {
			System.out.println("用A方法的finally");
		}
	}

	static void methodB() {
		try {
			System.out.println("进入方法B");
			return;
		} finally {
			System.out.println("调用B方法的finally");
		}
	}

	public static void main(String[] args) {
		try {
			methodA();
		} catch (Exception e) {
			System.out.println(e.getMessage());
		}
		
		methodB();
	}
}
输出：
进入方法A
用A方法的finally
制造异常
进入方法B
调用B方法的finally
````

* 可以抛出的异常必须是 Throwable 或其子类的实例，此语句在编译时将会产生语法错误：throw new String("want to throw")



![](13.png)

> 如何自定义异常类？
>
> 1. 继承于现有的异常结构：RuntimeException 、Exception
> 2. 提供全局常量：serialVersionUID
> 3. 提供重载的构造器

```java
public class MyException extends Exception{
	
	static final long serialVersionUID = -7034897193246939L;
	
	public MyException(){
		
	}
	
	public MyException(String msg){
		super(msg);
	}
}
```



##### 数据结构

* Collection接口：单列集合，用来存储一个一个的对象
  * List接口：存储有序的、可重复的数据。 -->“动态”数组
    * ArrayList：作为List接口的主要实现类；线程不安全的，效率高；底层使用Object[] elementData存储
    * LinkedList：对于频繁的插入、删除操作，使用此类效率比ArrayList高；底层使用双向链表存储
    * Vector：作为List接口的古老实现类；线程安全的，效率低；底层使用Object[] elementData存储
  * Set接口：存储无序的、不可重复的数据  -->高中讲的“集合”
    * HashSet：作为Set接口的主要实现类；线程不安全的；可以存储null值
    * LinkedHashSet：作为HashSet的子类；遍历其内部数据时，可以按照添加的顺序遍历，对于频繁的遍历操作，LinkedHashSet效率高于HashSet.
    * TreeSet：可以按照添加对象的指定属性，进行排序。

* Map接口：双列集合，用来存储一对(key - value)一对的数据
  * HashMap:作为Map的主要实现类；线程不安全的，效率高；存储null的key和value
    * LinkedHashMap:保证在遍历map元素时，可以按照添加的顺序实现遍历。原因：在原有的HashMap底层结构基础上，添加了一对指针，指向前一个和后一个元素。对于频繁的遍历操作，此类执行效率高于HashMap。
  * TreeMap:保证按照添加的key-value对进行排序，实现排序遍历。此时考虑key的自然排序或定制排序。底层使用红黑树
  * Hashtable:作为古老的实现类；线程安全的，效率低；不能存储null的key和value
  * Properties:常用来处理配置文件。key和value都是String类型



###### Collection

* 向Collection接口的实现类的对象中添加数据obj时，要求obj所在类要重写equals()

+ 迭代器Iterator：在调用it.next()方法之前必须要调用it.hasNext()进行检测。若不调用，且下一条记录无效，直接调用it.next()会抛出NoSuchElementException异常

* Iterator可以删除集合的元素，但是是遍历过程中通过迭代器对象的remove方法，不是集合对象的remove方法。 

* 如果还未调用next()或在上一次调用 next 方法之后已经调用了remove方法，再调用remove都会报IllegalStateException

* 使用 foreach 循环遍历集合元素，遍历集合的底层调用Iterator完成操作。



###### List

* 建议开发中使用带参的构造器：ArrayList list = new ArrayList(int capacity)

* jdk7中的ArrayList的对象的创建类似于单例的饿汉式，开始就创建了数组的长度，而jdk8中的ArrayList的对象的创建类似于单例的懒汉式，延迟了数组的创建，节省内存。



###### Set

* 向Set(主要指：HashSet、LinkedHashSet)中添加的数据，其所在的类一定要重写hashCode()和equals()；重写的hashCode()和equals()尽可能保持一致性：相等的对象必须具有相等的散列码

* 重写两个方法的小技巧：对象中用作 equals() 方法比较的 Field，都应该用来计算 hashCode 值。

* 向TreeSet中添加的数据，要求是相同类的对象，不可以容纳null元素



###### Map

* HashMap的底层：数组+链表 （jdk7及之前），数组+链表+红黑树 （jdk 8）

* Map中的key:无序的、不可重复的，使用Set存储所有的key ---> key所在的类要重写equals()和hashCode() （以HashMap为例）；Map中的value:无序的、可重复的，使用Collection存储所有的value --->value所在的类要重写equals()

* 存储过程，调用key1所在类的hashCode()计算key1哈希值，此哈希值经过某种算法计算以后，得到在Entry数组中的存放位置。
  *  如果此位置上的数据为空，此时的key1-value1添加成功
  * 如果此位置上的数据不为空，(意味着此位置上存在一个或多个数据(以链表形式存在)),比较key1和已经存在的一个或多个数据的哈希值
    * 如果key1的哈希值与已经存在的数据的哈希值都不相同，此时key1-value1添加成功
    * 如果key1的哈希值和已经存在的某一个数据(key2-value2)的哈希值相同，继续比较：调用key1所在类的equals(key2)方法比较
      * 如果equals()返回false:此时key1-value1添加成功
      * 如果equals()返回true:使用value1替换value2



##### 工具类

###### Object

* Object内置重要类9个（getClass、hashCode、equals、toString、wait、notify、notifyAll、finalize、clone）

* 另有一个registerNatives()方法使用了native特征修饰符修饰，我们看不到该方法的具体实现，并且也没有注释，native是与C++联合开发的时候用的！使用native关键字说明这个方法是原生函数，也就是这个方法是用C/C++语言实现的，并且被编译成了DLL，由java去调用。

```java
private static native void registerNatives();
    static {
        registerNatives();
    }
```

* getClass()方法返回当前运行时的类Class，即当前调用getClass()方法的对象对应的Class（字节码）。这个方法一般在反射场景中使用。

 

* hashCode()方法返回值是当前调用hashCode()方法的对象其内存地址所对应的的哈希值（hash value），并且这个方法在那些底层通过哈希表（hash table）实现的集合有很重要的意义，这个方法一般搭配equals()方法使用。

 

* toString()方法用于返回对象的字符串表示形式，Object类中的toString()方法返回的字符串形式为类全名@内存地址，建议所有的子类都重写toString()方法，因为toString()方法的初衷就是返回一个字符串，并且我们能够很容易的通过这个字符串来区别同一个类下的对象。

 

* equals()方法用于判断两个对象是否相等，Object类中的equals方法默认判断两个对象的地址引用是否相等。在实际开发中，一般需要根据业务逻辑重写equals方法，根据对象的属性值来判断两个对象在业务逻辑上是否相等。

 

* notify()方法使用了native特征修饰符修饰，表示该方法使用了其他语言来实现，所以我们看不到notify()方法的具体实现，但是通过notify()方法的注释，我们可以得知notify方法用于唤醒因线程同步进入阻塞状态的任意一个线程。

 

* notifyAll()方法和notify()方法基本差不多，不同的是notifyAll()方法不仅仅只唤醒一个因线程同步进入阻塞状态的线程，而是唤醒所有。

* 注意：notify()方法和notifyAll()方法需要在同步代码块或者同步方法中使用，因为这两个方法就是用来唤醒因线程同步锁进入阻塞状态的线程，如果不在线程同步环境下，那么notify()方法和notifyAll()方法需要唤醒的阻塞线程是哪一个呢？说白了就是不明确需要唤醒因哪一个线程同步锁进入阻塞状态的线程。而且如果不在同步代码块或者同步方法内使用这两个方法，编译会报错。

 

* wait(long)方法是native修饰的，即具体实现过程由其他语言（如C、C++）来实现。虽然我们不能通过源码来了解wait(long)方法的作用，但是我们可以通过注释来了解wait(long)方法的功能或者作用，从注释上可以知道，wait()方法的执行可以让当前正在执行的线程进入阻塞状态，直到其他线程调用notify()或者notifyAll()方法来唤醒这个线程，或者当过了timeout毫秒之后，自动从阻塞状态进入就绪状态。注意：wait()方法是同步监视器调用的，而不是线程本身调用的，同时也要弄清楚线程和同步监视器之间的关系。

* wait()方法从源码上可以得知，其实wait()方法本质上就是wait(0)，所以我们可以参考wait(long)方法来理解wait()方法的作用，因为wait()方法没有参数，即没有一个timeout值，所以当同步监视器调用了wait()方法后，进入阻塞的线程不存在过了timeout毫秒后自动进入就绪列队（状态）中，所以因为同步监视器调用wait()进入阻塞状态的线程只有当其他线程调用notify()或者notifyAll()方法来唤醒其，使其进入就绪状态。

* wait(long, int)方法其实是对线程的等待时间timeout进行了更加精细的控制。



* finalize() : void：当垃圾回收机制（Garbage Collection）确认了一个对象没有被引用时，那么垃圾回收机制就会回收这个对象，并在回收的前一刻这个对象调用finalize()方法，这类似C++中的析构函数，我们可以重写对象的finalize()方法来研究垃圾回收机制回收对象的过程。

 

* clone()方法用于克隆当前对象，克隆分为浅克隆和深克隆。Object类中的clone()方法是浅克隆对象，所以一般需要实现Cloneable接口，并重写clone方法，当复制的属性是引用数据类型时，进行递归克隆（基本数据类型直接复制一份，引用数据类型递归调用clone方法），重写完clone方法才实现真正意义上的克隆。

###### String

* String内部定义了final char[] value用于存储字符串

* String实现了Serializable接口：表示字符串是支持序列化的；也实现了Comparable接口：表示String可以比较大小。

* String、StringBuffer、StringBuilder三者的异同？
  * String:不可变的字符序列；底层使用char[]存储
  * StringBuffer:可变的字符序列；线程安全的，效率低；底层使用char[]存储
  * StringBuilder:可变的字符序列；jdk5.0新增的，线程不安全的，效率高；底层使用char[]存储

* 对比String、StringBuffer、StringBuilder三者的效率，从高到低排列：StringBuilder > StringBuffer > String

###### 日期类

* JDK 8之前日期和时间：java.util.Date类，java.sql.Date类

```java
Date date1 = new Date();
        System.out.println(date1.toString());//Sat Feb 16 16:35:31 GMT+08:00 2019

        System.out.println(date1.getTime());//1550306204104

        //构造器二：创建指定毫秒数的Date对象
        Date date2 = new Date(155030620410L);
        System.out.println(date2.toString());

        //创建java.sql.Date对象
        java.sql.Date date3 = new java.sql.Date(35235325345L);
        System.out.println(date3);//1971-02-13

        long time = System.currentTimeMillis();
        //返回当前时间与1970年1月1日0时0分0秒之间以毫秒为单位的时间差。
        //称为时间戳




        //实例化SimpleDateFormat:使用默认的构造器
        SimpleDateFormat sdf = new SimpleDateFormat();

        //格式化：日期 --->字符串
        Date date = new Date();
        System.out.println(date);

        String format = sdf.format(date);
        System.out.println(format);

        //解析：格式化的逆过程，字符串 ---> 日期
        String str = "2019/12/18 上午11:43";
        Date date1;
        date1 = sdf.parse(str);
        System.out.println(date1);

        //*************按照指定的方式格式化和解析：调用带参的构造器*****************
        SimpleDateFormat sdf1 = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
        //格式化
        String format1 = sdf1.format(date);
        System.out.println(format1);//2019-02-18 11:48:27
        //解析:要求字符串必须是符合SimpleDateFormat识别的格式(通过构造器参数体现),
        //否则，抛异常
        Date date2 = sdf1.parse("2020-02-18 11:48:27");
        System.out.println(date2);

输出为
Sat Jun 27 20:45:19 CST 2020
2020/6/27 下午8:45
Wed Dec 18 11:43:00 CST 2019
2020-06-27 08:45:19
Tue Feb 18 11:48:27 CST 2020
```

* JDK8：LocalDate、LocalTime、LocalDateTime 的使用

```java
//now():获取当前的日期、时间、日期+时间
        LocalDate localDate = LocalDate.now();
        LocalTime localTime = LocalTime.now();
        LocalDateTime localDateTime = LocalDateTime.now();

        System.out.println(localDate);
        System.out.println(localTime);
        System.out.println(localDateTime);

        //of():设置指定的年、月、日、时、分、秒。没有偏移量
        LocalDateTime localDateTime1 = LocalDateTime.of(2020, 10, 6, 13, 23, 43);
        System.out.println(localDateTime1);


        //getXxx()：获取相关的属性
        System.out.println(localDateTime.getDayOfMonth());
        System.out.println(localDateTime.getDayOfWeek());
        System.out.println(localDateTime.getMonth());
        System.out.println(localDateTime.getMonthValue());
        System.out.println(localDateTime.getMinute());

        //体现不可变性
        //withXxx():设置相关的属性
        LocalDate localDate1 = localDate.withDayOfMonth(22);
        System.out.println(localDate);
        System.out.println(localDate1);


        LocalDateTime localDateTime2 = localDateTime.withHour(4);
        System.out.println(localDateTime);
        System.out.println(localDateTime2);


        LocalDateTime localDateTime3 = localDateTime.plusMonths(3);
        System.out.println(localDateTime);
        System.out.println(localDateTime3);

        LocalDateTime localDateTime4 = localDateTime.minusDays(6);
        System.out.println(localDateTime);
        System.out.println(localDateTime4);

输出
2020-06-27
20:49:28.060056500
2020-06-27T20:49:28.060056500
2020-10-06T13:23:43
27
SATURDAY
JUNE
6
49
2020-06-27
2020-06-22
2020-06-27T20:49:28.060056500
2020-06-27T04:49:28.060056500
2020-06-27T20:49:28.060056500
2020-09-27T20:49:28.060056500
2020-06-27T20:49:28.060056500
2020-06-21T20:49:28.060056500

     //now():获取本初子午线对应的标准时间
        Instant instant = Instant.now();
        System.out.println(instant);//2019-02-18T07:29:41.719Z

        //添加时间的偏移量
        OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
        System.out.println(offsetDateTime);//2019-02-18T15:32:50.611+08:00

        //toEpochMilli():获取自1970年1月1日0时0分0秒（UTC）开始的毫秒数  ---> Date类的getTime()
        long milli = instant.toEpochMilli();
        System.out.println(milli);

        //ofEpochMilli():通过给定的毫秒数，获取Instant实例  -->Date(long millis)
        Instant instant1 = Instant.ofEpochMilli(1550475314878L);
        System.out.println(instant1);


//        方式一：预定义的标准格式。如：ISO_LOCAL_DATE_TIME;ISO_LOCAL_DATE;ISO_LOCAL_TIME
        DateTimeFormatter formatter = DateTimeFormatter.ISO_LOCAL_DATE_TIME;
        //格式化:日期-->字符串
        LocalDateTime localDateTime = LocalDateTime.now();
        String str1 = formatter.format(localDateTime);
        System.out.println(localDateTime);
        System.out.println(str1);//2019-02-18T15:42:18.797

        //解析：字符串 -->日期
        TemporalAccessor parse = formatter.parse("2019-02-18T15:42:18.797");
        System.out.println(parse);

//        方式二：
//        本地化相关的格式。如：ofLocalizedDateTime()
//        FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT :适用于LocalDateTime
        DateTimeFormatter formatter1 = DateTimeFormatter.ofLocalizedDateTime(FormatStyle.LONG);
        //格式化
        String str2 = formatter1.format(localDateTime);
        System.out.println(str2);//2019年2月18日 下午03时47分16秒


//      本地化相关的格式。如：ofLocalizedDate()
//      FormatStyle.FULL / FormatStyle.LONG / FormatStyle.MEDIUM / FormatStyle.SHORT : 适用于LocalDate
        DateTimeFormatter formatter2 = DateTimeFormatter.ofLocalizedDate(FormatStyle.MEDIUM);
        //格式化
        String str3 = formatter2.format(LocalDate.now());
        System.out.println(str3);//2019-2-18


//       重点： 方式三：自定义的格式。如：ofPattern(“yyyy-MM-dd hh:mm:ss”)
        DateTimeFormatter formatter3 = DateTimeFormatter.ofPattern("yyyy-MM-dd hh:mm:ss");
        //格式化
        String str4 = formatter3.format(LocalDateTime.now());
        System.out.println(str4);//2019-02-18 03:52:09

        //解析
        TemporalAccessor accessor = formatter3.parse("2019-02-18 03:52:09");
        System.out.println(accessor);{MinuteOfHour=52, MilliOfSecond=0, HourOfAmPm=3, MicroOfSecond=0, NanoOfSecond=0, SecondOfMinute=9},ISO resolved to 2019-02-18
```

###### 比较类

* 如何实现比较对象的大小？使用两个接口中的任何一个：Comparable 或 Comparator
* 实现 Comparable 的类必须实现 compareTo(Object obj) 方法，两个对象即通过 compareTo(Object obj) 方法的返回值来比较大小；实现Comparable接口的对象列表（和数组）可以通过 Collections.sort 或Arrays.sort进行自动排序。

* 使用 Comparator 的对象来排序，强行对多个对象进行整体排序的比较。重写compare(Object o1,Object o2)方法，比较o1和o2的大小

```java
 Arrays.sort(arr,new Comparator(){

            //按照字符串从大到小的顺序排列
            @Override
            public int compare(Object o1, Object o2) {
                if(o1 instanceof String && o2 instanceof  String){
                    String s1 = (String) o1;
                    String s2 = (String) o2;
                    return -s1.compareTo(s2);
                }
//                return 0;
                throw new RuntimeException("输入的数据类型不一致");
            }
        });
```

###### 枚举类

* 使用enum关键字定义的枚举类实现接口的情况
  * 情况一：实现接口，在enum类中实现抽象方法
  * 情况二：让枚举类的对象分别实现接口中的抽象方法

```java
import java.text.ParseException;

public class Test {
    public static void main(String[] args) throws ParseException {
        Season1 summer = Season1.SUMMER;
        //toString():返回枚举类对象的名称
        System.out.println(summer.toString());

//        System.out.println(Season1.class.getSuperclass());
        System.out.println("****************");
        //values():返回所有的枚举类对象构成的数组
        Season1[] values = Season1.values();
        for(int i = 0;i < values.length;i++){
            System.out.println(values[i]);
            values[i].show();
        }
        System.out.println("****************");
        Thread.State[] values1 = Thread.State.values();
        for (int i = 0; i < values1.length; i++) {
            System.out.println(values1[i]);
        }

        //valueOf(String objName):返回枚举类中对象名是objName的对象。
        Season1 winter = Season1.valueOf("WINTER");
        //如果没有objName的枚举类对象，则抛异常：IllegalArgumentException
//        Season1 winter = Season1.valueOf("WINTER1");
        System.out.println(winter);
        winter.show();
    }
}

interface Info{
    void show();
}

//使用enum关键字枚举类
enum Season1 implements Info{
    //1.提供当前枚举类的对象，多个对象之间用","隔开，末尾对象";"结束
    SPRING("春天","春暖花开"){
        @Override
        public void show() {
            System.out.println("春天在哪里？");
        }
    },
    SUMMER("夏天","夏日炎炎"){
        @Override
        public void show() {
            System.out.println("宁夏");
        }
    },
    AUTUMN("秋天","秋高气爽"){
        @Override
        public void show() {
            System.out.println("秋天不回来");
        }
    },
    WINTER("冬天","冰天雪地"){
        @Override
        public void show() {
            System.out.println("大约在冬季");
        }
    };

    //2.声明Season对象的属性:private final修饰
    private final String seasonName;
    private final String seasonDesc;

    //2.私有化类的构造器,并给对象属性赋值

    private Season1(String seasonName,String seasonDesc){
        this.seasonName = seasonName;
        this.seasonDesc = seasonDesc;
    }

    //4.其他诉求1：获取枚举类对象的属性
    public String getSeasonName() {
        return seasonName;
    }

    public String getSeasonDesc() {
        return seasonDesc;
    }
//    //4.其他诉求1：提供toString()
//
//    @Override
//    public String toString() {
//        return "Season1{" +
//                "seasonName='" + seasonName + '\'' +
//                ", seasonDesc='" + seasonDesc + '\'' +
//                '}';
//    }


//    @Override
//    public void show() {
//        System.out.println("这是一个季节");
//    }
}
输出
SUMMER
****************
SPRING
春天在哪里？
SUMMER
宁夏
AUTUMN
秋天不回来
WINTER
大约在冬季
****************
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
WINTER
大约在冬季
```

###### 其他类

* System，Math，BigInteger 和 BigDecimal

```java
String javaVersion = System.getProperty("java.version");
        System.out.println("java的version:" + javaVersion);

        String javaHome = System.getProperty("java.home");
        System.out.println("java的home:" + javaHome);

        String osName = System.getProperty("os.name");
        System.out.println("os的name:" + osName);

        String osVersion = System.getProperty("os.version");
        System.out.println("os的version:" + osVersion);

        String userName = System.getProperty("user.name");
        System.out.println("user的name:" + userName);

        String userHome = System.getProperty("user.home");
        System.out.println("user的home:" + userHome);

        String userDir = System.getProperty("user.dir");
        System.out.println("user的dir:" + userDir);

        BigInteger bi = new BigInteger("1243324112234324324325235245346567657653");
        BigDecimal bd = new BigDecimal("12435.351");
        BigDecimal bd2 = new BigDecimal("11");
        System.out.println(bi);
//         System.out.println(bd.divide(bd2));
        System.out.println(bd.divide(bd2, BigDecimal.ROUND_HALF_UP));
        System.out.println(bd.divide(bd2, 25, BigDecimal.ROUND_HALF_UP));
输出
1243324112234324324325235245346567657653
1130.486
1130.4864545454545454545454545
```

##### 文件和I/O

* Java 程序支持跨平台运行，因此路径分隔符要慎用。为了解决这个隐患， File 类提供了一个常量：public static final String separator根据操作系统，动态的提供分隔符。

![](14.png)

![](15.png)

* 流的体系结构

> 抽象基类					节点流（或文件流）											     缓冲流（处理流的一种）
>
> InputStream	FileInputStream(read(byte[] buffer))	BufferedInputStream (read(byte[] buffer))
>
> OutputStream	FileOutputStream(write(byte[] buffer,0,len)	BufferedOutputStream (write(byte[] buffer,0,len) / flush()
>
> Reader	FileReader (read(char[] cbuf))	BufferedReader (read(char[] cbuf) / readLine())
>
> Writer	FileWriter (write(char[] cbuf,0,len)	BufferedWriter (write(char[] cbuf,0,len) / flush()

![](17.png)

* 解码：字节、字节数组 --->字符数组、字符串

* 编码：字符数组、字符串 ---> 字节、字节数组

```java
@Test
    public void test2() throws Exception {
        //1.造文件、造流
        File file1 = new File("dbcp.txt");
        File file2 = new File("dbcp_gbk.txt");

        FileInputStream fis = new FileInputStream(file1);
        FileOutputStream fos = new FileOutputStream(file2);

        InputStreamReader isr = new InputStreamReader(fis,"utf-8");
        OutputStreamWriter osw = new OutputStreamWriter(fos,"gbk");

        //2.读写过程
        char[] cbuf = new char[20];
        int len;
        while((len = isr.read(cbuf)) != -1){
            osw.write(cbuf,0,len);
        }

        //3.关闭资源
        isr.close();
        osw.close();


    }


}
```

* 缓冲流作用：提供流的读取、写入的速度

```java
 //实现文件复制的方法
    public void copyFileWithBuffered(String srcPath,String destPath){
        BufferedInputStream bis = null;
        BufferedOutputStream bos = null;

        try {
            //1.造文件
            File srcFile = new File(srcPath);
            File destFile = new File(destPath);
            //2.造流
            //2.1 造节点流
            FileInputStream fis = new FileInputStream((srcFile));
            FileOutputStream fos = new FileOutputStream(destFile);
            //2.2 造缓冲流
            bis = new BufferedInputStream(fis);
            bos = new BufferedOutputStream(fos);

            //3.复制的细节：读取、写入
            byte[] buffer = new byte[1024];
            int len;
            while((len = bis.read(buffer)) != -1){
                bos.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //4.资源关闭
            //要求：先关闭外层的流，再关闭内层的流
            if(bos != null){
                try {
                    bos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if(bis != null){
                try {
                    bis.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            //说明：关闭外层流的同时，内层流也会自动的进行关闭。关于内层流的关闭，我们可以省略.
//        fos.close();
//        fis.close();
        }
    }

    @Test
    public void testCopyFileWithBuffered(){
        long start = System.currentTimeMillis();

        String srcPath = "C:\\Users\\Administrator\\Desktop\\01-视频.avi";
        String destPath = "C:\\Users\\Administrator\\Desktop\\03-视频.avi";


        copyFileWithBuffered(srcPath,destPath);


        long end = System.currentTimeMillis();

        System.out.println("复制操作花费的时间为：" + (end - start));//618 - 176
    }


    /*
    使用BufferedReader和BufferedWriter实现文本文件的复制

     */
    @Test
    public void testBufferedReaderBufferedWriter(){
        BufferedReader br = null;
        BufferedWriter bw = null;
        try {
            //创建文件和相应的流
            br = new BufferedReader(new FileReader(new File("dbcp.txt")));
            bw = new BufferedWriter(new FileWriter(new File("dbcp1.txt")));

            //读写操作
            //方式一：使用char[]数组
//            char[] cbuf = new char[1024];
//            int len;
//            while((len = br.read(cbuf)) != -1){
//                bw.write(cbuf,0,len);
//    //            bw.flush();
//            }

            //方式二：使用String
            String data;
            while((data = br.readLine()) != null){
                //方法一：
//                bw.write(data + "\n");//data中不包含换行符
                //方法二：
                bw.write(data);//data中不包含换行符
                bw.newLine();//提供换行的操作

            }


        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            if(bw != null){

                try {
                    bw.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if(br != null){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }

    }

}
```

* 标准的输入、输出流(System.in,System.out,System类的setIn(InputStream is) / setOut(PrintStream ps)方式重新指定输入和输出的流)

```java
从键盘输入字符串，要求将读取到的整行字符串转成大写输出。然后继续进行输入操作，直至当输入“e”或者“exit”时，退出程序。

    方法一：使用Scanner实现，调用next()返回一个字符串
    方法二：使用System.in实现。System.in  --->  转换流 ---> BufferedReader的readLine()
BufferedReader br = null;
        try {
            InputStreamReader isr = new InputStreamReader(System.in);
            br = new BufferedReader(isr);

            while (true) {
                System.out.println("请输入字符串：");
                String data = br.readLine();
                if ("e".equalsIgnoreCase(data) || "exit".equalsIgnoreCase(data)) {
                    System.out.println("程序结束");
                    break;
                }

                String upperCase = data.toUpperCase();
                System.out.println(upperCase);

            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
```

* 打印流(PrintStream,PrintWriter)

```java
@Test
    public void test2() {
        PrintStream ps = null;
        try {
            FileOutputStream fos = new FileOutputStream(new File("D:\\IO\\text.txt"));
            // 创建打印输出流,设置为自动刷新模式(写入换行符或字节 '\n' 时都会刷新输出缓冲区)
            ps = new PrintStream(fos, true);
            if (ps != null) {// 把标准输出流(控制台输出)改成文件
                System.setOut(ps);
            }


            for (int i = 0; i <= 255; i++) { // 输出ASCII字符
                System.out.print((char) i);
                if (i % 50 == 0) { // 每50个数据一行
                    System.out.println(); // 换行
                }
            }


        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (ps != null) {
                ps.close();
            }
        }

    }
```

* 数据流(DataInputStream,DataOutputStream)

```java
练习：将内存中的字符串、基本数据类型的变量写出到文件中。

    注意：处理异常的话，仍然应该使用try-catch-finally.
     */
    @Test
    public void test3() throws IOException {
        //1.
        DataOutputStream dos = new DataOutputStream(new FileOutputStream("data.txt"));
        //2.
        dos.writeUTF("刘建辰");
        dos.flush();//刷新操作，将内存中的数据写入文件
        dos.writeInt(23);
        dos.flush();
        dos.writeBoolean(true);
        dos.flush();
        //3.
        dos.close();


    }
 /*
    将文件中存储的基本数据类型变量和字符串读取到内存中，保存在变量中。

    注意点：读取不同类型的数据的顺序要与当初写入文件时，保存的数据的顺序一致！

     */
    @Test
    public void test4() throws IOException {
        //1.
        DataInputStream dis = new DataInputStream(new FileInputStream("data.txt"));
        //2.
        String name = dis.readUTF();
        int age = dis.readInt();
        boolean isMale = dis.readBoolean();

        System.out.println("name = " + name);
        System.out.println("age = " + age);
        System.out.println("isMale = " + isMale);

        //3.
        dis.close();

    }

}
```

* 对象流(ObjectInputStream,ObjectOutputStream)

> 对象序列化机制允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点。当其它程序获取了这种二进制流，就可以恢复成原来的Java对象。
>
> 需要满足如下的要求，方可序列化
>
> 1. 需要实现接口：Serializable
>
> 2. 当前类提供一个全局常量：private static final long serialVersionUID
>
> 3. 除了当前Person类需要实现Serializable接口之外，还必须保证其内部所有属性也必须是可序列化的。（默认情况下，基本数据类型可序列化）
>
> 补充：ObjectOutputStream和ObjectInputStream不能序列化static和transient修饰的成员变量

```java
public class ObjectInputOutputStreamTest {

    /*
    序列化过程：将内存中的java对象保存到磁盘中或通过网络传输出去
    使用ObjectOutputStream实现
     */
    @Test
    public void testObjectOutputStream(){
        ObjectOutputStream oos = null;

        try {
            //1.
            oos = new ObjectOutputStream(new FileOutputStream("object.dat"));
            //2.
            oos.writeObject(new String("我爱北京天安门"));
            oos.flush();//刷新操作

            oos.writeObject(new Person("王铭",23));
            oos.flush();

            oos.writeObject(new Person("张学良",23,1001,new Account(5000)));
            oos.flush();

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(oos != null){
                //3.
                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }

    }

    /*
    反序列化：将磁盘文件中的对象还原为内存中的一个java对象
    使用ObjectInputStream来实现
     */
    @Test
    public void testObjectInputStream(){
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("object.dat"));

            Object obj = ois.readObject();
            String str = (String) obj;

            Person p = (Person) ois.readObject();
            Person p1 = (Person) ois.readObject();

            System.out.println(str);
            System.out.println(p);
            System.out.println(p1);

        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if(ois != null){
                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }



    }

}
```

* 随机存取文件流(RandomAccessFile)

> 1. RandomAccessFile直接继承于java.lang.Object类，实现了DataInput和DataOutput接口
>
> 2. RandomAccessFile既可以作为一个输入流，又可以作为一个输出流
>
> 3. 如果RandomAccessFile作为输出流时，写出到的文件如果不存在，则在执行过程中自动创建。如果写出到的文件存在，则会对原有文件内容进行覆盖。（默认情况下，从头覆盖）
>
> 4. 可以通过相关的操作，实现RandomAccessFile“插入”数据的效果

```java
 public class RandomAccessFileTest {

    @Test
    public void test1() {

        RandomAccessFile raf1 = null;
        RandomAccessFile raf2 = null;
        try {
            //1.
            raf1 = new RandomAccessFile(new File("爱情与友情.jpg"),"r");
            raf2 = new RandomAccessFile(new File("爱情与友情1.jpg"),"rw");
            //2.
            byte[] buffer = new byte[1024];
            int len;
            while((len = raf1.read(buffer)) != -1){
                raf2.write(buffer,0,len);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            //3.
            if(raf1 != null){
                try {
                    raf1.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
            if(raf2 != null){
                try {
                    raf2.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }

            }
        }
    }

    @Test
    public void test2() throws IOException {

        RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");

        raf1.seek(3);//将指针调到角标为3的位置
        raf1.write("xyz".getBytes());//

        raf1.close();

    }
    /*
    使用RandomAccessFile实现数据的插入效果
     */
    @Test
    public void test3() throws IOException {

        RandomAccessFile raf1 = new RandomAccessFile("hello.txt","rw");

        raf1.seek(3);//将指针调到角标为3的位置
        //保存指针3后面的所有数据到StringBuilder中
        StringBuilder builder = new StringBuilder((int) new File("hello.txt").length());
        byte[] buffer = new byte[20];
        int len;
        while((len = raf1.read(buffer)) != -1){
            builder.append(new String(buffer,0,len)) ;
        }
        //调回指针，写入“xyz”
        raf1.seek(3);
        raf1.write("xyz".getBytes());

        //将StringBuilder中的数据写入到文件中
        raf1.write(builder.toString().getBytes());

        raf1.close();

        //思考：将StringBuilder替换为ByteArrayOutputStream
    }
}
```

###### NIO

> 非阻塞IO，面向缓冲区的，基于通道的IO操作。
>
> 同步请求，A调用B，B的处理是同步的，在处理完之前他不会通知A，只有处理完之后才会明确的通知A。
>
> 异步请求，A调用B，B的处理是异步的，B在接到请求后先告诉A我已经接到请求了，然后异步去处理，处理完之后通过回调等方式再通知A。
>
> 所以说，同步和异步最大的区别就是被调用方的执行方式和返回时机。同步指的是被调用方做完事情之后再返回，异步指的是被调用方先返回，然后再做事情，做完之后通过回调通知调用方。
>
> 阻塞请求，A调用B，A一直等着B的返回，别的事情什么也不干。
>
> 非阻塞请求，A调用B，A不用一直等着B的返回，先去忙别的事情了。
>
> 所以说，阻塞和非阻塞最大的区别就是在被调用方返回结果之前的这段时间内，调用方是否一直等待。阻塞指的是调用方一直等待别的事情什么都不做。非阻塞指的是调用方先去忙别的事情。
>
> 阻塞、非阻塞和同步、异步其实针对的对象是不一样的。阻塞、非阻塞说的是调用者，同步、异步说的是被调用者。
>
> 先来看同步场景中是如何包含阻塞和非阻塞情况的：
>
> 我们是用传统的水壶烧水（同步）。在水烧开之前我们一直做在水壶前面，等着水开。这就是阻塞的。
>
> 我们是用传统的水壶烧水（同步）。在水烧开之前我们先去客厅看电视了，但是水壶不会主动通知我们，需要我们时不时的去厨房看一下水有没有烧开。这就是非阻塞的。
>
> 再来看异步场景中是如何包含阻塞和非阻塞情况的：
>
> 我们是用带有提醒功能的水壶烧水（异步）。在水烧发出提醒之前我们一直做在水壶前面，等着水开。这就是阻塞的。
>
> 我们是用带有提醒功能的水壶烧水（异步）。在水烧发出提醒之前我们先去客厅看电视了，等水壶发出声音提醒我们。这就是非阻塞的。

* 在Java语言中，一共提供了三种IO模型，分别是阻塞IO（BIO）、非阻塞IO（NIO）、异步IO（AIO）。这里面的BIO和NIO都是同步的IO模型，即同步阻塞IO和同步非阻塞IO，异步IO指的是异步非阻塞IO。

> BIO （Blocking I/O）：有一排水壶在烧开水，BIO的工作模式就是，叫一个线程停留在一个水壶那，直到这个水壶烧开，才去处理下一个水壶。但是实际上线程在等待水壶烧开的时间段什么都没有做。
>
> NIO （New I/O）：NIO的做法是叫一个线程不断的轮询每个水壶的状态，看看是否有水壶的状态发生了改变，从而进行下一步的操作。
>
> AIO （ Asynchronous I/O）：为每个水壶上面装了一个开关，水烧开之后，水壶会自动通知我水烧开了。

* NIO的特性/BIO(IO)和NIO的区别

> IO流是阻塞的，NIO流是非阻塞的。
>
> IO 面向流(Stream oriented)，而 NIO 面向缓冲区(Buffer oriented)。
>
> IO没有选择器，NIO有选择器(Selector)
>
> 选择器：选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

* NIO和AIO的区别

> AIO 也就是 NIO 2。在 Java 7 中引入了 NIO 的改进版 NIO 2,它是异步非阻塞的IO模型。异步 IO 是基于事件和回调机制实现的，被调用者被调用之后会立即返回，当做完事请之后，会通过回调机制来通过调用者继续后续的操作。

* 什么是Netty

> Netty是对JDK的NIO api的繁杂、需具备多线程技能做制垫、安全性、可靠性性能不足，进而对NIO进行了一次封装。

![](16.png)

> 1. jdk 7.0 时，引入了 Path、Paths、Files三个类。此三个类声明在：java.nio.file包下。
>
> 2. Path可以看做是java.io.File类的升级版本。也可以表示文件或文件目录，与平台无关。
>
> 3. 如何实例化Path：使用Paths。
>
>    static Path get(String first, String … more) : 用于将多个字符串串连成路径
>
>    static Path get(URI uri): 返回指定uri对应的Path路径

```java
public class PathTest {

    //如何使用Paths实例化Path
    @Test
    public void test1() {
        Path path1 = Paths.get("d:\\nio\\hello.txt");//new File(String filepath)

        Path path2 = Paths.get("d:\\", "nio\\hello.txt");//new File(String parent,String filename);

        System.out.println(path1);
        System.out.println(path2);

        Path path3 = Paths.get("d:\\", "nio");
        System.out.println(path3);
    }

    //Path中的常用方法
    @Test
    public void test2() {
        Path path1 = Paths.get("d:\\", "nio\\nio1\\nio2\\hello.txt");
        Path path2 = Paths.get("hello.txt");

//		String toString() ： 返回调用 Path 对象的字符串表示形式
        System.out.println(path1);

//		boolean startsWith(String path) : 判断是否以 path 路径开始
        System.out.println(path1.startsWith("d:\\nio"));
//		boolean endsWith(String path) : 判断是否以 path 路径结束
        System.out.println(path1.endsWith("hello.txt"));
//		boolean isAbsolute() : 判断是否是绝对路径
        System.out.println(path1.isAbsolute() + "~");
        System.out.println(path2.isAbsolute() + "~");
//		Path getParent() ：返回Path对象包含整个路径，不包含 Path 对象指定的文件路径
        System.out.println(path1.getParent());
        System.out.println(path2.getParent());
//		Path getRoot() ：返回调用 Path 对象的根路径
        System.out.println(path1.getRoot());
        System.out.println(path2.getRoot());
//		Path getFileName() : 返回与调用 Path 对象关联的文件名
        System.out.println(path1.getFileName() + "~");
        System.out.println(path2.getFileName() + "~");
//		int getNameCount() : 返回Path 根目录后面元素的数量
//		Path getName(int idx) : 返回指定索引位置 idx 的路径名称
        for (int i = 0; i < path1.getNameCount(); i++) {
            System.out.println(path1.getName(i) + "*****");
        }

//		Path toAbsolutePath() : 作为绝对路径返回调用 Path 对象
        System.out.println(path1.toAbsolutePath());
        System.out.println(path2.toAbsolutePath());
//		Path resolve(Path p) :合并两个路径，返回合并后的路径对应的Path对象
        Path path3 = Paths.get("d:\\", "nio");
        Path path4 = Paths.get("nioo\\hi.txt");
        path3 = path3.resolve(path4);
        System.out.println(path3);

//		File toFile(): 将Path转化为File类的对象
        File file = path1.toFile();//Path--->File的转换

        Path newPath = file.toPath();//File--->Path的转换

    }


}



Path path1 = Paths.get("d:\\nio", "hello.txt");
		Path path2 = Paths.get("atguigu.txt");
		
//		Path copy(Path src, Path dest, CopyOption … how) : 文件的复制
		//要想复制成功，要求path1对应的物理上的文件存在。path1对应的文件没有要求。
//		Files.copy(path1, path2, StandardCopyOption.REPLACE_EXISTING);
		
//		Path createDirectory(Path path, FileAttribute<?> … attr) : 创建一个目录
		//要想执行成功，要求path对应的物理上的文件目录不存在。一旦存在，抛出异常。
		Path path3 = Paths.get("d:\\nio\\nio1");
//		Files.createDirectory(path3);
		
//		Path createFile(Path path, FileAttribute<?> … arr) : 创建一个文件
		//要想执行成功，要求path对应的物理上的文件不存在。一旦存在，抛出异常。
		Path path4 = Paths.get("d:\\nio\\hi.txt");
//		Files.createFile(path4);
		
//		void delete(Path path) : 删除一个文件/目录，如果不存在，执行报错
//		Files.delete(path4);
		
//		void deleteIfExists(Path path) : Path对应的文件/目录如果存在，执行删除.如果不存在，正常执行结束
		Files.deleteIfExists(path3);
		
//		Path move(Path src, Path dest, CopyOption…how) : 将 src 移动到 dest 位置
		//要想执行成功，src对应的物理上的文件需要存在，dest对应的文件没有要求。
//		Files.move(path1, path2, StandardCopyOption.ATOMIC_MOVE);
		
//		long size(Path path) : 返回 path 指定文件的大小
		long size = Files.size(path2);
		System.out.println(size);


Path path1 = Paths.get("d:\\nio", "hello.txt");
		Path path2 = Paths.get("atguigu.txt");
//		boolean exists(Path path, LinkOption … opts) : 判断文件是否存在
		System.out.println(Files.exists(path2, LinkOption.NOFOLLOW_LINKS));

//		boolean isDirectory(Path path, LinkOption … opts) : 判断是否是目录
		//不要求此path对应的物理文件存在。
		System.out.println(Files.isDirectory(path1, LinkOption.NOFOLLOW_LINKS));

//		boolean isRegularFile(Path path, LinkOption … opts) : 判断是否是文件

//		boolean isHidden(Path path) : 判断是否是隐藏文件
		//要求此path对应的物理上的文件需要存在。才可判断是否隐藏。否则，抛异常。
//		System.out.println(Files.isHidden(path1));

//		boolean isReadable(Path path) : 判断文件是否可读
		System.out.println(Files.isReadable(path1));
//		boolean isWritable(Path path) : 判断文件是否可写
		System.out.println(Files.isWritable(path1));
//		boolean notExists(Path path, LinkOption … opts) : 判断文件是否不存在
		System.out.println(Files.notExists(path1, LinkOption.NOFOLLOW_LINKS));

/**
	 * StandardOpenOption.READ:表示对应的Channel是可读的。
	 * StandardOpenOption.WRITE：表示对应的Channel是可写的。
	 * StandardOpenOption.CREATE：如果要写出的文件不存在，则创建。如果存在，忽略
	 * StandardOpenOption.CREATE_NEW：如果要写出的文件不存在，则创建。如果存在，抛异常
	 *
	 * @author shkstart 邮箱：shkstart@126.com
	 * @throws IOException
	 */
	@Test
	public void test3() throws IOException{
		Path path1 = Paths.get("d:\\nio", "hello.txt");

//		InputStream newInputStream(Path path, OpenOption…how):获取 InputStream 对象
		InputStream inputStream = Files.newInputStream(path1, StandardOpenOption.READ);

//		OutputStream newOutputStream(Path path, OpenOption…how) : 获取 OutputStream 对象
		OutputStream outputStream = Files.newOutputStream(path1, StandardOpenOption.WRITE,StandardOpenOption.CREATE);


//		SeekableByteChannel newByteChannel(Path path, OpenOption…how) : 获取与指定文件的连接，how 指定打开方式。
		SeekableByteChannel channel = Files.newByteChannel(path1, StandardOpenOption.READ,StandardOpenOption.WRITE,StandardOpenOption.CREATE);

//		DirectoryStream<Path>  newDirectoryStream(Path path) : 打开 path 指定的目录
		Path path2 = Paths.get("e:\\teach");
		DirectoryStream<Path> directoryStream = Files.newDirectoryStream(path2);
		Iterator<Path> iterator = directoryStream.iterator();
		while(iterator.hasNext()){
			System.out.println(iterator.next());
		}


	}
}
```

