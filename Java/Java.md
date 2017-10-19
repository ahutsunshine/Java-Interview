
### **1.以下代码有问题吗，是什么问题？**
1.1
```java
List<Object> obj = new ArrayList<Long>();
obj.add("I love Android!");
```

1.2
```java
Object[] objArray = new Long[1];
objArray[0] = "I love Android!";
```

**结果：**
```
1.1 编译出错。
1.2 编译可以，运行出错。
```

**原因：**

1.1 `List\<Object\>`和`ArrayList\<Long\>`没有继承关系。

1.2 数组是在运行时类型检查。

### **2.以下代码运行结果？**
2.1
```java
Long l1 = 128L;
Long l2 = 128L;
System.out.print(l1 == l2);
System.out.print(l1 == 128L);
```

2.2
```java
Long l1 = 127L;
Long l2 = 127L;
System.out.print(l1 == l2);
System.out.print(l1 == 127L);
```

**结果：**
```
2.1 false  true。   
2.2 true  true。
```

**原因：**

2.1 Long 包装类型常量 cache 为 -128 ～ 127 之间，所以 l1 和 l2 是两个对象，== 比较的是对象的地址，所以第一个打印为 false；接着由于包装类型在表达式中且表达式中至少有一个不是包装类型，所以 Long l1== 128L 中 l1 自动拆箱比较，所以数值比较为 true。

2.2 Long 包装类型 -128 - 127 之间的值 Java 维护在一个常量池中，所以 l1 和 l2 引用同一个对象。

还可参见第 16 题解析。

### **3.程序可以编译运行吗，结果是神马？**
```java
List<String>[] list = new List<String>[3];
```

**结果：**

不可以运行，编译报错。

**原因：**

创建泛型、参数化类型或者类型参数的数组是非法的，都会导致编译错误。

考察点：Java 默认是没有泛型数组的

### **4.是否存在使 `i>j || i<=j` 结果为 false 的数吗？**
**结果：**

存在。

**原因：**

数值 `NaN` 代表 not a number，无法用于比较，例如即使是 `i = Double.NaN; j = i;` 最后 `i == j` 的结果依旧为 false。

考察点：Java NaN。

### **5.下面程序的运行结果是什么？**
```java
public class Main {
    public static void main(String[] args) {
        TxtThread txtThread = new TxtThread();

        new Thread(txtThread).start();
        new Thread(txtThread).start();
        new Thread(txtThread).start();
    }

    static class TxtThread implements Runnable {
        int num = 100;
        String str = new String();

        @Override
        public void run() {
            synchronized (str) {
                while (num > 0) {
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException ex) {
                        Logger.getLogger(TxtThread.class.getName()).log(Level.SEVERE, null, ex);
                    }
                    System.out.println(Thread.currentThread().getName() + " this is " + num--);
                }
            }
        }
    }
}
```
**结果：**
```
Thread-0 this is 100
......	
......	
Thread-0 this is 1	
```

**原因：**

三个线程共享对一个 Runnable 对象，所以同步锁中其他两个线程没有执行机会。

### **6.下面这段代码中，第几行的 fobj/sobj 符合垃圾收集器的收集标准？**
6.1
```java
 1．fobj = new Object();   
 2．fobj.Method();   
 3．fobj = new Object();   
 4．fobj.Method();   
```

6.2
```java
1．Object sobj = new Object();   
2．Object sobj = null;   
3．Object sobj = new Object();   
4．sobj = new Object();   
```

6.3
```java
1．Object aobj = new Object ( ) ;   
2．Object bobj = new Object ( ) ;   
3．Object cobj = new Object ( ) ;   
4．aobj = bobj;   
5．aobj = cobj;   
6．cobj = null;   
7．aobj = null; 
```

6.4

```java
static Vector v = new Vector(10);
for (int i=0; i<100; i++) {
    Object aobj = new Object();
    v.add(aobj);
    aobj = null;
}
```

**结果及原因：**

6-1、第 3 行。因为第 3 行的 fobj 被赋了新值，产生了一个新的对象，即换了一块新的内存空间，也相当于为第 1 行中的 fobj 断开了引用，这种类型的题是最简单的。 

6-2、第 2、4 行。因为第 2 行为 sobj 赋值为 null，所以在此第 1 行的 sobj 符合垃圾收集器的收集标准。而第 4 行相当于为 sobj 赋值为 null，所以在此第 3 行的 sobj 也符合垃圾收集器的收集标准。 

6-3、第 4、7 行。行 1-3 分别创建了 Object 类的三个对象aobj、bobj、cobj；行 4 此时对象 aobj 的句柄指向 bobj，原来 aojb 指向的对象已经没有任何引用或变量指向，这时，就符合回收标准了；行 5 此时对象 aobj 的句柄指向 cobj，所以该行的执行不能使 aobj 符合垃圾收集器的收集标准；行 6 此时仍没有任何一个对象符合垃圾收集器的收集标准；行 7 对象 cobj 符合了垃圾收集器的收集标准，因为 cobj 的句柄指向单一的地址空间，在第 6 行的时候 cobj 已经被赋值为 null，但由于 cobj 同时还指向了 aobj（第5行），所以此时 cobj 并不符合垃圾收集器的收集标准，而在第 7 行 aobj 所指向的地址空间也被赋予了空值 null，这就说明由 cobj 所指向的地址空间已经被完全地赋予了空值，所以此时 cobj 最终符合了垃圾收集器的收集标准，但对于 aobj 和 bobj 仍然无法判断其是否符合收集标准，总之在 Java 语言中，判断一块内存空间是否符合垃圾收集器收集的标准只有两个：给对象赋予了空值 null，以下再没有调用过；给对象赋予了新值，既重新分配了内存空间。 

6-4 的　aobj = null　不会让　aobj 对应的实例回收，因为这仅仅是释放了对象引用，而对象实质还有被静态的　v 所持有。

### **7.分别回答以下问题？**
7-1 下面程序的运行结果是什么？
```java
String s1 = "abc";
StringBuffer s2 = new StringBuffer(s1);
System.out.println(s1.equals(s2));

StringBuffer s3 = new StringBuffer("abc");
System.out.println(s3.equals("abc"));
System.out.println(s3.toString().equals("abc"));
```
**解析：**

第一个打印为　false。
第二个打印为　false。
第三个打印为　true。

因为第一个考 String 的 equals 方式，equals 方法进行了 instance of 判断。第二个 s3.equals　为　Object　类型，所以实现为　＝＝　判断，故　false。第三个就是常规套路了。

7-2 如何实现字符串的反转及替换？ 

**解析：**

方法很多，可以自己写实现也可以使用 String 或 StringBuffer、StringBuilder 中的方法，不过有一道很常见的面试题是用递归实现字符串反转，代码如下所示：
```java
    public static String reverse(String originStr) {
        if(originStr == null || originStr.length() <= 1) 
            return originStr;
        return reverse(originStr.substring(1)) + originStr.charAt(0);
    }
```

7-3 怎样将 GB2312 编码的字符串转换为 ISO-8859-1 编码的字符串？ 

**解析：**
```java
String s1 = "你好";
String s2 = new String(s1.getBytes("GB2312"), "ISO-8859-1");
```

7-4 String、StringBuffer、StringBuilder 的区别是什么？String 为什么是不可变的？

**解析：**

String 是字符串常量，StringBuffer 和 StringBuilder 都是字符串变量，后两者的字符内容可变，而前者创建后内容不可变；StringBuffer 是线程安全的，而 StringBuilder 是非线程安全的，线程安全会带来额外的系统开销，所以 StringBuilder 的效率比 StringBuffer 高。

String 不可变是因为在 JDK 中 String 类被声明为一个 final 类，只有当字符串是不可变时字符串池才有可能实现，字符串池的实现可以在运行时节约很多 heap 空间，因为不同的字符串变量都指向池中的同一个字符串；如果字符串是可变的则会引起很严重的安全问题，譬如数据库的用户名密码都是以字符串的形式传入来获得数据库的连接，或者在 socket 编程中主机名和端口都是以字符串的形式传入，因为字符串是不可变的，所以它的值是不可改变的，否则黑客们可以钻到空子改变字符串指向的对象的值造成安全漏洞；因为字符串是不可变的，所以是多线程安全的，同一个字符串实例可以被多个线程共享，这样便不用因为线程安全问题而使用同步，字符串自己便是线程安全的；因为字符串是不可变的所以在它创建的时候 hashcode 就被缓存了，不需要重新计算，这就使得字符串很适合作为 Map 的键，字符串的处理速度要快过其它的键对象，这就是 HashMap 中的键往往都使用字符串的原因。

### **8.下面的题目结果是什么?**
```java
public class TestFool {
    public static String foo(){
        System.out.println("Test foo called");
        return "";
    }

    public static void main(String[] args) {
        TestFool obj = null;
        System.out.println(obj.foo());
    }
}
```
**结果：**

Test foo called。

**原因:**

jvm 内存里有栈区、堆区，栈区主要用来存放基础类型数据和局部变量，堆区主要存放 new 出来的对象。在堆区又有一个叫做方法区的内存区域用来存放常量、static 变量和 static 方法，还有类的信息，static 的变量和方法不依赖对象，即使对象没有创建，在类加载的时候已经存在信息了，jvm 识别出是 static 方法就直接调用了在方法区内存里的方法，没有报空指针异常。细节参见 10 题解析。

### **9.以下继承关系，运行结果是？**
```java
public class Base {
    public Base(){
        test();
    }
    
    public void test(){
    }
}

public class Child extends Base {
    private int a = 123;
    
    public Child(){
    }
    
    public void test(){
        System.out.println(a);
    }
}

public static void main(String[] args){
    Child c = new Child();
    c.test();
}
```
**结果：**
```
0 123
```
**原因：**

Child 直接继承于 Base，默认构造函数不显示调用 super 也会直接父类的默认构造函数，所以首先调用 `Base.java-->test()` 方法。这时 Child 类还没有构造完毕，a 基本数据类型还没有赋予值，a 又为成员变量默认值为 0。
如果在父类构造方法中调用了可被重写的方法则可能会出现意想不到的效果，慎重使用，最好只调用 private 方法。

### **10.以下继承关系，运行结果是？**
```java
public class Base {
	public static String s = "static_base";
	public String m = "base";
	
	public static void staticTest(){
		System.out.println("base static: "+s);
	}
}

public class Child extends Base {
	public static String s = "child_base";
	public String m = "child";
	
	public static void staticTest(){
		System.out.println("child static: "+s);
	}
}

public static void main(String[] args) {
	Child c = new Child();
	Base b = c;
	
	System.out.println(b.s);
	System.out.println(b.m);
	b.staticTest();
	
	System.out.println(c.s);
	System.out.println(c.m);
	c.staticTest();
}
```
**结果：**
```
static_base
base
base static: static_base
child_base
child
child static: child_base
```
**原因：**

当通过 b 访问时访问的是 Base 的变量和方法，当通过 c 访问时访问的是 Child 的变量和方法，这就是所谓的静态绑定，即访问绑定到变量的静态类型，静态绑定在程序编译阶段即可决定，而动态绑定则要等到程序运行时，实例变量、静态变量、静态方法、private 方法都是静态绑定。
继承重名是允许的，重名后实质有两个变量或者方法，对于 private 变量和方法，他们只能在类内被访问，访问的也永远是当前类（即子类中访问子类的，父类中访问父类的），仅仅是名字相同而已，没有任何关系；对于 public 变量和方法要看如何访问它，在类内部访问的是当前类的，单子类可以通过 super 明确指定访问父类的，在类外则要看访问变量的静态类型，静态类型是父类则访问父类的变量和方法，静态类型是子类则访问子类的变量和方法。

### **11.如果出现以下代码，编译运行会出现什么现象？**
```java
public class Base {
	protected void protect(){
	}
	
	public void open(){        
	}
}

public class Child extends Base {
//1放开会有啥现象
//    private void protect(){
//    }
	
//2放开会有啥现象
//    protected void open(){        
//    }
//3放开会有啥现象
	public void protect(){        
	}
}
```

**结果:**
```
如果 1 放开会在编译阶段报错；
如果 2 放开会在编译阶段报错；
如果 3 放开编译运行正常；
```
**原因：**

重载是指方法名相同但参数签名不同，重写是指子类重写父类相同签名的方法，但是重写方法不能比被重写方法限制有更严格的访问级别（即可见性重写）。

### **12.以下继承方式运行时会如何打印？**
```java
public class Base {
	public static int s;
	private int a;

	static {
		System.out.println("基类静态代码块, s:" + s);
		s = 1;
	}

	{
		System.out.println("基类实例代码块, a:" + a);
		a = 1;
	}

	public Base() {
		System.out.println("基类构造方法, a:" + a);
		a = 2;
	}

	protected void step() {
		System.out.println("base s: " + s + " , a : " + a);
	}

	public void action() {
		System.out.println("start");
		step();
		System.out.println("end");
	}
}

public class Child extends Base {
	public static int s;
	private int a;
	static {
		System.out.println("子类静态代码块， s:"+s);
		s = 10;
	}

	{
		System.out.println("子类实例代码块， a:"+a);
		a = 10;
	}

	public Child(){
		System.out.println("子类构造方法，a："+a);
		a = 20;
	}

	protected  void  step(){
		System.out.println("child s: "+s+" , a: "+a);
	}
}

public static void main(String[] args) {
    System.out.println("------ new Child()");
    Child c = new Child();
    System.out.println("\n------ c.action()");
    c.action();

    Base b = c;
    System.out.println("\n-------b.action()");
    b.action();

    System.out.println("\n ------- b.s: "+b.s);
    System.out.println("\n ------- c.s: "+c.s);
}
```

**结果:**
```
------ new Child()
基类静态代码块, s:0
子类静态代码块， s:0
基类实例代码块, a:0
基类构造方法, a:1
子类实例代码块， a:0
子类构造方法，a：10

------ c.action()
start
child s: 10 , a: 20
end

-------b.action()
start
child s: 10 , a: 20
end

------- b.s: 1
------- c.s: 10
```
**原因:**

其实考察的是 Java 类加载与初始化，加载就是在第一次使用类时动态把类加载到内存，加载一个类会看其父类是否已经加载，如果没有则会先加载父类。类加载过程如下：分配内存保存类信息->给类变量赋默认值->加载父类->设置父子关系->执行类初始化代码；而类初始化过程（先执行父类再执行子类，父类执行时子类静态变量是有默认值 0 的）如下：定义静态变量时的赋值语句->静态初始化代码块；这些 OK 以后就进入实例初始化过程：定义实例变量时的赋值语句->实例初始化代码块->构造方法。
所以上面的结果基本就是这个流程，只是寻找要执行的实例方法时是从对象的实际类型信息开始查找的，找不到才会查父类；对于变量无论是类变量还是实例变量都是静态绑定访问的。

### **13.内部类你知多少？**
java 内部类分为几种，各种自己有哪些特性？

**结果：**

静态内部类，成员内部类，方法内部类，匿名内部类；

匿名内部类访问外部类的局部变量，局部变量需要 final 修饰，1.8 之后该关键字被省略。

方法内部类只能在定义该内部类的方法内实例化，不可以在此方法外对其实例化。

静态内部类与其他成员相似，可以用 static 修饰内部类，这样的类称为静态内部类，静态内部类与静态内部方法相似，只能访问外部类的 static 成员，不能直接访问外部类的实例变量，与实例方法，只有通过对象引用才能访问。

成员内部类没有用 static 修饰且定义在在外部类类体中。

