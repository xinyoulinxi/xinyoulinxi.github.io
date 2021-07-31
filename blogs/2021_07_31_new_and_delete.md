在C++中，我们应该经常会用到`new`、`delete`，它们是C++的一个关键字，同时也是一个操作符，下面我将我对这两者的了解和学习做一个总结和探讨。

**一、`new`和`delete`的全过程**
======================
要了解C++中的`new`和`delete`，我们首先得对在我们使用`new`和`delete`的时候，这个操作到底背着我们做了哪些事情。
首先我们定义一个对象A：

```
struct A 
{
	size_t h;
};
```

当我们使用关键字`new`在堆上动态创建一个对象A时，比如 `A* p = new A()`，它实际上做了三件事：

 1. 向堆上申请一块内存空间（做够容纳对象A大小的数据）(`operator new`)
 2. 调用构造函数 （调用A的构造函数（如果A有的话））(`placement new`)
 3. 返回正确的指针

**当然，如果我们创建的是简单类型的变量，那么第二步会被省略。**
当我们`delete`的时候也是如此，比如我们`delete p` 的时候，其行为如下：

 1. 定位到指针p所指向的内存空间，然后根据其类型，调用其自带的析构函数（内置类型不用）
 2. 然后释放其内存空间（将这块内存空间标志为可用，然后还给操作系统）
 3. 将指针标记为无效（指向`NULL`）

`delete`先放下不谈，下面我们先主要谈一谈new这个操作符，之后，我们就可以很好的理解`delete`操作符


----------


**二、`new`的三种形态**
========




**1、`new operator` 和`operator new`**
----------------------------------




我们平常使用的`new`都是`new operator`，是由C++语言内建的，就像`sizeof`那样，不能改变意义，总是做相同的事情，其过程如上。
`new operator`总是做这三件事，无论如何你都不能改变其行为。
能够改变的是用来容纳对象的那块内存的分配行为，`new operator`调用某个函数，执行必要的内存分配动作，你可以重写或者重载这个函数，改变其行为。这个函数名称就叫`operator new` 。是不是感觉有点眩晕？
再详细说一下：

>  `new operator`就是**我们平时所使用的new**，其行为就是前面所说的三个步骤，我们不能更改它。但**具体到某一步骤中的行为，如果它不满足我们的具体要求时，我们是有可能更改它的。**
>  三个步骤中最后一步只是简单的做一个指针的类型转换，没什么可说的，并且在编译出的代码中也并不需要这种转换，只是人为的认识罢了。但前两步就有些内容了。
>  
> **`new operator`的第一步分配内存实际上是通过调用`operator new`来完成的**，这里的new实际上是像加减乘除一样的操作符，因此也是可以重载的。
> `operator new`默认情况下首先调用分配内存的代码，尝试得到一段堆上的空间，如果成功就返回，如果失败，则转而去调用一个`new_hander`，然后继续重复前面过程。
> 如果我们对这个过程不满意，就可以重载`operator new`，来达到我们希望的行为。

**比如函数 `operator new` 通常声明如下：**

`void * operator new (size_t size);`

其返回类型为`void*`。即返回一个指针，指向一块**原始的、未设置初始值**的内存。函数中的`size_t`参数表示需要分配多少内存，你可以将`operator new` 重载，加上额外的参数，但第一个参数类型必须总是`size_t`。或者你从来没有直接用过`operator new` .但是你可以像调用任何其他函数一样地去调用它。
比如使用如下的使用方式（首先要包含 `<memory>` 头文件）：

`void* rawMemory = operator new( sizeof ( string ) );`

这里的`operator new` 将返回一个指针，指针指向一块足够容纳`string`对象的内存空间。和`malloc`一样，`operator new` 的**唯一任务就是分配内存，它不知道什么是构造函数或者对象构造初始化之类的东西，它只负责分配内存。**取得`operator new` 返回的内存并将之转为一个真正的对象，是`new operator`的责任。
所以通过如上的讲解，我想你们应该已经大致明白了`new operator`的三步骤之一——内存分配是如何实现的。下面我将会更详细地说明`operator new`这个操作。

