---
title: jvm-05-class文件结构
date: 2018-02-25 09:26:39
tags: jvm笔记
---

概览： class类文件结构、字节码指令

<!--more-->

# 类文件结构

class文件是8位字节为单位的二进制流，如果使用16进制打开，2个16进制数字为1个字节。

1个字节=8个比特位，1个16进制数字=4个比特位。

只有两种数据类型：无符号数和表。  

无符号数以u1，u2，u4，u8代表1、2、4、8字节的无符号数，可以用来描述数字、索引、数量值或utf-8编码的字符串。   

表由多个无符号数或其他表构成的复核数据类型，用于描述层次关系的复合结构数据。

示例代码：

```java
public class HelloJvm {

    int i = 0;

    private int incr() {
        i++;
        return i;
    }
}
```

使用16进制打开的class文件：

{% asset_img cafebabe.png %}

## 魔数与class文件版本

头4个字节为魔数，`0xCAFEBABE`。紧接着的4个字节为class文件版本号，`0x0034`为版本52即jdk8。

## 常量池

常量池入口为u2类型的容量计数值，从1开始。 

每种常量池项目类型有着不同的数据结构，相同点是第一个数据项为tag表明是什么项目类型。

可以用 `javap -verbose` 将常量池计算出来

{% asset_img javap.png %}

## 类索引、父类索引和接口索引集合

类索引u2+父类索引u2+接口计数器u2+接口索引

## 字段表集合

字段表用于描述接口或者类中声明的变量。包括信息有：

* 字段作用域 public private protected
* 实例变量还是类变量 static
* 可变性 final
* 并发可见性 volatile
* 可否被序列化
* 字段数据类型 基本类型、对象、数组

字段表结构

| 类型           | 名称                              | 数量 |
| -------------- | --------------------------------- | ---- |
| u2             | access_flags 限定符               | 1    |
| u2             | name_index 简单名称               | 1    |
| u2             | descriptor_index 字段和方法描述符 | 1    |
| u2             | attributes_count 属性表计数       | 1    |
| attribute_info | attributes 属性                   | 1    |

**简单名称**：字段的变量名或方法名称

**全限定名称**：包括包路径的全称如 `com/stu/jvm/TestClass;`

**方法和字段描述符**：用来描述字段的数据类型、方法的参数列表和返回值。数组类型每一维度前置一个 `[`字符。
用描述符描述方法时，先参数列表，后返回值顺序描述

| 标识字符 | 含义                          |
| -------- | ----------------------------- |
| B        | byte                          |
| C        | char                          |
| D        | double                        |
| F        | float                         |
| I        | int                           |
| J        | long                          |
| S        | short                         |
| Z        | boolean                       |
| V        | void                          |
| L        | 对象类型，如Ljava/lang/Object |

## 方法表集合

方法表结构同字段表一样，仅访问标识可选项有所区别。

## 属性表集合

在class文件、字段表、方法表都可以有自己的属性表集合，用于描述场景专有信息。其中关键部分属性：

| 属性               | 使用位置           | 含义                                         |
| ------------------ | ------------------ | -------------------------------------------- |
| Code               | 方法表             | Java代码编译后的字节码指令                   |
| Exceptions         | 方法表             | 方法抛出的异常                               |
| LineNumberTable    | Code属性           | Java源码行号和字节码指令的对应关系           |
| LocalVariableTable | Code属性           | 方法局部变量描述                             |
| SourceFile         | 类文件             | 源文件名称                                   |
| ConstantValue      | 字段表             | final关键字定义的常量值                      |
| InnerClasses       | 类文件             | 内部类列表                                   |
| Signature          | 类、方法表、字段表 | 用于支持泛型情况的方法签名，记录泛型签名信息 |


# 字节码指令简介

指令执行流程：

```java
do {
    PC寄存器值加1；
    从字节码流中取出操作码；
    if ( 需要操作数 ) 从字节码流中取出操作数；
    执行操作码定义的操作；
} while ( 字节码流 > 0 )
```

## 字节码和数据类型

大多数指令包含对应操作数据类型信息。

`i = int ; l = long ; s = short ; b = byte ; c = char ; f = float ; d = double ; a = reference ;`

## 加载和存储指令

加载和存储指令用于将数据和栈帧中局部变量表和操作数栈之间来回传输。

* 将局部变量加载到操作栈： iload、iload<n>、lload、fload...
* 将数值从操作数栈存储到局部变量表： istore、lstore...
* 将常量加载到操作数栈：bipush、sipush、ldc、ldc_w、iconst_m1、lconst_<l>...
* 扩充局部变量表访问索引：wide

## 运算指令

用于堆两个操作数栈上的值进行特定运算，并把结果重新存入操作栈顶。分为对整型数据进行运算 和 对浮点型数据运算指令。

* 加法: iadd、ladd...
* 减法：isub、lsub...
* 乘法：imul、lmul...
* 除法：idiv、ldiv...
* 求余: irem、lrem...
* 取反: ineg、lneg...
* 位移：ishl、ishr、iushr...
* 按位或： ior、lor...
* 按位与： iand、land...
* 按位异或： ixor、lor...
* 自增： iinc
* 比较： dcmpg、dcmpl...

## 类型转换指令

用于将两种不同数值类型进行互相转换。

* 宽化类型转换，无需显示的转换指令
* 窄化类型转换：i2b、i2c、i2s... 很可能导致数值精度丢失

## 对象创建与访问指令

* 创建类实例：new
* 创建数组：newarray、anewarray、multianewarray
* 访问类字段和实例字段：getfield、putfield、getstatic、putstatic
* 加载数组元素到操作数栈：baload、caload、iaload...
* 将操作数栈值存储到数组元素：bastore、iastore...
* 取数组长度：arraylength
* 检查类实例类型：instanceof、checkcast

## 操作数栈管理指令

* 将操作数栈栈顶的一个或两个元素出栈：pop、pop2
* 复制栈顶数值并复制重新压入栈顶：dup、dup2、dup_x1、dup_x2...
* 栈顶两个数值呼唤: swap

## 控制转移指令

让虚拟机有条件或无条件的从指定位置而不是控制转移指令的下一条指令继续执行程序，可看作修改PC寄存器的值。

* 条件分支：ifeq、iflt、ifle、ifnull...
* 复核条件分支： tableswitch、lookupswitch
* 无条件分支：goto、goto_w、jsr、ret

## 方法调用和返回指令

* 调用方法实例：invokevirtual
* 调用接口方法：invokeinterface
* 调用需要特殊处理的实例方法，如初始化方法、私有方法和父类方法：invokespecial
* 调用类方法：invokestatic
* 调用运行时动态解析出的方法：invokedynamic

方法调用与数据类型无关，返回指令则有关如：return、ireturn、lreturn、areturn...

## 异常处理指令

athrow指令实现，处理异常catch语句采用异常表完成。

## 同步指令

虚拟机支持方法级别的同步和方法内部一段指令序列的同步，同步结构使用管程 `Monitor` 实现。

方法级的同步是隐式的，虚拟机通过方法表结构中的 `ACC_SYNCHRONIZED` 得知一个方法是否为同步方法。方法调用时，执行线程须先成功持有管城，其他线程无法获取同一管程。方法完成或异常退出是释放管程。

同步一段指令，有 `monitorenter` 和 `monitorexit` 指令支持 `synchronized` 关键字语义。