更多参见[java之内部类----非静态内部类、静态内部类、局部内部类、匿名内部类](http://www.cnblogs.com/Gaojiecai/p/4041663.html)一文。

### **14.下面程序的运行结果是什么？**
```java
public enum Size {
	SMALL, MEDIUM, LARGE
}

Size size = Size.SMALL;
System.out.println(size==Size.SMALL);
System.out.println(size.equals(Size.SMALL));
System.out.println(size==Size.MEDIUM);
System.out.println(size.toString());
System.out.println(size.name());
```

**结果：**
```
true
true
false
SMALL
SMALL
```
**原因：**

枚举可以使用 == 或者 equals 进行比较，结果都一样；枚举也是有顺序的，可以比较大小；枚举变量的 toString 方法和 name 方法返回值一样，都是其字面值；枚举的本质也是类，在编译阶段编译器自动做了很多事情，所以使用起来便捷。

### **15. 请粗略描述下面这些方法调用执行流程和最终返回值？**
```java
public static int test(){
	int ret = 0;
	try{
		return ret;
	}finally{
		ret = 2;
	}
}

public static int test(){
	int ret = 0;
	try{
		int a = 5/0;
		return ret;
	}finally{
		return 2;
	}
}

public static void test(){
	try{
		int a = 5/0;
	}finally{
		throw new RuntimeException("hello");
	}
}
```

**结果：**

1、返回 0，执行到 try 的 return ret; 语句前会先将返回值 ret 保存在一个临时变量中，然后才执行 finally 语句，最后 try 再返回那个临时变量，finally 中对 ret 的修改不会被返回。

2、返回 2，5/0 会触发 ArithmeticException 异常，但是 finally 中有 return 语句，finally 中 return 不仅会覆盖 try 和 catch 内的返回值，还会掩盖 try 和 catch 内的异常，就像异常没有发生一样，所以这个方法就会返回 2，而不再向上传递异常了。

3、运行抛出 hello 异常，因为如果 finally 中抛出了异常，则原异常就会被掩盖。

**总结：**

为避免混淆我们应该避免在 finally 中使用 return 语句或者抛出异常，如果调用的其他代码可能抛出异常，则应该捕获异常并进行处理。

### **16.java1.5开始的自动装箱和开箱机制是编译器特性还是虚拟机运行时特性？分别是怎么实现的？**
```java
Integer i1 = 100;
Integer i2 = 100;
Integer i3 = 200;
Integer i4 = 200;
System.out.println(i1 == i2);
System.out.println(i3 == i4);

Double d1 = 100.0;
Double d2 = 100.0;
Double d3 = 200.0;
Double d4 = 200.0;
System.out.println(d1 == d2);
System.out.println(d3 == d4);

Boolean b1 = false;
Boolean b2 = false;
Boolean b3 = true;
Boolean b4 = true;
System.out.println(b1 == b2);
System.out.println(b3 == b4);

Integer a = 1;
Integer b = 2;
Integer c = 3;
Integer d = 3;
Integer e = 321;
Integer f = 321;
Long g = 3L;
Long h = 2L;
System.out.println(c == d);
System.out.println(e == f);
System.out.println(c == (a + b));
System.out.println(c.equals(a + b));
System.out.println(g == (a + b));
System.out.println(g.equals(a + b));
System.out.println(g.equals(a + h));

Integer a = 444;
int b = 444;
System.out.println(a==b);
System.out.println(a.equals(b));
```

**结果：**
```
true
false
false
false
true
true
true
false
true
true
true
false
true

true
true
```
**原因:**

第一第二没什么解释的，第三句 a+b 包含运算符所以自动拆箱，所以比较值，对于 c.equals(a+b) 会先自动触发自动拆箱再触发自动装箱再比较。

java 1.5 开始的自动装箱拆箱机制其实是编译器自动完成的替换，装箱阶段自动替换为了 valueOf 方法，拆箱阶段自动替换为了 xxxValue 方法。对于 Integer 类型的 valueOf 方法参数如果是 -128~127 之间的值会直接返回内部缓存池中已经存在对象的引用，参数是其他范围值则返回新建对象；而 Double 类型与 Integer 类型一样会调用到 Double 的 valueOf 方法，但是 Double 的区别在于不管传入的参数值是多少都会 new 一个对象来表达该数值（因为在指定范围内浮点型数据个数是不确定的，整型等个数是确定的，所以可以 Cache）。
注意：Integer、Short、Byte、Character、Long 的 valueOf 方法实现类似，而 Double 和 Float 比较特殊，每次返回新包装对象。
对于两边都是包装类型的比较 == 比较的是引用，equals 比较的是值，对于两边有一边是表达式（包含算数运算）则 == 比较的是数值（自动触发拆箱过程），对于包装类型 equals 方法不会进行类型转换。

### **17.String 消消乐，下面程序的执行结果分别是什么？**
```java
String stra = "ABC";
String strb = new String("ABC");
System.out.println(stra == strb);		//false
System.out.println(stra.equals(strb));		//true

String str1 = "123";
System.out.println("123" == str1.substring(0));		//true
System.out.println("23" == str1.substring(1));		//false

String str3 = new String("ijk");
String str4 = str3.substring(0);
System.out.println(str3 == str4);		//true
System.out.println((new String("ijk") == str4));		//false

String str5 = "NPM";
String str6 = "npm".toUpperCase();
System.out.println(str5 == str6);		//false
System.out.println(str5.equals(str6));		//true

String str7 = new String("TTT");
String str8 = "ttt".toUpperCase();
System.out.println(str7 == str8);		//false
System.out.println(str7.equals(str8));		//true

String str9 = "a1";
String str10 = "a" + 1;
System.out.println(str9 == str10);		//true

String str11 = "ab";
String str12 = "b";
String str13 = "a" + str12;
System.out.println(str11 == str13);		//false

String str14 = "ab";
final String str15 = "b";
String str16 = "a" + str15; 
System.out.println(str14 == str16);		//true


private static String getBB() {   
    return "b";   
}
String str17 = "ab";   
final String str18 = getBB();   
String str19 = "a" + str18;   
System.out.println(str17 == str19);		//false


String str20 = "ab";
String str21 = "a";   
String str22 = "b";   
String str23 = str21 + str22;   
System.out.println(str23 == str20);   		//false
System.out.println(str23.intern() == str20);		//true
System.out.println(str23 == str20.intern());		//false
System.out.println(str23.intern() == str20.intern());		//true
```
**结果：**
```
见题目中注释部分，基于 JDK 1.7 版本分析。
```

**原因：**

对于 stra 与 strb 来说没啥分析的，显式的创建了新对象，使用 == 总是不等，equals 被重写过，值判断。

对于 str1 的 substring 来说其实跳进去看下这个方法的实现就明白了，里面有个 index == 0 判断，等于 0 就直接返回当前对象，否则新 new 一个。

对于 str3 与 str4 来说就没啥分析的必要了，参见上面两个分析结果。

对于 str5、str6、str7、str8 来说实质都一样，toUpperCase 方法内部创建了新字符串对象。

对于 str9 与 str10 来说当两个字符串常量连接时（相加）得到的新字符串依然是字符串常量且保存在常量池中。

对于 str11 到 str13 来说当字符串常量与 String 类型变量连接时得到的新字符串不再保存在常量池中，而是在堆中新建一个 String 对象来存放，很明显常量池中要求的存放的是常量，有String类型变量当然不能存在常量池中了。

对于 str14 到 str16 来说此处是字符串常量与 String 类型常量连接，得到的新字符串依然保存在常量池中。

对于 str17 到 str19 来说`final String str18 = getBB()`其实与`final String str18 = new String(“b”)`是一样的，也就是说 return “b” 会在堆中创建一个 String 对象保存 ”b”，虽然 str18 被定义成了 final，所以可见看见，并非定义为 final 的就保存在常量池中，很明显此处 str18 常量引用的 String 对象保存在堆中，因为 getBB() 得到的 String 已经保存在堆中了，final 的 String 引用并不会改变 String 已经保存在堆中这个事实。

**str9 到 str19 深刻的说明了我们在代码中使用 String 时应该有的优化技巧，你懂得！特别说明 String 的 + 和 += 在编译后实质被自动优化为了 StringBuilder 和 append 调用，但是如果在循环等情况下调用 + 或者 += 就是在不停的 new StringBuilder 对象 append 了。**

对于 str20 到 str23 来说`str23 == str20`就是上面刚刚分析的；而对于调用 intern 方法如果字符串常量池中已经包含一个等于此 String 对象的字符串（用 equals(Object) 方法确定）则返回字符串常量池中的字符串，否则将此 String 对象添加到字符串常量池中，并返回此 String 对象的引用，所以`str23.intern() == str20`实质是常量比较返回 true，`str23 == str20.intern()`中 str23 就是上面说的堆中新对象，相当于一个新对象和一个常量比较，所以返回 false，`str23.intern() == str20.intern()` 就没啥说的了，指定相等。

要玩明白 Java String 对象的核心其实就是玩明白字符串的堆栈和常量池，虚拟机为每个被装载的类型维护一个常量池，常量池就是该类型所用常量的一个有序集合，包括直接常量（String、Integer 和 Floating Point常量）和对其他类型、字段和方法的符号引用，池中的数据项就像数组一样是通过索引访问的；由于常量池存储了相应类型所用到的所有类型、字段和方法的符号引用，所以它在 Java 程序的动态链接中起着核心的作用。


### **18.什么是 java 泛型？泛型的核心原理是什么？泛型的好处是什么？请分别简单谈谈您的理解！**

1、Java 泛型实质就是一种语法约束，泛型是Java SE 1.5的新特性，泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

2、泛型的核心原理其实就是泛型的 T 类型参数的原理，Java 编译器在编译阶段会将泛型代码转换为普通的非泛型代码，实质就是擦除类型参数 T 替换为限定类型（无限定类型替换为Object）且插入必要的强制类型转换操作，所以核心其实就是 Test<String> 被擦除还原为 Test，同时 java 编译器是通过先检查代码中泛型的类型然后再进行类型擦除，最后进行编译的。
编译器在编译时擦除所有类型相关信息，所以在运行时不存在任何类型相关信息，例如`List<String>`在运行时仅用一个`List`来表示，这样做的目的是确保能和 Java 5 之前的版本开发二进制类库进行兼容，我们无法在运行时访问到类型参数，因为编译器已经把泛型类型转换成了原始类型。

3、泛型的好处其实就是约束，可以让我们编写的接口、类、方法等脱离具体类型限定而适用于更加广泛的类型，从而使代码达到低耦合、高复用、高可读、安全可靠的特点，避免主动向下转型带来的恶心可读性和隐晦的转换异常，尽可能的将类型问题暴露在 IDE 提示和编译阶段。

### **19.下面两个方法有什么区别，有什么问题？**

```java
public static <T> T get1(T t1, T t2) {  
	if(t1.compareTo(t2) >= 0);
	return t1;  
}  

public static <T extends Comparable> T get2(T t1, T t2) {
	if(t1.compareTo(t2) >= 0);  
	return t1;  
}
```
结果：

get1 方法直接编译错误，因为编译器在编译前首先进行了泛型检查和泛型擦除才编译，所以等到真正编译时 T 由于没有类型限定自动擦除为 Object 类型，所以只能调用 Object 的方法，而 Object 没有 compareTo 方法。

get2 方法添加了泛型类型限定可以正常使用，因为限定类型为 Comparable 接口，其存在 compareTo 方法，所以 t1、t2 擦除后被强转成功。所以类型限定在泛型类、泛型接口和泛型方法中都可以使用，不过不管该限定是类还是接口都使用 extends 和 & 符号，如果限定类型既有接口也有类则类必须只有一个且放在首位，如果泛型类型变量有多个限定则原始类型就用第一个边界的类型变量来替换。

### **20.下面程序的运行结果是什么？为什么？**

```java
ArrayList<String> arrayList1 = new ArrayList<String>();  
arrayList1.add("demo");  
ArrayList<Integer> arrayList2 = new ArrayList<Integer>();  
arrayList2.add(123);  
System.out.println(arrayList1.getClass() == arrayList2.getClass()); 


ArrayList<Integer> arrayList3 = new ArrayList<Integer>();  
arrayList3.add(123);  
arrayList3.getClass().getMethod("add", Object.class).invoke(arrayList3, "demo");  
for (int i=0;i<arrayList3.size();i++) {  
	System.out.println(arrayList3.get(i));  
}
```
结果：

第一个返回 true，因为 arrayList1 是`ArrayList<String>`泛型类型只能存储字符串，arrayList2 是`ArrayList<Integer>`泛型类型只能存储整形，而在编译后运行时泛型类型 String 和 Integer 都被擦除只剩下原始类型，所以都是 ArrayList 类，故相等。

第二个正常打印数字 123 和字符串 demo，因为 arrayList3 是`ArrayList<Integer>`泛型类型只能存储整形，但是编译擦除后成了 ArrayList，add 参数擦除替换为 Object，所以可以 add String 类型。

### **21.请尝试解释下面程序中每行代码执行情况及原因？**

```java
public class Test{
	public static <T> T add(T x, T y){  
        return y;
    }
  
    public static void main(String[] args) {
		int t0 = Test.add(10, 20.8);
		int t1 = Test.add(10, 20);
		Number t2 = Test.add(100, 22.2);
		Object t3 = Test.add(121, "abc");

		int t4 = Test.<Integer>add(10, 20);
		int t5 = Test.<Integer>add(100, 22.2);
		Number t6 = Test.<Number>add(121, 22.2);
    }
}  
```
结果：
```
t0 编译直接报错，add 的两个参数一个是 Integer，一个是 Float，所以取同一父类的最小级为 Number，故 T 为 Number 类型，而 t0 类型为 int，所以类型错误。
t1 执行赋值成功，add 的两个参数都是 Integer，所以 T 为 Integer 类型。
t2 执行赋值成功，add 的两个参数一个是 Integer，一个是 Float，所以取同一父类的最小级为 Number，故 T 为 Number 类型。 
t3 执行赋值成功，add 的两个参数一个是 Integer，一个是 Float，所以取同一父类的最小级为 Object，故 T 为 Object 类型。
t4 执行赋值成功，add 指定了泛型类型为 Integer，所以只能 add 为 Integer 类型或者其子类的参数。
t5 编译直接报错，add 指定了泛型类型为 Integer，所以只能 add 为 Integer 类型或者其子类的参数，不能为 Float。 
t6 执行赋值成功，add 指定了泛型类型为 Number，所以只能 add 为 Number 类型或者其子类的参数，Integer 和 Float 均为其子类，所以可以 add 成功。 
```
t0、t1、t2、t3 其实演示了调用泛型方法不指定泛型的几种情况，t4、t5、t6 演示了调用泛型方法指定泛型的情况。
在调用泛型方法的时可以指定泛型，也可以不指定泛型；在不指定泛型时泛型变量的类型为该方法中的几种类型的同一个父类的最小级（直到 Object），在指定泛型时该方法中的几种类型必须是该泛型实例类型或者其子类。

切记，java 编译器是通过先检查代码中泛型的类型，然后再进行类型擦除，再进行编译的。

### **22.请尝试解释下面程序中每行代码执行情况及原因？**

```java
ArrayList<String> arrayList1 = new ArrayList();		// 编译警告
arrayList1.add("1");	// 编译通过  
arrayList1.add(1);		// 编译错误
String str1 = arrayList1.get(0);		//返回类型就是 String  

ArrayList arrayList2 = new ArrayList<String>();  	// 编译警告
arrayList2.add("1");	// 编译通过  
arrayList2.add(1);		// 编译通过  
Object object = arrayList2.get(0);	// 返回类型就是 Object  

List<String> arrayList3 = new ArrayList<>();	//JDK7的新特性，会自动推断泛型
arrayList3.add("123");	//编译通过
arrayList3.add(123);	//编译错误
  
new ArrayList<String>().add("11");	// 编译通过
new ArrayList<String>().add(22);	// 编译错误  
String string = new ArrayList<String>().get(0); // 返回类型就是 String  

ArrayList<String> arrayList4 = new ArrayList<Object>();	// 编译错误  
ArrayList<Object> arrayList5 = new ArrayList<String>();	// 编译错误  
```

解释：

arrayList1 其实可以实现与完全使用泛型参数一样的效果，arrayList2 完全没了泛型特性，arrayList3 是 JDK 7 支持的中规中矩写法，由于类型检查就是针对引用的，与引用的对象无关，所以有了这些结果。

arrayList4 与 arrayList5 是因为泛型中参数化类型是不支持继承关系的，切记（注意与 arrayList1 和 arrayList2 对比理解）。至于不支持的原因如下：

对于 arrayList4 首先我们假设 `new ArrayList<Object>()` 申请的内存赋值给了引用类型是`ArrayList<Object>`或 ArrayList 的 temp，接着我们向 temp 中添加了几个元素（支持 add 任何类型），
然后让`ArrayList<String> arrayList4 = temp`，这时候调用 arrayList4 的 get 返回的都是 String 类型（类型检测是根据引用来决定的），锅来了，temp 中存了各种奇怪的类型啊，
arrayList4 拿回来直接用就可能出现 ClassCastException 异常啊，卧槽，Java 设计泛型的目的之一就是为了避免出现这种锅啊，所以 Java 直接不允许进行这样的引用传递，所以直接编译报错，扼杀在摇篮。

同理对于 arrayList5 来说类似 arrayList4，只不过最终 get 出来是从 String 转为 Object，不会出现 ClassCastException 异常，但是一样卧槽啊，我还得自己转，毛线，泛型出现之一也是为了解决这个锅啊，所以 Java 直接不允许进行这样的引用传递，所以直接编译报错，扼杀在摇篮。

### **23.请尝试解释下面程序中注释问题？**

```java
ArrayList<String> arrayList = new ArrayList<String>();    
if(arrayList instanceof ArrayList<String>) { //TODO }	//1.是否可以运行及结果原因
if(arrayList instanceof ArrayList<?>) { //TODO }	//2.是否可以运行及结果原因

public class Problem<T> extends Exception{ //TODO }		//3.是否可以运行及结果原因

public static <T extends Throwable> void test(Class<T> t) {  
	try{  
		//TODO 
	}catch(T e){	//4.是否可以运行及结果原因
		//TODO
	}
}

public static <T extends Throwable> void test1(T t) throws T {  
    try{  
        //TODO
    }catch(Throwable realCause){  
        t.initCause(realCause);  
        throw t;   			//5.是否可以运行及结果原因
    }
}

ArrayList listn = new ArrayList(); //已过时，取出来时需要自己强制转换，引入泛型代码之前的方式，在引入泛型后集合都已经重写以迎合泛型
```
解答：

1 编译报错，因为类型擦除之后，ArrayList<String> 只剩下原始类型，泛型信息String不存在了，所以进行类型查询的时候使用具体泛型类型是错误的。

2 可以运行，因为 java 限定了通配符这种类型查询的方式。

3 编译报错，因为不能出现继承异常的泛型类也不能捕获泛型类的异常对象，因为异常都是在运行时捕获和抛出的，而在编译的时候泛型信息全都会被擦除掉，所以多个 catcch 中可能会存在一样的原始类型捕获，这是不符合异常的语法。

4 编译报错，不可以在 catch 子句中使用泛型变量，因为泛型信息在编译的时候已经变为原始类型，也就是说上面的 T 会变为原始类型 Throwable，假设同时有多个 catch 则就不符合异常的语法。

5 可以运行，在异常声明中可以使用泛型类型变量。

### **24.请尝试解释下面程序编译到运行的现象及原因？**

```java
List<String>[] ls1 = new ArrayList<String>[10];	//1
List<String>[] ls2 = new ArrayList[10];	//2
List<?>[] lsa = new List<?>[10];	//3
```
结果：

1 编译错误。

2 正常运行，但是编译有警告。

3 正常运行。

因为 Java 规定数组的类型不可以是泛型类型变量，除非是采用通配符的方式。数组是允许把一个子类数组赋给一个父类数组变量的，譬如 Father 继承自 Son，则可以定义`Father[] son=new Son[10];`，如果 Java 允许泛型数组这会出现如下代码：
```java
List<String>[] ls1 = new ArrayList<String>[10];
Object[] oj = ls1;
```
也就是我们能在数组 ls1 里存放任何类的对象且能够通过编译，因为在编译阶段 ls1 被认为是一个Object[]，也就是 ls1 里面可以放一个 int、也可以放一个 String，当我们运行阶段取出里面的 int 并强制转换为 String 则会出现 ClassCastException，这明显违反了泛型引入的原则，所以 Java 不允许创建泛型数组。对于 2 可以编译通过但是会得到警告的解释其实是因为编译器确实不让我们实例化泛型数组，但是允许我们创建对这种数组的引用，所以我们可以创建非泛型数组转型给泛型引用。对于 3 通配符的方式，最后取出数据是要做显式的类型转换的，所以并不会存在上一个例子的问题。

提示：直接使用`ArrayList<ArrayList<String>>`最安全且有效。

### **25.请尝试解释下面程序编译到运行的现象及原因？**

```java
a1 = new T(); //1

public<T> T[] func(T[] a){  
	T[] a2 = new T[2]; //2  
	return a2;
}

public static <T extends Comparable> T[] func1(T[] a) {  
	T[] a3 == (T[]) Array.newInstance(a.getClass().getComponentType(), 2);	//3
	return a3;
}

class Bean<T super Student> { //TODO }	//4
```
解答：

1 编译时报错，不能实例化泛型类型，类型擦除会使这个操作做成 new Object()。

2 编译时报错，不能创建泛型数组，擦除会使这个方法构造一个 Object[2] 数组。

3 可以运行，可以用反射构造泛型对象和数组。

4 编译时报错，因为 Java 类型参数限定只有 extends 形式，没有 super 形式。

### **26.请尝试解释下面程序编译到运行的现象及原因？**

```java
public class Test1<T> {    
    public static T value;   //1
    public static  T test1(T param){ //2
        return null;    
    }    
}

public class Test2<T> {    
    public static <T> T test2(T param){	//3
        return null;
    }    
}

class Test<T> {  //4 这个类可以运行吗？
    public boolean equals(T value) {  
        return true;  
    }     
}
```
解答：

1 编译错误，2 编译错误，3 正常运行。

因为泛型类中的静态方法和静态变量不可以使用泛型类所声明的泛型类型参数，泛型类中的泛型参数的实例化是在定义对象的时候指定的，而静态变量和静态方法不需要使用对象来调用，对象都没创建，如何确定这个泛型参数是何种类型，所以当然是错误的；而 3 之所以成功是因为在泛型方法中使用的 T 是自己在方法中定义的 T，而不是泛型类中的 T。

4 无法编译通过，因为擦除后方法 boolean equals(T) 变成了方法 boolean equals(Object)，这与 Object.equals 方法是冲突的，除非重新命名引发错误的方法。

### **27.下面程序有什么问题，该如何修复？**

```java
public class Test {  
    public static void main(String[] args) throws Exception{  
        List<Integer> listInteger = new ArrayList<Integer>();   
        printCollection(listInteger);     
    }   
    public static void printCollection(Collection<Object> collection) {  
        for(Object obj:collection){  
            System.out.println(obj);  
        }    
    }  
}
```
解答：

语句`printCollection(listInteger);`编译报错，因为泛型的参数是没有继承关系的。
修复方式就是使用 ？通配符，`printCollection(Collection<?> collection)`，因为在方法`printCollection(Collection<?> collection)`中不可以出现与参数类型有关的方法，譬如`collection.add()`，因为程序调用这个方法的时候传入的参数不知道是什么类型的，但是可以调用与参数类型无关的方法，譬如 collection.size()。

### **28.下面程序编译运行会有什么现象？**

```java
Vector<? extends Number> x1 = new Vector<Integer>();	//正确
Vector<? extends Number> x2 = new Vector<String>();	//编译错误

Vector<? super Integer> y1 = new Vector<Number>();	//正确
Vector<? super Integer> y2 = new Vector<Byte>();	//编译错误
```
解释：

本题主要考察泛型中的`?`通配符的上下边界扩展问题。

通配符对于上边界有如下限制：`Vector<? extends 类型1> x = new Vector<类型2>();`中的类型1指定一个数据类型，则类型2就只能是类型1或者是类型1的子类。

通配符对于下边界有如下限制：`Vector<? super 类型1> x = new Vector<类型2>();`中的类型1指定一个数据类型，则类型2就只能是类型1或者是类型1的父类。

### **29.什么是泛型中的限定通配符和非限定通配符？**

解答：

限定通配符对类型进行了限制，有两种限定通配符，一种是`<? extends T>`通过确保类型必须是 T 的子类来设定类型的上界，另一种是`<? super T>`它通过确保类型必须是 T 的父类来设定类型的下界，泛型类型必须用限定内的类型来进行初始化，否则会导致编译错误。
另一方面`<?>`表示非限定通配符，因为`<?>`可以用任意类型来替代。

### **30.简单说说 Java 中`List<Object>`和原始类型`List`之间的区别？**

解答：

区别一：原始类型和带泛型参数类型`<Object>`之间的主要区别是在编译时编译器不会对原始类型进行类型安全检查，却会对带参数的类型进行检查，通过使用 Object 作为类型可以告知编译器该方法可以接受任何类型的对象（比如 String 或 Integer）。

区别二：我们可以把任何带参数的类型传递给原始类型 List，但却不能把`List<String>`传递给接受`List<Object>`的方法，因为会产生编译错误。

### **31.Java 中`List<?>`和`List<Object>`之间的区别是什么？**

解答：

这道题跟上一道题看起来很像，实质上却完全不同。`List<?>`是一个未知类型的`List`，而`List<Object>`其实是任意类型的`List`。我们可以把`List<String>`, `List<Integer>`赋值给`List<?>`，却不能把`List<String>`赋值给`List<Object>`。譬如：
```java
List<?> listOfAnyType;
List<Object> listOfObject = new ArrayList<Object>();
List<String> listOfString = new ArrayList<String>();
List<Integer> listOfInteger = new ArrayList<Integer>();
listOfAnyType = listOfString; //legal
listOfAnyType = listOfInteger; //legal
listOfObjectType = (List<Object>) listOfString; //compiler error – in-convertible types
```
所以通配符形式都可以用类型参数的形式来替代，通配符能做的用类型参数都能做。
通配符形式可以减少类型参数，形式上往往更为简单，可读性也更好，所以能用通配符的就用通配符。
如果类型参数之间有依赖关系或者返回值依赖类型参数或者需要写操作则只能用类型参数。

### **32.`List<? extends T>`和`List <? super T>`之间有什么区别？**

解答：

有时面试官会用这个问题来评估你对泛型的理解，而不是直接问你什么是限定通配符和非限定通配符，这两个`List`的声明都是限定通配符的例子，`List<? extends T>`可以接受任何继承自 T 的类型的`List`，而`List<? super T>`可以接受任何 T 的父类构成的`List`。
例如`List<? extends Number>`可以接受`List<Integer>`或`List<Float>`。Java 容器类的实现中有很多这种用法，比如 Collections 中就有如下一些方法：
```java
public static <T extends Comparable<? super T>> void sort(List<T> list)
public static <T> void sort(List<T> list, Comparator<? super T> c)
public static <T> void copy(List<? super T> dest, List<? extends T> src)
public static <T> T max(Collection<? extends T> coll, Comparator<? super T> comp)
```

### **33.`<T extends E>`和`<? extends E>`有什么关系？**

解答：

它们用的地方不一样，`<T extends E>`用于定义类型参数，声明了一个类型参数 T，可放在泛型类定义中类名后面、泛型方法返回值前面。
`<? extends E>`用于实例化类型参数，用于实例化泛型变量中的类型参数，只是这个具体类型是未知的，只知道它是 E 或 E 的某个子类型。
虽然它们不一样，但两种写法经常可以达到相同的目的，譬如：
```java
public void addAll(Bean<? extends E> c)
public <T extends E> void addAll(Bean<T> c) 
```

### **34.解释下面程序执行情况和原因？**

```java
DynamicArray<Integer> ints = new DynamicArray<>();
DynamicArray<? extends Number> numbers = ints; 
Integer a = 200;
numbers.add(a);		//这三行add现象？
numbers.add((Number)a);
numbers.add((Object)a);

public void copyTo(DynamicArray<? super E> dest){
    for(int i=0; i<size; i++){
        dest.add(get(i));	//这行add现象？
    }
}
```
解析：

上面三种 add 方法都是非法的，无论是 Integer，还是 Number 或 Object，编译器都会报错。因为`?`表示类型安全无知，`? extends Number`表示是Number的某个子类型，但不知道具体子类型，
如果允许写入，Java 就无法确保类型安全性，所以直接禁止。

最后方法的 add 是合法的，因为`<? super E>`形式与`<? extends E>`正好相反，超类型通配符表示 E 的某个父类型，有了它我们就可以更灵活的写入了。

### **35.Arraylist 的动态扩容机制是如何自动增加的？简单说说你理解的增加流程！**

解析：

当在 ArrayList 中增加一个对象时 Java 会去检查 Arraylist 以确保已存在的数组中有足够的容量来存储这个新对象，如果没有足够容量就新建一个长度更长的数组（原来的1.5倍），旧的数组就会使用 Arrays.copyOf 方法被复制到新的数组中去，现有的数组引用指向了新的数组。下面代码展示为 `Java 1.8` 中通过 `ArrayList.add` 方法添加元素时，内部会自动扩容，扩容流程如下：
```java
//确保容量够用，内部会尝试扩容，如果需要
ensureCapacityInternal(size + 1)

//在未指定容量的情况下，容量为DEFAULT_CAPACITY = 10
//并且在第一次使用时创建容器数组，在存储过一次数据后，数组的真实容量至少DEFAULT_CAPACITY
 private void ensureCapacityInternal(int minCapacity) {
    //判断当前的元素容器是否是初始的空数组
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        //如果是默认的空数组，则 minCapacity 至少为DEFAULT_CAPACITY
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }

    ensureExplicitCapacity(minCapacity);
}

//通过该方法进行真实准确扩容尝试的操作
 private void ensureExplicitCapacity(int minCapacity) {
        modCount++;//记录List的结构修改的次数

        //需要扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
}

//扩容操作
 private void grow(int minCapacity) {
    //原来的容量
    int oldCapacity = elementData.length;
    
    //新的容量 = 原来的容量 + （原来的容量的一半）
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    
    //如果计算的新的容量比指定的扩容容量小，那么就使用指定的容量
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    
    //如果新的容量大于MAX_ARRAY_SIZE(Integer.MAX_VALUE - 8)
    //那么就使用hugeCapacity进行容量分配
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = (minCapacity > MAX_ARRAY_SIZE) ? Integer.MAX_VALUE : MAX_ARRAY_SIZE;
    
    //创建长度为newCapacity的数组，并复制原来的元素到新的容器，完成ArrayList的内部扩容
    elementData = Arrays.copyOf(elementData, newCapacity);
 }
```

### **36.下面这些方法可以正常运行吗？为什么？**
```java
public void remove1(ArrayList<Integer> list) {
    for(Integer a : list){
        if(a <= 10){
            list.remove(a);
        }
    }
}

public static void remove2(ArrayList<Integer> list) {
    Iterator<Integer> it = list.iterator();
    while(it.hasNext()){
        if(it.next() <= 10) {
            it.remove();
        }
    }
}

public static void remove3(ArrayList<Integer> list) {
    Iterator<Integer> it = list.iterator();
    while(it.hasNext()) {
        it.remove();    
    }
}

public static void remove4(ArrayList<Integer> list) {
    Iterator<Integer> it = list.iterator();
    while(it.hasNext()) {
        it.next();
        it.remove();
        it.remove();
    }
}
```

解析：

remove1 方法会抛出 ConcurrentModificationException 异常，这是迭代器的一个陷阱，foreach 遍历编译后实质会替换为迭代器实现（普通for循环不会抛这个异常，因为list.size方法一般不会变，所以只会漏删除），因为迭代器内部会维护一些索引位置数据，要求在迭代过程中容器不能发生结构性变化（添加、插入、删除，修改数据不算），否则这些索引位置数据就失效了，避免的方式就是使用迭代器的 remove 方法。

remove2 方法可以正常运行，无任何错误。

remove3 方法会抛出 IllegalStateException 异常，因为使用迭代器的 remove 方法前必须先调用 next 方法，next 方法会检测容器是否发生了结构性变化，然后更新 cursor 和 lastRet 值，直接不调用 next 而 remove 会导致相关值不正确。

remove4 方法会抛出 IllegalStateException 异常，理由同 remove3，remove 调用一次后 lastRet 值会重置为 -1，没有调用 next 去设置 lastRet 的情况下再直接调一次 remove 自然就状态异常了。

当然了，上面四个写法的具体官方解答可参见 ArrayList 中迭代器部分源码，如下：
```java
private class Itr implements Iterator<E> {
	int cursor;       // index of next element to return
	int lastRet = -1; // index of last element returned; -1 if no such
	int expectedModCount = modCount;

	public boolean hasNext() {
		return cursor != size;
	}

	@SuppressWarnings("unchecked")
	public E next() {
		checkForComodification();
		int i = cursor;
		if (i >= size)
			throw new NoSuchElementException();
		Object[] elementData = ArrayList.this.elementData;
		if (i >= elementData.length)
			throw new ConcurrentModificationException();
		cursor = i + 1;
		return (E) elementData[lastRet = i];
	}

	public void remove() {
		if (lastRet < 0)
			throw new IllegalStateException();
		checkForComodification();

		try {
			ArrayList.this.remove(lastRet);
			cursor = lastRet;
			lastRet = -1;
			expectedModCount = modCount;
		} catch (IndexOutOfBoundsException ex) {
			throw new ConcurrentModificationException();
		}
	}

	final void checkForComodification() {
		if (modCount != expectedModCount)
			throw new ConcurrentModificationException();
	}
}
```

### **37.简要解释下面程序的执行现象和结果？**
```java
ArrayList<Integer> list = new ArrayList<Integer>();
list.add(1);
list.add(2);
list.add(3);
Integer[] array1 = new Integer[3];
list.toArray(array1);
Integer[] array2 = list.toArray(new Integer[0]);
System.out.println(Arrays.equals(array1, array2));	//1 结果是什么？为什么？

Integer[] array = {1, 2, 3};
List<Integer> list = Arrays.asList(array);
list.add(4);	//2 结果是什么？为什么？

Integer[] array = {1, 2, 3};
List<Integer> list = new ArrayList<Integer>(Arrays.asList(array));
list.add(4);	//3 结果是什么？为什么？
```

解析：

1 输出为 true，因为 ArrayList 有两个方法可以返回数组`Object[] toArray()`和`<T> T[] toArray(T[] a)`，第一个方法返回的数组是通过 Arrays.copyOf 实现的，第二个方法如果参数数组长度足以容纳所有元素就使用参数数组，否则新建一个数组返回，所以结果为 true。

2 会抛出 UnsupportedOperationException 异常，因为 Arrays 的 asList 方法返回的是一个 Arrays 内部类的 ArrayList 对象，这个对象没有实现 add、remove 等方法，只实现了 set 等方法，所以通过 Arrays.asList 转换的列表不具备结构可变性。

3 当然可以正常运行咯，不可变结构的 Arrays 的 ArrayList 通过构造放入了真正的万能 ArrayList，自然就可以操作咯。

### **38.简单解释一下 Collection 和 Collections 的区别？**

解析：

java.util.Collection 是一个集合接口，它提供了对集合对象进行基本操作的通用接口方法，在 Java 类库中有很多具体的实现，意义是为各种具体的集合提供最大化的统一操作方式。
譬如 Collection 的实现类有 List、Set 等，List 的实现类有 LinkedList、ArrayList、Vector 等，Vector 的实现类有 Stack 等，不过切记 Map 是自立门户的，其提供了转换为 Collection 的方法，但是自己不是 Collection 的子类。
 
java.util.Collections 是一个包装类，它包含有各种有关集合操作的静态多态方法，此类构造 private 不能实例化，就像一个工具类，服务于 Java 的 Collection 框架，其提供的方法大概可以分为对容器接口对象进行操作类（查找和替换、排序和调整顺序、添加和修改）和返回一个容器接口对象类（适配器将其他类型的数据转换为容器接口对象、装饰器修饰一个给定容器接口对象增加某种性质）。

### **39.解释一下 ArrayList、Vector、Stack、LinkedList 的实现和区别及特点和适用场景？**

解析：

首先他们都是 List 家族的儿子，List 又是 Collection 的子接口，Collection 又是 Iterable 的子接口，所以他们都具备 Iterable 和 Collection 和 List 的基本特性。

ArrayList 是一个动态数组队列，随机访问效率高，随机插入、删除效率低。LinkedList 是一个双向链表，它也可以被当作堆栈、队列或双端队列进行操作，随机访问效率低，但随机插入、随机删除效率略好。Vector 是矢量队列，和 ArrayList 一样是一个动态数组，但是 Vector 是线程安全的。Stack 继承于 Vector，特性是先进后出(FILO, FirstIn Last Out)。

从线程安全角度看 Vector、Stack 是线程安全的，ArrayList、LinkedList 是非线程安全的。

从实现角度看 LinkedList 是双向链表结构，ArrayList、Vector、Stack 是内存数组结构。

从动态扩容角度看由于 ArrayList 和 Vector（Stack 继承自 Vector，只在 Vector 的基础上添加了几个 Stack 相关的方法，故之后不再对 Stack 做特别的说明）使用数组实现，当数组长度不够时，其内部会创建一个更大的数组，然后将原数组中的数据拷贝至新数组中，而 LinkedList 是双向链表结构，内存不用连续，所以用多少申请多少。

从效率方面来说 Vector、ArrayList、Stack 是基于数组实现的，是根据索引来访问元素，Vector（Stack）和 ArrayList 最大的区别就是 synchronization 同步的使用，抛开两个只在序列化过程中使用的方法不说，没有一个 ArrayList 的方法是同步的，相反，绝大多数 Vector（Stack）的方法法都是直接或者间接的同步的，因此就造成 ArrayList 比 Vector（Stack）更快些，不过在最新的 JVM 中，这两个类的速度差别是很小的，几乎可以忽略不计；而 LinkedList 是双向链表实现，根据索引访问元素时需要遍历寻找，性能略差。所以 ArrayList 适合大量随机访问，LinkList 适合频繁删除插入操作。

从差异角度看 LinkedList 还具备 Deque 双端队列的特性，其实现了 Deque 接口，Deque 继承自 Queue 队列接口，其实也挺好理解，因为 LinkedList 是的实现是双向链表结构，所以实现队列特性实在是太容易了。

### **40.简单介绍下 List 、Map、Set、Queue 的区别和关系？**

解析：

List、Set、Queue 都继承自 Collection 接口，而 Map 则不是（继承自 Object），所以容器类有两个根接口，分别是 Collection 和 Map，Collection 表示单个元素的集合，Map 表示键值对的集合。

List 的主要特点就是有序性和元素可空性，他维护了元素的特定顺序，其主要实现类有 ArrayList 和 LinkList。ArrayList 底层由数组实现，允许元素随机访问，但是向 ArrayList 列表中间插入删除元素需要移位复制速度略慢；LinkList 底层由双向链表实现，适合频繁向列表中插入删除元素，随机访问需要遍历所以速度略慢，适合当做堆栈、队列、双向队列使用。

Set 的主要特性就是唯一性和元素可空性，存入 Set 的每个元素都必须唯一，加入 Set 的元素都必须确保对象的唯一性，Set 不保证维护元素的有序性，其主要实现类有 HashSet、LinkHashSet、TreeSet。HashSet 是为快速查找元素而设计，存入 HashSet 的元素必须定义 hashCode 方法，其实质可以理解为是 HashMap 的包装类，_所以 HashSet 的值还具备可 null 性_；LinkHashSet 具备 HashSet 的查找速度且通过链表保证了元素的插入顺序（实质为 HashSet 的子类），迭代时是有序的，同理存入 LinkHashSet 的元素必须定义 hashCode 方法；TreeSet 实质是 TreeMap 的包装类，_所以 TreeSet 的值不备可 null 性_，其保证了元素的有序性，底层为红黑树结构，存入 TreeSet 的元素必须实现 Comparable 接口；不过特别注意 EnumSet 的实现和 EnumMap 没有一点关系。

Queue 的主要特性就是队列和元素不可空性，其主要的实现类有 LinkedList、PriorityQueue。LinkedList 保证了按照元素的插入顺序进行操作；PriorityQueue 按照优先级进行插入抽取操作，元素可以通过实现 Comparable 接口来保证优先顺序。Deque 是 Queue 的子接口，表示更为通用的双端队列，有明确的在头或尾进行查看、添加和删除的方法，ArrayDeque 基于循环数组实现，效率更高一些。

Map 自立门户，但是也提供了嫁接到 Collection 相关方法，其主要特性就是维护键值对关联和查找特性，其主要实现类有 HashTab、HashMap、LinkedHashMap、TreeMap。HashTab 类似 HashMap，_但是不允许键为 null 和值为 null_，比 HashMap 慢，因为为同步操作；HashMap 是基于散列列表的实现，_其键和值都可以为 null_；LinkedHashMap 类似 HashMap，_其键和值都可以为 null_，其有序性为插入顺序或者最近最少使用的次序（LRU 算法的核心就是这个），之所以能有序，是因为每个元素还加入到了一个双向链表中；TreeMap 是基于红黑树算法实现的，查看键值对时会被排序，存入的元素必须实现 Comparable 接口，_但是不允许键为 null，值可以为 null_；如果键为枚举类型可以使用专门的实现类 EnumMap，它使用效率更高的数组实现。

从数据结构角度看集合的区别有如下：

动态数组：ArrayList 内部是动态数组，HashMap 内部的链表数组也是动态扩展的，ArrayDeque 和 PriorityQueue 内部也都是动态扩展的数组。

链表：LinkedList 是用双向链表实现的，HashMap 中映射到同一个链表数组的键值对是通过单向链表链接起来的，LinkedHashMap 中每个元素还加入到了一个双向链表中以维护插入或访问顺序。

哈希表：HashMap 是用哈希表实现的，HashSet, LinkedHashSet 和 LinkedHashMap 基于 HashMap，内部当然也是哈希表。

排序二叉树：TreeMap 是用红黑树(基于排序二叉树)实现的，TreeSet 内部使用 TreeMap，当然也是红黑树，红黑树能保持元素的顺序且综合性能很高。

堆：PriorityQueue 是用堆实现的，堆逻辑上是树，物理上是动态数组，堆可以高效地解决一些其他数据结构难以解决的问题。

循环数组：ArrayDeque 是用循环数组实现的，通过对头尾变量的维护，实现了高效的队列操作。

位向量：EnumSet 是用位向量实现的，对于只有两种状态且需要进行集合运算的数据使用位向量进行表示、位运算进行处理，精简且高效。

### **41.简单说说 HashMap 的底层原理？**

答案：

当我们往 HashMap 中 put 元素时，先根据 key 的 hash 值得到这个元素在数组中的位置（即下标），然后把这个元素放到对应的位置中，如果这个元素所在的位子上已经存放有其他元素就在同一个位子上的元素以链表的形式存放，新加入的放在链头，从 HashMap 中 get 元素时先计算 key 的 hashcode，找到数组中对应位置的某一元素，然后通过 key 的 equals 方法在对应位置的链表中找到需要的元素，所以 HashMap 的数据结构是数组和链表的结合。

解析：

HashMap 底层是基于哈希表的 Map 接口的非同步实现，实际是一个链表散列数据结构（即数组和链表的结合体）。
首先由于数组存储区间是连续的，占用内存严重，故空间复杂度大，但二分查找时间复杂度小（O(1)），所以寻址容易，插入和删除困难。而链表存储区间离散，占用内存比较宽松，故空间复杂度小，但时间复杂度大（O(N)），所以寻址困难，插入和删除容易。
所以就产生了一种新的数据结构------哈希表，哈希表既满足了数据的查找方便，同时不占用太多的内容空间，使用也十分方便，哈希表有多种不同的实现方法，HashMap 采用的是链表的数组实现方式，具体如下：

首先 HashMap 里面实现了一个静态内部类 Entry(key、value、next)，HashMap 的基础就是一个 Entry[] 线性数组，Map 的内容都保存在 Entry[] 里面，而 HashMap 用的线性数组却是随机存储的原因如下：
```java
// 存储时
int hash = key.hashCode(); //每个 key 的 hash 是一个固定的 int 值 
int index = hash % Entry[].length; 
Entry[index] = value;

// 取值时
int hash = key.hashCode(); 
int index = hash % Entry[].length; 
return Entry[index];
```
当我们通过 put 向 HashMap 添加多个元素时会遇到两个 key 通过`hash % Entry[].length`计算得到相同 index 的情况，这时具有相同 index 的元素就会被放在线性数组 index 位置，然后其 next 属性指向上个同 index 的 Entry 元素形成链表结构（譬如第一个键值对 A 进来，通过计算其 key 的 hash 得到的 index = 0，记做 Entry[0] = A，接着第二个键值对 B 进来，通过计算其 index 也等于 0，这时候 B.next = A, Entry[0] = B，如果又进来 C 且 index 也等于 0 则 C.next = B, Entry[0] = C）。
当我们通过 get 从 HashMap 获取元素时首先会定位到数组元素，接着再遍历该元素处的链表获取真实元素。
当 key 为 null 时 HashMap 特殊处理总是放在 Entry[] 数组的第一个元素。
HashMap 使用 Key 对象的 hashCode() 和 equals() 方法去决定 key-value 对的索引，当我们试着从 HashMap 中获取值的时候，这些方法也会被用到，所以 equals() 和 hashCode() 的实现应该遵循以下规则：
如果`o1.equals(o2)`则`o1.hashCode() == o2.hashCode()`必须为 true，或者如果`o1.hashCode() == o2.hashCode()`则不意味着`o1.equals(o2)`会为true，这也就是衍生问题 Java 中重写 equals 方法的同时为什么要重写 hashCode 方法的答案。

关于 HashMap 的 hash 函数算法巧妙之处可以参见本文链接：http://pengranxiang.iteye.com/blog/543893

### **42.简单解释一下 Comparable 和 Comparator 的区别和场景？**

解析：

Comparable 对实现它的每个类的对象进行整体排序，这个接口需要类本身去实现，若一个类实现了 Comparable 接口，实现 Comparable 接口的类的对象的 List 列表(或数组)可以通过 Collections.sort（或 Arrays.sort）进行排序，此外实现 Comparable 接口的类的对象可以用作有序映射(如TreeMap)中的键或有序集合(如TreeSet)中的元素，而不需要指定比较器，
实现 Comparable 接口必须修改自身的类（即在自身类中实现接口中相应的方法），如果我们使用的类无法修改（如SDK中一个没有实现Comparable的类），我们又想排序，就得用到 Comparator 这个接口了（策略模式）。
所以如果你正在编写一个值类，它具有非常明显的内在排序关系，比如按字母顺序、按数值顺序或者按年代顺序，那你就应该坚决考虑实现 Comparable 这个接口，
若一个类实现了 Comparable 接口就意味着该类支持排序，而 Comparator 是比较器，我们若需要控制某个类的次序，可以建立一个该类的比较器来进行排序。
Comparable 比较固定，和一个具体类相绑定，而 Comparator 比较灵活，可以被用于各个需要比较功能的类使用。

### **43.简单说说 Iterator 和 ListIterator 的区别？**

解析：

ListIterator 有 add() 方法，可以向 List 中添加对象，而 Iterator 不能。

ListIterator 和 Iterator 都有 hasNext() 和 next() 方法，可以实现顺序向后遍历，但是 ListIterator 有 hasPrevious() 和 previous() 方法，可以实现逆向（顺序向前）遍历，Iterator 就不可以。

ListIterator 可以定位当前的索引位置，通过 nextIndex() 和 previousIndex() 可以实现，Iterator 没有此功能。

都可实现删除对象，但是 ListIterator 可以实现对象的修改，通过 set() 方法可以实现，Iierator 仅能遍历，不能修改。

容器类提供的迭代器都会在迭代中间进行结构性变化检测，如果容器发生了结构性变化，就会抛出 ConcurrentModificationException，所以不能在迭代中间直接调用容器类提供的 add、remove 方法，如需添加和删除，应调用迭代器的相关方法。

### **44.请实现一个极简 LRU 算法容器？**

解析：

看起来是一道很难的题目，其实静下来你会发现想考察的其实就是 LRU 的原理和 LinkedHashMap 容器知识，当然，你要是厉害不依赖 LinkedHashMap 自己纯手写撸一个也不介意。
LinkedHashMap 支持插入顺序或者访问顺序，LRU 算法其实就要用到它访问顺序的特性，即对一个键执行 get、put 操作后其对应的键值对会移到链表末尾，所以最末尾的是最近访问的，最开始的最久没被访问的。
LRU 是一种流行的替换算法，它的全称是 Least Recently Used，最近最少使用，它的思路是最近刚被使用的很快再次被用的可能性最高，而最久没被访问的很快再次被用的可能性最低，所以被优先清理。
下面给出极简 LRU 缓存算法容器：
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private int maxEntries;
    
	//maxEntries 最大缓存个数
    public LRUCache(int maxEntries){
        super(16, 0.75f, true);
        this.maxEntries = maxEntries;
    }
    
	//在添加元素到 LinkedHashMap 后会调用这个方法，传递的参数是最久没被访问的键值对，如果这个方法返回 true 则这个最久的键值对就会被删除，LinkedHashMap 的实现总是返回 false，所有容量没有限制。
    @Override
    protected boolean removeEldestEntry(Entry<K, V> eldest) {
        return size() > maxEntries;
    }
}   
```

### **45.简单说说你对 Java 的 transient 关键字理解？**

解析：

我们都知道一个对象只要实现了 Serilizable 接口就可以被序列化了，java 的这种序列化模式使得我们可以不必关心具体序列化的过程，只要这个类实现了 Serilizable 接口则类的所有属性和方法都会自动序列化，而实际开发中我们常常会遇到这样的问题，这个类的有些属性需要序列化，而其他属性不需要被序列化，对于不要被序列化的属性就可以加上 transient 关键字。其主要的特性如下：

一旦变量被 transient 修饰就不再是对象持久化的一部分，该变量内容在序列化后无法获得访问；transient 关键字只能修饰变量而不能修饰方法和类（注意本地变量是不能被 transient 关键字修饰的）；变量如果是用户自定义类变量，则该类需要实现 Serializable 接口；被 transient 关键字修饰的变量不再能被序列化，一个静态变量不管是否被 transient 修饰均不能被序列化（一个静态变量不管是否被 transient 修饰均不能被序列化，反序列化后类中 static 型变量的值为当前 JVM 中对应 static 变量的值，这个值是 JVM 中的不是反序列化得出的）；在 Java 中对象的序列化可以通过实现两种接口来实现，若实现的是 Serializable 接口则所有的序列化将会自动进行，若实现的是 Externalizable 接口则没有任何东西可以自动序列化，需要在 writeExternal 方法中进行手工指定所要序列化的变量，这与是否被 transient 修饰无关。

### **46.简单谈谈你所知道的 Java IO 流特性？**

解析：

java.io 包下面的流基本都是装饰器模式的实现，提供了各种类型流操作的便携性，常见的流分类如下：
```
以二进制字节方式读写的流（字节流没有编码的概念，转换字节需要考虑编码，不能按行处理，使用不太方便）主要有：