**当编译器看到这段代码**：
```
string *ps=new string("memory");
```

它会执行一些更详细的代码，这些代码大致会做出如下行为：

```
void* memory=operator new(sizeof(string));   //取得原始内存，用于放置一个string对象

call string::string("memory") on memory      //在memory中将内存中对象初始化
 
string *ps=static_cast<string*>(memory);     //让ps指向新完成的对象

```
转换成正常代码的话大致是这样：

```
string *ps = (string*)malloc(sizeof(A));
ps->string::string("memory");
return ps;
```
虽然从效果上看，这三段代码也得到了一个有效的指向堆上的`string` 构造完成的对象的指针`ps`。
但区别在于，当`malloc`失败时，它不会调用分配内存失败处理程序`new_handler`，而使用`new`的话是有错误处理的。

**不管如何，注意第二步，调用一个构造函数。**
身为程序员没有权利绕过`new operator`这么去使用构造函数，但是编译器却就是是这么干的。（关于这点下面的`placement new` 将会讲到）
我们无力去改变`new operator` 的主要行为，但是却可以对其执行的过程中的步骤进行改变
如果我们对`operator new`过程不满意，就可以自己重载一下`operator new`，来设置我们希望的行为。例如：

```
class A 
{
public:
	void * operator new(size_t size)
	{
		cout << "call operator new " << endl;
		return ::operator new(size);
	}
};
A * a = new A();
```

这里通过`::operator new`调用了原有的全局的`new`，实现了在分配内存之前输出一句话。当然全局的`operator new` 也是可以重载的，但这样就不可以用`new`来分配内存了，而只能使用`malloc`：



```
void *operator new(size_t size)
{
	cout << "call global new operator" << endl;
	return malloc(size);
}
```


   **相应的，delete也有`delete operator`和`operator delete`之分，后者也是可以重载的。**
  **并且，如果重载了`operator new`，就应该也相应的重载`operator delete`，这是良好的编程习惯。**


----------


**2、`placement new`**
-------------



> 有时候你真的会想直接调用一个构造函数，去针对一个已经被定义的对象调用其构造函数生成对象，但这没有什么意义，因为构造函数用来对对象进行初始化，而一个对象只能只能初始化一次。
> 但是你偶尔会有一些分配好的原始内存，你想要在上面构建已知的对象，这样的话，你就需要用到`placement new`

`placement new`是用来实现定位构造的，因此可以实现`new operator`三步操作中的第二步，也就是在取得了一块可以容纳指定类型对象的原始内存后，在这块内存上构造出一个对象
有点类似于 `ps->string::string("memory")   `

但是这并不是标准的写法，正确的写法是使用`placement new`：

```
#include <new.h>

void main()
{
   char s[sizeof(string)];
   string* p = (string*)s;
   new(p) string("memory"); //p->string::string("memory");
   cout << (*p).size() << endl;  //6
}
```

首先是对头文件`<memory>` 的引用，这是必须的，这样才可以使用`placement new` ，这里的`new(p) string("memory");` 实际上就是`placement new`，它实现了在指定的内存地址上调用制定类型的构造函数去构造一个对象的功能，后面的`string("memory")` 就是**对构造函数的显式调用。**
除非必要，不要对`placement new`  进行直接使用，这毕竟不是用来构造对象的正式写法，只不过是new operator的其中一个步骤而已

 **使用`new operator`地编译器会自动生成对`placement new`的调用的代码，因此也会相应的生成使用`delete`时调用析构函数的代码但是如果像我们刚才那样使用了`placement new`，则必须手工调用其析构函数：`p->~string()`**

> 当我们觉得默认的`new operator`对内存的管理不能满足我们的需要，而希望自己手工的管理内存时，`placement new`就有用了。STL中的`allocator`就使用了这种方式，借助`placement new`来实现更灵活有效的内存管理


在《STL源码剖析》中，SGI STL自行架构了一个空间配置器，与标准规范也不相同，其名称为alloc，而且不接受任何参数。比如如你要在程序中明白采用SGI配置器，不能这样写：

