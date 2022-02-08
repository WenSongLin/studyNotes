# lang包常用类

# 1. 回顾

> 1. Error 与Exception 区别?

```java
共同点:
  1. 父类都是Throwable
  2. 程序里面不正常的现象。    
不同点:
  1. 处理方式: 
      Error 不能处理。JVM 程序终止。
      Exception: java语言具有健壮性。 有异常处理机制的。 
```



> 2. 异常分为哪几类?  常见的异常都有哪些? 处理方式?

```java
Exception 分为2大类 ： RuntimeException   编译时异常
 1. RuntimeException  不推荐处理。提前预判数据。
 2. 编译时: 必须要去处理。
     1. 捕获 try..catch...finally
     2. 抛出 throws     
```



> 3. throws   vs  throw

```java
1. 控制流程 
2. 异常信息传递： 声明抛出一个父级异常对象   Casued By
3. 与自定义异常综合使用。    
```



#  2. 包装类  null

> 概念： 对基本类型的包装。

```java
基本类型: 8种
包装类: 8个
byte short     char       int         long  float  double  boolean
Byte Short    Character   Integer     Long  Float   Double   Boolean

原因:
   java语言面向对象的语言。我们不能使用基本类型创建对象。不符合java语言的特性的。
   为了丰富基本类型的操作，提供8个包装类型。    
基本 vs包装: int  vs Integer
    1. 共同点 都属于整型  都代表整型的数据。
    2. Integer 里面属性和方法，null    int: 0
```



## 2.1 Integer

```java
public final class Integer
extends Number
implements Comparable<Integer>
```



> 常用构造方法

```java
Integer(int value) 
Integer(String s)   String 转 int      "200"
    
private final int value;  维护包装类数据    
```



> 常用功能方法

```java
Integer.valueOf(200)
```

```java
public static Integer valueOf(int i) {//100    200
        if (i >= IntegerCache.low && i <= IntegerCache.high)// -128 - 127
            return IntegerCache.cache[i + (-IntegerCache.low)];
        return new Integer(i);
    }
 private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)  // 循环存储数组元素  -128-127
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

        private IntegerCache() {}
    }
```



```java
int num2 = Integer.parseInt(numStr);
```

> parseInt 底层代码

```java


 public static int parseInt(String s, int radix)
                throws NumberFormatException
    {
        /*
         * WARNING: This method may be invoked early during VM initialization
         * before IntegerCache is initialized. Care must be taken to not use
         * the valueOf method.
         */

        if (s == null) {
            throw new NumberFormatException("null");
        }

        if (radix < Character.MIN_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " less than Character.MIN_RADIX");
        }

        if (radix > Character.MAX_RADIX) {
            throw new NumberFormatException("radix " + radix +
                                            " greater than Character.MAX_RADIX");
        }

        int result = 0;
        boolean negative = false;
        int i = 0, len = s.length();//获得字符串的长度 (字符个数)
        int limit = -Integer.MAX_VALUE;
        int multmin;
        int digit;

        if (len > 0) {
            char firstChar = s.charAt(0);// 3  0-2 获得指定索引处的字符数据
            if (firstChar < '0') { // Possible leading "+" or "-"
                if (firstChar == '-') {
                    negative = true;
                    limit = Integer.MIN_VALUE;
                } else if (firstChar != '+')
                    throw NumberFormatException.forInputString(s);

                if (len == 1) // Cannot have lone "+" or "-"
                    throw NumberFormatException.forInputString(s);
                i++;
            }
            multmin = limit / radix;
            while (i < len) {
                // Accumulating negatively avoids surprises near MAX_VALUE
                digit = Character.digit(s.charAt(i++),radix);
                //将指定字符转换成指定进制下的数据  10  0-9  '1' 1
                //1 
                if (digit < 0) {
                    throw NumberFormatException.forInputString(s);
                }
                if (result < multmin) {
                    throw NumberFormatException.forInputString(s);
                }
                result *= radix; // 0   -10  -120
                if (result < limit + digit) {
                    throw NumberFormatException.forInputString(s);
                }
                result -= digit;//result=result-digit;  -1  -12  -123
            }
        } else {
            throw NumberFormatException.forInputString(s);
        }
        return negative ? result : -result;
    }
```



## 2.2 装箱 vs  拆箱

> 当基本类型的数据 自动 转换成  包装类对象。  包装类.valurOf(基本类型 变量)

```java
Integer num1 = 100;
Integer num1 = Integer.valueOf(100);
```

> 拆箱:  包装类对象 转换 成 基本类型的数据

```java
int a = num1.intValue();
//拆箱
System.out.println(Integer.compare(num1, num2));
```



