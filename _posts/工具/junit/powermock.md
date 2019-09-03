---
title: Powermockito 的基本使用
date: 2019-08-29 14:47:38
description: Powermockito 的基本使用
categories:
  - 工具
  - junit
tags:
  - junit

export_on_save:
  markdown: true

---

## 1. 为什么要使用 Mock 工具

在做单元测试的时候，我们会发现我们要测试的方法会引用很多外部依赖的对象，比如：（发送邮件，网络通讯，远程服务, 文件系统等等）。 而我们没法控制这些外部依赖的对象，为了解决这个问题，我们就需要用到 Mock 工具来模拟这些外部依赖的对象，来完成单元测试。

## 2. 为什么要使用 PowerMock

现如今比较流行的 Mock 工具如 jMock 、EasyMock 、Mockito 等都有一个共同的缺点：不能 mock 静态、final、私有方法等。而 PowerMock 能够完美的弥补以上三个 Mock 工具的不足。

## 3. PowerMock 简介

PowerMock 是一个扩展了其它如 EasyMock 等 mock 框架的、功能更加强大的框架。PowerMock 使用一个自定义类加载器和字节码操作来模拟静态方法，构造函数，final 类和方法，私有方法，去除静态初始化器等等。通过使用自定义的类加载器，简化采用的 IDE 或持续集成服务器不需要做任何改变。熟悉 PowerMock 支持的 mock 框架的开发人员会发现 PowerMock 很容易使用，因为对于静态方法和构造器来说，整个的期望 API 是一样的。PowerMock 旨在用少量的方法和注解扩展现有的 API 来实现额外的功能。目前 PowerMock 支持 EasyMock 和 Mockito。

## 4. PowerMock 入门

PowerMock 有两个重要的注解：

```java
 @RunWith(PowerMockRunner.class)

 @PrepareForTest( { YourClassWithEgStaticMethod.class })
```

如果你的测试用例里没有使用注解@PrepareForTest，那么可以不用加注解@RunWith(PowerMockRunner.class)，反之亦然。当你需要使用 PowerMock 强大功能（Mock 静态、final、私有方法等）的时候，就需要加注解@PrepareForTest。

## 5. PowerMock 基本用法

### 5.1. 普通 Mock： Mock 参数传递的对象

测试目标代码：

```java
 public boolean callArgumentInstance(File file) {
      return file.exists();
 }
```

测试用例代码：

```java
 @Test
 public void testCallArgumentInstance() {

     File file = PowerMockito.mock(File.class);

     ClassUnderTest underTest = new ClassUnderTest();

     PowerMockito.when(file.exists()).thenReturn(true);

     Assert.assertTrue(underTest.callArgumentInstance(file));
 }
```

说明：普通 Mock 不需要加@RunWith 和@PrepareForTest 注解。

### 5.2. Mock 方法内部 new 出来的对象

测试目标代码：

```java

 public class ClassUnderTest {

     public boolean callInternalInstance(String path) {

         File file = new File(path);

         return file.exists();

     }
 }
```

测试用例代码：

```java
 @RunWith(PowerMockRunner.class)
 public class TestClassUnderTest {

     @Test
     @PrepareForTest(ClassUnderTest.class)
     public void testCallInternalInstance() throws Exception {

         File file = PowerMockito.mock(File.class);

         ClassUnderTest underTest = new ClassUnderTest();

         PowerMockito.whenNew(File.class).withArguments("bbb").thenReturn(file);

         PowerMockito.when(file.exists()).thenReturn(true);

         Assert.assertTrue(underTest.callInternalInstance("bbb"));
     }
 }
```

说明：当使用 PowerMockito.whenNew 方法时，必须加注解@PrepareForTest 和@RunWith。注解@PrepareForTest 里写的类是需要 mock 的 new 对象代码所在的类。

### 5.3. Mock 普通对象的 final 方法

测试目标代码：

```java
 public class ClassUnderTest {

     public boolean callFinalMethod(ClassDependency refer) {

         return refer.isAlive();

     }
 }


 public class ClassDependency {

     public final boolean isAlive() {

         // do something

         return false;

     }
 }
```

测试用例代码：

```java
 @RunWith(PowerMockRunner.class)
 public class TestClassUnderTest {

     @Test
     @PrepareForTest(ClassDependency.class)
     public void testCallFinalMethod() {

         ClassDependency depencency =  PowerMockito.mock(ClassDependency.class);

         ClassUnderTest underTest = new ClassUnderTest();

         PowerMockito.when(depencency.isAlive()).thenReturn(true);

         Assert.assertTrue(underTest.callFinalMethod(depencency));

     }
 }
```

