# **内存分配**
- Author：WeicongLee
- Data：2019.05.02

## **Java内存分配主要包括以下几个区域：**
1. ### 寄存器：我们在程序中无法控制
2. ### 栈：存放基本类型的数据和对象的引用
3. ### 堆：存放用new产生的数据，如对象
4. ### 静态域：存放对象中用static定义的静态成员
5. ### 常量池：存放常量
6. ### 非RAM（随机存取存储器）存储：硬盘等永久存储空间

## **一、Java内存中的栈**
1）特点：线程私有，局部变量表，操作栈，动态链接，方法出口，**对象引用（指向堆）**
2）在函数中定义的一些**基本类型的变量数据和对象的引用变量**都在函数的栈内存中分配
3）引用变量是普通变量，定义时在栈中分配，引用变量在程序运行到其作用域（方法域）之外后被释放
4）栈中的变量指向堆内存的变量，可以理解为引用就是Java中的指针
#### 当在一段代码块定义一个变量时，Java就在栈中为这个变量分配内存空间，*当该变量退出该**作用域**后，Java会自动释放掉为该变量所分配的内存空间*，该内存空间可以立即被另作他用。


## **二、Java内存中的堆**
1）特点：线程共享，所有的对象，数组
2）在堆中分配的内存，由Java虚拟机的自动垃圾回收器来管理
3）数组和对象本身在堆中分配，即使程序运行到使用new产生数组或者对象的语句所在的代码块之外，数组和对象本身占据的内存也不会被释放
4）数组和对象在**没有引用变量指向它的时候，才变为垃圾**，不能再被使用，但仍然占据内存空间不不放，**在随后的一个不确定的时间被垃圾回收器释放掉**。这也是Java比较占内存的原因。

## **三、方法区**
1）特点：线程共享，存储类的信息、常量、静态变量、编译后的代码
2）在不同JDK版本中，方法区中存储的数据是不一样的
3）JDK1.6之前，运行时**常量池**是方法区的一个部分，同时方法区里面存储了类的**元数据信息、静态变量、即时编译器编译后的代码**等（即加载类时需要加载的 信息，包括版本、field、方法、接口等信息）；
### 4）JDK1.7后，**JVM已经将运行时常量池从方法区中移了出来**，在JVM堆开辟了一块区域存放常量池。

## **四、常量池**
常量池指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。除了包含代码中所定义的各种基本类型（如int、long等）和对象型（如String及数组）的 **常量值(final)** 还包含一些以文本形式出现的符号引用，比如：
- 类和接口的全限定名
- 字段的名称和描述符
- 方法的名称和描述符

*虚拟机必须为**每个被装载的类**维护一个常量池。* 常量池就是该类型所用到常量的一个有序集合，包括直接常量（String、Integer和Floating point常量）和其他类型，字段和方法的符号引用，**字符串常量池是全局共享的**。

对于String常量，它的值是在常量池中的。而**JVM中的常量池在内存当中是以表的形式存在的**，对于String类型，有一张固定长度的表用来存储文字字符串值，注意：**该表只存储文字字符串值，不存储符号引用**。

## **五、堆与栈的区别**
### Java的堆是一个运行时数据区。堆是由垃圾回收来负责的。
### 1）堆
- 存放的数据：基本类型的变量数据和对象的引用变量
- 优势：可以动态地分配内存大小、生存期也不必事先告诉编译器，因为它是在运行时动态分配内存的，Java的垃圾收集器会自动收走这些不再使用的数据
- 缺点：由于要在运行时动态分配内存，存取速度较慢
### 2）栈
- 存放的数据：所有的对象，数组
- 优势：存取速度比堆要快，仅次于寄存器，栈数据可以共享
- 缺点：存在栈中的的数据大小与生存期必须是确定的，缺乏灵活性

### 3）总结：
这里我们主要关心栈、堆和常量池。对于栈和常量池的对象可以共享，对于堆中的对象不可以共享。栈中的数据大小和生命周期是可以确定的，当没有引用指向数据时，这个数据就会消失。堆中的对象由垃圾回收器负责回收，因此大小和生命周期不需确定，具有很大的灵活性。

## **六、字符串内存分配**
对于字符串，其**对象的引用都是存储在栈中的**，*如果是编译器已经创建好（直接用双引号定义的）的就存储在常量池中，如果运行期（new出来的）才能确定的就存储在堆中*。对于equals相等的字符串，**在常量池中永远只有一份，在堆中有多份**。

### 面试题：String str4 = new String(“abc”) 创建多少个对象？
1） 在常量池中查找是否有“abc”对象
- 有则返回对应的引用实例
- 没有则创建对应的实例对象

2）在堆中 new 一个 String("abc") 对象

3）将对象地址赋值给str4,创建一个引用

所以，常量池中没有“abc”字面量则创建两个对象，否则创建一个对象，以及创建一个引用