```
vector<int , std::alloctor<int> > iv;
```
必须这样写：

```
vector<int , std::alloc<int> > iv;

```
STL对每一个容器都已经使用了缺省的空间配置器alloc，例如下面的vector声明：

```
template<class T,class Alloc = std::alloc>
class vector
{
   ...
};
```

当然，STL中针对不同类型的数据，提供了一个萃取的方案，可以根据传入数据的不同类型，自动获取其类型，然后调用模版函数中的正确构造和析构方案。
而且STL并不依赖于C++提供的默认内存分配方案，而是自己构建了一个内存池，在创建之初就向操作系统申请了很大一整块的内存放入自己的内存池中，然后在容器申请内存进行添加元素的时候直接从内存池中进行原始内存的获取，再进行构造添加。这样大大提高了内存的管理效率，而且有效的抑制了内存碎片的产生。

为了实现这种方案，STL使用了`placement new`，然后在自己管理的内存空间上直接使用`placement new` 来构造对象，以达到`new operator` 所具有的功能。
**比如用`placement new` 构成的`construct`函数构造对象**
```
template <class T1, class T2>
inline void construct(T1* p, const T2& value)
{
   new(p) T1(value);
}
```
代码中后半截T1(value)便是`placement new`语法中调用构造函数的写法，如果传入的对象`value`正是所要求的类型T1，那么这里就相当于调用拷贝构造函数。

**然后用自写的 `delete` 构成的 `destory`析构对象，释放内存**

```
template <class T>
inline void destory(T* pointer)
{
   pointer->~T();
}
```

**在书中，`destory()`有两个版本**
第一个版本接受一个指针，将指针所指向的对象析构掉，这很简单，直接调用对象的析构函数就是。
第二个版本接受first和last两个迭代器，将[first，last]范围内的对象都析构掉。但是，考虑一下，如果所传入的对象是非简单类型，这样做是必要的。
但如果传入的是简单类型，或者根本没有必要调用析构函数的自定义类型（例如只包含数个int成员的结构体），那么再逐一调用析构函数是很浪费时间的。
如果可以直接判断指针所指之物的型别的话，就很方便了，但C++并不支持对“指针所指之物的型别”的判断，也不支持对“对象析构函数是否没有调用价值”进行判断。为此，STL使用了一种称为“type traits”（类型萃取）的技巧，在编译阶段就判断出所传入的类型是否需要调用析构函数：（以下代码为《STL源码剖析》代码）：

```
template <class ForwardIterator>
inline void destory(ForwardIterator first, ForwardIterator last)
{
   __destory(first, last, value_type(first));
}
其中value_type()用于取出迭代器所指向的对象的类型信息
template<class ForwardIterator, class T>
inline void __destory(ForwardIterator first, ForwardIterator last, T*)
{
   typedef typename __type_traits<T>::has_trivial_destructor trivial_destructor;
   __destory_aux(first, last, trivial_destructor());
}
//如果需要调用析构函数：
template<class ForwardIterator>
inline void __destory_aux(ForwardIterator first, ForwardIterator last, __false_type)
{
   for(; first < last; ++first)
       destory(&*first); 
}
//如果不需要，就什么也不做：
tempalte<class ForwardIterator>
inline void __destory_aux(ForwardIterator first, ForwardIterator last, __true_type)
{}
```
这里的关键在于`__type_traits<T>`这个模板类上，它根据不同的T类型定义出不同的`has_trivial_destructor`的结果，如果T是简单类型，就定义为`__true_type`类型，否则就定义为`__false_type`类型。具体的实现就不说太多了，`__true_type`和`__false_type` 都只是一个空类，没有任何内容，但是对编译器来说，就可以很好的特化这个函数。

STL中的`type_traits`（类型萃取）机制充分借助**模板特化**的功能，实现了在程序编译期通过编译器来决定为每一处函数调用使用哪一个特化版本，于是在不增加编程复杂性的前提下大大提高了程序的运行效率。

