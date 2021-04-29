## Effective C++   （学习笔记详解）

[TOC]

### 01 ：视C++为一个语言联邦

C++的四个次语言：

C：C++是在C语言的基础上发展而来的；

Object-Oriented C++：这是C++中不同于C的部分，这里主要指面向对象。class（包括构造函数和析构函数）、封装、继承、多态、virtual函数（动态绑定）...等等；

Template C++：C++中的泛型编程；

STL：标准模板库，对容器、迭代器。算法以及函数对象的规约有极佳的紧密配合与协调。

### 02： 尽量const、enum、inline替换#define

```c++
#define ASPECT_RATIO 1.653
```

记号 `ASPECT_RATIO` 可能不会被编译器看见，或者在编译器开始处理源码之前就被预处理器移走了，即你所使用的名称可能并未进入记号表。

解决办法：用`const`代替`#define`。

#### 1、const

以`const`替换`#define`时，注意两个特殊情况：

1）定义常量指针。

​	由于常量定义式通常被放在头文件内（以便被不同的源码含入)，因此有必要将指针（而不只是指针所指之物）声明为const。例如若要在头文件内定义一个常量的`char*-based`字符串，你必须写`const`两次：

```c++
const char* const authorName ="Scott Meyers" ;
```

​	另外，`string`对象通常比`char*-based`合宜，所以上述的`authorName`往往定义成这样更好些:

```c++
const std::string authorName ("Scott Meyers") ;
```

2) `class` 专属常量

为了将常量的作用域限制于 `class` 内，必须让它成为 `class` 的一个成员。而为确保此常量至多只有一份实体，你必须让它成为一个`static`成员：

```c++
class GamePlayer {
private:
	static const int NumTurns = 5; //常量声明式
	int scores [NumTurns];         //使用该常量
	...
};
```

然而你所看到的是`NumTurns`的声明式而非定义式。通常C++要求你对你所使用的任何东西提供一个定义式，但如果它是个`class`专属常量又是`static`且为整数类型（ `integral type`，例如 `ints`, `chars`），则需特殊处理。

只要不取它们的地址，你可以声明并使用它们而无须提供定义式。

但如果你取某个`class`专属常量的地址，或纵使你不取其地址而你的编译器却（不正确地）坚持要看到一个定义式，你就必须另外提供定义式如下:

```c++
const int GamePlayer::NumTurns;  //NurmTurns的定义;
```

把这个式子放进一个实现文件而非头文件。~~由于class常量已在声明时获得初值（例如先前声明NumTurns时为它设初值5），因此定义时不可以再设初值。~~

另外，请注意，无法利用`#define`创建一个`class`专属常量，因为`#defines`并不重视作用域。一旦宏被定义，它就在其后的编译过程中有效。这意味`#defines`不仅不能够用来定义`class`专属常量，也不能够提供任何封装性，也就是说没有所谓`private #define`这样的东西。

而`const`成员变量是可以被封装的，``NumTurns`就是。

3）旧式C++编译器，可能不支持类内初始化，不允许静态成员在其声明式上获得初值，因此静态常量必须要在类外初始化。如下：

```c++
class Game {
private:
    static const int NumTurns;   // static class 常量声明，位于头文件内
    int scores[NumTurns];
};

const int Game::NumTurns = 5;    // static class 常量定义，位于实现文件内
```

**综上，const好处**

> 1. <u>define直接常量替换，出现编译错误不易定位 (不知道常量是哪个变量)</u>
> 2. <u>define没有作用域，不能够提供任何封装性，const有作用域提供了封装性</u>



#### 2、enum

如果编译器（错误地）不允许“`static`整数型`class`常量”完成初值设定，可采用`enum hack`，理论基础是：

**一个属于枚举类型的数值可权充`ints`被使用**

```c++
class GamePlayer {
private:
	enum {NumTurns = 5};           //enum把NumTurns定义为一个枚举常量
	int scores [NumTurns];         
	...
};
```

**`enum hack`好处：**

> 1. `enum hack`的行为更像`#define`而不是`const`，如果你不希望别人得到你的常量成员的指针或引用，可以用`enum hack`替代之。（为什么不直接用`#define`呢？首先，因为`#define`是字符串替换，所以不利于程序调试。其次，`#define`的可视范围难以控制，没有作用域，不能够提供任何封装性。
> 2. 使用`enum hack`不会导致非必要的内存分配。（`#define`也不会）
> 3. `enum hack`是模板元编程的基本技术，大量的代码在使用，很实用。



