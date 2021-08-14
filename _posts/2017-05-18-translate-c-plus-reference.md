---
layout: post
author: "ylvoid"
title:  "【分析】深入探究 C++ 引用"
subtitle: ""
date:   2017-07-09 11:30:08
tags:
    - 技术细节
    - c++

---
争论
--

> 在 c/c++ 中，访问一个变量只能通过两种方式被访问，传递，或者查询。这两种方式是：
> 
>  - 通过值 访问 / 传递变量
>  - 通过地址 访问 / 传递变量 – 这种方法就是指针

关于引用，我翻看了一些网上的博客和外文文章，看到两种观点：

 - 引用为一个参数的别名（当然，C++ primer 上也是这么说的）
 
 - 引用其实就是一个常量指针，指向这个对象（所以必须被初始化才能使用，和常量指针很类似），由编译器进行解释自行翻译成常量指针

个人看法
----

其实以上的两种说法都有一定的正确性，但是我们其实可以两个都当作正确的（只要能够正确使用就行），当然我还是认为后一个观点正确一点（仅仅是个人观点），因为我认为C++ 中并没有所谓的别名的这个特性，如开头所说：

> 在 c/c++ 中，访问一个变量只能通过两种方式被访问，传递，或者查询，分别是：通过值 访问，通过地址 访问


很多的引用别名的特性其实都可以通过常量指针的特性得到解释，引用的基本特性如下：


 - 引用不可以为空，必须在初始化的时候绑定上一个对象，即必须初始化
 - 引用不可以改变指向（奇怪的特性）
 - 对引用的操作是直接反应到所绑定变量之上（同样经过 * 取地址的常量指针也具有这个特性）
 - 引用的大小是所指向的变量的大小（和指针类似，不过这里要给pointer 加上 *在进行sizeof（））


比如最基本的，定义一个引用对象，必须进行初始化，测试代码如下：

```
#include<iostream>
using namespace std;


int main() {
	int a = 5;
//  int &b;   //编译器报错,提示需要初始化引用
//  int *const b; //编译器同样报错，提示需要初始化 
	int &b = a;
	int *const &c = &a;
	cout << a <<" "<< b <<"  "<< *c << endl;
	// print  5 5 5

	a++;

	cout << a << " " << b << "  " << *c << endl;
	//print 6 6 6

	return 0;
}
```


可以看到，这个引用的特性可以用常量指针的形式得到解释，定义一个`int &b = a;`  类似于定义一个`int *const &c = &a;`。

使用的时候，类似的，编译器会给这个引用解释成 `int * const` 进行解释

针对**不可以改变引用的指向**，同样可以用常量指针进行解释。由于对引用的使用都是给这个引用解释成 `int * const` 然后给上*进行操作，所以sizeof（）的结果也是原对象的大小。

最重要的一点让我认为引用不是对象的别名的就是，**引用也会占用内存空间**（当然啦，不占内存空间的对象应该不存在），如下：


```
#include<iostream>
using namespace std;

class A {
	int &i;   // int *const i;  
	int &j;   // int *const j;  
	int &k;   // int *const k;   
};
int main() {

	cout << "sizeof(A): " << sizeof(A) << endl;
	//print   sizeof(A): 12 
	system("pause");
	return 0;
}
```

**引用的地址空间和原对象的地址一样：**

```
#include<iostream>
using namespace std;

int main() {
	int a = 5;
	int &b = a;
	cout << &a << endl;
	//00CFFA00
	cout << &b << endl;
	//00CFFA00
	system("pause");
	return 0;
}
```

    最后还有最重要的一点，引用对虚函数的支持，这个我想已经没有什么好说的了。
  
    如果引用只是一个对象的别名，那么就应该绑定到彻底。
  
    而只有指针可以提供虚函数的动态支持。
    
   如下代码：
   

```
#include<iostream>
using namespace std;

class A
{
public:
	virtual void print() { cout << "A" << endl; }
};
class B : public A
{
public:
	virtual void print() { cout << "B" << endl; }
};

class C : public B
{
public:
	virtual void print() { cout << "C" << endl; }
};
int main()
{
	C c1;
	A &a1 = c1;
	a1.print(); // print C  
	A a2 = c1;
	a2.print(); // print A  
	system("pause");
	return 0;
}
```
最后再贴一点stack overflow上关于引用和指针的描述：




>  - 指针可以重新分配任意次数，而参考在绑定后无法重新配置。
>  -  指针可以指向无处（NULL），而引用总是引用一个对象。
>  -  你可以用指针来取代引用的地址。
>  -  没有“参考算术”（但是您可以使用引用指向的对象的地址，并在其中进行指针算术&obj + 5），如下所示。
> 
> 澄清一个误解： C ++标准非常小心，以避免指示编译器如何实现引用，但每个C ++编译器都将引用作为指针。也就是说，一个声明如： int &ri = i; 如果没有完全优化，则分配与指针相同的存储空间，并将地址i放入该存储。 所以，一个指针和一个引用都占用了相同的内存量。
>  
- 作为基本规则， 在函数参数和返回类型中使用引用来定义有用的和自洽的接口。

> -  使用指针来实现算法和数据结构。

   

总结
--
     看到这里我们应该已经可以知道了，我对这两种说法都没有意见，只是对翻译有一点点意见。
     引用只是一种被语言特性劫持了的常量指针，具体实现就是用编译器自动替换转换成常量指针的地址取值，一点自己的理解，可能有很多的偏颇。
     但是我提供了一点新的思路
  

  
 