这些都是《STL源码剖析》上的二、三章的内容，我综合了一下别人博客和书上的内容，提了一下，主要还是为了举出`placement new` 在实际中的运用。通过以上的东西我想大家都已经完全理解了`placement new` 的作用和使用情况，就不再深入了。

**3、对三个形式的new的总结：**
------------------

> 如果你希望将对象产生于heap，就是得`new operator`,它不但分配内存而为该对象调用一个构造函数。、
> 
> 如果你只是打算分配内存，请用`operator new`,就没有构造函数被调用。
> 
> 如果你打算在heap object产生自己决定的内存分配方式，请写一个自己的`operator new`。并使用`new operator`，它将会自动调用你所写的operator new。
> 
> 如果你打算在已经分配（并拥有指针）的内存构造对象，请使用`placement new` 。


三、深入理解 new [ ] 
===========
我们对new[]的使用一般都是用来动态创建一个数组，比如：

```
int *lis = new int[100];

......

delete lis;
```
严格的说，上述代码是不正确的，因为我们在分配内存时使用的是new[]，而并不是简单的`new`，但释放内存时却用的是`delete`。正确的写法是使用`delete[]`：

```
delete[] lis;
```
但是，上述错误的代码似乎也能编译执行，内存也被很好的释放了，并不会带来什么错误。
事实上，new与new[]、delete与delete[]是有区别的，特别是当用来处理复杂类型的时候。

**下面我们就来深入讲一讲new、new[]的不同之处**

加入我们对我们自己写的类型A进行`new[]`内存分配：

```
class A 
	{
	private:
		int value;
	public:
		void test() {};
		A(int v):value(v) {};
		A() = default;
		~A() {};//注意，必须有默认的析构函数
	};

	void test() 
	{
		A *lis = new A[10];
		delete[] lis;
	}
```
上述代码做了如下工作：

 - 通过`new[]`在堆上分配了10个连续的A对象大小的内存空间
 - 在这个内存空间上依次对10个A对象调用默认构造函数（必须带有自带的默认构造函数，否则会报错）
 - 通过`delete[]` 依次调用分配的10个对象的析构函数
 - 释放内存空间
 - 将`lis`指针指向`NULL`

当我们对动态分配的数组调用delete[]时，其行为根据所申请的变量类型会有所不同。
如果p指向简单类型，如`int`、`char`等，其结果就是这块内存被回收，此时使用delete[]与delete没有区别
但如果p指向的是**复杂类型**，`delete[]`会针对动态分配得到的每个对象调用析构函数，然后再释放内存。
到这里，我们很容易看出一个问题——`delete[]`是如何知道要为多少个对象调用析构函数的？
**我们试着重载一下类A的`operator new[]`试试：**


 

```
	class A 
	{
	private:
		int value;
	public:
		A(int v):value(v) {};
		A() {
			cout << "creat A" << endl;
		}
		void* operator new[] (size_t size)
		{
			void *p = operator new(size);
			cout << "class operator new size is: " << size << endl;
			return p;
	
		}
		~A() {
			cout << "delete A" << endl;
		};
	};

	void test() 
	{
		cout << "sizeof(A) is : " << sizeof(A) << endl;
		A *lis = new A[3];
		delete[] lis;
	}
```
上面的程序输出结果如下：

```
sizeof(A) is : 4
class operator new size is: 16
creat A
creat A
creat A
delete A
delete A
delete A
```

可以看到，构造和析构都是在意料之中，但是申请的内存空间大小却和我们想的有所不同，每个A类大小为4，但是3个A类却分配了16字节的内存大小，也就是说，在处理复杂类型（存在析构函数）的时候，系统为我们多分配了4个字节。
通过如下代码：

```
		cout << "sizeof(A) is : " << sizeof(A) << endl;
		A *lis = new A[3];
		int * count = (int*)lis;
		count--;
		cout << *count << endl; // 3
		delete[] lis;
```

 我们发现，多分配的4个字节的数据为3，正好就是我们分配的A的数量
 于是，我们也可以有理由去认为`new [] operator` 的行为相当于下面的伪代码（直接转载的）：