InputStream、OutputStream: 二进制字节读写抽象类型的基类。

FileInputStream、FileOutputStream: 输入输出目标是文件的流，构造支持 File 类型和 String 文件名类型及追加覆盖模式，以 byte 或 byte 数组读写文件，FileOutputStream 没有缓冲，没有重写flush，调用 flush 没有任何效果，数据只是传递给了操作系统，但操作系统什么时候保存到硬盘上是不一定的，按字节读取效率低。

ByteArrayInputStream、ByteArrayOutputStream: 输入输出目标是字节数组的流，数组的长度是根据数据内容动态扩展的，ByteArrayOutputStream 无缓存，同理 flush 无效。

DataInputStream、DataOutputStream: 按基本类型和字符串而非只是字节读写流装饰类，FilterInputStream、FilterOutputStream 装饰基类的子类，在写入时 DataOutputStream 会将这些类型的数据转换为其对应的二进制字节，必须按照字节读取，效率较低。

BufferedInputStream、BufferedOutputStream: 对输入输出流提供缓冲功能的装饰类，BufferedInputStream 内部有个字节数组作为缓冲区，读取时先从这个缓冲区读，缓冲区读完了再调用包装的流读，BufferedOutputStream 的构造方法也有两个，默认的缓冲区大小也是 8192，它的 flush 方法会将缓冲区的内容写到包装的流中。

