---
title: C# 中的、 、.、[]、、=和as
date: 2020-03-01 16:51:55.0
updated: 2021-01-08 17:04:06.419
url: https://maoxian.fun/archives/c中的和as
categories: 
- 程序
- C#
tags: 
---

## (?) 可空类型

通常来说，编程语言中一般对于数据的引用类型分为值类型和引用类型。例如bool、int等为值类型，而类一般为引用类型。值类型即存储的是数值，引用存储的是对数值地址的引用。从而引用类型可以为空（null），值类型不可为空，使用前未初始化则会出现警告或被默认初始化为该类型的默认值。

(?)符号的作用即**修饰值类型**，使其可空（可为null）。

举个例子：

```c#
int a = null; // 该行报错，Vs提示：无法将null转换为“int”，因为后者是不可为null的值类型”
int? b = null; // 无报错和警告
```

对应的类型：

```c#
Console.WriteLine(typeof(int)); // System.Int32
Console.WriteLine(typeof(int?)); // System.Nullable[System.Int32]
```

可以看出来int类型即.Net类型中的System.Int32，而int?则是由Nullable类型对int的封装，使其可空，查阅[MSDN文档](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable?view=netframework-4.8)对Nullable的说明如下

> **注解**
> 如果可以为类型赋值或将其分配到 `null`，则称该类型为 null，这意味着该类型没有任何值。 默认情况下，所有引用类型（如 [String](https://docs.microsoft.com/zh-cn/dotnet/api/system.string?view=netframework-4.8)）都可以为 null，但所有值类型（如 [Int32](https://docs.microsoft.com/zh-cn/dotnet/api/system.int32?view=netframework-4.8)）都不是。
> 在 C# 和 Visual Basic 中，通过在值类型后使用 `?` 表示法，将值类型标记为可以为 null。 例如，在 Visual Basic 中 C# `int?` 或 `Integer?` 声明可以 `null` 分配的整数值类型。
> [Nullable](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable?view=netframework-4.8) 类提供 [Nullable](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable-1?view=netframework-4.8) 结构的互补支持。 [Nullable](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable?view=netframework-4.8) 类支持获取可以为 null 的类型的基础类型，以及针对其基础值类型不支持泛型比较和相等运算的可以为 null 的类型对的比较和相等性运算。
> **装箱和取消装箱**
> 当对可以为 null 的类型进行装箱时，公共语言运行时将自动框 [Nullable](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable-1?view=netframework-4.8) 对象的基础值，而不是 [Nullable](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable-1?view=netframework-4.8) 对象本身。 也就是说，如果 [HasValue](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable-1.hasvalue?view=netframework-4.8) 属性为 `true`，则 [Value](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable-1.value?view=netframework-4.8) 属性的内容为装箱。 如果 `HasValue` 属性为 `false`，则 `null` 为装箱。 如果可以为 null 的类型的基础值为取消装箱，则公共语言运行时将创建一个新的 [Nullable](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable-1?view=netframework-4.8) 结构，该结构已初始化为基础值。
>
> [MSDN](https://docs.microsoft.com/zh-cn/dotnet/api/system.nullable?view=netframework-4.8)

## (?:) 三目运算符

这个其实不需要怎么解释，也不是c#特有的语法，就是常用的三目运算符。

```c#
int a = 7;
bool x = (a < 17 ? true : false); // true
```

条件 ? 返回值1 : 返回值2，这是三目运算符的基本格式，问号前面也就是三目运算的条件，如果该条件为真（true）返回（返回值1），否则返回（返回值2）。和大部分编程语言相同，需要注意一个三目运算符的两个返回值的类型都必须相同，也就是返回值1和2的类型必须一致。因为在编译期就已经决定好该运算的返回值类型，如果不同，则编译时无法确定返回类型，就会报错。

```c#
Console.WriteLine(7 < 17 ? 37 : new DateTime()); // 报错，提示：无法确定条件表达式的类型，因为“int”和“System.DateTime”之间没有隐式转换
```

可以看到当返回值类型不同时（int和DateTime类型）将会提示这两种类型无法通过隐式的类型转换变成相同类型。*（DateTime类型是时间类型，这里只是为了区分WriteLine函数的参数类型和返回值间的类型）*

## (?.)和(?[]) Null条件运算符

```c#
List<int> list1 = null; // 声明空的int类型列表list1
List<int> list2 = new List<int>(); // 声明int类型列表list2
list1.Add(7); // 为list1添加一个元素 
list2.Add(7); // 为list2添加一个元素 
```

代码中声明两个列表对象，其中一个为null，分别调用Add()函数给列表添加元素，在Vs编译时没有报错，代码编辑器中也没有红线警告，但是当编译运行后，代码运行至list1.Add(7);时，抛出NullReferenceException空指针异常。

很容易的看出由于list1为null并且没有做null检查。改下代码

```c#
List<int> list1 = null; // 声明空的int类型列表list1
List<int> list2 = new List<int>(); // 声明int类型列表list2
if (null != list1)
{
	// 如果list1 不为null 则添加一个元素 
	list1.Add(7);
}
list2.Add(7); // 为list2添加一个元素 
```

编译运行，程序正常运行，无异常无报错。通过调试诊断工具或者肉眼调试可以知道list1依然为null并且没有执行list1.Add(7)语句，list2中成功添加值为7的元素。

相同的操作，如果使用(?.)语法则变成这样

```c#
List list1 = null; // 声明空的int类型列表list1
List list2 = new List(); // 声明int类型列表list2
list1?.Add(7); // 如果list1 不为null 则添加一个元素 
list2?.Add(7); // 为list2添加一个元素 
```

代码执行后的结果与上一段代码相同。

?. 语法，首先判断问号前的对象是否为空，如果为空(null)则返回null并且停止执行其后的代码链，list1为null，不执行.Add()函数；如过不为null则继续执行其后的代码链，如list2.Add(7)。

```c#
List list1 = null; // 声明空的int类型列表list1
List list2 = new List(); // 声明int类型列表list2
list1?.Add(7); // 如果list1 不为null 则添加一个元素 
list2?.Add(7); // 为list2添加一个元素 
int? a = list1?[0]; // a = null
int? b = list2?[0]; // b = 7
```

?[]也是同样的意思，如果问号前的对象为空则返回null，否则返回对应的值。

## (??)和(??=) Null合并运算符

```c#
List<int> list1 = null; // 声明空的int类型列表list1
List<int> list2 = new List<int>(); // 声明int类型列表list2
List<int> list3 = list1 ?? list2; // list3 = list2
```

?? 运算符，若左值为null，返回右值；否则返回左值。

## (as) 类型转换

as语法对类对象进行强制转换

```c#
object obj = new A();
B b = new B();
// 有两个相互独立的类A和类B

A a1 = obj as A; // obj转换成功 
A a2 = b as A; // b转换失败，a2为null

//将obj和b两个对象转成A类对象，若转换失败，则为null，不会抛出异常，传统的类型强制转换语法，在转换失败时会抛出异常而as语法不会 
```