说明： 当需要 mock final 方法的时候，必须加注解@PrepareForTest 和@RunWith。注解@PrepareForTest 里写的类是 final 方法所在的类。

### 5.4. Mock 普通类的静态方法

测试目标代码：

```java
 public class ClassUnderTest {

     public boolean callStaticMethod() {

         return ClassDependency.isExist();

     }
 }


 public class ClassDependency {

     public static boolean isExist() {

         // do something

         return false;

     }
 }
```

测试用例代码：

```java

 @RunWith(PowerMockRunner.class)
 public class TestClassUnderTest {

     @Test
     @PrepareForTest(ClassDependency.class)
     public void testCallStaticMethod() {

         ClassUnderTest underTest = new ClassUnderTest();

         PowerMockito.mockStatic(ClassDependency.class);

         PowerMockito.when(ClassDependency.isExist()).thenReturn(true);

         Assert.assertTrue(underTest.callStaticMethod());

     }
 }
```

说明：当需要 mock 静态方法的时候，必须加注解@PrepareForTest 和@RunWith。注解@PrepareForTest 里写的类是静态方法所在的类。

### 5.5. Mock 私有方法

测试目标代码：

```java
 public class ClassUnderTest {

     public boolean callPrivateMethod() {

         return isExist();

     }

     private boolean isExist() {

         return false;

     }
 }
```

测试用例代码：

```java
 @RunWith(PowerMockRunner.class)
 public class TestClassUnderTest {

     @Test
     @PrepareForTest(ClassUnderTest.class)
     public void testCallPrivateMethod() throws Exception {

        ClassUnderTest underTest = PowerMockito.mock(ClassUnderTest.class);

        PowerMockito.when(underTest.callPrivateMethod()).thenCallRealMethod();

        PowerMockito.when(underTest, "isExist").thenReturn(true);

        Assert.assertTrue(underTest.callPrivateMethod());

     }
 }
```

说明：和 Mock 普通方法一样，只是需要加注解@PrepareForTest(ClassUnderTest.class)，注解里写的类是私有方法所在的类。

### 5.6. Mock 系统类的静态和 final 方法

测试目标代码：

```java
 public class ClassUnderTest {

     public boolean callSystemFinalMethod(String str) {

         return str.isEmpty();

     }

     public String callSystemStaticMethod(String str) {

         return System.getProperty(str);

     }
 }
```

测试用例代码：

```java
 @RunWith(PowerMockRunner.class)
 public class TestClassUnderTest {

   @Test
   @PrepareForTest(ClassUnderTest.class)
   public void testCallSystemStaticMethod() {

       ClassUnderTest underTest = new ClassUnderTest();

       PowerMockito.mockStatic(System.class);

       PowerMockito.when(System.getProperty("aaa")).thenReturn("bbb");

       Assert.assertEquals("bbb", underTest.callJDKStaticMethod("aaa"));

   }
 }
```

说明：和 Mock 普通对象的静态方法、final 方法一样，只不过注解@PrepareForTest 里写的类不一样 ，注解里写的类是需要调用系统方法所在的类。

## 6. 无所不能的 PowerMock

### 6.1. 验证静态方法：

```java
  PowerMockito.verifyStatic();
  Static.firstStaticMethod(param);
```

### 6.2. 扩展验证:

```java
 PowerMockito.verifyStatic(Mockito.times(2)); //  被调用2次
 Static.thirdStaticMethod(Mockito.anyInt()); // 以任何整数值被调用
```

### 6.3. 更多的 Mock 方法

http://code.google.com/p/powermock/wiki/MockitoUsage

## 7. PowerMock 简单实现原理

    •  当某个测试方法被注解@PrepareForTest标注以后，在运行测试用例时，会创建一个新  org.powermock.core.classloader.MockClassLoader实例，然后加载该测试用例使用到的类（统类除外）。

    •   PowerMock会根据你的mock要求，去修改写在注解@PrepareForTest里的class文件（当前测试类会自动加入注解中），以满足特殊的mock需求。例如：去除final方法的final标识，在静态方法最前面加入自己的虚拟实现等。

    •   如果需要mock的是系统类的final方法和静态方法，PowerMock不会直接修改系统类的class 件，而是修改调用系统类的class文件，以满足mock需求。
