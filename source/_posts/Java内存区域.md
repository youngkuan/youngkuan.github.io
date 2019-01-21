---
title: Java内存区域
date: 2019-01-10 12:04:56
tags: 
    - Java堆
    - 方法区
    - Java虚拟机栈
categories: 深入理解Java虚拟机
---

# Java内存区域

Java虚拟机在执行Java程序的过程中，把其所管理的内存划分成多个区域，如下图所示。每个数据区域都有各自的用途，有的区域随着虚拟机进程的启动而存在，是线程公有的；有些区域依赖于特定的线程而存在，是线程私有的。
![image](Java内存区域/运行时内存.png)
<!-- more -->
## 程序计数器

程序计数器是一小块内存区域，可以看作是当前线程执行的字节码的行号指示器。每条线程都独立拥有一个程序计数器，因此程序计数器是线程私有的。

## Java虚拟机栈

Java虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行时都会创建一个栈帧（stack frame）用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每个方法从被调用开始执行到执行完成返回的过程，就对应这一个栈帧在虚拟机栈中入栈和出栈的过程。Java内存区域经常被简单地划分为堆内存和栈内存，其中的栈内存粗略来讲就是指的Java虚拟机栈，当然实际上内存划分肯定更加复杂。虚拟机栈与程序计数器一样的线程私有的。  
局部变量表存放了编译器可知的各种基本数据类型（boolean、byte、char、short、int、float、long、double）、对象引用（reference类型，直观上相当于C语言中的指针，存放的是对象地址）。64位长度的long和double类型的数据会占用2个局部变量空间（Slot），其余的数据类型只占用1个Slot。  
与Java虚拟机栈相关的两个异常：如果线程请求的栈深度大于虚拟机所允许的深度，就会抛出StackOverflowError异常，一般Java程序中递归调用一个方法而不设置终止条件就会出现这个异常，例如下面样例程序求解斐波那契数列，注释了终止条件就会出现异常；如果虚拟机栈可以动态扩展，但是扩展时无法申请到足够的内存空间，就会抛出OutOfMemoryError异常。

```
public int getFibonacci(int n) {
    // if(n == 0||n == 1){
    //     return n;
    // }
    return getFibonacciByRecursion(n-1)+getFibonacciByRecursion(n-2);
}
```
## 本地方法栈

本地方法栈（Native Method Stack）与虚拟机栈所发挥的作用相似，只不过区别在于虚拟机栈为虚拟机执行Java方法服务，本地方法栈为虚拟机中使用到的Native方法服务。有的虚拟机，如Sun HotSpot虚拟机将本地方法栈和虚拟机栈合二为一。  
那什么是Native方法呢？简单来讲，一个native method就是一个java调用非java代码的接口，也就是该方法由非java语言实现，比如C语言。那么可以知道，在定义一个native方法时，不需要提供实现，只需要定义方法名，参数以及返回类型，如下面实例：
```
public class IHaveNatives {
    native public void Native1( int x ) ;
    native static public long Native2() ;
    native synchronized private float Native3( Object o ) ;
    native void Native4( int[] ary ) throws Exception ;
} 

```
## Java堆

Java堆（Java Heap）一般是Java虚拟机所管理的内存中最大的一块，被Java虚拟机进程下的所有线程共享的一块内存区域。Java堆的唯一目的就是存放对象实例，几乎所有的对象实例都要在堆上分配内存。Java虚拟机规范中描述：所有的对象实例以及数组都要在堆上分配（The heap is the runtime data area from which memory for all class instances and arrays is allocated），但是随着JIT编译器的发展以及逃逸分析技术的成熟，栈上分配、标量替换优化技术使得所有对象在堆上分配内存就变得不那么“绝对”。  
Java堆是垃圾收集器管理的主要区域，很多时候也被称之为“GC 堆（Garbage Collected Heap）”。从垃圾回收的角度来看，目前垃圾收集器都采用分代算法，所以Java堆还细分为：新生代和老年代，其中新生代又可以分为Eden空间、From Survivor空间、To Survivor空间，HotSpot虚拟机默认Eden和Survivor的大小比例是8:1。  
根据Java虚拟机规范规定，Java堆可以处在物理上不连续的内存空间中，只要逻辑连续就行了。目前主流的虚拟机都可以通过命令来自主设置Java堆的大小，其中命令-Xms1024m指的是Java堆的初始大小，-Xmx2048m指的是分配给Java堆的最大内存。
## 方法区

方法区和Java堆一样也是所有线程共享的内存区域，它用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。有些在HotSpot虚拟机上开发、部署程序的开发者习惯称方法区为“永久代（Permanent Generation）”，这是由于HotSpot虚拟机的垃圾收集器可以像管理Java堆一样管理方法区这块内存区域，省去了专门为方法区编写内存管理代码的工作。
## 运行时常量池

运行时常量池（Runtime Constant Pool）是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息是常量池，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池。  
运行时常量池相对于Class文件常量池的另一个重要特征是具备动态性，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，运行期间也可能将新的常量放入池中，比如String类中的intern()方法。

## 直接内存

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁使用。  
自从JDK1.4中引入NIO（New Input/Output）类，提出一种基于通道（Channel）和缓冲区（Buffer）的I/O方式，它可以使用Native方法直接分配堆外内存，然后通过一个存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作。

## 对象创建过程

虚拟机遇到一条new指令，首先检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有就必须先执行相关的类加载过程。  
在类加载检查通过之后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后就可以完全确定，为对象分配空间的任务就是把一块确定大小的内存从Java堆中划分出来。
假设Java堆中内存是绝对规整的，所有用过的内存都放在一边，空闲的内存放在另一边，中间放着一个指针作为分界点的指示器，那么分配内存就是把指针往空闲内存的方向挪动一段与对象大小相等的距离，这种分配方式称为“指针碰撞（Bump the Pointer）”；如果Java堆中的内存不规整，那么虚拟机就必须维护一个列表，记录哪些内存块可用，在为对象分配内存的时候就从列表中寻找一块足够大的空间划分给对象实例，并更新表上的记录，这种分配方式称为“空闲列表（Free List）”。