---
title: jvm-08-虚拟机字节码执行引擎
date: 2018-02-27 07:22:57
tags: jvm笔记
---

概览：虚拟机的方法调用和字节码执行，运行时栈帧结构，基于栈的字节码解析执行引擎

输入的是字节码文件，处理过程是字节码解析的等效过程，输出执行结果。

<!--more-->

# 运行时栈帧结构

栈帧是用于支持虚拟机进行方法调用和方法执行的数据结构，是虚拟机栈的栈元素。栈帧存储了方法的局部变量表、操作数栈、动态连接和方法返回地址信息。

## 局部变量表

局部变量表是一组变量值存储空间，用于存放方法参数和方法内局部变量。

* 局部变量表容量以变量槽 `Slot` 为最小单位，一般为4字节32位。long和double占用两个连续的 `Slot` 
* 虚拟机通过索引定位方式使用局部变量表
* 方法执行时，虚拟机使用局部变量表完成参数传递，若实例方法，第0位索引为实例引用 `this`
* 局部变量没有类变量那样的准备阶段

## 操作数栈

操作栈，是一个后入先出的栈。栈元素为任意Java数据类型。

* 方法执行过程中，各种字节码指令往操作数栈写入和提取内容，即出栈、入栈操作。
* 操作数栈元素数据类型与字节码指令序列严格匹配。
* 虚拟机优化处理，零两个栈帧共享部分操作数栈区域。

## 动态链接

栈帧包括一个指向运行时常量池中该栈帧所属方法的引用，持有这个应用是为了支持方法调用过程中的动态连接。

* 字节码中方法调用以常量池中指向方法的符号引用作为参数。
* 符号引用一部分会在类加载阶段或第一次使用时转化为直接引用，静态解析。
* 另一部分将在每一次运行期间转化为直接引用，称为动态连接。

## 方法返回

方法的两种退出方式：

1. 正常完成出口：遇到返回的字节码指令，将返回值传递给上层方法调用者。
2. 异常完成出口：方法执行中遇到异常，且在方法的异常表中无匹配的异常处理器

# 方法调用

不等同方法执行，任务是确定被调用方法的版本。

## 解析

类解析阶段，会将非虚方法的符号引用转化为直接引用。符合“编译期可知，运行期不变”的方法，包括静态方法和私有方法两类。

5条方法调用字节码指令：

1. invokestatic: 调用静态方法
2. invokespecial: 调用实力构造器方法、私有方法和父方法
3. invokevirtual: 调用所有虚方法
4. invokeinterface: 调用接口方法
5. invokedynamic: 先运行时动态解析出调用点限定符所引用的方法，再执行

解析调用为静态过程，分派调用则可能是静态或动态。

## 分派

因为重载和重写的原因，虚拟机如何确定正确的目标方法

1. 静态分派

`Human man = new Man()`

Human为静态类型，Man为实际类型。编译阶段，Javac编译器根据参数静态类型决定使用哪个重载版本。

所有依赖静态类型来定位方法执行版本的分派动作为静态分派。典型应用是方法重载。

2. 动态分派

运行期根据实际类型确定方法执行版本的分派过程，和重写有关。

invokevitual 指定的多态查找过程：

    2.1 找到操作数栈顶的第一个元素所指向对象的实际类型，记作C
    2.2 若C中找到常量描述符和简单名称都相符的方法，有则返回
    2.3 按照继承关系自下往上对父类进行搜索匹配
    2.4 否则，抛出异常。

3. 单分派和多份派

方法的接收者和方法的参数统称为方法的宗量。

根据分派基于多少种宗量，将分派划分为单分派和多份派两种。

4. 虚拟机动态分派的实现

动态分派优化手段：使用虚方法表所以代替元数据查找以提高性能。

## 动态类型语言支持

1. 动态类型语言

动态类型语言关键特征是类型检查主体过程是在运行期而不是编译器。

2. jdk7与动态类型

方法符号引用再编译时产生，而动态类型语言只有在运行期才能确定接收者类型。虚拟机上动态类型语言实现方式有：编译时留占位符类型，运行时动态生成字节码实现具体类型到占位符类型适配。

3. java.lang.invoke包

目的：在单词依靠符号一用来确定调用的目标方法这种方式之外，提供新的动态确定目标方法的机制，成为 `MethodHandle`。

```java

public class MethodHandleTest {

    static class ClassA{
        public void println(String s) {
            System.out.println(s);
        }
    }

    public static void main(String[] args) throws Throwable {
        Object obj = System.currentTimeMillis() % 2 == 0 ? System.out : new ClassA();
        getPrintlnMH(obj).invokeExact("hello");
    }

    private static MethodHandle getPrintlnMH(Object receiver) throws NoSuchMethodException, IllegalAccessException {
        MethodType methodType = MethodType.methodType(void.class, String.class);
        return MethodHandles
                .lookup()
                .findVirtual(receiver.getClass(), "println", methodType)
                .bindTo(receiver);
    }

}

```

4. invokedynamic指令

每一处含有 `invokedynamic` 指令位置称做 动态调用点

```java

public class InvokeDynamicTest {
    public static void main(String[] args) throws Throwable {
        INDY_BootstrapMethod().invokeExact("hello");
    }
    public static void testMethod(String s) {
        System.out.println("hello s:" + s);
    }

    public static CallSite BootstrapMethod(MethodHandles.Lookup lookup, String name, MethodType methodType) throws Throwable{
        return new ConstantCallSite(lookup.findStatic(InvokeDynamicTest.class, name, methodType));
    }

    private static MethodType MT_BootstrapMethod() {
        return MethodType.fromMethodDescriptorString(
                "(Ljava/lang/invoke/MethodHandles$Lookup;Ljava/lang/String;Ljava/lang/invoke/MethodType;)Ljava/lang/invoke/CallSite;",
                null);
    }

    private static MethodHandle MH_BootstrapMethod() throws Throwable {
        return MethodHandles.lookup().findStatic(InvokeDynamicTest.class, "BootstrapMethod", MT_BootstrapMethod());
    }

    private static MethodHandle INDY_BootstrapMethod() throws Throwable {
        CallSite callSite = (CallSite) MH_BootstrapMethod().invokeWithArguments(
                MethodHandles.lookup(),
                "testMethod",
                MethodType.fromMethodDescriptorString("(Ljava/lang/String;)V", null));
        return callSite.dynamicInvoker();
    }
}

```

5. 掌控方法分派规则

程序员可以通过super关键字或MethodHandle掌控方法分派的具体版本。

# 基于栈的字节码解析执行引擎

虚拟机是如何执行方法中的字节码指令的。

## 解释执行

编译过程： 程序源码 -> 词法分析 -> 单词流 -> 语法分析 -> 抽象语法树

传统编译分支：  抽象语法树 -> 优化器 -> 中间代码 -> 生成器 -> 目标代码

解释执行分支：  抽象语法树 -> 指令流 -> 解释器 -> 解释执行

## 基于栈的指令集与基于寄存器的指令集

1. 基于栈的指令集

优点：可移植，不受硬件约束；代码更紧凑；实现叫简单；
缺点：执行速度稍慢，需要指令数较多

2. 基于寄存器的指令集

优点：执行速度快
缺点：依赖寄存器硬件

## 基于栈的解释器执行过程

整个运算过程中的中间变量都以操作数的出栈、入栈为信息交换途径。