PipedInputStream、PipedOutputStream：分别是管道输入输出流，作用是让多线程可以通过管道进行线程间的通讯，在使用管道通信时，必须将 PipedOutputStream 和 PipedInputStream 配套使用。

PrintStream：继承于 FilterOutputStream 来装饰其它输出流的流。提供了自动 flush 和字符集设置功能。

以文本字符方式读写的流（文本文件中编码非常重要）主要有：

Reader、Writer：字符流的抽象基类。

InputStreamReader、OutputStreamWriter：适配器类，输入是 InputStream，输出是 OutputStream，将字节流转换为字符流，一个重要的参数是编码类型，如果没有传则为系统默认编码。

FileReader、FileWriter：输入输出目标是文件的字符流，InputStreamReader、OutputStreamWriter 的子类，需要注意的是 FileReader、FileWriter 不能指定编码类型，只能使用默认编码，如果需要指定编码类型可以使用 InputStreamReader、OutputStreamWriter。

CharArrayReader、CharArrayWriter: 输入输出目标是 char 动态调整数组的字符流，类似 ByteArrayInputStream、ByteArrayOutputStream。

StringReader、StringWriter：输入输出目标是 String 的字符流，与 CharArrayReader、CharArrayWriter 类似。

BufferedReader、BufferedWriter：装饰类，对输入输出流提供缓冲以及按行读写功能，FileReader、FileWriter 是没有缓冲的、也不能按行读写，所以一般应该在它们的外面包上对应的缓冲类。

PrintWriter：装饰类，可直接指定文件名作为参数、可以指定编码类型、可以自动缓冲、可以自动将多种类型转换为字符串，在输出到文件时可以优先选择该类。

PipedReader、PipedWriter：分别是字符管道输入输出流，作用是让多线程可以通过管道进行线程间的通讯，在使用管道通信时，必须将 PipedReader、PipedWriter 配套使用。
```

### **47.请简单谈谈你对 Java 的序列化与反序列化理解？**

解析：

序列化就是将对象转化为字节流，反序列化就是将字节流转化为对象，要让一个类支持序列化只要让这个类实现接口 java.io.Serializable 即可，Serializable 只是一个没有定义任何方法的标记接口，声明实现 Serializable 接口后保存读取对象就可以使用 ObjectOutputStream、ObjectInputStream 流了，ObjectOutputStream 是 OutputStream 的子类，但实现了 ObjectOutput 接口，ObjectOutput 是 DataOutput 的子接口，增加了一个 writeObject(Object obj) 方法将对象转化为字节写到流中，ObjectInputStream 是 InputStream 的子类，实现了ObjectInput 接口，ObjectInput 是 DataInput 的子接口，增加了一个 readObject() 方法从流中读取字节转为对象，序列化和反序列化的实质在于 ObjectOutputStream 的 writeObject 和 ObjectInputStream 的 readObject 方法实现，常见的 String、Date、Double、ArrayList、LinkedList、HashMap、TreeMap 等都默认实现了 Serializable。

有时候我们对象有些字段的值可能与内存位置（hashcode）、当前时间等有关，所以我们不想序列化他，因为反序列化后的值是没有意义的，或者有时候如果类中的字段表示的是类的实现细节而非逻辑信息则默认序列化也是不适合的，所以我们需要定制序列化。Java 提供的定制主要有 transient 关键字方式和实现 writeObject、readObject 方式及 Externalizable 接口 readResolve、writeReplace 方式，还可以将字段声明为 transient 后通过 writeObject、readObject 方法来自己保存该字段。

默认情况下 Java 会根据类中一系列信息自动生成一个版本号，在反序列化时如果类的定义发生了变化版本号就会变化，也就与反序列化流中的版本号不匹配导致会抛出异常，所以我们为了更好的控制和性能问题会自定义 serialVersionUID 版本号来避免类定义发生变化后反序列化版本号不匹配异常问题，如果版本号一样时流中有该字段而类定义中没有则该字段会被忽略，如果类定义中有而流中没有则该字段会被设为默认值，如果对于同名的字段类型变了则会抛出 InvalidClassException。

### **48.Java 线程优先级是怎么定义的，Java 线程有几种状态？**

解析：

Java 线程的优先级定义为从 1 到 10 的等级，默认为 5，设置和获取线程优先级的方法是 setPriority(int newPriority) 和 getPriority()，Java 的这个优先级会被映射到操作系统中线程的优先级，不过由于操作系统各不相同，不一定都是 10 个优先级，所以 Java中 不同的优先级可能会被映射到操作系统中相同的优先级，同时优先级对操作系统而言更多的是一种建议和提示而非强制，所以我们不要过于依赖优先级。

Java Thread 可以通过 State getState() 来获取线程状态，Thread.State 枚举定义了 NEW、RUNNABLE、BLOCKED、WAITING、TIMED_WAITING、TERMINATED 六种线程状态，其实真正严格意义来说线程只有就绪、阻塞、运行三种状态，Java 线程之所以有六种状态其实是站在 Thread 对象实例的角度来看待的，具体解释如下：

NEW（新建），表示线程 Thread 刚被创建，还没调用 start 方法。

RUNNABLE（运行，实质对应就绪和运行状态），表示 Thread 线程正在 JVM 中运行，也就是说处于就绪和运行状态的线程在 Thread 中都表现为 RUNNABLE。

BLOCKED（阻塞，实质对应阻塞状态），表示等待监视锁可以重新进行同步代码块中执行，此线程需要获得某个锁才能继续执行，而这个锁目前被其他线程持有，所以进入了被动的等待状态，直到抢到了那个锁才会再次进入就绪状态。处于受阻塞状态的某一线程正在等待监视器锁，以便进入一个同步的块或方法，或者在调用 wait 之后再次进入同步的块或方法。

WAITING（等待，实质对应阻塞状态），表示此线程正处于无限期的主动等待中，直到有人唤醒它它才会再次进入就绪状态。某一线程因为调用下不带超时值的 wait、不带超时值的 join、LockSupport.park 会进入等待状态，处于等待状态的线程正等待另一个线程以执行特定操作，例如已经在某一对象上调用了 Object.wait() 的线程正等待另一个线程以便在该对象上调用 Object.notify() 或 Object.notifyAll()，或者已经调用了 Thread.join() 的线程正在等待指定线程终止。

TIMED_WAITING（有限等待，实质对应阻塞状态），表示此线程正处于有限期的主动等待中，要么有人唤醒它，要么等待够了一定时间之后才会再次进入就绪状态，譬如调用带有超时的 sleep、join、wait 方法可能导致线程处于等待状态。

TERMINATED（终止），表示线程执行完毕，已经退出。

### **49.简单说说你对 Java synchronized 的理解？**

解析：

synchronized 关键字主要用来解决多线程共享内存的并发同步问题，可以用来修饰类的实例方法、静态方法、代码块；synchronized 实例方法实际保护的是同一个对象的方法调用，当为不同对象时多线程是可以同时访问同一个 synchronized 方法的；synchronized 静态方法和 synchronized 实例方法保护的是不同对象，不同的两个线程可以同时一个执行 synchronized 静态方法，另一个执行 synchronized 实例方法，因为 synchronized 静态方法保护的是 class 类对象，synchronized 实例方法保护的是 this 实例对象；synchronized 代码块同步的可以是任何对象，因为任何对象都有一个锁和等待队列。

synchronized 具备可重入性，对同一个线程在获得锁之后在调用其他需要同样锁的代码时可以直接调用，其可重入性是通过记录锁的持有线程和持有数量来实现的，调用 synchronized 代码时检查对象是否已经被锁，是则检查是否被当前线程锁定，是则计数加一，不是则加入等待队列，释放时计数减一直到为零释放锁。

synchronized 还具备内存可见性，除了实现原子操作避免竞态以外对于明显是原子操作的方法（譬如一个 boolean 状态变量 state 的 get 和 set 方法）也可以通过 synchronized 来保证并发的可见性，在释放锁时所有写入都会写回内存，而获得锁后都会从内存读取最新数据；不过对于已经是原子性的操作为了保证内存可见性而使用 synchronized 的成本会比较高，轻量级的选择应该是使用 volatile 修饰，一旦修饰 java 就会在操作对应变量时插入特殊指令保证可见性。

synchronized 是重量级锁，其语义底层是通过一个 monitor 监视器对象来完成，其实 wait、notify 等方法也依赖于 monitor 对象，所以这就是为什么只有在同步的块或者方法中才能调用 wait、notify 等方法，否则会抛出 IllegalMonitorStateException 异常的原因，监视器锁（monitor）的本质依赖于底层操作系统的互斥锁（Mutex Lock）实现，而操作系统实现线程之间的切换需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，所以这就是为什么 synchronized 效率低且重量级的原因（Java 1.6 进行了优化，但是相比其他锁机制还是略显偏重）。

synchronized 在发生异常时会自动释放线程占用的锁资源，Lock 需要在异常时主动释放，synchronized 在锁等待状态下无法响应中断而 Lock 可以。

还可以参考 JVM 官方文档：http://docs.oracle.com/javase/specs/jvms/se8/html/jvms-3.html#jvms-3.14

### **50.什么是死锁？请模拟写出一段 Java 死锁的核心代码？如何避免死锁？**

解析：

关于什么是死锁其实就是陷入互相等待谁都无法执行的一种状态，譬如有 A B 两个线程，A 持有 LockA 锁且在等待 LockB 锁，而 B 持有锁 LockB 且在等待 LockA 锁，LockA LockB 陷入互相等待导致谁都无法执行的一种现象。当发生死锁可以采用 jstack 命令进行分析，Android 上面可以采用 Android Device Monitor Thread 进行分析。

关于死锁的核心代码例子如下：
```java
public class ThreadLoop {
    private static Object mLockA = new Object();
    private static Object mLockB = new Object();

    private static void startThreadA() {
        new Thread() {
            @Override
            public void run() {
                Log.i("YYYY", "startThreadA running!");
                synchronized (mLockA) {
                    Log.i("YYYY", "startThreadA mLockA enter!");
                    ThreadLoop.sleep(200);
                    synchronized (mLockB) {
                        Log.i("YYYY", "startThreadA mLockB enter!");
                    }
                }
                Log.i("YYYY", "startThreadA over!");
            }
        }.start();
    }

    private static void startThreadB() {
        new Thread() {
            @Override
            public void run() {
                Log.i("YYYY", "startThreadB running!");
                synchronized (mLockB) {
                    Log.i("YYYY", "startThreadB mLockB enter!");
                    ThreadLoop.sleep(200);
                    synchronized (mLockA) {
                        Log.i("YYYY", "startThreadB mLockA enter!");
                    }
                }
                Log.i("YYYY", "startThreadB over!");
            }
        }.start();
    }

    private static void sleep(long time) {
        try {
            Thread.sleep(time);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void start() {
        startThreadA();
        startThreadB();
    }
}
```
运行结果可能如下：
```
startThreadA running!
startThreadA mLockA enter!
startThreadB running!
startThreadB mLockB enter!
//陷入死锁...
```

避免死锁其实主要有如下几个经验：

考虑加锁顺序：当多个线程需要相同的一些锁但每个线程又按照不同顺序加锁则很容易发生死锁（如上面死锁的例子），如果能确保所有的线程都是按照相同的顺序获得锁则发生死锁的情况就不存在了。

考虑加锁时限：可以在尝试获取锁的时候加一个超时时间，若一个线程没有在给定的时限内成功获得所有需要的锁则会进行回退并释放所有已经获得的锁，然后等待一段随机的时间再重试，这段随机的等待时间让其它线程有机会尝试获取相同的这些锁，并且让该应用在没有获得锁的时候可以继续运行。

### **51.下面程序有什么问题？该如何修改？**
```java
public class SyncCollection {
    private static void putThread(final List<String> list) {
        new Thread() {
            @Override
            public void run() {
                for (int index=0; index<1000; index++) {
                    list.add("data-"+index);
                }
            }
        }.start();
    }

    private static void loopThread(final List<String> list) {
        new Thread() {
            @Override
            public void run() {
                while (true) {
                    for (String item : list) {
                        Log.i("YYYY", "item is "+item);
                    }
                }
            }
        }.start();
    }

    public static void run() {
        final List<String> list = Collections.synchronizedList(new ArrayList<String>());
        putThread(list);
        loopThread(list);
    }
}
```

解析：

该程序运行会在 loopThread 方法的 for 循环迭代器里面抛出 ConcurrentModificationException 异常，因为在遍历容器时容器结构发生了变化（add 操作），所以会抛出该异常，如果要避免这个问题需要在遍历容器时给整个容器对象加锁，如下：
```java
......
while (true) {
        synchronized (list) {
            for (String item : list) {
                Log.i("YYYY", "item is "+item);
            }
        }
    }
}
......
```
这样就保证了 for 循环迭代的同步性，之所以只给这个方法添加 synchronized (list) 是因为 list 对象其实就是 Collections.synchronizedList 返回的同步 List 里面 add 等操作用的锁对象，所以不用给上面的 putThread 方法再添加。

### **52.简单谈谈 Java 并发协作的 wait、notify、notifyAll 等方法的特点和场景？**

解析：

首先并发协作的 wait、notify、notifyAll 等方法是定义在 java 的 Object 类中，而非 Thread，wait 有一个重载方法，参数 0 表示无限等待，更加重要的是在等待期间均可被中断抛出 InterruptedException（很重要），每个对象都有一把锁和等待队列，线程在进入 synchronized 时如果尝试获取锁失败就会把当前线程加入锁的等待队列，其实每个对象除过有用于锁的等待队列外还有一个条件队列，条件队列就是用来进行线程间的协作的，调用 wait 方法就会把当前线程放入这个条件队列并阻塞，然后等待其他线程通过 notify 或者 notifyAll 触发这个条件（自己无法触发）来执行，惟一的区别就是 notify 会从条件队列选择一个线程触发条件并且从队列移除而 notifyAll 会触发条件队列里所有等待的线程并从队列移除。

wait 和 notify、notifyAll 只能在 synchronized 函数或者对象中调用，被上锁的对象一般是多线程共享的对象，如果调用 wait 和 notify、notifyAll 方法时当前线程没有持有对象锁则会抛出 IllegalMonitorStateException 异常。切记代码执行到 synchronized 锁起来的 wait 方法时当前线程会释放对象锁，因为 wait 的具体实现过程是先把当前线程放入条件等待队列、释放对象锁、阻塞等待（线程状态变为 WAITING 或 TIMED_WATING），等待时间到了或者被其他线程 notify、notifyAll 以后从条件队列中移除然后要重新竞争对象锁，竞争到就变为 RUNNABLE 状态，否则该线程被加入对象锁队列变为 BLOCKED 状态。切记调用 notify、notifyAll 会把条件队列中等待的线程移除但是不会释放对象锁，只有在包含 notify、notifyAll 的 synchronized 方法或者代码块执行完毕才能轮到等待的线程执行。

除过我们要保证 wait 和 notify、notifyAll 应该在被 synchronized 的背景下和那个被多线程共享的对象上调用以外还要尽可能保证永远在条件循环而不是 if 语句中使用 wait，因为线程从 wait 调用中返回后不代表其等待的条件就一定成立，所以我们在使用 wait 时应该尽量使用如下模板：
```java
// The standard idiom for calling the wait method in Java 
synchronized (sharedObject) { 
    while (condition) { 
    sharedObject.wait(); 
        // (Releases lock, and reacquires on wakeup) 
    } 
    // do action based upon condition e.g. take or put into queue 
}
```
在条件循环里使用 wait 的目的是在线程被唤醒的前后都持续检查条件是否被满足，如果条件并未改变而 wait 被调用之前 notify 的唤醒通知就来了，那么这个线程并不能保证被唤醒且有可能会导致死锁问题（建立在全局项目超过两个线程以上）。譬如假设有两个生产者 A、B，一个消费者 C，在生产消费者模式中如果对生产者 A、B 不使用条件循环而简单 if 判断中调用 wait 就会出事，当空间满了后 A、B 都被 wait，当 C 取走一个数据后如果调用了 notifyAll 则 A、B 都将被唤醒，假设 A 被唤醒后往空间放入一个数据且空间满了，而此时 B 也会放置一个数据，所以发生空间炸裂错误。
（提示：如上也解答了并发的另一个面试题 ---- Java 多线程为什么使用 while 循环来调用 wait 方法？）
（其实 Thread 的 join 方法实现也是条件循环，核心代码如下：`while (isAlive()) {lock.wait(0);}`）

并发协作其实在 java.util.concurrent 包下已经提供了很多不错且高效的封装实现类了，不过我们依然可以自己使用 wait 和 notify、notifyAll 来解决生产消费者场景、并发等待等场景问题。

### **53.说说你知道的 Java 停止线程的方式及优劣点？**

解析：

关于停止 Java 线程的常见方式及优劣点主要如下：

[不推荐]使用 Thread 的 stop() 方式结束线程已经不推荐了，因为它是一种恶意的中断，一旦执行 stop 方法就会立即终止当前正在运行的线程，所以它无法保证线程逻辑是否完整（譬如线程中重要的资源清理等逻辑代码还没来得及执行就被 stop 掉了，这是非常危险的）；同时会破坏原子性操作，因为一般任何进行加锁的代码块都是为了保护数据的一致性，而调用 stop 后导致该线程所持有的所有锁被突然释放，这样就导致被保护的数据有可能呈现不一致性，其他线程在使用这些被破坏的数据时有可能导致一些很奇怪的应用程序错误。

[推荐]使用 Thread 的 interrupt() 方式中断线程是要看场景决定的，Java 的中断并不是 stop 一样强迫终止一个线程，而是一种给对应线程传递取消信号的协作机制，由对应线程决定如何及何时退出。每个线程都有一个中断标志位（Native 实现），线程的对象方法 isInterrupted() 方法就是返回该标志是否为 true；线程的静态方法 interrupted() 方法也是返回该标志是否为 true，但是它会清空标志位；调用 interrupt() 方法对线程的效果取决于线程的当前状态和在进行的 IO 操作，所以这种方式中断线程需要看场合决定。如果线程处于 RUNNABLE 状态且无 IO 操作则 interrupt() 方法只会设置线程的中断标志位而无其他任何作用，所以这种情况下我们需要在线程运行的合适位置检查中断标记来中断线程，譬如 while 循环条件为 !Thread.isInterrupted()；如果线程处于 WAITING、TIMED_WAITING 状态则 interrupt() 方法会使该线程抛出 InterruptedException 异常且在异常抛出后中断标志会被清空而不是设置，这也就是为什么我们使用 wait、join、sleep 方法时必须要处理受检查的 InterruptedException 异常原因，所以我们一般需要在捕获这种异常后进行线程的终止回收逻辑同时最好再调用一下 Thread.currentThread().interrupt() 设置下当前线程中断标志位；如果线程处于 BLOCKED 锁等待状态则 interrupt() 方法只会设置线程的中断标志位而无其他任何作用，所以 interrupt 无法让一个处于等待锁的线程真正中断，譬如当线程 A、B 共用 LockM 锁，A 如果在获取 LockM 锁后调用 B 的 interrupt() 后接着执行其他耗时操作后才释放 LockM 锁，而在 A 获得 LockM 锁的期间 B 刚好处于锁等待状态，这时即便 B 的锁代码块中显式调用了 !Thread.isInterrupted() 方法也不会中断，只能等到 B 获取锁后才可以中断；如果线程处于 NEW、TERMINATE 状态则 interrupt() 方法没有任何效果，中断标志都不会被设置，这自然是一种没有意义的操作；如果线程处于 IO 操作阻塞情况则 interrupted() 方法的效果取决于当前阻塞的 IO 类，如果类实现了 InterruptibleChannel 接口（一般都是 NIO 的通道操作有实现此接口）则可中断且使 IO 通道关闭且会收到 ClosedByInterruptException 异常且会设置中断状态，如果阻塞的 IO 类是 NIO 的 Selector 则也会被中断且设置标志，如果 IO 类操作是常见的 ServerSocket 的 accept、InputStream 的 read 等调用则只会设置线程的中断标志而不会中断。所以说如果不明白自己的线程在做什么就不要想当然的调用 interrupt()，因为不一定会终止线程，更多的是一种标志协作。

[推荐]自定义中断信号量方式，这种方式中断 RUNNABLE 状态的线程也是很常见的，使用 volatile 的原子性共享标记变量来告诉线程必须停止正在运行的任务。

[推荐]使用 java.util.concurrent 并发包下面提供的方法（很多实质还是 interrupt()），譬如 Future 的 boolean cancel(boolean mayInterruptIfRunning) 方法、ExecutorService 的 shutdown()、shutdownNow() 方法等。

### **54.简单谈谈你对 Java 虚拟机内存模型 JMM 的认识和理解及并发中的原子性、可见性、有序性的理解？**

解析：

这是一个很泛很大且很有水准的面试题，也算是对并发基础原理实质的一个深度问题，想要在面试中简短的回答好不是特别容易，本解析也仅供参考，具体理解可自己查阅其他资料。

Java 内存模型主要目标是定义程序中变量（此处变量特指实例字段、静态字段等，但不包括局部变量和函数参数，因为这两种是线程私有无法共享）在虚拟机中存储到内存与从内存读取出来的规则细节，Java 内存模型规定所有变量都存储在主内存中，每条线程还有自己的工作内存，工作内存保存了该线程使用到的变量到主内存副本拷贝，线程对变量的所有操作（读取、赋值）都必须在工作内存中进行而不能直接读写主内存中的变量，不同线程之间无法相互直接访问对方工作内存中的变量，线程间变量值的传递均需要在主内存来完成。

Java 内存模型对主内存与工作内存之间的具体交互协议定义了八种操作，具体如下：
```
lock（锁定）：作用于主内存变量，把一个变量标识为一条线程独占状态。
unlock（解锁）：作用于主内存变量，把一个处于锁定状态的变量释放出来，释放后的变量才可以被其他线程锁定。
read（读取）：作用于主内存变量，把一个变量从主内存传输到线程的工作内存中，以便随后的 load 动作使用。
load（载入）：作用于工作内存变量，把 read 操作从主内存中得到的变量值放入工作内存的变量副本中。
use（使用）：作用于工作内存变量，把工作内存中的一个变量值传递给执行引擎，每当虚拟机遇到一个需要使用变量值的字节码指令时执行此操作。
assign（赋值）：作用于工作内存变量，把一个从执行引擎接收的值赋值给工作内存的变量，每当虚拟机遇到一个需要给变量进行赋值的字节码指令时执行此操作。
store（存储）：作用于工作内存变量，把工作内存中一个变量的值传递到主内存中，以便后续 write 操作。
write（写入）：作用于主内存变量，把 store 操作从工作内存中得到的值放入主内存变量中。
```
如果要把一个变量从主内存复制到工作内存就必须按顺序执行 read 和 load 操作，从工作内存同步回主内存就必须顺序执行 store 和 write 操作，但是 JVM 只要求了操作的顺序而没有要求上述操作必须保证连续性，所以实质执行中 read 和 load 间及 store 和 write 间是可以插入其他指令的。

Java 内存模型还会对指令进行重排序操作，在执行程序时为了提高性能编译器和处理器经常会对指令进行重排序操作，重排序主要分下面几类：
```
编译器优化重排序：编译器在不改变单线程程序语义的前提下可以重新安排语句的执行顺序。
指令级并行重排序：现代处理器采用了指令级并行技术来将多条指令重叠执行，如果不存在数据依赖性处理器可以改变语句对应机器指令的执行顺序。
内存系统重排序：由于处理器使用缓存和读写缓冲区使得加载和存储操作看上去可能是在乱序执行。
```

其实 Java JMM 内存模型是围绕并发编程中原子性、可见性、有序性三个特征来建立的，关于原子性、可见性、有序性的理解如下：

原子性：就是说一个操作不能被打断，要么执行完要么不执行，类似事务操作，Java 基本类型数据的访问大都是原子操作，long 和 double 类型是 64 位，在 32 位 JVM 中会将 64 位数据的读写操作分成两次 32 位来处理，所以 long 和 double 在 32 位 JVM 中是非原子操作，也就是说在并发访问时是线程非安全的，要想保证原子性就得对访问该数据的地方进行同步操作，譬如 synchronized 等。

可见性：就是说当一个线程对共享变量做了修改后其他线程可以立即感知到该共享变量的改变，从 Java 内存模型我们就能看出来多线程访问共享变量都要经过线程工作内存到主存的复制和主存到线程工作内存的复制操作，所以普通共享变量就无法保证可见性了；Java 提供了 volatile 修饰符来保证变量的可见性，每次使用 volatile 变量都会主动从主存中刷新，除此之外 synchronized、Lock、final 都可以保证变量的可见性。

有序性：就是说 Java 内存模型中的指令重排不会影响单线程的执行顺序，但是会影响多线程并发执行的正确性，所以在并发中我们必须要想办法保证并发代码的有序性；在 Java 里可以通过 volatile 关键字保证一定的有序性，还可以通过 synchronized、Lock 来保证有序性，因为 synchronized、Lock 保证了每一时刻只有一个线程执行同步代码相当于单线程执行，所以自然不会有有序性的问题；除此之外 Java 内存模型通过 happens-before 原则如果能推导出来两个操作的执行顺序就能先天保证有序性，否则无法保证，关于 happens-before 原则可以查阅相关资料。

所以说如果想让 Java 并发程序正确的执行必须保证原子性、有序性、可见性，只要三者中有任意一个不满足并发都无法正确执行。

（关于该问题答案还可参考：www.ifeve.com/java-memory-model-1/）

### **55.说说你对 Java 的 volatile 关键字的理解和使用场景举例？**

解析：

一旦一个并发共享变量（类的成员变量、静态成员变量）被 volatile 关键字修饰就具备了可见性（即一个线程修改了一个变量的值对于另一个线程来说是立即可见的）和有序性（即禁止进行指令重排序），实质是在生成的汇编代码中多了一个 lock 前缀指令。
譬如我们经常会使用标记法中断线程，如下：
```java
boolean stop = false;
//线程1
while(!stop){
    //do some thread things...
}
 
//线程2
stop = true;
```
这段代码其实大多数时候是可以中断线程1的，但是依然存在一定小概率无法中断线程1，因为每个线程都有自己的工作内存，当线程1运行时会对主存的 stop 变量拷贝一份放置到自己的工作内存使用，当线程2更改了 stop 变量的值后还未来得及写回主存中而接着做其他事情了，此时线程1可能无法立即感知到 stop 变量的改变而无法中断自己造成错误的逻辑，当我们对 stop 变量添加 volatile 修饰符后就不会存在上面的问题了，因为 volatile 会强制线程修改变量的改变立即回写到主存中，当线程2修改 stop 变量值时会导致线程1的工作内存中 stop 缓存失效进而主动去主存中重新读取 stop 值。

volatile 有序性的保证有两层含义，当程序执行到 volatile 变量的读或者写操作时在其前面的操作的更改肯定已经全部进行且结果已经对后面的操作可见，在其后面的操作肯定还没有进行，在进行指令优化时不能将对 volatile 变量访问的语句放在其后面执行，也不能把 volatile 变量后面的语句放到其前面执行。

volatile 常见的使用场景为多线程自定义条件标记变量中断和单例模式的 double check 等。

### **56.什么是原子变量？为什么需要他们？Java 提供了哪些原子变量类型？**

解析：

原子变量就是原子操作，是在多线程环境下避免数据不一致必须的手段，譬如`int++`就不是一个原子操作，因为当一个线程读取它的值并加 1 时另外一个线程有可能会读到之前的值而引发错误，所以为了解决这个问题就必须保证增加操作是原子操作，在 JDK1.5 之前可以使用 synchronized 同步技术来做到原子操作（成本太高，需要获取释放锁，获取不到锁还要等待，还有线程的上下文切换操作），而在 JDK1.5 的 java.util.concurrent.atomic 包中 Java 提供了可以自动保证是原子操作且不需要使用 synchronized 同步的实现类。

synchronized 是一种阻塞式算法的悲观锁，但是原子变量的更新逻辑是非阻塞式乐观的，synchronized 得不到锁就会进入等待状态和有线程切换开销，原子变量更新冲突时会循环重试，不会阻塞和上下文切换开销。

java 提供的原子变量类型都在 java.util.concurrent.atomic 包下面，基本的有 AtomicBoolean（原子布尔类型）、AtomicInteger（原子Integer类型）、AtomicLong（原子Long类型）、AtomicReference（原子引用类型），针对基本的对应的数组类型有 AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray，为了便于以原子方式更新对象中字段而增加的类型有 AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicReferenceFieldUpdater，AtomicReference 还有两个类似的类 AtomicMarkableReference、AtomicStampedReference，对于 char、short、float、double 类型如果我们需要可以转换（floatToIntBits）为 int 或者 long 来使用。

### **57.简单谈谈你对 CAS 的认识和理解？**

解析：

CAS（compare and swap）实质就是比较交换策略，设计并发算法时常用到的一种技术，java.util.concurrent 包基本都建立在 CAS 之上，现代的处理器基本都支持 CAS，只是不同厂商实现不同而已；CAS 有内存值V、旧的预期值A、要修改的值B三个操作数，当且仅当预期值A和内存值V相同时才会将内存值修改为B并返回 true，否则什么也不做且返回 false。

CAS 是通过 Unsafe 类来实现的，Unsafe 提供了硬件级别的原子操作，关于 CAS 的方法主要是 native 的 compareAndSwapObject、compareAndSwapInt、compareAndSwapLong，其比较交换是一组原子操作，因为是硬件级别的操作，所以效率会高一些。

CAS 最常见和基础的使用地方在 java.util.concurrent.atomic 包下面，譬如 AtomicInteger 使用 CAS 的实质如下（基于JDK 1.8之前，1.8开始已经再次优化了，看不见阻塞的逻辑了）：
```java
public class AtomicInteger extends Number implements java.io.Serializable {
    //Unsafe 是 CAS 的核心类
    private static final sun.misc.Unsafe unsafe = sun.misc.Unsafe.getUnsafe();
    //valueOffset 在 static 代码块中通过 Unsafe 获取 value 变量在内存中的偏移地址。
    //因为 Unsafe 是通过 valueOffset 内存偏移地址来获取数据的原始值的。
    private static final long valueOffset;
    //获取变量的偏移地址
    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (ReflectiveOperationException e) {
            throw new Error(e);
        }
    }
    //volatile修饰保证了 value 变量的可见性和有序性
    private volatile int value;

