## Effective C++   （学习笔记详解04）

### 04 确定对象被使用前已先被初始化

#### 1. 内置类型

读取未初始化的值会导致不明确的行为。

比如：`int x`；在某些语境下`x`保证被初始化（为0），但在其他语境中却不保证。

处理办法：**永远在使用对象之前先把它初始化。对于无任何成员的内置类型，必须手工初始化**。

```c++
int x = 0;                              //对int进行手工初始化
const char* text = "A C-style string" ; //对指针进行手工初始化
double d;
std: :cin >> d;                         //以读取input stream的方式完成初始化.
```



#### 2. 非内置类型

至于内置类型以外的对象的初始化，主要由构造函数来实现，<u>**确保每一个构造函数都将对象的每一个成员初始化**。</u>

**不要混淆了初始化和赋值**，如下考虑一个用来表现通讯簿的`class`，其构造函数如下：

```c++
class PhoneNumber{...};
    class ABEntry{   
    public:
     ABEntry(const std::string& name, const std::string& address, 
            const std::List<PhoneNumber>& phones);    
    private:
        std::string theName;
        std::string theAddress;        
        std::List<PhoneNumber> thePhones;
        int numTimesConsulted;
   
    };

    ABEntry::ABEntry(const std::string& name, const std::string& address, const std::List<PhoneNumber>& phones){
        theName = name;               //构造函数进行赋值而并非是初始化
        theAddress = address
        thePhones = phones;
        numTimesConsulted = 0;
    }
```



**`ABEntry`构造函数初始化的最佳写法是使用member initialization list（成员初始化列表）的方式实现：**

```c++
   ABEntry::ABEntry(const std::string& name, const std::string& address, const std::List<PhoneNumber>& phones)
     : theName(name),                 //初始化
       theAddress(address), 
 	   thePhones(phones), 
	   numTimesConsulted(0)
   { }                                //构造函数本体不必进行任何操作                
```

 这个构造函数比上一个的最终结果效率更高！上面那个基于赋值的那个构造函数，首先调用一个`default`默认构造函数为成员变量设置初始值，再进行赋值操作。`default`默认构造函数赋初值这一步被浪费了。成员初值列中针对各个成员变量而设的实参，被拿去作为各成员变量之构造函数的实参。

> - 前者，赋值是先定义变量，在定义的时候已经调用的变量的默认构造函数之后是用了赋值操作符；
> - 后者，初始化时直接调用了拷贝构造函数



综上：

> 1. 构造函数体内的是赋值，初始化列表中才是初始化；
>
> 2. 构造函数最好使用成员初值列（member initialization list），而不要在构造函数本体内使用赋值操作。
>
> 3. 成员初值列列出的成员变量，初始化顺序要和声明顺序相同。即，其排列次序应该和它们在`class`中的声明次序相同（C++有着十分固定的”成员初始化次序”。`base classes`更早于其`derived classes`被初始化，而`class`的成员变量总是以其声明次序被初始化）。
>
>    成员初始化顺序，基类-->子类的声明次序
>
> 4. 若成员变量为`const`或`references`，则一定得使用初始化列表进行初始化，因为他们肯定需要初值，只能被初始化，而不能被赋值。（条款5当中会说明）



#### 3. 跨编译单元之初始化次序

编译单元：指产出单一目标文件的那些源码，基本上它是单一源码文件加上其所含入的头文件。

1）函数内的`static`对象称为`local static`对象，其他`static`对象称为`non-local static`对象。

2）C++对“定义于不同编译单元内的`non-local static`对象”的初始化次序并无明确定义。

**解决办法：**

<u>将每个`non-local static` 对象搬到自己的专属函数内（该对象在此函数内被声明为`static`）。这些函数返回一个`reference`来指向它所包含的对象，然后用户调用这些函数，并不直接使用这些对象。</u>

**即，将非静态局部对象转化为局部的静态对象，这样能够保证其在何时进行初始化。**

<u>基础在于：C++保证，函数内的`local static`对象会在”该函数被调用期间” ”首次遇上该对象之定义式”时被初始化</u>。

#### 总结

> 1. 为内置型对象进行手工初始化，因为C++不保证初始化它们。
> 2. 构造函数最好使用成员初值列（member initialization list），而不要在构造函数本体内使用赋值操作。初值列列出的成员变量，其排列次序应该和它们在 `class` 中的声明次序相同。
> 3. 为免除“跨编译单元之初始化次序”问题，请以`local static`对象替换`non-local static` 对象。