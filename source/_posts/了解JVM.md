title: 了解JVM
date: 2015-09-08 17:45:19
categories: 技术
tags: [JVM,Stack,Heap]
---
一般在JVM中，内存分为两个部分，Stack（栈）和Heap（堆）。
<font color="red">Stack（栈）</font>是JVM的<font color="red">内存指令区</font>。Stack管理很简单，push一定长度字节的数据或者指令，Stack指针压栈相应的字节位移；pop一定字节长度数据或者指令，Stack指针弹栈。Stack的速度很快，管理很简单，并且每次操作的数据或者指令字节长度是已知的。所以Java 基本数据类型，Java 指令代码，常量都保存在Stack中。
<br>
<font color="red">Heap（堆）</font>是JVM的<font color="red">内存数据区</font>。Heap 的管理很复杂，每次分配不定长的内存空间，专门用来保存对象的实例。在Heap 中分配一定的内存来保存对象实例，实际上也只是保存对象实例的属性值，属性的类型和对象本身的类型标记等，并不保存对象的方法（方法是指令，保存在Stack中）,在Heap 中分配一定的内存保存对象实例和对象的序列化比较类似。而对象实例在Heap 中分配好以后，需要在Stack中保存一个4字节的Heap 内存地址，用来定位该对象实例在Heap 中的位置，便于找到该对象实例。