    public final int incrementAndGet() {
        //原子更新成功为止才返回
        for (;;) {
            int current = get();
            int next = current + 1;
            if (compareAndSet(current, next))
                return next;
            }
        }
    }
}
```
可以发现基于 CAS 可以实现乐观的非阻塞算法的 Java atomic 原子变量，除此之外还可以实现悲观阻塞式算法，譬如锁机制等。

CAS 策略算法其实有个致命的 ABA 问题，如果一个变量 V 初次读取时值为 A，并且在准备赋值时检查到仍然是 A，而这时候又在多线程的场景下我们是无法确认这段时间之内是否值被其他线程先修改为 B 接着改回了 A，所以 CAS 就会在这种场景下误认为变量从来没被修改过，也就是 ABA 问题了，这个问题一般是没啥影响的，如果程序逻辑需要处理 ABA 场景就可以使用 AtomicStampedReference，在修改值的同时传入一个唯一标记（譬如时间戳），只有值和标记都想等才进行修改，使用 AtomicMarkableReference 也可以，只不过标记是 boolean 类型。

### **58.说说 synchronized 与 Lock 的区别及优劣点？你用过 Java Lock 的哪些实现类？**

解析：

关于 synchronized 与 Lock 的区别及优劣点主要如下：
synchronized 是 Java 的一个内置特性关键字，而 Lock 是 Java 的一个接口类；synchronized 在发生异常时会自动释放线程占用的锁，而 Lock 在发生异常时（不发生也一样）需要主动在 
finally 中调用 unLock() 去释放锁；Lock 可以让等待锁的线程响应中断，而 synchronized 无法响应中断，会一直等待下去；Lock 可以知道有没有成功获取到锁，而 synchronized 无法办到；Lock 可以提高多线程进行读操作的效率，而 synchronized 不可以；在性能上来说如果竞争资源不激烈则两者差距不大，如果竞争资源非常激烈（很多线程同时抢占）时 Lock 的性能远远好于 synchronized；不过要注意只能中断阻塞中的线程，一旦获取到锁进入运行状态就无法中断；Lock 和 synchronized 都可以保证保证内存可见性。

Lock 接口及其主要实现类都位于 java.util.concurrent.locks 包下，Lock 的主要方法如下：
```java
//切记Lock不会主动释放锁
public interface Lock {
    //用来获取锁，如果锁已被其他线程获取则进行等待，等待过程无法中断。
    void lock();
    //用来获取锁，如果线程正在等待获取锁则这个线程能够响应中断，即可以中断线程的等待状态。
    void lockInterruptibly() throws InterruptedException;
    //用来尝试获取锁，如果获取成功则返回true，如果获取失败（即锁已被其他线程获取）则返回false，该方法会立即返回，在拿不到锁时也不会一直在那等待。
    boolean tryLock();
    //用来尝试获取锁，区别在于拿不到锁时会等待一定的时间，在时间期限之内如果还拿不到锁，就返回false，如果如果一开始拿到锁或者在等待期间内拿到了锁则返回true，等待期间可被中断。
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
    //用来释放锁，Lock 不会主动释放锁，需要手动调用此方法，一般写在 finally 语句中。
    void unlock();
    //创建一个 Lock 的 Condition。
    Condition newCondition();
}
```
ReentrantLock 和 synchronized 一样是一个可重入锁，也是唯一一个实现了 Lock 接口的类且提供了更多的方法的锁实现类，synchronized 是非公平锁，而 ReentrantLock 默认也是非公平锁，不过提供了构造参数实现公平锁，所谓的公平其实就是等待时间最长的线程优先获得锁，公平锁会影响性能。

ReadWriteLock 接口及其主要实现类都位于 java.util.concurrent.locks 包下，ReadWriteLock 的主要方法如下：
```java
//将文件的读写操作分开，分成2个锁来分配给线程，从而使得多个线程可以同时进行读操作。
public interface ReadWriteLock {
    //获取读锁
    Lock readLock();
    //获取写锁
    Lock writeLock();
}
```
ReentrantReadWriteLock 是 ReadWriteLock 接口的实现类，里面提供了很多丰富的方法，主要的 readLock() 和 writeLock() 方法用来获取读锁和写锁，效率比 synchronized 高很多，尤其在并发读文件情况下；不过要注意的是如果有一个线程已经占用了读锁则此时其他线程如果要申请写锁则申请写锁的线程会一直等待释放读锁，如果有一个线程已经占用了写锁则此时其他线程如果申请写锁或者读锁则申请的线程会一直等待释放写锁。

### **59.简单说说你所了解的 Java 锁分类和特点有哪些？**

解析：

其实对于 Java 锁的分类没有严格意义的规则，我们常说的分类一般都是依据锁的特性、锁的设计、锁的状态等进行归纳整理的，所以常见的分类如下：

公平锁、非公平锁：公平锁指多个线程按照申请锁的顺序来获取锁，非公平锁就是没有顺序完全随机，所以能会造成优先级反转或者饥饿现象；synchronized 就是非公平锁，ReentrantLock（使用 CAS 和 AQS 实现） 通过构造参数可以决定是非公平锁还是公平锁，默认构造是非公平锁；非公平锁的吞吐量性能比公平锁大好。

可重入锁：又名递归锁，指在同一个线程在外层方法获取锁的时候在进入内层方法会自动获取锁，synchronized 和 ReentrantLock 都是可重入锁，可重入锁可以在一定程度避免死锁。

独享锁、共享锁：独享锁是指该锁一次只能被一个线程持有，共享锁指该锁可以被多个线程持有；synchronized 和 ReentrantLock 都是独享锁，ReadWriteLock 的读锁是共享锁，写锁是独占锁；ReentrantLock 的独享锁和共享锁也是通过 AQS 来实现的。

互斥锁、读写锁：其实就是独享锁、共享锁的具体说法；互斥锁实质就是 ReentrantLock，读写锁实质就是 ReadWriteLock。

乐观锁、悲观锁：这个分类不是具体锁的分类，而是看待并发同步的角度；悲观锁认为对于同一个数据的并发操作一定是会发生修改的（哪怕实质没修改也认为会修改），因此对于同一个数据的并发操作悲观锁采取加锁的形式，因为悲观锁认为不加锁的操作一定有问题；乐观锁则认为对于同一个数据的并发操作是不会发生修改的，在更新数据的时候会采用不断的尝试更新，乐观锁认为不加锁的并发操作是没事的；由此可以看出悲观锁适合写操作非常多的场景，乐观锁适合读操作非常多的场景，不加锁会带来大量的性能提升，悲观锁在 java 中很常见，乐观锁其实就是基于 CAS 的无锁编程，譬如 java 的原子类就是通过 CAS 自旋实现的。

分段锁：实质是一种锁的设计策略，不是具体的锁，对于 ConcurrentHashMap 而言其并发的实现就是通过分段锁的形式来实现高效并发操作；当要 put 元素时并不是对整个 hashmap 加锁，而是先通过 hashcode 知道它要放在哪个分段，然后对分段进行加锁，所以多线程 put 元素时只要放在的不是同一个分段就做到了真正的并行插入，但是统计 size 时就需要获取所有的分段锁才能统计；分段锁的设计是为了细化锁的粒度。

偏向锁、轻量级锁、重量级锁：这种分类是按照锁状态来归纳的，并且是针对 synchronized 的，java 1.6 为了减少获取锁和释放锁带来的性能问题而引入的一种状态，其状态会随着竞争情况逐渐升级，锁可以升级但不能降级，意味着偏向锁升级成轻量级锁后无法降为偏向锁，这种升级无法降级的策略目的就是为了提高获得锁和释放锁的效率。

自旋锁：其实是相对于互斥锁的概念，互斥锁线程会进入 WAITING 状态和 RUNNABLE 状态的切换，涉及上下文切换、cpu 抢占等开销，自旋锁的线程一直是 RUNNABLE 状态的，一直在那循环检测锁标志位，机制不重复，但是自旋锁加锁全程消耗 cpu，起始开销虽然低于互斥锁，但随着持锁时间加锁开销是线性增长。

可中断锁：synchronized 是不可中断的，Lock 是可中断的，这里的可中断建立在阻塞等待中断，运行中是无法中断的。

### **60.简单谈谈你对 Java 并发显式协作 Condition 的 await、signal、signalAll 理解？同时说说他们和普通并发协作 wait、notify、notifyAll 等方法的区别？**

解析：

我们知道 java　显式锁　Lock 可以认为是对　synchronized　的升级，所以　java　1.5　出现的显式协作 Condition 接口的 await、signal、signalAll 也可以说是普通并发协作 wait、notify、notifyAll 的升级；普通并发协作 wait、notify、notifyAll 需要与　synchronized　配合使用，显式协作 Condition 的 await、signal、signalAll 需要与显式锁　Lock 配合使用（Lock.newCondition()），调用　await、signal、signalAll　方法都必须在　lock　保护之内。

和　wait　一样，await　在进入等待队列后会释放锁和　cpu，当被其他线程唤醒或者超时或中断后都需要重新获取锁，获取锁后才会从　await　方法中退出，await　同样和　wait　一样存在等待返回不代表条件成立的问题，所以也需要主动循环条件判断；await　提供了比　wait　更加强大的机制，譬如提供了可中断或者不可中断的　await　机制等；特别注意　Condition　也有　wait、notify、notifyAll　方法，因为其也是　Object，所以在使用显式协作机制时千万不要和 synchronized 情况下的协作机制混合使用，避免出现诡异问题。

使用实例参见：https://stackoverflow.com/questions/2536692/a-simple-scenario-using-wait-and-notify-in-java

### **61.简单谈谈你对　ThreadLocal　的认识？其与同步机制有何区别？其实现原理及使用场景和存在的问题有哪些?**

解析：

这算是作为　Android　开发在面试时被问得比较多的一道题了，其实是很简单的，只要你用心体会过就知道怎么回事了；总之　ThreadLocal　不使用来解决对象共享访问问题的，而是线程本地变量，其　set 的对象作用域为当前线程内部，生命周期伴随线程执行而终止，多个线程间不共享，切记将　ThreadLocal　理解成多线程变量副本的认知是绝对错误的，没有副本这一操作，ThreadLocal　中　set　进去的对象依然是引用方式而不是复制拷贝，所以谈不上副本，所以一般不建议　ThreadLocal　的　set　参数传递共享对象；ThreadLocal　特别适合会被多线程调用框架的编写，可以很容易解决框架中当前线程本地变量的效果；ThreadLocal 是一个泛型类，其提供的主要方法如下：
```java
public T get() :获取值，如果没有返回　null。
public void set(T) :设置值。
protected T initialValue() :用于提供初始值，当调用　get　方法时如果之前没有设置过则会调用该方法获取初始值，默认为　null。
public void remove() :删掉当前线程对应的值，如果删掉后再次调用　get　则会再调用　initialValue　获取初值。
```

ThreadLocal　和同步机制面对的是不同场景的问题，严格说两者没有任何关系，同步机制是为了同步多线程对相同资源的并发访问，是为了线程间数据共享的问题，而　ThreadLocal　是隔离多线程数据共享，从根本上就不在多个线程之间共享资源，这样自然就不需要多线程的同步机制了，所以还是一句话，两者没有一毛钱的关系，需求场景不同，甚至需求有些互斥。

每个线程　Thread　都有一个类型为　ThreadLocalMap　的　Map，调用　set　实际上是在线程自己的　Map　里设置了一个条目，键为当前　ThreadLocal　实例，值为　set　方法的泛型　value　参数，别的线程此时调用　get　是拿不到的，只有当前线程拿到当前线程设置的，ThreadLocalMap　是一个专门用于　ThreadLocal　的一个静态内部类，与一般的　Map 不同（和一般　Map　也没有继承等任何关系），它的键类型为`WeakReference<ThreadLocal>`；其　get　方法的实现是拿到当前线程的　ThreadLocalMap　引用，以当前　ThreadLocal　实例为键从　Map　中获取条目取其　value　值，如果　Map　中没有就返回　initialValue　方法初始化的值；所以通过每个　Thread　自己的　Map　存取　value　就达到了线程本地变量的效果。

ThreadLocal　的使用场景核心原则就是按线程多实例（每个线程对应一个实例）的对象访问，并且这个对象很多地方都会用到，譬如　Android 的　Looper　实现等，达到每个线程只初始化一次对应实例等效果，其他应用场景譬如解决数据库连接，Session　管理等。

ThreadLocal　使用要注意的问题主要如下：

一般使用　ThreadLocal　都不会将共享变量放到线程的　ThreadLocal　中，因为存放共享变量依旧需要我们手动解决并发问题且和直接传递使用没有啥实质区别，一般存放到　ThreadLocal　的变量都是当前线程本身就独一无二的一个变量，其他线程本身就无法访问。

使用　ThreadLocal　配合线程池使用时要特别注意，线程池中的线程是复用的机制，线程池中线程在执行完一个任务执行下一个任务（复用线程情况下，线程池没满未复用不会）时其中的　ThreadLocal　对象并不会被清空，所以　set　的值会被带到下一个任务中，所以这种情况下一定要记得使用　ThreadLocal　前先复位初值或者使用后　remove　清空。

### **62.什么是 CopyOnWriteArrayList，它与 ArrayList 有何不同？同理说说　CopyOnWriteArraySet。**

解析：

CopyOnWriteArrayList　是　ArrayList　的一个线程安全的变体，其中所有可变操作（add、set　等等）都是通过对底层数组进行一次新的复制来实现的，相比较于　ArrayList　它的写操作要慢一些，因为　CopyOnWriteArrayList　中写操作需要大面积复制数组，所以性能肯定很差，但是读操作因为操作的对象和写操作不是同一个对象，读之间也不需要加锁，读和写之间的同步处理只是在写完后通过一个简单的　"="　将引用指向新的数组对象上来，这个几乎不需要时间，这样读操作就很快很安全且适合在多线程里使用且不会发生　ConcurrentModificationException，不过其迭代器不支持修改操作和依赖迭代器修改的操作（譬如 Collections　的　sort 方法），因此　CopyOnWriteArrayList　适合使用在读操作远远大于写操作且数据量不是特别庞大（每次写都是一次复制）的场景里，比如缓存等。

CopyOnWriteArrayList　的内部实现依然是一个　volatile　的数组，这个数组每次是以原子方式被整体更新的（譬如　add, remove, set, sort　操作都是基于锁机制的，jdk　和　openjdk　的锁方式不一样，openjdk　使用　ReentrantLock，jdk　使用　synchronized），每次修改操作都会新建一个数组将原数组复制过来修改后将引用指向新建的数组，这就是写时拷贝，所有读操作都是直接访问当前引用数组。我们常见的同步线程安全方式都是　synchronized 和　Lock　和循环　CAS，而　CopyOnWriteArrayList　的写时拷贝对于线程安全提供了新的思路，只是这种方式代价略大。

对于　CopyOnWriteArraySet　实现了 Set 接口，线程安全，其特点就是不包含重复元素，其内部实现是通过　CopyOnWriteArrayList　来的，而　Android　的　ArraySet　（Java 没有这个类）内部实现依然是数组而不是基于　ArrayList，注意区分，TreeSet\HashSet\LinkedHashSet 是非线程安全的，所以　CopyOnWriteArraySet　同样适合读多于写且数据集不大的场合，在大多数场合下这些写时拷贝的线程安全集合效率都比　Collections 中同步集合构造器低。

### **63.说说你对　java Map　家族中　ConcurrentHashMap　的理解？**

解析：

Map　的　HashMap　和　LinkedHashMap　等都是非线程安全的，而　Hashtable 和　Collections　的同步　Map 方式都是使用了方法及别的粗粒度锁机制，ConcurrentHashMap　却另辟蹊径实现了一种比较高效的无序线程安全　Map，并发迭代器不会抛出异常。

jdk 1.6　与　1.7 采用了分段锁　Segment(ReentrantLock　的子类)　与　HashEntry　结合的机制，只有在同一个分段内才会存在竞争，不同的分段锁之间没有竞争，所以分段锁提高了并发的效率，但是由于不是对　Map　整体加锁所以导致一些需要扫描整个　Map　的方法（size, containsValue）需要使用特殊的实现，另一些方法(clear)甚至直接放弃了对一致性的要求，所以说这些版本的　ConcurrentHashMap　实现是弱一致性的；Segment　数组的每个元素类似　HashMap　的结构，即内部拥有一个　HashEntry　数组，数组中每个元素又是一个链表，所以　ConcurrentHashMap　的并发度就是分段锁 Segment 数组的长度，默认为　16，构造可设置。

jdk 1.8　对　ConcurrentHashMap　进行了重新实现，由原来的一千多行代码变成了六千多行，放弃了分段锁的思路而采用了　Node + CAS + synchronized 的新思路来保证线程的安全，所以对于　Node 的锁粒度明显比　1.6　中对于分段锁的粒度还要小。

关于具体深度理解可以参考 [深入并发包 ConcurrentHashMap](http://www.importnew.com/26049.html) 一文。

### **64.说说你对　ConcurrentSkipListMap　和　ConcurrentSkipListSet　的理解？**

解析：

基于细粒度锁线程安全的　ConcurrentHashMap　是不支持有序的，而　TreeMap 和　TreeSet 是有序的但线程不安全，所以　java 并发包提供了与之对应线程安全有序的　ConcurrentSkipListMap（ConcurrentNavigableMap　接口的实现类）　和　ConcurrentSkipListSet；TreeSet　是基于　TreeMap　实现的，ConcurrentSkipListSet　是基于　ConcurrentSkipListMap　实现的（value　用一个 True　值替换了而已），ConcurrentSkipListMap　是基于　SkipList　和　CAS 实现的，主要操作的复杂度都是 O(log(N))，SkipList　是基于跳跃表的一种数据结构算法，使用这个算法可以更易于实现高效并发算法；ConcurrentHashMap　没有使用锁，所有操作都是无阻塞的，所有操作（包括写）都可以并行，迭代器不会抛出异常（ConcurrentModificationException），数据是弱一致的，迭代可能反映最新修改也可能不反映，一些方法（putAll, clear）不是原子的。

ConcurrentSkipListMap　使用的　SkipList　跳表结构是基于链表的，在链表的基础上加了多层索引结构而已，高层索引节点少，低层多，对于每个索引节点有两个指针，一个向右指向下一个同层的索引节点，另一个向下指向下一层索引节点或者基本链表节点，快速查找时由高层到低层每次二分法查找，速度非常块。

想要了解更多可以查看[跳表（SkipList）及ConcurrentSkipListMap源码解析](http://blog.csdn.net/sunxianghuang/article/details/52221913)一文。

### **65.简单说说你所知道或者用过的　java　队列都有哪些？各自有何特点？**

解析：

java　的队列主要分为　Queue　和　Deque　双端队列，Queue　扩展了　Collection　接口，Deque　扩展了　Queue　接口，他们都是　java　的集合类接口，其支持　Collection　操作外还支持队列或者双端队列提供的其他操作。

Queue　接口的每个方法都提供两种形式，一种在操作失败时抛出异常，另一种操作失败时返回特殊值，特有的接口方法如下：
```java
boolean add(E e)    //插入队列，如果立即返回且未超容返回　true，否则抛出异常
boolean offer(E e)  //插入队列，不会抛出异常
E remove()      //获取并移除此队列的头，如果队列已空执行此操作抛出异常
E poll()        //获取并移除此队列的头，如果队列已空执行则返回 null
E element()     //获取但不移除此队列的头，如果队列已空执行此操作抛出异常
E peek()        //获取但不移除此队列的头，如果队列已空执行则返回 null
```
所以我们使用　Queue　时要尽量避免使用会抛出异常的方法，值得注意的是　LinkedList 类也实现了　Queue　和　Deque，所以要留意。

Deque　接口是一个线性的双端队列，支持在两端插入和删除元素，大多数时候　Deque　的实现对于容量没有限制，但是该接口可支持容量限制或者不限制操作，Deque　接口的每个方法都提供两种形式，一种在操作失败时抛出异常，另一种操作失败时返回特殊值，特有的接口方法如下：
```java
void addFirst(E e)
boolean offerFirst(E e)
void addLast(E e)
boolean offerLast(E e)
E removeFirst()
E pollFirst()
E removeLast()
E pollLast()
E getFirst()
E peekFirst()
E getLast()
E peekLast()
```
Deque 接口继承自　Queue　的方法在大多数情况下等同上面的操作，所以建议使用　Deque 时尽量使用其特有的方法，值得注意的是　LinkedList 类也实现了　Queue　和　Deque，所以要留意。

关于　Queue　与　Deque　的实现类主要可以分为非线程安全和线程安全两大类，非线程安全的队列主要如下：

|非线程安全队列|说明|
|--|--|
|LinkedList|实现了　Deque　与　List，基于双向链表实现。|
|PriorityQueue|实现了　Queue，基于数组且基于　Comparable　或　Comparator　的无限容量优先级队列。|
|ArrayDeque|实现了　Deque，基于循环数组且无限容量比　LinkedList　效率高的双端队列。|

线程安全的并发队列（位于　java　并发包下面）主要有如下（下面这些对列迭代都不抛出　ConcurrentModificationException，都是弱一致的），其又可分类为无锁实现并发和有锁实现并发，或者分类为阻塞队列和非阻塞队列，又或者分类为有界队列或者无界队列：

|并发安全队列|特性|说明|
|--|--|--|
|ConcurrentLinkedQueue|无锁非阻塞并发队列|基于单向链表无界和循环 CAS　无锁并发实现，size　方法不是常量运算，并发弱一致，先进先出，尾部入队，头部出队。|
|ConcurrentLinkedDeque|无锁非阻塞并发队列|基于双向链表无界和循环　CAS　无锁并发实现，size　方法不是常量运算，并发弱一致，两端可入队出队，类似　LinkedList。|
|ArrayBlockingQueue|基于循环数组线程安全有界阻塞先进先出队列|阻塞队列是指当队列到达高低界限时如果继续操作则需要阻塞等待界限解除，其实现依赖同一个　ReentrantLock　和两个　Condition（一个队列满条件，一个队列空条件），并发安全是因为很多操作都基于　ReentrantLock　锁实现，队列的容量和使用非公平锁还是公平锁都是通过构造方法控制的，阻塞是可以被中断的，阻塞等待也是可以设置超时机制的；典型用法就是生产消费者，供求关系一旦到达高低界限都会自动阻塞调整。|
|LinkedBlockingQueue|基于单向链表线程安全阻塞先进先出队列|可指定长度也可不指定，默认无长度，其实现依赖两个　ReentrantLock（一个take锁，一个put锁）和两个　Condition（一个队列满条件，一个队列空条件），并发安全是因为很多操作都基于　ReentrantLock　锁实现，阻塞是可以被中断的，阻塞等待也是可以设置超时机制的，存在一个哨兵节点维持头节点；典型用法就是生产消费者。|
|LinkedBlockingDeque|基于双向链表线程安全阻塞队列|可指定长度也可不指定，默认无长度，其实现依赖一个　ReentrantLock　和两个　Condition（一个队列满条件，一个队列空条件），并发安全是因为很多操作都基于　ReentrantLock　锁实现，阻塞是可以被中断的，阻塞等待也是可以设置超时机制的，不存在头哨兵节点；典型用法就是生产消费者。|
|PriorityBlockingQueue|优先级阻塞并发队列|基于数组且基于　Comparable　或　Comparator　的无限容量并发阻塞优先级队列，放入的元素不可为空；如果队列已经空了继续进行取操作则会阻塞，如果没空且放置操作一直进行则可能会造成队列爆掉，所以该队列不会阻塞数据生产者，其大多数操作方法实现依赖一个　ReentrantLock　锁和一个　Condition（队列空条件），不过要特别注意其每次添加元素遇到队列数组扩容操作时会有一个基于　CAS　的自旋锁操作以避免并发申请调整大小，由于需要在扩容申请新内存的过程中退出锁，所以不可以将任务简单的委托给一个　PriorityQueue　来完成，为了保证兼容性在序列话的过程中使用了锁，为了增加保证兼容性，在序列化的过程中需要付出双倍的代价（先转换成一个　PriorityQueue　然后再序列化）。|
|DelayQueue|延时阻塞并发优先级队列|无界队列，故生产者永远不会被阻塞，消费者元素延迟时间未到才会阻塞，队列中的元素只有当其指定的延迟时间到了才能够从队列中获取到该元素，其实现依赖一把　ReentrantLock　锁和一个　Condition　条件，其存储基于　PriorityQueue　队列。|
|SynchronousQueue|其他阻塞队列|基于无锁算法保证并发安全及性能，是一个没有数据缓冲的阻塞队列，生产者对其插入操作必须等待消费者的移除操作，反过来也一样，不像　ArrayBlockingQueue　或　LinkedListBlockingQueue，SynchronousQueue　内部并没有数据缓存空间，不能调用　peek　方法来看队列中是否有数据元素，因为数据元素只有当你试着取走的时候才可能存在，不取走而只想偷窥一下是不行的，当然遍历这个队列的操作也是不允许的，可以理解为生产者和消费者互相等待对方握手然后一起离开；一个典型的使用场景是在　Executors.newCachedThreadPool()　里面，这个线程池根据需要（新任务到来时）创建新的线程，如果有空闲线程则会重复使用，线程空闲了　60　秒后会被回收。|
|LinkedTransferQueue|其他阻塞队列|基于　CAS 和单向链表无界先进先出队列，不怎么常用，jdk 1.7 出现，具体可以[参考此文](http://www.cnblogs.com/rockman12352/p/3790245.html)。|

### **66.简单谈谈你对　Java　的　Runnable、Callable、Future、Executor、ExecutorService、Executors、FutureTask 认识和理解？**

解析：

Runnable、Callable 都表示要执行的异步任务的接口，都提供了一个接口方法，Runnable 没有返回值且不会抛出异常，Callable 有返回值且会抛出异常；Executor 是一个执行任务接口，其定义了一个接收 Runnable 对象的方法；ExecutorService 是继承 Executor 接口执行任务的更广泛的接口，其提供了生命周期管理的方法以及可跟踪一个或多个异步任务执行状况返回 Future 的方法；AbstractExecutorService 是 ExecutorService 接口执行方法的默认抽象实现类；ScheduledExecutorService 是继承 ExecutorService 拓展的一个可定时调度任务接口；ScheduledThreadPoolExecutor 是 ScheduledExecutorService 接口的实现类，用来表示一个可定时调度任务的线程池；ThreadPoolExecutor 线程池是基于 AbstractExecutorService 的实现类；可以通过调用 Executors 以其提供的静态工厂方法来创建线程池并返回一个 ExecutorService 对象；Future 表示异步任务的状态结果；FutureTask 是 Future 和 Runnable 的实现类，所以 FutureTask 可以交给 Executor 执行，也可以由调用线程直接执行 FutureTask.run 方法，FutureTask 的 run 方法中又会调用 Callable 的 call 方法，FutureTask 有一个回调函数 done 方法，当一个任务执行结束后会回调这个方法，我们可以在 done 方法中调用 FutureTask 的 get 方法来获得计算的结果，这样就不会因为主动调用 get 被阻塞等待了。
```java
public interface Runnable {
    //无返回值且不会抛出异常的任务实现方法
    public abstract void run();
}

