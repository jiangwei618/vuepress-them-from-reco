# Java库的两个设计模式

（1）装饰模式：装饰模式在Java语言中的最著名的应用莫过于Java I/O标准库的设计了。

（2）适配器模式：适配器模式是Java I/O库中第二个最重要的设计模式。

## 1. 装饰模式的应用


### 1.1. InputStream类型中的装饰模式

```plantuml
node Inputstream
node ByteArrayInputStream
node FileInputStream
node FilterInputStream
node ObjectInputStream
node PipelineInputStream
node SequenceInputStream
node StringBufferInputStream
node BufferedInputStream
node DataInputStream
node LineNumberInputStre
node PushbackInputStream

Inputstream--> ByteArrayInputStream
Inputstream--> FileInputStream
Inputstream--> FilterInputStream
Inputstream--> ObjectInputStream
Inputstream--> PipelineInputStream
Inputstream--> SequenceInputStream
Inputstream--> StringBufferInputStream

FilterInputStream--> BufferedInputStream
FilterInputStream--> DataInputStream
FilterInputStream--> LineNumberInputStream
FilterInputStream--> PushbackInputStream

```
<center><strong>结构图：</strong></center>


**装饰模式的各个角色**：

（1）抽象构件（Component）角色：由InputStream扮演。这是一个抽象类，为各种子类型流处理器提供统一的接口。

（2）具体构件（ConcreteComponent）角色：由ByteArrayInputStream、FileInputStream、PipedInputStream以及StringBufferInputStream等原始流处理器扮演。他们实现了抽象构件角色所规定的接口，可以被链接流处理器所装饰。

（3）抽象装饰（Decorator）角色：由FilterInputStream扮演。它实现了InputStream所规定的接口。

（4）具体装饰（ConcreteDecorator）角色：由几个类扮演，分别是DataInputStream、BufferInputStream以及两个不常用的类LineNumberInputStream和PushBackInputStream

注意：StringBufferInputStream、LineNumberInputStream已经过时，不再推荐使用。

### 1.2. OutputStream类型中的装饰模式

```plantuml
node Outputstream
node ByteArrayOutputStream
node FileOutputStream
node FilterOutputStream
node ObjectOutputStream
node PipelineOutputStream
node SequenceOutputStream
node StringBufferOutputStream
node BufferedOutputStream
node DataOutputStream
node PrintStream

Outputstream--> ByteArrayOutputStream
Outputstream--> FileOutputStream
Outputstream--> FilterOutputStream
Outputstream--> ObjectOutputStream
Outputstream--> PipelineOutputStream
Outputstream--> SequenceOutputStream
Outputstream--> StringBufferOutputStream

FilterOutputStream--> BufferedOutputStream
FilterOutputStream--> DataOutputStream
FilterOutputStream--> PrintStream

```
<center><strong>结构图：</strong></center>


**装饰模式的各个角色：**

（1）抽象构件（Component）角色：由OutputStream扮演。这是一个抽象类，为各种的子类型流处理器提供统一的接口。

（2）具体构件（ConcreteComponent）角色：由ByteArrayOutputStream、FileOutputStream以及PipedOutputStream等扮演，它们均实现了OutputStream所声明的接口。

（3）抽象装饰（Decorator）角色：由FilterOutputStream扮演。它有与OutputStream相同的接口，而这正是装饰类的关键。

（4）具体装饰（ConcreteDecorator）角色：由几个类扮演，分别是BufferedOutputStream、DataOutputStream，以及PrintStream。

### 1.3. Reader类型中的装饰模式
结构图：
```plantuml
node Reader
node BufferReader
node CharArrayReader
node FilterReader
node InputStreamReader
node PipedReader  
node StringReader
node LineNumberReader
node PushbackReader
node FileReader

Reader--> BufferReader
Reader--> CharArrayReader
Reader--> FilterReader
Reader--> InputStreamReader
Reader--> PipedReader  
Reader--> StringReader

BufferReader--> LineNumberReader
FilterReader--> PushbackReader
InputStreamReader--> FileReader

```
<center><strong>结构图：</strong></center>

装饰模式的各个角色：

（1）抽象构件（Component）角色： 由Reader扮演。这是一个抽象类，为各种的子类型流处理器提供统一的接口。

（2）具体构件（ConcreteComponent）角色：有CharArrayReader、InputStreamReader、PipedReader以及StringReader等扮演，它们均实现了Reader所声明的接口。

（3）抽象装饰（Decorator）角色：由BufferedReader以及FilterReader扮演。这两者有着与Readeer相同的接口，而这正是装饰类的关键。

（4）具体装饰（ConcreteD）角色：分别是LineNumberReader作为BufferedReader的具体装饰角色，PushbackReader作为FilterReader的具体装潢角色。

### 1.4. Writer类型中的装饰模式
```plantuml
node Writer
node BufferWriter
node CharArrayWriter
node FilterWriter
node OutputStreamWriter
node PipedWriter
node StringWriter
node FileWriter

Writer--> BufferWriter
Writer--> CharArrayWriter
Writer--> FilterWriter
Writer--> OutputStreamWriter
Writer--> PipedWriter
Writer--> StringWriter

OutputStreamWriter--> FileWriter

```

**装饰模式的各个角色：**

（1）抽象构件（Component）角色：由Writer扮演。这是一个抽象类，为各种的子类型流处理器提供统一的接口。

（2）具体构件（ConcreteComponent）角色：由CharArrayWriter、OutputStreamWriter、PipedWriter以及StringWriter等扮演，它们均实现了Reader所声明的接口。

（3）抽象装饰（Decorator）角色：由BufferedWriter、FilterWriter以及PrintWriter扮演，它们有着与Writer相同的接口。

（4）具体装饰（ConcreteDecorator）角色：是与抽象装饰角色合并的。由于抽象装饰角色与具体装饰角色发生合并，因为装饰模式在这里被简化了。

## 2. 装饰模式和适配器模式的对比


（1）装饰模式和适配器模式，都是通过封装其他对象达到设计目的的。

（2）理想的装饰模式在对被装饰对象进行功能增强时，要求具体构件角色、装饰角色的接口与抽象构件角色的接口完全一致；而适配器模式则不然，一般而言，适配器模式并不要求对源对象的功能进行增强，只是利用源对象的功能而已，但是会改变源对象的接口，以便和目标接口相符合。

（3）装饰模式有透明和半透明两种，区别就在于接口是否完全一致。关于装饰模式的重要的事实是，很难找到理想的装饰模式。一般而言，对一个对象进行功能增强的同时，都会导致加入新的行为，因此，装饰角色的接口比抽象构件角色的接口宽是很难避免的，这种现象存在于Java I/O库中多有的类型的链接流处理器中。一个装饰类提供的新的方法越多，它离纯装饰模式的距离就越远，离适配器模式的距离也就越近。

## 3. 适配器模式的应用


### 3.1. InputStream原始流处理器中的适配器模式


![](https://gitee.com/jiangwei_618/note/blob/master/assets/image/1JMX.md-2019-08-06-14-59-25.png)