## 2.3 字符串转int

```java
String numStr = "+456";
//将字符串转int
int num = new Integer(numStr);
int num1 = Integer.valueOf(numStr);
int num2 = Integer.parseInt(numStr);//parseInt()
System.out.println(num);
System.out.println(num1);
System.out.println(num2);
```



```java
private static int demo5(String numStr, int radix) {

        //获得字符串里面每个字符数据
        Objects.requireNonNull(numStr);
        //获得字符串的长度
        int length = numStr.length();
        if (length == 0) {
            throw new NumberFormatException("字符串长度为0，无法解析！");
        }

        int index = 0;
        int result = 0;
        boolean flag = false;

        //第一个字符
        char firstChar = numStr.charAt(0);//获得字符串指定索引的字符数据
        if (firstChar == '-') {
            flag = true;
            index++;
        } else if (firstChar == '+') {
            index++;
        } else if (firstChar < 48 || firstChar > 57) {
            throw new NumberFormatException("字符串里面包含非数字的数据");
        }

        while (index < length) {
            char ch = numStr.charAt(index++);//字符数据  48-57
            if (ch < 48 || ch > 57) {
                throw new NumberFormatException("字符串里面包含非数字的数据");
            }
            int num = ch - 48;
            result = result * radix + num;
        }
        return flag ? -result : result;
    }
```





## 2.4 Character

> 常用方法

```java
private static void demo1() {
        Character ch1 = new Character('1');
        Character ch2 = 97;
        Character ch3 = Character.valueOf('a');

        System.out.println(ch1);
        System.out.println(ch2);
        System.out.println(ch3);

        //常用的静态方法
        System.out.println(Character.isDigit(ch1));//判断字符是否是数字
        System.out.println(Character.isLetter(ch2));
        System.out.println(Character.toLowerCase(Character.toUpperCase(ch2)));
        System.out.println(Character.isWhitespace(' '));

        //parseInt()
        System.out.println(Character.digit('a', 16));//获得指定进制里面的数据
        
    }
```

## 2.5 成员变量_包装类

```java
@Setter
@Getter
public class Rank {

    private Integer id;
    private Integer personCount;
    private Double increase;//0.0

    public static void main(String[] args) {
        //获得昨天的数据
        Rank rank2 = null;
        try {
            Rank rank1 = new Rank();
            rank1.setId(1);
            rank1.setPersonCount(100);

            //今天数据
            rank2 = new Rank();
            rank2.setId(2);
            rank2.setPersonCount(200);//
            System.out.println(3 / 0);

            double result = (200 - 100) / 100;
            rank2.setIncrease(result);
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(rank2.getIncrease());// 没有赋值导致出现 获得默认值 0.0  null

    }
```



# 3. Math

```java
static double PI;
static double abs(double a)  
static int max(int a, int b)  
static double random() 
static long round(double a)  
static double pow(double a, double b)     
```



# 4. Object



```java
1. protected Object clone()  克隆对象的副本  
2. boolean equals(Object obj)  比较2个对象是否一致 默认比较是地址值  ==
3. int hashCode()   获得对象的hash值 hash值不等 2个对象肯定不等  hash值相等 2个不等
4. String toString()   将对象转换成String
5. protected void finalize()  GC  默认调用对象的 finalize方法 回收对象
6. Class<?> getClass()  获得正在运行的类或者接口的Class类对象
    jvm--->class文件--->肯定会创建与class文件一致的Class类对象
7. void notify()/notifyAll() 在另外一个线程里面唤醒另外一个wait的线程
8. void wait()/wait(long timeout) /wait(long timeout, int nanos)  当前线程等待      
```



## 4.1 比较2个对象

> 重写equals 必须重写 hashcode

```java
//1. jvm先调用hashcode比较对象
//2个对象hashcode不一样  2个对象就肯定不一样  true 不执行equals
//2个对象haashcode有可能一样 不能完全信任hashcode 依然调用equals
//不同对象的hashcode是有可能相等的
```



```java
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
public class User extends Object {
    //类里面所有的属性全部使用包装类修饰
    private Integer id;
    private String name;
    private Integer age;

    //重写父类的equals的方法 必然重写hashcode
    //比较规则: 当id  name  age的数据都相等的时候  就认为2个对象是相等的

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        User user = (User) o;
        return
                Objects.equals(name, user.name );
    }

    @Override
    public int hashCode() {
        return Objects.hash( name);
    }
}
```



