---
title: JVM入门、概念理解
tag:
	- Java
	- Jvm
---

### 类加载时机

编译得到的`.class`文件并不会立即加载JVM中。只有符合如下5种情况才会立即加载到虚拟机中。

* 创建类的实例（`new`的方式）。
	* 访问某个类或者接口的静态变量，
	* 对类或者接口的静态变量赋值
	* 调用类的静态方法
* 反射创建类
* 初始化某个类的子类，则其父类也会被初始化。
* Java虚拟机在启动时被标明为启动类的类，直接使用`java.exe`命令运行某个主类（包含`main`方法的类）
* 使用JDK1.7动态语言支持时会立即加载

Java类是动态加载的，它并不会一次性吧所有的类加载再运行，而只是保证程序运行的基础类会立即加载到Jvm中。其他的类则在需要的时候才加载。
这也是为了节省内存开销。

### 栈帧

在一条线程中，只有目前正在执行的那个方法的栈帧是活动的。这个栈帧就被称作是当前栈帧，这个栈帧对于的方法就是当前方法。这个方法所在的类就是当前类。对局部变量表和操作数的各种操作，都通常指的是对当前栈帧的对局部变量表和操作数栈进行的操作。

栈帧是线程本地私有的数据，不可能在一个栈帧中引用另一个线程的栈帧。

### 局部变量表

局部变量使用索引进行定位访问，第一个局部变量的索引值为0，局部变量的索引值是从0至小于局部变量表最大容量的所有整数`[0 ~ constantPoolCount-1]`。

Java虚拟机使用局部变量表来完成方法调用时参数传递，当一个方法被调用的时候，它的参数将会传递至从0开始的连续的局部变量表位置上。特别的，当一个实例方法被调用的时候，第0个布局变量一定是用来存储被调用的实例方法所在的对象的引用（即Java中的`this`关键字）。后续的其他参数将会传递至从1开始的连续的布局变量表位置上。



### 方法调用指令



`invokevirtual`指令用于调用对象的实例方法，根据对象的实际类型进行分派。

`invokeinterface`指令用于调用接口方法，它会在运行时搜索一个实现了这个接口方法的对象，找出合适的方法进行调用。

`invokespecail`指令用于调用一些特殊处理的实例方法，包括实例初始化方法、私有方法、父类方法。

`invokestatic`指令用于调用类方法（`static`方法）。



### 常量池tag说明



| 常量类型                    | 值   |
| --------------------------- | ---- |
| CONSTANT_Class              | 7    |
| CONSTANT_Fieldref           | 9    |
| CONSTANT_Methodref          | 10   |
| CONSTANT_InterfaceMethodref | 11   |
| CONSTANT_String             | 8    |
| CONSTANT_Integer            | 3    |
| CONSTANT_Float              | 4    |
| CONSTANT_Long               | 5    |
| CONSTANT_Double             | 6    |
| CONSTANT_NameAndType        | 12   |
| CONSTANT_Utf8               | 1    |
| CONSTANT_MethodHandle       | 15   |
| CONSTANT_MethodType         | 16   |
| CONSTANT_InvokeDynamic      | 18   |



## Java虚拟机指令集



为了方便理解，用一个统一的格式说明Java虚拟机的每个指令。

### 描述格式


| 指令		| 	xxx		|
| ---------- | -------- |
| 操作       | 简要描述 |
| 格式       | <b>助记符</b><br>操作数1<br>操作数2<br>…… |
| 结构       | 助记符 = 操作码 |
| 操作数栈   | …，value1，value2 -><br>…，result |
| 描述       | 关于操作数栈内容、常量池项、指令操作和结果类型等信息的详细描述。 |
| 链接时异常 | 如果执行该指令可能抛出任何链接时异常，那么每一个可能抛出的异常都需要在此进行描述。 |
| 运行时异常 | 如果执行该指令可能抛出任何运行时异常，那么每一个可能抛出的异常都需要在此进行描述。<br>除了在此列出的链接时、运行时异常以及`VirtualMachineError`或其子类之外，指令不得再抛出其他任何异常。 |
| 注意       | 某些并非本规范对该指令强制约束的注释，将会在这里进行描述。 |



### 指令集



#### aaload指令



| 指令       |  aaload		|
| ---------- | ------------------------------------------------------------ |
| 操作       | 从数组中加载一个`reference`类型数据到操作栈                |
| 格式       | aaload              |
| 结构       | aaload = 50（0x32）                             |
| 操作数栈   | …，arrayref，index -><br>…，value                     |
| 描述       | `arrayref`必须是一个`reference`类型的数据，它指向一个组件类型为`reference`的数组，`index`必须为`int`类型。指令执行后，`arrrayref`和`index`同时从操作数栈出栈，`index`作为索引定位到数组中的`reference`类型值将压入到操作数栈中。 |
| 运行时异常 | 如果`arrayref`为`null`,`aaload`指令将抛出`NullPointerException`异常。<br>另外，如果`index`不在`arrayref`所代表的数组上下界范围中，另外将抛出`ArrayIndexOutOfBoundException`异常。 |
| 注意       | 无 |





#### aastore指令



| 指令       | aastore                                                      |
| ---------- | ------------------------------------------------------------ |
| 操作       | 从操作数栈读取一个reference类型数据存入到数组中              |
| 格式       | aastore                                                      |
| 结构       | aastore = 83（0x53）                                         |
| 操作数栈   | …，arrayref，index，value -><br>…                            |
| 描述       | `arrayref`必须是一个`reference`类型的数据，它指向一个组件类型为`reference`的数组，`index`必须为`int`类型，value必须为reference类型。指令执行后，`arrrayref`、`index`和value同时从操作数栈出栈，value存储到`index`作为索引定位的数组元素中。<br>运行时，value的实际类型必须与arrayref所代表的数组的组件类型匹配。具体地说，reference类型值value（记作S）能匹配组件类型为reference（记作T）的数组的前提是：<br><br/>**如果S是类型（Class Type），那么 **   <br/>         如果T也是类类型，那S必须与T是同一个类类型，或者S是T所代表的类型的子类<br/>          如果T是接口类型，那么S必须实现了T接口<br>**如果S是接口类型（Interface Type），那么**<br>           如果T是类类型，那么T只能是`Object`。<br>           如果T是接口类型，那么T与S应当是相同的接口，或者T是S的父接口。<br>**如果S是数组类型（Array Type），假设为`SC[]`的形式，这个数组的组件为SC，那么：**<br>           如果T是类类型，那么T只能是`Object`。<br>           如果T是数组类型，假设为`TC[]`的形式，这个数组的组件类型为TC，那么下面的两条规则之一必须成立：<br>                          TC和SC是同一个原始类型。<br>                          TC和SC都是reference类型，并且SC能与TC类型相匹配。<br>            如果T是接口类型，那么T必须是数组类型所实现的接口之一。 |
| 运行时异常 | 如果`arrayref`为`null`,`aastore`指令将抛出`NullPointerException`异常。<br>另外，如果`index`不在`arrayref`所代表的数组上下界范围中，另外将抛出`ArrayIndexOutOfBoundException`异常。<br>另外，如果arrayref不为`null`，并且value的实例类型与数组组件类型不能相匹配，aastore指令将抛出`ArrayStoreException`异常。 |
| 注意       | 无                                                           |



