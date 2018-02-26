---
title: jvm-07-虚拟机类加载机制
date: 2018-02-26 15:19:54
tags: jvm笔记
---

类加载机制是虚拟机将class文件加载到内存，并对数据进行校验、转换解析和初始化，最终形成可以被虚拟机直接使用的java类型的过程。

<!--more-->

# 类加载时机

生命周期7个阶段: 加载 -> 验证 -> 准备 -> 解析 -> 初始化 -> 使用 -> 卸载

加载时机5中情况：

1. 遇到 `new、getstatic、putstatic或invokestatic` 这4条指令。
2. 使用 `java.lang.reflect` 方法对类进行反射调用
3. 初始化一个类是，须先触发父类初始化
4. 虚拟机启动时，指定的主类初始化
5. 动态语言生成的类方法

有且仅有再主动引用该类时才会进行初始化。

# 类加载过程

类加载全过程： 加载 -> 验证 -> 准备 -> 解析 -> 初始化

## 加载

1. 通过类全限定名来获取此类的二进制字节流
2. 将字节流代表的静态存储结构转化为方法区的运行时数据结构
3. 内存中生成一个代码这个类的 `java.lang.Class` 对象，作为方法区此类的各自数据的访问入口

## 验证

确保class文件符合要求，不会尾号虚拟机自身安全。

1. 文件格式验证
2. 元数据验证
3. 字节码验证
4. 符号引用验证

## 准备

正式为类变量分配内存并设置类变量初始值的阶段，这些变量所使用内存将在方法区中分配。

## 解析

解析阶段是虚拟机将常量池内符号引用替换为直接引用的过程。

* 符号引用：以一组符号来描述所引用目标，可以是字面量，能定位到目标即可。
* 直接引用：可以是直接指向目标的指针、相对偏移量或能间接定位到目标的句柄。

1. 类或接口的解析

    当前类D，要将符号引用N解析为一个类或接口C的直接引用，三个步骤
    1.1 若C不是数组类型，将N全限定名传递给D的类加载器去加载类C。
    1.2 若C是数组类型，且元素类型为对象，将按1.1规则加载元素类型。
    1.3 经上，C在虚拟机中已成有效类或几口，然后进行符号引用验证。

2. 字段解析

    先对字段表内class_info符号引用解析，将这个字段所属类或接口表示为C。
    2.1 若C包含简单名称和描述都匹配的字段，则返回
    2.2 若C中实现接口，按继承关系自下往上搜索匹配，找到则返回
    2.3 按照继承关系从下往上搜索父类，匹配则返回
    2.4 否则查找失败，java.lang.NoSuchFieldError

3. 类方法解析

    先对字段表内class_info符号引用解析，将这个字段所属类或接口表示为C。
    3.1 若C是接口，抛出异常
    3.2 C中查找简单名称和描述符匹配的方法，有则返回
    3.3 父类中递归查找
    3.4 实现的接口列表及父接口中递归查找
    3.5 java.lang.NoSuchMethodError

4. 接口方法解析    

    先对字段表内class_info符号引用解析，将这个字段所属类或接口表示为C。
    4.1 若C是个类，抛异常
    4.2 C中查找简单名称和描述符匹配的方法，有则返回
    4.3 父接口中递归查找，有则返回
    4.4 java.lang.NoSuchMethodError

## 初始化

真正开始执行中定义的程序代码：初始化阶段是执行类构造器 `<clinit>()` 方法的过程。

*  `<clinit>()` 由编译器自动收集类中变量的赋值动作和静态语句块中语句合并产生
*  `<clinit>()` 不需要显示调用父类构造器，但会保证父类的`<clinit>()` 方法先执行。
*  虚拟机会包装一个类的 `<clinit>()`  在多线程环境被正确的加锁、同步。

# 类加载器

通过一个类的全限定名来获取描述此类的二进制流，实现这一动作的代码模块即类加载器。

## 类与类加载器

对任一类，需要由加载它的类加载器和类本身共同确定唯一性。

```java

public class ClassLoaderTest {
    public static void main(String[] args) throws ClassNotFoundException {

        ClassLoader classLoader = new ClassLoader() {
            @Override
            public Class<?> loadClass(String name) throws ClassNotFoundException {
                String fileName = name.substring(name.lastIndexOf(".") + 1) + ".class";
                InputStream is = getClass().getResourceAsStream(fileName);
                if (null == is) return super.loadClass(name);
                try {
                    byte[] bytes = new byte[is.available()];
                    is.read(bytes);
                    return defineClass(name, bytes, 0, bytes.length);
                } catch (IOException e) {
                    throw new ClassNotFoundException(name);
                }
            }
        };

        Object obj = classLoader.loadClass("com.stu.jvm.ClassLoaderTest");
        System.out.println(obj.getClass());     // class java.lang.Class
        System.out.println(obj instanceof ClassLoaderTest); // false
        System.out.println(ClassLoaderTest.class);  // class com.stu.jvm.ClassLoaderTest
    }
}

```

## 双亲委派模型

3种系统提供的类加载器：

1. 启动类加载器 `Bootstrap ClassLoader`:负责加载 lib 目录中的或路径参数指定类库。
2. 扩展类加载器 `Extension ClassLoader` 由 `sun.misc.Lanuncher$ExtClassLoader`实现，负责加载\lib\ext目录中的类库。
3. 应用程序类加载器 `Application ClassLoader`: 由`sun.misc.Lanuncher$AppClassLoader`,系统类默认加载器。

双亲委派模型： 自定义类加载器 -> 应用程序类加载器 -> 扩展类加载器 -> 启动类加载器

工作过程：一个类加载器收到类加载请求，将请求委派给父类加载器完成，所有加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完成加载请求时（搜索范围内没有找到所需类），子加载器才会尝试自己去加载。

## 破坏双亲委派模型

1. JNDI服务，需要启动类加载器去加载，调用独立厂商实现并部署在应用程序的 `ClassPath`下的JNDI接口提供这（SPI）的代码。设计出线程上下文类加载器（Thread Context ClassLoader）,父类加载器请求子类加载器去完成类加载动作。
2. 对程序动态性追求而致，OSGi实现模块化热部署的关键是自定义的类加载器机制实现。