```java
public static void main(String[] args) {

        //比较2个对象: 内存地址值
        User user1 = new User();
        User user2 = new User();

        System.out.println(user1 == user2);//false
        System.out.println(user1.equals(user2));//false

        System.out.println(user1.hashCode());
        System.out.println(user2.hashCode());

        System.out.println("------------------------");
        //根据属性值比较
        //规则: id  name  age 2个对象是相等的
        User user3 = new User(1, "张三", 20);
        User user4 = new User(1, "张三", 20);

        System.out.println(user3.equals(user4));

        //user3 和user4 哈希值必然一样
        System.out.println(user3.hashCode());
        System.out.println(user4.hashCode());

        //1. jvm先调用hashcode比较对象
        //2个对象hashcode不一样  2个对象就肯定不一样  true 不执行equals
        //2个对象haashcode有可能一样 不能完全信任hashcode 依然调用equals
        //不同对象的hashcode是有可能相等的

        String str1 = new String("Ma");
        String str2 = new String("NB");
        System.out.println(str1.equals(str2));
        System.out.println(str1.hashCode());
        System.out.println(str2.hashCode());
  }
```





## 4.2  克隆

> 获得指定对象的副本。  ==浅克隆==  和 深克隆。

```java
@Setter
@Getter
@AllArgsConstructor
@NoArgsConstructor
@ToString
//Cloneable 标识性接口  jvm可以按照自己规则克隆对象。不执行构造
public class Student implements Cloneable{

    private Integer id;

    private String name;
    private Integer age;
    private String[] hobby;

    //private Computer computer;

    //重写父类的clone的方法
    @Override
    public Student clone() {

        Student student = null;
        try {
            student = (Student) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
            System.out.println("必须让当前类对象支持被克隆,当前类必须实现Cloneable接口");
        }
        return student;
    }
}
```

```java
 //出现的问题:
//引用类型: 一个对象的数据改变  不要影响其他数据
//1. 深克隆  遍历式执行复制
//2. 序列化方法  读写对象
```

```java
@Override
public Student clone() {

    Student student = null;
    try {
        student = (Student) super.clone();
        student.setHobby(hobby.clone());
        //遍历式执行复制
        //或者使用序列化操作对象

    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
        System.out.println("必须让当前类对象支持被克隆,当前类必须实现Cloneable接口");
    }
    return student;
}
```



![](pic\克隆.png)

## 4.2 引用

```java
 public static void main(String[] args) {
        String[] hobby = {"game", "code", "music"};
        Object obj = new Object();
        Student student = new Student(1001, "张三", 30, hobby);
        //student  是一个变量,对象，引用?
        //回收对象: 回收引用指向的堆内存里面对象
        //1.强引用: 有用且必须的对象
        // 在程序运行期间  jvm内存1G  占据了1020M student可能占据5M内存
        //OOM out of memory Error   GC不会回收强引用
        //2.软引用: 有用非必须对象  SoftReference
        //一直检测内存  当内存不足的时候 自动回收之前那些对象
         //2.软引用: 有用非必须对象  SoftReference
        //一直检测内存  当内存不足的时候 自动回收之前那些对象

        SoftReference<Student> softReference = new SoftReference<>(student);
        System.out.println(softReference.get());

        student = null;//student 从有用变为无用对象
        System.gc();
        //        if(内存不足){
        //            System.gc();
        //        }
        System.out.println(softReference.get());
     
        //3.弱引用: 有用非必须对象  引用关联的对象活不过下一次GC

//        WeakReference<Student> weakReference = new WeakReference<>(student);
//        System.out.println(weakReference.get());
//        student = null;//将有用变为无用对象
//        System.gc();
//        System.out.println(weakReference.get());


        //4.虚引用  GC活动轨迹
        ReferenceQueue<Student> queue = new ReferenceQueue<>();
        PhantomReference<Student> phantomReference = new PhantomReference<>(student, queue);
        System.out.println(phantomReference.get());//null

    }
```





# 5. Class类

```java
public static void main(String[] args) {

        //1. 对象.getClass() (不推荐 与耦合度台高)
        //获得jvm里面Student.class所对应的Class类对象

//        Student student = new Student();
//        Class clazz = student.getClass();
//        System.out.println(clazz.toString());

        //2. 静态class属性  比较推荐  访问本项目里面的类或者接口的class
//        Class aClass = Student.class;
//        System.out.println(aClass);

        //3.利用Class.forName("类/接口全限定名称") 类/接口路径
        //加载第三方里面的类库的类或者接口的字节码  缺少jar
        try {
            Class aClass = Class.forName("com.javasm.clone_.Student");
            System.out.println(aClass);
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
            System.out.println("jvm里面肯定没有加载Student字节码文件");
        }
    }
}
```





