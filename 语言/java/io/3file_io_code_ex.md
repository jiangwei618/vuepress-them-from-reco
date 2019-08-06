# 文件流

## 1. 文件读取

&emsp;&emsp;如果想要打开一个文件用于字符输入，可以使用FileReader。为了提供速度，我们可能希望对那个文件进行缓冲，那么我们将所产生的引用传递给一个BufferedReader对象。

代码示例（BufferedInputFile.java）如下：

### 1.1. 缓冲输入文件

```
import java.io.*;
class BufferedInputFile
{
	public static String read(String filename) throws IOException
	{
		//Reading input by lines
		BufferedReader in = new BufferedReader(new FileReader(filename));
		String s;
		StringBuilder sb = new StringBuilder(); //注意这里使用的StringBuilder是JDK5.0引入的，它和StringBuffer的唯一区别是它不是线程安全的，因而性能更高
		while((s = in.readLine()) != null)
		{
			sb.append(s + "\n");
		}
		in.close();
		return sb.toString();
 
	}
	public static void main(String[] args) throws IOException
	{
		System.out.println(read("BufferedInputFile.java"));
	}
}
```

### 1.2. 从内存输入内容


使用StringReader读取字符，或者使用DataIn或者使用DataInputStream读取字节。代码如下：

* 从内存输入：使用StringReader，无中文乱码问题
```
import java.io.*;
class MemoryInput
{
	public static void main(String[] args) throws IOException
	{
		StringReader in = new StringReader(BufferedInputFile.read("MemoryInput.java"));
		int c;
		while((c = in.read()) != -1)
		{
			System.out.print((char)c);
		}
	}
}
```

* 从内存输入；使用DataInputStream，这是一个面向字节的I/O类（不是面向字符的），有中文乱码问题

```
class FormattedMemoryInput
{
	public static void main(String[] args) throws IOException
	{
		try
		{
			DataInputStream in = new DataInputStream(new ByteArrayInputStream(BufferedInputFile.read("MemoryInput.java").getBytes()));
			while(true)
			{
				System.out.print((char)in.readByte());
			}			
		}
		catch (EOFException e)
		{
			System.err.println("End Of Stream");
		}
 
	}
}
```

* 从内存输入；判断文件结尾
```
class TestEOF
{
	public static void main(String[] args) throws IOException
	{
		DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream("MemoryInput.java")));
		while(in.available() != 0)
		{
			System.out.print((char)in.readByte());
		}
	}
}
```

## 2. 基本的文件输出


FileWriter对象可以向文件写入数据。通常会 用BufferedWriter将其包装起来 用以缓冲输出（缓冲往往能显著地增加I/O操作的性能）。在本例中，为了提供格式化机制，它被装饰成PrintWriter。安装这种方式创建的数据可作为普通文本读取。代码如下：

* 基本的文件输出
```
import java.io.*;
class BasicFileOutput
{
	public static void main(String[] args) throws IOException
	{
		BufferedReader in = new BufferedReader(new StringReader(BufferedInputFile.read("BasicFileOutput.java")));
		//PrintWriter out = new PrintWriter(new BufferedWriter(new FileWriter("BasicFileOutput.out")));
		//也可以使用简化的方式，依旧是在进程缓存，而不必自己去实现。遗憾的是，其他常见的写入任务都没有快捷方式
		//因此，典型的I/O人就包含大量的冗余文本
		PrintWriter out = new PrintWriter("BasicFileOutput.out");
		int lineCount = 1;
		String s;
		while((s = in.readLine()) != null)
		{
			out.println(lineCount++ + ": " + s);
		}
		out.close();
		//Show the stored file:
		System.out.println(BufferedInputFile.read("BasicFileOutput.out"));
	}
}
```

* 存储和恢复数据

PrintWriter可以对数据进行格式化，以便人们的阅读。如果为了输出可供另一个”流“恢复的数据，则需要用DataOutputStream写入数据，并用DataInputStream恢复数据。当然，这些流可以是任何形式，但在下面的示例中使用的是一个文件，并且对于读写都进行了缓冲处理。代码如下：

```
import java.io.*;
class StoringAndRecoveringData
{
	public static void main(String[] args) throws IOException
	{
		DataOutputStream out = new DataOutputStream(new BufferedOutputStream(new FileOutputStream("Data.txt")));
		out.writeDouble(3.14159);
		out.writeUTF("你好");
		out.writeDouble(1.111222);
		out.writeUTF("'That's True");
		out.close();
 
		DataInputStream in = new DataInputStream(new BufferedInputStream(new FileInputStream("Data.txt")));
		System.out.println(in.readDouble());
		System.out.println(in.readUTF());
		System.out.println(in.readDouble());
		System.out.println(in.readUTF());
	}
}
```

## 3. 用GZIP进行简单压缩


GZIP接口非常简单，如果我们只想对单个数据流（而不是一系列互异数据）进行压缩，它可能比较适合。代码如下： 

//用GZIP进行简单压缩
```
import java.io.*;
import java.util.zip.*;
class GZIPcompress
{
	public static void main(String[] args) throws IOException
	{
		if(args.length == 0)
		{
			System.out.println("Usage:\nGZIPcompress file\nUses GZIP compression to compress the file to test.gz");
			System.exit(1);
		}
		//写压缩文件入磁盘
		BufferedReader in = new BufferedReader(new FileReader(args[0]));
		BufferedOutputStream out = new BufferedOutputStream(
			new GZIPOutputStream(new FileOutputStream("test.gz")));
		System.out.println("Writing file");
		int c;
		while((c = in.read()) != -1)
		{
			out.write(c);
		}
		in.close();
		out.close();
		//从磁盘读取压缩文件
		System.out.println("Reading file");
		BufferedReader in2 = new BufferedReader(
			new InputStreamReader(new GZIPInputStream(new FileInputStream("test.gz"))));
		String s;
		while(( s = in2.readLine()) != null)
		{
			System.out.println(s);
		}
	}
}
```