public interface Callable<V> {
    ////有返回值且会抛出异常的任务实现方法
    V call() throws Exception;
}

public interface Executor {
    //最简单的执行服务接口，可执行一个无返回的Runnable，接口无限定，方法实现可以使线程池也可以是每次new线程执行等，只是一个规范
    void execute(Runnable command);
}

public interface ExecutorService extends Executor {
    //关闭执行服务，不阻塞，返回不代表所有任务已结束，表示不再接收新任务，但是已经提交的任务还会继续执行，即使任务还未开始执行
    void shutdown();
    //关闭执行服务，不阻塞，返回不代表所有任务已结束，表示不再接收新任务且已提交但尚未执行的任务会被终止，对于正在执行的任务一般会调用interrupt方法尝试中断，该方法会返回已提交但尚未执行的任务列表
    List<Runnable> shutdownNow();
    //调用 shutdown 和 shutdownNow 后该方法一定返回 true，返回不代表所有任务已结束
    boolean isShutdown();
    //获取所有任务是否已经结束，当调用shutdown或者shutdownNow方法后，并且所有提交的任务完成后返回为true
    boolean isTerminated();
    //阻塞等待所有任务结束，可以设置timeout，如果超时了所有任务还未完成则返回false，反之
    boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
    //提交一个Callable有返回值类型的任务，返回值为Future，提交后立即返回，返回不代表任务已经执行，通过Future可以查询提交的任务状态、获取返回值、取消任务等
    <T> Future<T> submit(Callable<T> task);
    //提交一个Runnable无返回值类型的任务，返回值为Future，提交后立即返回，返回不代表任务已经执行，通过Future可以查询提交的任务状态、获取返回值（获取到的值取决于参数result）、取消任务等
    <T> Future<T> submit(Runnable task, T result);
    //提交一个Runnable无返回值类型的任务，返回值为Future，提交后立即返回，返回不代表任务已经执行，通过Future可以查询提交的任务状态、获取返回值（获取到的值为null）、取消任务等
    Future<?> submit(Runnable task);
    //阻塞批量提交任务，提交的任务容器列表和返回的Future列表存在顺序上有对应关系，当所有任务都完成时、调用线程被中断时返回
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException;
    //阻塞批量提交任务，超时则会取消所有没执行的任务，提交的任务容器列表和返回的Future列表存在顺序上有对应关系，当所有任务都完成时调用线程被中断时或者超过时限时都会返回结果，超过时限后，任何尚未完成的任务都会被取消，作为invokeAll的返回值，每个任务要么正常地完成，要么被取消
    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException;
    //阻塞批量提交任务，有一类多线程编程模式场景为启动多个线程，相互独立的（无同步）去计算一个结果，当某一个线程得到结果之后，立刻终止所有线程，因为只需要一个结果就够了，这个方法就是用来解决这个问题的；也就是说只有有一个任务成功就会阻塞返回这个任务的结果，其他任务被取消
    <T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException;
    //阻塞批量提交任务，同上，只是如果没有任何任务在规定timeout内返回则抛出异常
    <T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}

//Timer的优质替代品
public interface ScheduledExecutorService extends ExecutorService {
    //安排所提交的Runnable任务在initDelay指定的时间后执行。
    public ScheduledFuture<?> schedule(Runnable command, long delay, TimeUnit unit);
    //安排所提交的Callable任务在initDelay指定的时间后执行。
    public <V> ScheduledFuture<V> schedule(Callable<V> callable, long delay, TimeUnit unit);
    //安排所提交的Runnable任务按指定的间隔重复执行，如果任务执行过程中抛出了异常则ScheduledExecutorService就会停止执行任务且不会再周期地执行该任务了，所以你如果想保住任务都一直被周期执行，那么catch一切可能的异常。
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command, long initialDelay, long period, TimeUnit unit);
    //安排所提交的Runnable任务在每次执行完后，等待delay所指定的时间后重复执行，如果任务执行过程中抛出了异常则ScheduledExecutorService就会停止执行任务且不会再周期地执行该任务了，所以你如果想保住任务都一直被周期执行，那么catch一切可能的异常。
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command, long initialDelay, long delay, TimeUnit unit);
    //终止可以使用返回的ScheduleFuture对象上的cancel方法，另外也一种选择是在run或者execute里根据条件抛异常，可参考 http://www.oschina.net/question/1158769_119659?sort=time 一文。
}

public interface Future<V> {
    //用于取消异步任务，如果任务已经完成、已经取消、或者不能取消则返回false，反之true，如果任务还未开始则不再运行，如果任务已经运行则不一定能取消，mayInterruptIfRunning可以在任务运行中尝试中断任务，但不一定成功
    boolean cancel(boolean mayInterruptIfRunning);
    //表示任务是否被取消，只要cancel为true则该方法返回true，即便执行的任务线程还未真正结束
    boolean isCancelled();
    //表示任务是否结束，不管什么原因（正常结束、异常、中断、取消等）
    boolean isDone();
    //用于返回异步任务最终的执行结果，如果任务还未执行完（包含任务被中断、异常等）则阻塞等待
    V get() throws InterruptedException, ExecutionException;
    //用于返回异步任务最终的执行结果，如果任务还未执行完（包含任务被中断、异常等）或者到设置的timeout时间则阻塞等待
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}

//线程池静态工厂类，提供方便的创建一些预配置的线程池，包装转换等操作。
public class Executors {
    //创建一个基于ThreadPoolExecutor参数为LinkedBlockingQueue的定长线程池，可控制线程最大并发数且可重用，超出的线程会在队列中等待。
    public static ExecutorService newFixedThreadPool(int nThreads) {...}
    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {...}
    //1.8引入，创建持有足够线程的线程池来支持给定的并行级别，并通过使用多个队列减少竞争，它需要穿一个并行级别的参数，如果不传则被设定为默认的CPU数量。
    public static ExecutorService newWorkStealingPool(int parallelism) {...}
    public static ExecutorService newWorkStealingPool() {...}
    //创建一个基于ThreadPoolExecutor参数为LinkedBlockingQueue的单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。
    public static ExecutorService newSingleThreadExecutor() {...}
    public static ExecutorService newSingleThreadExecutor(ThreadFactory threadFactory) {...}
    //创建一个基于ThreadPoolExecutor参数为SynchronousQueue可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收则新建线程。
    public static ExecutorService newCachedThreadPool() {...}
    public static ExecutorService newCachedThreadPool(ThreadFactory threadFactory) {...}
    //创建一个单例线程池，定期或延时执行任务。
    public static ScheduledExecutorService newSingleThreadScheduledExecutor() {...}
    public static ScheduledExecutorService newSingleThreadScheduledExecutor(ThreadFactory threadFactory) {...}
    //创建一个基于ScheduledThreadPoolExecutor定长线程池，支持定时及周期性任务执行。
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {...}
    public static ScheduledExecutorService newScheduledThreadPool(
            int corePoolSize, ThreadFactory threadFactory) {...}
    //将普通线程池包装成不可配置的线程池，如果不想线程池被不明所以的后人修改可以调用这个方法。
    public static ExecutorService unconfigurableExecutorService(ExecutorService executor) {...}
    public static ScheduledExecutorService unconfigurableScheduledExecutorService(ScheduledExecutorService executor) {...}
    //返回用于创建新线程的线程工厂
    public static ThreadFactory defaultThreadFactory() {...}
    //返回用于创建新线程的线程工厂，这些新线程与当前线程具有相同的权限
    public static ThreadFactory privilegedThreadFactory() {...}
    //一堆适配器接口方法转换操作
    public static <T> Callable<T> callable(Runnable task, T result) {...}
    public static Callable<Object> callable(Runnable task) {...}
    public static Callable<Object> callable(final PrivilegedAction<?> action) {...}
    public static Callable<Object> callable(final PrivilegedExceptionAction<?> action) {...}
    public static <T> Callable<T> privilegedCallable(Callable<T> callable) {...}
    public static <T> Callable<T> privilegedCallableUsingCurrentClassLoader(Callable<T> callable) {...}