#### 3、inline

`#define`宏函数容易造成误用

```c++
//define误用举例
#define MAX(a, b) f((a) > (b) ? (a) : (b))  //以a和b的较大值调用f

int a = 5, b = 0;
MAX(++a, b);          //a++调用2次
MAX(++a, b+10);       //a++调用一次

//若采用template inline函数
template<typename T>
inline void callwithMax (const T& a,const T& b)
{
    f(a > b ? a : b);  //callwithMax 是个真正的函数，它遵守作用域和访问规则
}
```

**注意**

> 1. **对于单纯的常量，最好以const对象或enums替换#define**
> 2. **对于形似函数的宏，最好改成内联函数 inline替换 #defines**



### 03 ：尽可能使用const



#### 1、**`const`语法使用**

如果关键字`const`出现在星号左边，表示被指物是常量；如果出现在星号右边，表示指针自身是常量；如果出现在星号两边，表示被指物和指针两者都是常量。

```c++
char greeting [] = "Hello";
char* p = greeting;              //non-const pointer, non-const data
const char* p = greeting;        // non-const pointer, const data
char* const p = greeting;        // const pointer, non-const data
const char* const p = greeting;  // const pointer, const data

```

如果被指物是常量，将关键字`const`写在类型之前，或把它写在类型之后、星号之前。两种写法的意义相同，所以下列两个函数接受的参数类型是一样的：

```c++
void f1 (const widget* pw);      //f1获得一个指针，指向一个常量的（不变的）widget对象.
void f2(widget const * pw);      //f2也是
```

#### 2、STL迭代器

STL迭代器就像个`T*`指针。声明迭代器为`const`就像声明指针为`const`一样（即声明--个`T* const`指针），表示这个迭代器不得指向不同的东西，但它所指的东西的值是可以改动的。

如果你希望迭代器所指的东西不可被改动（即希望STL模拟一个`const T*`指针），你需要的是`const_iterator`：

```c++
std::vector<int> vec;
...
const std::vector<int>::iterator iter = vec.begin ( );  //iter的作用像个T* const
*iter = 10;                                             //没问题，改变iter所指物
++iter;                                                 //错误! iter是const

std::vector<int>::const_iterator cIter = vec.begin ( ); //cIter的作用像个const T*
*cIter = 10;                                            //错误!*cIter是const
++cIter;                                                //没问题，改变cIter.
```

#### 3、const成员函数

a.  确认类中哪些成员函数可以修改数据成员

const对象只能调用const 对象成员函数，非const对象既可以调用普通成员函数也可以调用const成员函数。（这是因为`this`指针可以转化为`const this`，但是`const this`不能转化为非`const this`）

如果两个成员函数只是常量性，可以被重载。C++的重要特性；

真实程序中，const对象大多用于 **通过指向常量的指针** 或 **通过常量引用** 的传递结果。

b.  更改了“指针所指物"的成员函数不算是`const`，但是如果只有指针（而非其所指物）属于对象，则此函数为`bitwise constness`不会发生编译器异议。

用`mutable`关键字修饰的成员变量，将永远处于可变状态， 哪怕是在一个`const`函数中。`mutable`释放掉非静态成员变量的`bitwise constness`约束。

**总结**

1） 将某些东西声明为`const`可帮助编译器侦测出错误用法。`const`可被施加于任何作用域内的对象、函数参数、函数返回类型、成员函数本体。
2） 编译器强制实施`bitwise constness`，但你编写程序时应该使用“概念上的常量性”( conceptual constness）。

3)  当`const`和`non-const`成员函数有着实质等价的实现时，令`non-const`版本调用`const`t版本可避免代码重复。