```
template <class T>
T* New[](int count)
{
   int size = sizeof(T) * count + 4;
   void* p = T::operator new[](size);
   *(int*)p = count;
   T* pt = (T*)((int)p + 4);
   for(int i = 0; i < count; i++)
       new(&pt[i]) T();
   return pt;
}
```
从中可以看到它分配了比预期多4个字节的内存并用它来保存对象的个数，然后对于后面每一块空间使用placement new来调用无参构造函数，这也就解释了为什么这种情况下类必须有无参构造函数，最后再将首地址返回。
类似的，我们很容易写出相应的delete[]的实现代码：

```
template <class T>
void Delete[](T* pt)
{
   int count = ((int*)pt)[-1];
   for(int i = 0; i < count; i++)
       pt[i].~T();
   void* p = (void*)((int)pt – 4);
   T::operator delete[](p);
}
```

由此可见，在默认情况下（简单类型）`operator new[]`与`operator new`的行为是相同的，`operator delete[]`与`operator delete`也是，不同的是`new operator`与`new[] operator`、`delete operator`与`delete[] operator`。当然，我们可以根据不同的需要来选择重载带有和不带有“[]”的`operator new`和`operator delete`，以满足不同的具体需求。

把前面的A类中的析构函数注释掉，再来看输出：

```
sizeof(A) is : 4
class operator new size is: 12
creat A
creat A
creat A
-33686019
```

这一次，new[]就只申请了12个字节，看来，需要多申请4个字节的类型差不多如下：

 - 显式的声明了析构函数的
 - 拥有需要调用析构函数的类的成员的
 - 继承自拥有析构函数的类的

类似的，动态申请简单类型的数组时，也不会多申请4个字节。
于是在这两种情况下，释放内存时使用`delete`或`delete[]`都可以，但为养成良好的习惯，我们还是应该注意只要是动态分配的数组，释放时就使用`delete[]`。

**最后，大家肯定还是会想，那么对于简单类型，`delete[]`如何知道应该释放多少内存呢？**

说实在的，我也不是太清楚，但应该涉及到`malloc()`和`free()`的原理了，我大致猜测一下，可能的情况应该如下：

 - `malloc()`返回的指针，其指针头部都保留几个bit去储存数组的信息
 - `malloc()`开辟的空间比实际需要的空间大，多出来的部分用来储存数组的信息
 -  `malloc()`的时候，直接在符号表里对这个内存地址给定大小size，`delete`的时候直接释放那样大小的区域
 - ......

**四、delete与内存释放**
==============
相信，通过以上那么多的对new的讲解，大家已经完全理解了new这个操作符在使用的时候做了哪些工作，同样的，delete也大致如此：
函数 `operator delete`对于内建的`delete operator`(操作符)就好像 `operator new` 对于`new operator`一样。

```
string *ps;
...
delete ps; //使用delete operator.
```

内存释放动作是由operator delete执行的,通常声明如下：

```
void operator delete(void* memoryToBeDeallocated);
```
因此 delete ps;

会造成编译器生成大致代码如下：

```
ps->~string();//调用析造函数
operator delete(ps);//释放对象所占用的内存
```
这里告诉我们，如果只打算**处理原始的、未设初值的内存**，应该完全回避 `new operator`和`delete operator`。

改为调用`operator new`取得内存并以`operator delete`归还系统。

如：

```
void* buffer=operator new (50*sizeof(char));//分配内存，放置50个char,没有调用构造函数

......

operator delete(buffer); //释放内存，而没有直接调用析构函数。

```

**这组行为类似`malloc` 和 `free`。**

如果使用了`placement new` ，在某块内存中产生对象，你应该避免那块内存使用`delete operator`（操作符）。
因为`delete operator`会调用`operator delete`来释放内存，但是该内存所含的对象最初并不是由`operator new` 分配来的。（OK？）`placement new`只是构造这个指针的对象然后返回它接收的指针而已，谁知道那个指针从哪里来呢？所以为了抵消该对象的构造函数的影响，使用`placement new` 时应该直接调用该对象的析构函数。

最后
我想关于`delete`和`new`的东西已经讲的差不多了，有补充的话会写在后面的博客里面。