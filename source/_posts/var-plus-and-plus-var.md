---
title: 分析在Java中i++和++i的区别
date: 2012-10-01 06:01:35
tags:
mp3: http://link.hhtjim.com/163/33541347.mp3
cover: http://odwjyz4z6.bkt.clouddn.com/1-131123121938-50.jpg
---
刚接触编程时，i++与++i这类问题确实难以理解，即使知道它俩区别，还是不能做到学以致用，今天让我带领新手把它给摸透了，看它葫芦里卖的是什么药，高手可以飘过哦~~

什么是i++，什么是++i
> i++和++i都是i=i+1的缩写，唯一区别是执行与引用的顺序。i++先引用后计算，    ++i先计算后引用

示例
```
int i=1;
System.out.println(i++); // “先引用，后计算”：首先被println方法引用打印结果1，当println方法执行完毕后进行计算i=i+1。
```
```
int i=1;
System.out.println(++i); // “先计算,后引用”：首先计算i=i+1（这时i=2），再被println方法引用打印结果2

```

题目
```
public class Demo {
	int count = 9;

	public void result() {
		System.out.println("result=" + count++);
	}

	public static void main(String args[]) {
		Demo test = new Demo();
		test.result(); // 进入result方法，count++先引用：打印结果9，再计算：count=count+1，方法执行完后count=10
		test.result(); // 进入result方法，count++先引用：打印结果10，再计算：count=count+1，方法执行完后count=11，但等于11后我们没对它进行任何处理了。
	}
}
```
输出结果
```
result=9
result=10
```

<hr>
有这样一道面试题：
Int i=0; i=i++;执行这2句话后变量i的值为（A）
A. 0 B. 1 C. 2 D. 3

查看编译后生成的字节码得出结论：java编译器对于i++会先将i的值保存至另一变量 然后再对i++，另一变量仍没有改变。 而对于++i 是先对i++ 然后保存到另一变量 然后赋值。
```
iconst_0 // 将int型0推送至栈顶
istore_1 // 将栈顶int型数值存入第二个本地变量 i=0(int i=0)
iload_1  // 将第二个int型本地变量推送至栈顶然后将i推送至栈顶   0
iinc 1 1 // 将指定int型变量增加指定值（i++, i--, i+=2）完成i++
istore_1 // 将栈顶int型数值存入第二个本地变量将栈元素赋值给了i，i=0
getstatic java/lang/System/out Ljava/io/PrintStream;
iload_1
```