    //样例可参考 http://blog.csdn.net/a369414641/article/details/48342253 一文。
}
```

### **67.谈谈你对 java 线程池 ThreadPoolExecutor 与 ScheduledThreadPoolExecutor 的理解及相关构造参数的含义？**

解析：

线程池由任务队列和工作线程组成，它可以重用线程来避免线程创建的开销，在任务过多时通过排队避免创建过多线程来减少系统资源消耗和竞争，确保任务有序完成；ThreadPoolExecutor 继承自 AbstractExecutorService 实现了 ExecutorService 接口，ScheduledThreadPoolExecutor 继承自 ThreadPoolExecutor 实现了 ExecutorService 和 ScheduledExecutorService 接口；ThreadPoolExecutor 有一些重要的参数来控制我们合理的使用线程池，所以我们有必要看下这些参数的含义：
```java
//有多个构造方法，最终都指向这个最多参数的构造方法
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              ThreadFactory threadFactory,
                              RejectedExecutionHandler handler) {...}
```
corePoolSize：核心运行的线程个数，也就是当超过这个范围的时候就需要将新的异步任务放入到等待队列中，小于这个数时添加进来的异步任务一般直接新建 Thread 执行；

maximumPoolSize：最大线程个数，当大于了这个值就会将准备新加的异步任务由一个丢弃处理机制来处理，大于 corePoolSize 且小于 maximumPoolSize 则新建 Thread 执行，但是当通过 newFixedThreadPool 创建的时候，corePoolSize 和 maximumPoolSize 是一样的，而 corePoolSize 是先执行的，所以他会先被放入等待队列而不会执行到下面的丢弃处理中；

workQueue：任务等待队列，当达到 corePoolSize 的时候就向该等待队列放入线程信息（默认为一个LinkedBlockingQueue）；

keepAliveTime：默认是 0，当线程没有任务处理后空闲线程保持多长时间，不推荐使用；

threadFactory：是构造 Thread 的方法，一个接口类，可以使用默认的 default 实现，也可以自己去包装和传递，主要实现 newThread 方法即可；

handler：当参数 maximumPoolSize 达到后丢弃处理的方法实现，java 提供了 5 种丢弃处理的方法，当然你也可以自己弄，主要是要实现接口 RejectedExecutionHandler 中 rejectedExecution(Runnabler, ThreadPoolExecutor e) 方法，java 默认使用的是 AbortPolicy，他的作用是当出现这种情况的时候抛出一个异常；

通常得到线程池后会调用其中的 submit 或 execute 方法去提交执行异步任务，其实 submit 方法最终会调用 execute 方法来进行操作，只是他提供了一个 Future 来托管返回值的处理而已，当你调用需要有返回值的信息时用它来处理是比较好的，这个 Future 会包装 Callable 信息。

ScheduledThreadPoolExecutor 是基于 ThreadPoolExecutor 实现的，其调用线程池的构造使用了 DelayedWorkQueue 阻塞队列实现而已。

一般性的需求推荐通过 Executors 工厂类创建线程池，其对 ThreadPoolExecutor 已经做了合适的封装，很方便使用。

深入感兴趣的可以查看[ThreadPoolExecutor的原理](http://blog.csdn.net/u010723709/article/details/50372322)一文或者[线程池源码分析-ThreadPoolExecutor](https://yq.aliyun.com/articles/39090)一文。

### **68.简单说说 CompletionService 的作用和使用场景及原理？**

解析：

CompletionService 是一个泛型接口，其实现类是 ExecutorCompletionService，其主要解决的场景是主线程提交多个任务然后希望有任务完成就处理结果，并且按照任务完成顺序逐个处理（譬如并发请求返回刷新UI的操作就可以谁请求成功就开始刷而不用等待所有OK才刷等）。CompletionService 接口如下：
```java
public interface CompletionService<V> {
    //同线程池接口方法含义
    Future<V> submit(Callable<V> task);
    Future<V> submit(Runnable task, V result);
    //阻塞等待获取下一个完成任务的结果
    Future<V> take() throws InterruptedException;
    //直接获取下一个完成任务的结果，如果没有已经完成的任务则返回null
    Future<V> poll();
    //最多等待timeout后返回，如果没有已经完成的任务则返回null
    Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException;
}
```
ExecutorCompletionService 实现类依赖于 Executor 完成实际的任务提交执行，自己主要负责结果的排队处理，AbstractExecutorService 的 invokAny 实现就依赖此类，ExecutorCompletionService 内部有一个额外的队列，每个提交给 Executor 的任务都是通过继承 FutureTask 封装过的，FutureTask 在任务结束后会回调 done 方法，所以 ExecutorCompletionService 就在继承 FutureTask 封装重写的 done 方法中将当前 FutureTask 加入额外队列，然后我们通过其 take 或者 poll 方法获取的实质就是从这个额外队列中取数据。

### **69.除过常见的 wait、notify、显示 Condition 并发协作方式外你还知道哪些 java 的并发协作工具类？**

解析：

除过基础的并发协作操作以外 java 提供的便捷并发协作工具类主要如下：

CountDownLatch：允许一个或者多个线程等待其他线程完成操作，平时我们在主线程等待其他线程结束一般都是在主线程调用其他 Thread 的 join 方法进行阻塞等待，而 join 的实质就是循环判断线程 isAlive 就进行 wait 操作，直到被 join 的线程终止后线程的 this.notifyAll 方法（JVM实现，外部不可见）会被调用进而触发 wait 结束而使 join 阻塞退出；并发包提供的 CountDownLatch 也可以实现 join 的功能，而且还提供了比 join 更多的功能，CountDownLatch 构造函数接收一个 int 参数作为计数器，如果你想等待 N 个计数点完成就传 N，当我们每个点完成时调用 CountDownLatch 的 countDown 方法则 N 就会自动减一，CountDownLatch 的 await 会阻塞当前线程直到 N 变为零为止，其采用 CAS 和 AQS（AbstractQueuedSynchronizer）实现。

CyclicBarrier：同步可循换使用的屏障，让一组线程到达一个屏障（同步点）时被阻塞，直到最后一个线程到达屏障时才会开门，所有被屏障拦截的线程才会继续干活；CyclicBarrier 默认的构造方法是 CyclicBarrier(int parties)，其参数表示屏障拦截的线程数量，每个线程调用 await 方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞；CyclicBarrier 还提供一个高级构造函数 CyclicBarrier(int parties, Runnable barrierAction) 用于在线程到达屏障时优先执行 barrierAction，方便处理复杂的业务场景；CyclicBarrier 适合并发计算数据汇总场景，其与 CountDownLatch 的主要区别是 CyclicBarrier 计数器可以 reset 而 CountDownLatch 只能用一次，其次 CyclicBarrier 提供了比 CountDownLatch 多的操作方法。

Semaphore：信号量，用来控制同时访问特定资源的线程数量，它通过协调各个线程以保证合理的使用公共资源，可以用做流量控制，譬如数据库连接场景控制等；Semaphore 的构造方法 Semaphore(int permits) 接收一个整型参数，表示可用的许可证数量，即最大并发数量，使用方法就是在线程里面首先调用 acquire 方法获取一个许可，使用完后接着调用 release 归还一个许可，还可以使用 tryAcquire 尝试获取许可，其还提供了一些状态数量获取方法，不再说明。

Exchanger：两个线程进行数据交换的交换者，其提供一个同步点，在这个同步点两个线程可以交换彼此的数据，这两个线程通过 exchange 方法交换数据，如果第一个线程先执行 exchange 方法则它会一直等待第二个线程也执行 exchange 方法，当两个线程都到达同步点时就可以交换数据将本线程生产的数据传递给对方；使用场景为并发校验等，可以进行 AB 互相 review 保障。

具体实例细节可以参考[Java并发工具类详解](http://blog.csdn.net/sunxianghuang/article/details/52277394)一文。

### **70.简单说说 ReentrantLock 和 Condition 的原理？**

解析：

ReentrantLock 锁的实现原理依赖于 CAS、AQS、LockSupport 等。

LockSupport 也位于并发包下，其提供了一组静态方法：
```java
public class LockSupport {
    //使指定线程恢复运行状态
    public static void unpark(Thread thread) {...}
    //使当前线程放弃CPU进入WAITING等待状态不往下继续执行，特别注意 park 系列方法是可以响应中断的，当有中断时 park 方法会返回且线程中断标记被设置，park 还有可能无缘无故返回，程序应该主动检测 park 等待的条件是否满足。
    public static void park(Object blocker) {...}
    public static void parkNanos(Object blocker, long nanos) {...}
    public static void parkUntil(Object blocker, long deadline) {...}
    public static Object getBlocker(Thread t) {...}
    public static void park() {...}
    public static void parkNanos(long nanos) {...}
    public static void parkUntil(long deadline) {...}
}
```
LockSupport 的 park、unpark 实现都和 CAS 类似基于 Unsafe 类的方法，Unsafe 最终调用操作系统 API。

AQS(AbstractQueuedSynchronizer) 并发工具抽象类利用了 CAS 和 LockSupport 来实现，ReentrantLock、ReentrantReadWriteLock、Semaphore、CountDownLatch 都是基于 AQS 来实现的，通过 AQS 可以简化并发工具的实现，AQS 封装了一个 int 的状态，给子类提供了查询和设置的方法，如下：
```java
private volatile int state;
protected final int getState();
protected final void setState(int newState);
protected final boolean compareAndSetState(int expect, int update);
```
用于实现锁时 AQS 可以保存锁的当前持有线程，提供的方法如下：
```java
private transient Thread exclusiveOwnerThread;
protected final void setExclusiveOwnerThread(Thread thread);
protected final Thread getExclusiveOwnerThread();
```
AQS 内部维护了一个等待队列，借助 CAS 方法实现了无阻塞算法更新。

ReentrantLock 内部有三个基于 AQS 的内部静态类，如下：
```java
abstract static class Sync extends AbstractQueuedSynchronizer;  //抽象类
static final class NonfairSync extends Sync;    //非公平实现
static final class FairSync extends Sync;   //公平实现
```
ReentrantLock 内部有一个 Sync 成员变量在构造方法中被赋值，构造方法决定了使用 NonfairSync 实例还是 FairSync 实例，默认为 NonfairSync 非公平实例，接着 ReentrantLock 的 lock 其实调用了 Sync 的 lock，unlock 调用了 Sync 的 release(1)，所以其实质是基于 AQS 的。

AQS 具体细节可以参考[AbstractQueuedSynchronizer的介绍和原理分析](http://ifeve.com/introduce-abstractqueuedsynchronizer/)和[队列同步器 AQS 详解](http://blog.csdn.net/sunxianghuang/article/details/52287968)一文。

ReentrantLock 可以查看 [ReentrantLock实现原理深入探究](http://www.importnew.com/22924.html)。

### **71.简单说说 java instanceof 的作用与 Class.isInstance 的区别？**

解析：

java 中的 instanceof 运算符用来在运行时指出对象是否是特定类的一个实例，通过返回一个布尔值来指出这个对象是否是这个特定类或者是它的子类的一个实例；用法为`result = object instanceof class`，参数 result 布尔类型，object 为必选项的实例，class 为必选项的任意已定义的对象类，如果 object 是 class 的一个实例则 instanceof 运算符返回 true，如果 object 不是指定类的一个实例或者 object 是 null 则返回 false；但是 instanceof 在 java 的编译状态和运行状态是有区别的，在编译状态中 class 可以是 object 对象的父类、自身类、子类，在这三种情况下 java 编译时不会报错，在运行转态中 class 可以是 object 对象的父类、自身类但不能是子类，当为父类、自生类的情况下 result 结果为 true，为子类的情况下为 false。

Class.inInstance(obj) 表明这个对象能不能被转化为这个类，一个对象是本身类的一个对象，一个对象能被转化为本身类所继承类（父类的父类等）和实现的接口（接口的父接口）强转，所有对象都能被 Object 的强转，凡是 null 有关的都是 false。

### **72.什么是反射？如何提高 java 反射的效率？反射的优缺点是什么？**

解析：

反射是在运行时而非编译时动态获取类型的信息（譬如接口信息、成员信息、方法信息、构造方法信息等）然后依据这些动态获取到的信息创建对象、访问修改成员、调用方法等。

通过调用`Class.forName(clzss)`方法可以访问返回一个以指定字符串 clzss 为类名的类对象，因为 java 里面任何 class 都要装载在虚拟机上才能运行，所以那个方法的作用就是装载类用的，也可以直接通过`类名.Class`获取 Class 类对象，还可以通过`实例.getClass()`方法获取 Class 类对象；Class 类提供了许多方法，譬如可以获取与类名称有关的信息，可以获取类中定义的字段 Field（静态和实例变量都被称为字段，可获取 public 与非 public 的字段）、Field 也提供了许多获取字段或者设置字段具体信息的操作，可以获取类中定义的方法 Method（静态方法和非静态方法都是方法，可获取 public 与非 public 的方法）、Method 也提供了许多获取方法信息、修饰符、参数、返回值、注解等、调用对象方法的操作；`Class.newInstance()`方法可以创建对象实例（只是用默认无参构造），不过也提供了一些其他方法获取所有构造方法；Class 还提供了类型检查、类型判断、修饰符、父类、接口、注解、内部类等操作的方法。

从 JVM 的角度使用关键字 new 创建一个类的时候这个类可以没有被加载，但是使用 newInstance() 方法的时候就必须保证这个类已经加载且这个类已经连接了，而完成上面两个步骤是 Class 的静态方法 forName() 所完成的，这个静态方法调用了启动类加载器（即加载 java API 的那个加载器），所以 newInstance() 实际上是把 new 这个方式分解为两步，即首先调用 Class 加载方法加载某个类然后实例化。

提高反射效率要考虑的问题如下，首先保证反射 API 最小化，譬如尽量使用 getMethod 直接获取而不是 getMethods 遍历查找获取；其次需要多次动态创建一个类的实例时尽可能的使用缓存。

反射虽然很灵活，有些时候能够使得写的代码变的大幅精简，但是在用的时候一定要注意具体的应用场景，因为反射的优缺点如下： 

|特点|说明|
|----|----|
|优点|能够运行时动态获取类的实例，大大提高系统的灵活性和扩展性；与 Java 动态编译相结合可以实现无比强大的功能；|
|缺点|使用反射的性能较低；使用反射相对来说不安全；破坏了类的封装性，可以通过反射获取这个类的私有方法和属性；有些情况下反射受版本兼容问题而不稳定，譬如 Android；运行时检查，故难提前发现问题；| 

### **73.Java 泛型在编译时被擦出后还有办法在运行时获取泛型信息吗？**

解析：

有办法，正因为泛型在编译时擦出所以会有如下现象：
```java
Class c1 = new ArrayList<Integer>().getClass();  
Class c2 = new ArrayList<String>().getClass();   
System.out.println(c1 == c2);  //true
```
`ArrayList<Integer>`和`ArrayList<String>`在编译的时候是完全不同的类型，我们无法在写代码时把一个 String 类型的实例加到`ArrayList<Integer>`中，但是在程序运行时的确会输出 true，因为不管是`ArrayList<Integer>`还是`ArrayList<String>`在编译时都会被编译器擦除成了`ArrayList`。

运行时被擦出的泛型信息获取其实主要依赖反射包下的 ParameterizedType 泛型参数类型接口，应用也比较广泛，譬如 Gson 框架就是这么搞的，如下是一段获取代码示例：
```java
Map<String, Integer> map = new HashMap<String, Integer>() {};  //大括号非常重要，相当于匿名内部类
Type type = map.getClass().getGenericSuperclass();  
ParameterizedType parameterizedType = ParameterizedType.class.cast(type);  
for (Type typeArgument : parameterizedType.getActualTypeArguments()) {  
    System.out.println(typeArgument.getTypeName());  
}  
/* Output 
java.lang.String 
java.lang.Integer 
*/  
```
上面这段代码展示了如何获取 map 这个实例所对应类的泛型信息，其中最重要的就是第一行 map 实例的创建是一个匿名内部类且为 HashMap 的子类，泛型参数限定为 String 和 Integer，Java 引入泛型擦除的原因是避免因为引入泛型而导致运行时创建不必要的类，所以我们可以通过定义类的方式在类信息中保留泛型信息，从而在运行时获得这些泛型信息，所以 Java 的泛型擦除是有范围的，即类定义中的泛型是不会被擦除的。 

感兴趣的可以去看下[Gson 官方源码解析的代码](https://github.com/google/gson/blob/2b15334a4981d4e0bd4f2c2d34b8978a4167f36a/gson/src/main/java/com/google/gson/reflect/TypeToken.java#L77)。

额外知识点补充[《Java中的Type详解》](http://loveshisong.cn/%E7%BC%96%E7%A8%8B%E6%8A%80%E6%9C%AF/2016-02-16-Type%E8%AF%A6%E8%A7%A3.html)。

### **74.java 反射的基本原理是什么？**

解析：

java 反射机制是围绕 Class 类展开的，JVM 使用 ClassLoader 将 class 字节码文件加载到方法区内存中，如下：
```java
Class clazz = ClassLoader.getSystemClassLoader().loadClass("com.package.DemoClass");
```
可见 ClassLoader 根据类的完全限定名加载类并返回了一个 Class 对象，而 java 反射的所有起源都是从这个 Class 类开始的；为了提高反射的性能，Class 类内部有一个 useCaches 静态变量来标记是否使用缓存，这个值可以通过外部配置项 sun.reflect.noCaches 进行开关，Class 类内部提供了一个 ReflectionData 内部类用来存放反射数据的缓存，并声明了一个 reflectionData 字段，由于稍后进行按需延迟加载并缓存，所以这个域并没有指向一个实例化的 ReflectionData 对象；反射方法真正调用的主要流程都是先从 Class 内部的 reflectionData 缓存中读取数据，如果没有缓存数据则从 JVM 中去请求数据然后设置到缓存中供下次使用；在 JVM 中实质 Class 文件会以字节码存在，反射基本操作其实就是查找匹配调用返回的过程。

具体可以参考[《JAVA反射原理》](http://www.cnblogs.com/techspace/p/6931397.html)和[《Java反射在JVM的实现》](http://www.importnew.com/21211.html)。

### **75.简单谈谈 java 类加载器的加载机制？**

解析：

通过 java 命令运行 java 程序的步骤就是指定包含 main 方法的完整类名以及一个 classpath 类路径，类路径可以有多个，对于直接的 class 文件路径就是 class 文件的根目录，对于 jar 包文件路径是 jar 包的完整路径，包含 jar 包名字；java 运行时会根据类的完全限定名寻找并加载，寻找的方式基本就是在系统类和指定的路径中寻找，如果是 class 文件的根目录则直接查看是否有对应的子目录及文件，如果是 jar 包则首先在内存中解压文件，然后再查看是否有对应的类；负责类加载的类就是 ClassLoader 类加载器，它的输入是完全限定的类名，输出是 Class 对象，java 虚拟机中可以安装多个类加载器，系统默认主要有三个类加载器，每个类负责加载特定位置的类，也可以自定义类加载器，自定义的加载器必须继承 ClassLoader，如下：

|加载器|说明|
|----|----|
|启动类加载器(Bootstrap ClassLoader)|此加载器为虚拟机实现的一部分，不是 java 语言实现的，一般为 C++ 实现，主要负责加载 java 基础类（譬如`<JAVA_HOME>/lib/rt.jar`，常用的 String、List 等都位于此包下），启动类加载器无法被 java 程序直接引用。|
|扩展类加载器(Extension ClassLoader)|此加载器实现类为`sun.misc.Launcher$ExtClassLoader`，负责加载 java 的一些扩展类（一般为`<JAVA_HOME>/lib/ext`目录下的 jar 包），开发者可直接使用。|
|应用程序类加载器(Application ClassLoader)|此加载器实现类为`sun.misc.Launcher$AppClassLoader`，负责加载应用程序的类，包括自己写的和引用的第三方类库，即 classpath 类路径中指定的类，开发者可直接使用，一个程序运行时会创建一个这个加载器，程序中用到加载器的地方如果没有特殊指定一般都是这个加载器，所以也被称为 System 系统类加载器。|

这三个加载器具备父子委派关系（非继承父子关系），在 java 中每个类都是由某个类加载器的实体来载入的，所以在 Class 类的实体中都会有字段记录载入它的类加载器的实体（当为 null 时，其指 Bootstrap ClassLoader），在 java 类加载器中除了引导类加载器（既 Bootstrap ClassLoader），所有的类加载器都有一个父类加载器（因为他们本身自己就是 java 类），子 ClassLoader 有一个变量 parent 指向父 ClassLoader，在子 ClassLoader 加载类时一般会先通过父 ClassLoader 加载，所以在加载一个 class 文件时首先会判断是否已经加载过了，加载过则直接返回 Class 对象（一个类只会被一个 ClassLoader 加载一次），没加载过则先让父 ClassLoader 去加载，如果加载成功返回得到的 Class 对象，父没有加载成功则尝试自己加载，自己加载不成功则抛出 ClassNotFoundException，整个这个加载流程就是双亲委派模型，即优先让父 ClassLoader 加载；双亲委派可以从优先级的策略上避免 Java 类库被覆盖的问题，例如类 java.long.Object 存放在 rt.jar 中，无论哪个类加载器要加载这个类最终都会委派给启动类加载器进行加载，因此 Object 类在程序的各种类加载器环境中都是同一个类，相反如果我们自己写了一个类名为 java.long.Object 且放在了程序的 classpath 中，那系统中将会出现多个不同的 Object 类，java 类型体系中最基础的行为也无法保证，所以一般遵循双亲委派的加载器就不会存在这个问题。

类加载机制中的双亲委派模型只是一般情况下的机制，有些时候我们可以自定义加载顺序（不建议）就不用遵守双亲委派模型了，同时以 java 开头的类也不能被自定义类加载器加载，这是 java 安全机制保证的；ClassLoader 一般是系统提供的，不需要自己实现，不过通过自定义 ClassLoader 可以实现一些灵活强大的功能，譬如热部署（不重启 Java 程序的情况下动态替换类实现）、应用的模块化和隔离化（不同 ClassLoader 可以加载相同的类，但是互相隔离互不影响，tomcat 就是利用这个特性管理多 web 应用的）、灵活加载等，通过自定义类加载器我们可以加载其它位置的类或 jar，自定义类加载器主要步骤为继承 java.lang.ClassLoader 然后重写父类的 findClass 方法，之所以一般只重写这一个方法是因为 JDK 已经在 loadClass 方法中帮我们实现了 ClassLoader 搜索类的算法，当在 loadClass 方法中搜索不到类时 loadClass 方法会主动调用 findClass 方法来搜索类，所以我们只需重写该方法即可，如没有特殊的要求，一般不建议重写 loadClass 搜索类的算法。

JVM 在判定两个 Class 是否相同时不仅会判断两个类名是否相同而且会判断是否由同一个类加载器实例加载的，只有两者同时满足的情况下 JVM 才认为这两个 Class 是相同的，就算两个 Class 是同一份 class 字节码文件，如果被两个不同的 ClassLoader 实例所加载 JVM 也会认为它们是两个不同 Class，比如字节码文件 Simple.class 被 ClassLoaderA 和 ClassLoaderB 这两个类加载器分别加载并分别得到了 Class 实例，而对于 JVM 来说它们是两个不同的实例对象，但它们确实是同一份字节码文件，当试图将这个 Class 实例生成具体的对象进行转换时就会抛运行时异常 java.lang.ClassCaseException 提示这是两个不同的类型（切记，Android 开发中我已经踩坑过一次）；此外一个 ClassLoader 创建时如果没有指定 parent 则 parent 默认就是 AppClassLoader。

关于 Java 类加载器的细节可以参考[《深入分析Java ClassLoader原理》](http://blog.csdn.net/xyang81/article/details/7292380)一文。

### **76.java 能不能自己写个类也叫 java.lang.String？**

解析：

一般情况下不能，因为类加载采用双亲委托机制，这样可以保证 parent 类加载器优先，也就是总是使用 parent 类加载器能找到的类，这样总是使用 java 系统提供的 String 类，因为每个类加载器加载类时先委托给其上级类加载器，java.lang.String 在 BootStrap 中最先加载，但是我们可以自己写一个类加载器（譬如 parent 设置 null 等）来加载我们自己写的 java.lang.String 类，当编写自己的类加载器时我们首先让自定义的类加载器继承 ClassLoader，然后重写 loadClass 方法与 findClass 方法，loadClass 中先调用父类的 loadClass，然后调用 findClass，通常情况下只重写覆盖 findClass 就可以了，当然我们还可以重写 defineClass 方法让自定义的类加载器用于解密自己写的已加密的 class 字节码，这样即使别人拥有该 class 文件也无法被系统的类加载器正常加载，切记类的包路径和类加载器决定类的唯一性。

### **77.简单谈谈类加载顺序流程及实例初始化机制？**

解析：

谈类初始化机制实质是要包含类加载机制的，类加载机制实质主要就是类加载器原理，类加载器原理的核心是双亲委派父优先级加载，只有这样的机制才保证了类被加载的唯一性，只是类加载器机制是宏观的概述，往细了说就涉及类初始化机制。类从被加载到虚拟机内存中开始到卸载出内存为止的整个生命周期包括了加载、验证、准备、解析、初始化、使用和卸载这 7 个阶段，其中验证、准备和解析这三个部分统称为连接，加载、验证、准备、初始化和卸载这五个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班的“开始”（仅仅指的是开始，而非执行或者结束，因为这些阶段通常都是互相交叉的混合进行，通常会在一个阶段执行的过程中调用或者激活另一个阶段），而解析阶段则不一定（它在某些情况下可以在初始化阶段之后再开始，这是为了支持 Java 语言的运行时绑定。

对于生命周期中的加载阶段虚拟机规范中没强行约束，这点可以交给虚拟机的的具体实现自由把握，但是对于生命周期中的初始化阶段虚拟机规范严格规定了如下几种情况（如果类未初始化会对类进行初始化）：

1. 创建类的实例；
2. 访问类的静态变量（除常量，被 final 修辞的静态变量），因为常量一种特殊的变量，编译器把他们当作值（value）而不是域（field）来对待，如果代码中用到了常变量编译器并不会生成字节码来从对象中载入域的值，而是直接把这个值插入到字节码中，这是一种很有用的优化，但是如果你需要改变 final 域的值那么每一块用到那个域的代码都需要重新编译；
3. 访问类的静态方法；
4. 反射（例如 Class.forName("com.package.Test")）；
5. 当初始化一个类时发现其父类还未初始化则先触发父类的初始化；
6. 虚拟机启动时定义了 main() 方法的那个类先初始化；

以上情况称为称对一个类进行主动引用，除此种情况之外被称为被动引用（譬如子类调用父类的静态变量，子类不会被初始化，只有父类被初始化，对于静态字段只有直接定义这个字段的类才会被初始化；通过数组定义来引用类不会触发类的初始化；访问类的常量不会初始化类），被动引用均不会触发类的初始化，接口的加载过程与类的加载过程稍有不同，接口中不能使用 static{} 块，当一个接口在初始化时，并不要求其父接口全部都完成了初始化，只有真正在使用到父接口时（例如引用接口中定义的常量）才会初始化。

加载阶段是类加载过程的第一个阶段，虚拟机需要完成三件事情，通过一个类的全限定名来获取定义此类的二进制字节流，将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构，在 Java 堆中生成一个代表这个类的 java.lang.Class 对象作为方法区这些数据的访问入口；该阶段使用类加载器完成，加载阶段与连接阶段的部分内容（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段尚未完成连接阶段可能已经开始。 

验证是连接阶段的第一步，目的是为了确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全；不同虚拟机对类验证的实现可能有所不同，但大致都会完成下面四个阶段的验证：文件格式验证、元数据验证、字节码验证和符号引用验证，验证阶段对于虚拟机的类加载机制来说不一定是必要的阶段，如果所运行的全部代码确认是安全的则可以关闭大部分的类验证措施以缩短虚拟机类加载时间。

准备阶段是为类的静态变量分配内存并将其初始化为默认值，这些内存都将在方法区中进行分配，准备阶段不分配类中的实例变量的内存，实例变量将会在对象实例化时随着对象一起分配在 Java 堆中，譬如`public static int value=123;`在准备阶段 value 初始值为 0 ，在初始化阶段才会变为 123。

解析阶段是虚拟机将常量池内的符号引用替换为直接引用的过程；符号引用以一组符号来描述所引用的目标，符号可以是任何形式的字面量，只要使用时能无歧义地定位到目标即可，符号引用与虚拟机实现的内存布局无关，引用的目标并不一定已经加载到内存中；直接引用可以是直接指向目标的指针、相对偏移量或是一个能间接定位到目标的句柄，直接引用是与虚拟机实现的内存布局相关的，如果有了直接引用则引用的目标必定已经在内存中存在。

初始化是类加载过程的最后一步，前面的类加载过程除了在加载阶段用户应用程序可以通过自定义类加载器参与之外，其余动作完全由虚拟机主导和控制，到了初始化阶段才真正开始执行类中定义的 Java 程序代码，初始化阶段是执行类构造器`<clinit>()`方法的过程，`<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块（static{}块）中的语句合并产生的。

属性、方法、构造方法和块都是类中的成员，在创建类的对象时各成员的执行顺序为：

1. 父类静态成员和静态初始化快，按在代码中出现的顺序依次执行；
2. 子类静态成员和静态初始化块，按在代码中出现的顺序依次执行；
3. 父类的实例成员和实例初始化块，按在代码中出现的顺序依次执行；
4. 执行父类的构造方法；
5. 子类实例成员和实例初始化块，按在代码中出现的顺序依次执行；
6. 执行子类的构造方法；

非静态初始化块主要是用于对象的初始化操作，在每次创建对象的时都要调用一次，其执行顺序在构造方法之前；由于非静态成员不能在静态方法中使用，同样也不能在静态初始化块中，因此静态初始化块主要用于初始化静态变量和静态方法，静态初始化块只在类第一次加载到内存时调用一次，并非一定要创建对象才执行，静态初始化块比非静态初始化块先执行。

### **78.说说下面程序的运行结果及原因？**
```java
class Father {
    {
        System.out.println("FB0 ......");
    }

    static {
        System.out.println("FSB0 ......");
        //d\a无法在此打印，执行在此时d\a还位初始化
    }

    static int a = 1;

    static {
        System.out.println("FSB1 a:"+a);
        //d无法在此打印，执行在此时d还位初始化
    }

    int b = 1;
    static {
        a++;
        System.out.println("FSB2 a:"+a);
        //d无法在此打印，执行在此时d还位初始化
    }

    final static int c = 1;
    static {
        System.out.println("FSB3 c:"+c);
        //d无法在此打印，执行在此时d还位初始化
    }

    public Father() {
        a++;
        b++;
        System.out.println("F CONSTTRUCTOR a:"+a+", b:"+b+", c:"+c+", d:"+d);
    }

    {
        a++;
        b++;
        System.out.println("FB4 a:"+a+", b:"+b+", c:"+c);
        //d无法在此打印，执行在此时d还位初始化
    }

    final int d = 1;

    public void func1() {
        a++;
        b++;
        System.out.println("F func1 a:"+a+", b:"+b+", c:"+c+", d:"+d);
    }

    public static void func2() {
        a++;
        System.out.println("F func1 a:"+a+", c:"+c);
    }
}

class Child extends Father {
    {
        System.out.println("CB0 ......");
    }

    static {
        System.out.println("CSB0 ......");
        //d\a无法在此打印，执行在此时d\a还位初始化
    }

    static int a = 1;

    static {
        System.out.println("CSB1 a:"+a);
        //d无法在此打印，执行在此时d还位初始化
    }

    int b = 1;
    static {
        a++;
        System.out.println("CSB2 a:"+a);
        //d无法在此打印，执行在此时d还位初始化
    }

    final static int c = 1;
    static {
        System.out.println("CSB3 c:"+c);
        //d无法在此打印，执行在此时d还位初始化
    }

    public Child() {
        a++;
        b++;
        System.out.println("C CONSTTRUCTOR a:"+a+", b:"+b+", c:"+c+", d:"+d);
    }

    {
        a++;
        b++;
        System.out.println("CB4 a:"+a+", b:"+b+", c:"+c);
        //d无法在此打印，执行在此时d还位初始化
    }

    final int d = 1;

    public void func1() {
        a++;
        b++;
        System.out.println("C func1 a:"+a+", b:"+b+", c:"+c+", d:"+d);
    }

    public static void func2() {
        a++;
        System.out.println("C func1 a:"+a+", c:"+c);
    }

    public static class BBQ {
        static {
            System.out.println("BBQ");
        }
    }
}

public class Demo {
    public static void main(String[] args) {
        Child.BBQ bbq = new Child.BBQ();
        Father.func2();
        Child.func2();
        Father father = new Child();
        father.func1();
    }
}
```
解析：

此题考察类实例化初始化流程，所得结果如下，原因如上面 77 题所示：
```
BBQ
FSB0 ......
FSB1 a:1
FSB2 a:2
FSB3 c:1
F func1 a:3, c:1
CSB0 ......
CSB1 a:1
CSB2 a:2
CSB3 c:1
C func1 a:3, c:1
FB0 ......
FB4 a:4, b:2, c:1
F CONSTTRUCTOR a:5, b:3, c:1, d:1
CB0 ......
CB4 a:4, b:2, c:1
C CONSTTRUCTOR a:5, b:3, c:1, d:1
C func1 a:6, b:4, c:1, d:1
```

### **79.下面程序的运行结果是什么，为什么？**
```java
class SingleTon {
    private static SingleTon singleTon = new SingleTon();
    public static int count1;
    public static int count2 = 0;

    private SingleTon() {
        count1++;
        count2++;
    }

    public static SingleTon getInstance() {
        return singleTon;
    }
}

public class Demo {
    public static void main(String[] args) {
        SingleTon singleTon = SingleTon.getInstance();
        System.out.println("count1=" + singleTon.count1);
        System.out.println("count2=" + singleTon.count2);
    }
}
```

解析：
```
count1=1
count2=0
```
原因为`SingleTon singleTon = SingleTon.getInstance();`调用了类的静态方法，所以触发类的初始化，类加载的时候在准备过程中为类的静态变量分配内存并初始化默认值`singleton=null，count1=0，count2=0`，类初始化时为类的静态变量赋值和执行静态代码快，singleton 赋值为 new SingleTon() 调用类的构造方法，调用类的构造方法后 count=1 且 count2=1，继续为 count1 与 count2 赋值，此时 count1 没有赋值操作，所有 count1 为 1，但是 count2 执行赋值操作就变为 0。

### **80.下面程序的运行结果是什么，为什么？**
```java
class SuperClass {
    static {
        System.out.println("superclass init");
    }
    public static int value = 123;
}

class SubClass extends SuperClass {
    static {
        System.out.println("subclass init");
    }
}

class ConstClass {
    static {
        System.out.println("constclass init");
    }
    public static final String HELLOWORLD = "hello world";
}

public class Demo {
    public static void main(String[] args) {
        System.out.println(SubClass.value); //1
        SubClass[] array = new SubClass[10];    //2
        System.out.println(ConstClass.HELLOWORLD);  //3
        new SubClass();     //4
    }
}
```

解析：
```
superclass init
123
hello world
subclass init
```
因为被动引用不会触发类的初始化，主动引用才会触发，上面 1 为子类调用父类的静态变量，子类不会被初始化，只有父类被初始化，对于静态字段只有直接定义这个字段的类才会被初始化；2 为通过数组定义来引用类，所以不会触发类的初始化；3 为访问类的常量，所以不会初始化类；4 就是主动引用了，自然会初始化了。

### **81.下面程序的输出结果是什么？**
```java
class Singleton {
    public static class Inner {
        public static Singleton testInstance = new Singleton(3);
        static {
            System.out.println("SI static.");
        }
    }

    public static Singleton getInstance() {
        return Inner.testInstance;
    }

    public Singleton(int i) {
        System.out.println("construct i="+i);
    }

    static {
        System.out.println("S static.");
    }

    public static Singleton testOut = new Singleton(1);
}

public class Demo {
    public static void main(String args[]){
        Singleton t = new Singleton(2);
        Singleton.getInstance();
    }
}
```

解析：
```
S static.
construct i=1
construct i=2
construct i=3
SI static.
```

### **82.下面程序的执行结果是什么？**
```java
public class Demo {
    private String baseName = "base";

    public Demo() {
        callName();
    }

    public void callName() {
        System.out.println(baseName);
    }

    static class Sub extends Demo {
        private String baseName = "sub";

        public void callName() {
            System.out.println(baseName) ;
        }
    }

    public static void main(String[] args) {
        Demo demo = new Sub();
    }
}
```

解析：
```
null
```
当虚拟机创建一个新的实例时，都需要在堆中为保存对象的实例分配内存，所有在对象的类中和它的超类中声明的变量（包括隐藏的实例变量）都要分配内存，一旦虚拟机为新的对象准备好堆内存，它立即把实例变量初始化为默认的初始值，一旦虚拟机完成了为新的对象分配内存和为实例变量初始化为默赋予正确认的初始值后，接下来就会调用对象的实例初始化方法。

在初始化 Sub 对象前首先在堆区开辟内存并将子类中的 baseName 和父类中的 baseName（已被隐藏）均赋为 null，接下来执行对象的初始化过程，由于 Sub 类的构造函数没有写，初始化首先调用 Base 类的，故 baseName = “base” 是父类的 baseName 赋值，接着父类构造函数里调用 callName() 实质是多态调用子类 Sub 的 callName() 方法，而此时父类构造还未执行完毕，暂时轮不到子类一般属性初始化和构造调用，所以子类的 baseName 变量还未赋值，所以还是 null。

### **83.简单说说 java 的 Class.forName 和 ClassLoader.loadClass 方法的区别？**

解析：

一个 Java 类加载到 JVM 中会经过三个步骤，装载（查找和导入类或接口的二进制数据）、链接（校验：检查导入类或接口的二进制数据的正确性；准备：给类的静态变量分配并初始化存储空间；解析：将符号引用转成直接引用；）、初始化（激活类的静态变量的初始化 Java 代码和静态 Java 代码块）。

对于 Class.forName 方法来说：
```java
//Class.forName(className) 方法内部实际调用的方法是 Class.forName(className, true, classloader);
public static Class<?> forName(String name, boolean initialize, ClassLoader loader)
```
三个参数的含义分别为：
```
name：要加载 Class 的名字
initialize：是否要初始化
loader：指定的 classLoader
```
对于 ClassLoader.loadClass 方法来说：
```java
//ClassLoader.loadClass(className) 方法内部实际调用的方法是 ClassLoader.loadClass(className, false);
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException
```
两个参数的含义分别为：
```
name：class 的名字
resolve：是否要进行链接
```
所以通过传入的参数可以知道 Class.forName 方法执行之后已经对被加载类的静态变量分配完了存储空间，而 ClassLoader.loadClass 方法并没有一定执行完链接这一步；当想动态加载一个类且这个类又存在静态代码块或者静态变量而你在加载时就想同时初始化这些静态代码块则应偏向于使用 Class.forName 方法。

### **84.什么是 Java 的注解？**

解析：

注解是给程序添加一些信息，用`@`字符开头，这些信息用于修饰它后面紧挨着的其他代码元素，比如类、接口、字段、方法、方法参数、构造方法等，注解可以被编译器、程序运行时环境及其他工具使用，用于增强或者修改程序行为等。

Java 注解可以分为内置注解和自定义注解，常见的内置注解有`@Override`、`@Deprecated`、`@SuppressWarnings`等，我们会发现注解基本都是通过一个简单的`@`声明就可以达到某种效果，这些都是声明式编程风格，在这种风格下程序都由三个组件组成：
```
1、声明的关键字和语法本身；
2、系统框架库，负责解释执行声明式的语句；
3、应用程序，使用声明式风格写程序；
```
这种风格降低了编程的难度，为应用程序员提供了更加高级的语言，使我们可以站在更高层次角度思考解决问题而不用过多的关心底层实现，所以注解一般适合框架开发，用来指定规则给使用框架的人创建便利。

### **85.Java 如何创建自定义注解？注解是如何工作的？**

解析：

下面是一个自定义注解`@Todo`：
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@interface Todo {
    public enum Priority {LOW, MEDIUM, HIGH}
    
    public enum Status {STARTED, NOT_STARTED}

    String author() default "Yash";
    Priority priority() default Priority.LOW;
    Status status() default Status.NOT_STARTED;
}
```
下面的例子演示了如何使用上面的自定义注解：
```java
@Todo(priority = Todo.Priority.MEDIUM, author = "Yashwant", status = Todo.Status.STARTED)
public void incompleteMethod1() {
//Some business logic is written
//But it’s not complete yet
}
```
接着我们解释下上面这个自定义注解，Java 在 java.lang.annotation 包下提供了几种元注解用来注解其他的注解：
```java
//注解是否将包含在 JavaDoc 中，表示是否将注解信息添加在 java 文档中。
@Documented

//什么时候使用该注解，定义该注解的生命周期。
//取值 RetentionPolicy.SOURCE 表示在编译阶段丢弃，这些注解在编译结束之后就不再有任何意义，所以它们不会写入字节码，@Override、@SuppressWarnings 都属于这类注解。
//取值 RetentionPolicy.CLASS 在类加载的时候丢弃，在字节码文件的处理中有用，注解默认使用这种方式。
//取值 RetentionPolicy.RUNTIME 表示始终不会丢弃，运行期也保留该注解，因此可以使用反射机制读取该注解的信息，我们自定义的注解通常使用这种方式。
@Retention

//表示该注解用于什么地方，如果不明确指出，该注解可以放在任何地方，可以同时指定多个地方。
//取值 ElementType.TYPE 用于描述类、接口或 enum 声明。
//取值 ElementType.FIELD 用于描述实例变量。
//取值 ElementType.METHOD 用于描述方法。
//取值 ElementType.PARAMETER 用于描述参数。
//取值 ElementType.CONSTRUCTOR 用于描述构造方法。
//取值 ElementType.LOCAL_VARIABLE 用于描述本地局部变量。
//取值 ElementType.ANNOTATION_TYPE 用于另一个注释。
//取值 ElementType.PACKAGE 用于记录 java 文件的 package 信息。
@Target

//是否允许子类继承该注解，如果父类使用了注解子类默认是不继承的。
@Inherited
```
注解的可用的类型包括以所有基本类型、String、Class、enum、Annotation 及以上类型的数组形式，元素不能有不确定的值（即要么有默认值，要么在使用注解的时候提供元素的值），而且元素不能使用 null 作为默认值，注解在只有一个元素且该元素的名称是 value 的情况下在使用注解的时候可以省略 “value=” 直接写值即可，如下：
```java
@interface Author {
    String value();
}

@Author("Yashwant")
public void someMethod() {
}
```
所以上面就是自定义注解并将其应用在业务逻辑上的流程，接着我们需要使用反射机制写一个用户程序调用我们的注解，如下代码使用了上面的注解：
```java
Class businessLogicClass = BusinessLogic.class;
for(Method method : businessLogicClass.getMethods()) {
    Todo todoAnnotation = (Todo)method.getAnnotation(Todo.class);
    if(todoAnnotation != null) {
        System.out.println(" Method Name : " + method.getName());
        System.out.println(" Author : " + todoAnnotation.author());
        System.out.println(" Priority : " + todoAnnotation.priority());
        System.out.println(" Status : " + todoAnnotation.status());
    }
}
所以说注解的工作原理实质对于 RetentionPolicy.CLASS 类型是通过注解处理器（AbstractProcessor等），RetentionPolicy.RUNTIME 类型通过反射获取注解信息进行解析处理。

### **86.？**

http://www.importnew.com/1796.html

http://www.cnblogs.com/javaee6/p/3714716.html
http://www.cnblogs.com/tengpan-cn/p/5869099.html
https://www.2cto.com/kf/201608/535046.html
http://blog.csdn.net/qq_16216221/article/details/71600535


### **80.？**

解析：

http://www.codes51.com/itwd/1166147.html
http://blog.csdn.net/u010590685/article/details/47066865

### **81.**

解析：



http://zangweiren.iteye.com/category/34977
 http://www.cnblogs.com/ggmfengyangdi/p/5761632.html




http://www.importnew.com/15246.html  java注解处理器
＼finalize原理＼垃圾回收、ASM、AOP（动态代理为基础），微信题，java SPI

### **.谈谈 Java 的 NIO 与内存映射，，**

java面试大全
http://www.importnew.com/22083.html
http://bbs.51cto.com/thread-1473565-1.html


http://blog.csdn.net/ck1600259860/article/details/50730871
