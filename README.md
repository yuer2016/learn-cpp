# C++

- [C++](#c)
  - [C++ Code Style](#c-code-style)
  - [宏定义和条件编译](#宏定义和条件编译)
    - [预处理编程](#预处理编程)
    - [包含文件（include）](#包含文件include)
    - [宏定义（#define/#undef）](#宏定义defineundef)
    - [条件编译（#if/#else/#endif）](#条件编译ifelseendif)
  - [自动类型推导](#自动类型推导)
  - [指针](#指针)
    - [所有权](#所有权)
    - [指针声明语法](#指针声明语法)
    - [引用](#引用)
    - [引用与指针的区别](#引用与指针的区别)
    - [函数指针](#函数指针)
    - [动态内存分配](#动态内存分配)
    - [数组和指针](#数组和指针)
    - [指针运算](#指针运算)
  - [结构体](#结构体)
    - [方法声明和调用的语法](#方法声明和调用的语法)
    - [方法的定义和结构体分离](#方法的定义和结构体分离)
  - [类](#类)
    - [类的生命周期](#类的生命周期)
    - [初始化类的成员](#初始化类的成员)
    - [解构对象](#解构对象)
    - [复制类](#复制类)
      - [赋值操作符](#赋值操作符)
      - [复制构造函数](#复制构造函数)
      - [彻底禁止复制](#彻底禁止复制)
    - [继承和多态](#继承和多态)
    - [多态对象的销毁](#多态对象的销毁)
    - [对象切割问题](#对象切割问题)
    - [与子类共享代码](#与子类共享代码)
    - [属于类的数据](#属于类的数据)
    - [如何实现多态](#如何实现多态)
    - [类的最佳实践](#类的最佳实践)
  - [C++ 模板](#c-模板)
    - [类型推断](#类型推断)
    - [模板类](#模板类)
  - [容器](#容器)
  - [标准库](#标准库)

## C++ Code Style

***任何人都能写出机器能看懂的代码，但只有优秀的程序员才能写出人能看懂的代码.***

1. 变量、函数名和名字空间用 snake_case，全局变量加 “g_” 前缀；
2. 自定义类名用 CamelCase，成员函数用 snake_case，成员变量加 “m_” 前缀；
3. 宏和常量应当全大写，单词之间用下划线连接；
4. 尽量不要用下划线作为变量的前缀或者后缀（比如 \_local、name_），很难识别。

```C++
#define MAX_PATH_LEN 256 //常量，全大写

int g_sys_flag; // 全局变量，加 g_ 前缀

namespace linux_sys { // 名字空间，全小写
  void get_rlimit_core(); // 函数，全小写
}

class FilePath final // 类名，首字母大写
{
  public:
    void set_path(const string& str); // 函数，全小写
  private:
    string m_path; // 成员变量，m_前缀
    int m_level;  // 成员变量，m_前缀
}
```

变量函数的名字长度与它的作用域成正比，也就是说，局部变量函数名可以短一点，而全局变量函数名应该长一点

## 宏定义和条件编译

### 预处理编程

预处理阶段编程的操作目标是 **源码**，用各种指令控制预处理器，把源码改造成另一种形式。

C++ 语言有近百个关键字，预处理指令只有十来个，预处理指令都以符号 **#** 开头，虽然都在一个源文件里，但它不属于 C++ 语言，它走的是预处理器，不受 C++ 语法规则的约束。因此，预处理编程也就不用太遵守 C++ 代码的风格。

一般来说，预处理指令不应该受 C++ 代码缩进层次的影响，不管是在函数、类里，还是在 if、for 等语句里，永远是 **顶格写**。

单独的一个 **#** 也是一个预处理指令，叫 “空指令”，可以当作特别的预处理空行。而 **#** 与后面的指令之间也可以有空格，从而实现缩进，方便排版。

```C++
# // 预处理空行
#if __linux__ // 预处理检查宏是否存在
# define HAS_LINUX 1 // 宏定义，有缩进
#endif // 预处理条件语句结束
# // 预处理空行
```

```shell
g++ test03.cpp -E -o a.cxx #输出预处理后的源码
```

### 包含文件（include）

最常用的预处理指令 “#include”，它的作用是 “包含文件”。
注意，不是 “包含头文件”，而是可以包含任意的文件。
也就是说，只要愿意，使用 “#include” 可以把源码、普通文本，甚至是图片、音频、视频都引进来。

```C++
#include "a.out" // 完全合法的预处理包含指令，你可以试试
```

因为 “include” 不做什么检查，只是单纯的合并文件，在写头文件的时候，为了防止代码被重复包含，通常要加上 “#ifndef/#define/#endif” 来保护整个头文件。

```C++
#ifndef _XXX_H_INCLUDED_
#define _XXX_H_INCLUDED_
 // 头文件内容
#endif // _XXX_H_INCLUDED_
```

一个应用示例

```C++
static uint32_t calc_table[] = { // 非常大的一个数组，有几十行
0x00000000, 0x77073096, 0xee0e612c, 0x990951ba,
0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3,
0x0edb8832, 0x79dcb8a4, 0xe0d5e91e, 0x97d2d988,
0x09b64c2b, 0x7eb17cbd, 0xe7b82d07, 0x90bf1d91,
};

static uint32_t calc_table[] = {
# include "calc_values.inc" // 非常大的一个数组，细节被隐藏
}
```

### 宏定义（#define/#undef）

“#define”，用来定义一个源码级别的 “文本替换”，也就是常说的 “宏定义”。

“#define” 在预处理阶段可以无视 C++ 语法限制，替换任何文字，定义常量、变量，实现函数功能，为类型起别名（typedef），减少重复代码等。

因为宏的展开、替换发生在预处理阶段，不涉及函数调用、参数传递、指针寻址，没有任何运行期的效率损失，所以对于一些调用频繁的小代码片段来说，用宏来封装的效果比 inline 关键字要更好，因为它真的是源码级别的无条件内联。

```C++
#define ngx_tolower(c) ((c >= 'A' && c <= 'Z') ? (c | 0x20) : c)
#define ngx_toupper(c) ((c >= 'a' && c <= 'z') ? (c & ~0x20) : c)
#define ngx_memzero(buf, n) (void) memset(buf, 0, n)
```

其次，宏是没有作用域概念的，永远是全局生效，对于一些用来简化代码、起临时作用的宏，最好是用完后尽快用 “#undef” 取消定义，避免冲突的风险。

```C++
#define CUBE(a) (a) * (a) * (a) // 定义一个简单的求立方的宏
cout << CUBE(10) << endl; // 使用宏简化代码
cout << CUBE(15) << endl; // 使用宏简化代码
#undef CUBE // 使用完毕后立即取消定义

// 宏定义前先检查
#ifdef AUTH_PWD // 检查是否已经有宏定义
# undef AUTH_PWD // 取消宏定义
#endif // 宏定义检查结束
#define AUTH_PWD "xxx" // 重新宏定义

// 用宏定义常量
#define MAX_BUF_LEN 65535
#define VERSION "1.0.18"

// 用宏来代替直接定义名字空间（namespace）
#define BEGIN_NAMESPACE(x) namespace x {
#define END_NAMESPACE(x) }

BEGIN_NAMESPACE(my_own)
// functions and classes
END_NAMESPACE(my_own)
```

### 条件编译（#if/#else/#endif）

条件编译有两个要点，一个是条件指令“#if”，另一个是后面的“判断依据”，也就是定义好的各种宏，而这个 “判断依据” 是条件编译里最关键的部分。

```C++
#ifdef __cplusplus // 定义了这个宏就是在用C++编译
extern "C" { // 函数按照C的方式去处理
#endif

void a_c_function(int a);

#ifdef __cplusplus // 检查是否是C++编译
} // extern "C" 结束
#endif

#if __cplusplus >= 201402 // 检查C++标准的版本号
cout << "c++14 or later" << endl; // 201402就是C++14
#elif __cplusplus >= 201103 // 检查C++标准的版本号
cout << "c++11 or before" << endl; // 201103是C++11
#else // __cplusplus < 201103 // 199711是C++98
# error "c++ is too old" // 太低则预处理报错
#endif // __cplusplus >= 201402 // 预处理语句结束
```

## 自动类型推导

因为 C++ 是一种静态强类型的语言，任何变量都要有一个确定的类型，否则就不能用。
在 “自动类型推导” 出现之前，写代码时只能 “手动推导”，也就是说，在声明变量的时候，必须要明确地给出类型。
这在变量类型简单的时候还好说，比如 int、double，但在泛型编程的时候，麻烦就来了。
因为泛型编程里会有很多模板参数，有的类型还有内部子类型，一下子就把 C++ 原本简洁的类型体系给搞复杂了，这就迫使我们去和编译器“斗智斗勇”，只有写对了类型，编译器才会“放行”（编译通过）

```C++
int i = 0; // 整数变量，类型很容易知道
double x = 1.0; // 浮点数变量，类型很容易知道
std::string str = "hello"; // 字符串变量，有了名字空间，麻烦了一点

std::map<int, std::string> m = // 关联数组，名字空间加模板参数，很麻烦
 {{1,"a"}, {2,"b"}}; // 使用初始化列表的形式

std::map<int, std::string>::const_iterator // 内部子类型，超级麻烦
iter = m.begin();
```

```C++
auto i = 0; // 自动推导为int类型
auto x = 1.0; // 自动推导为double类型
auto str = "hello"; // 自动推导为const char [6]类型
std::map<int, std::string> m = {{1,"a"}, {2,"b"}}; // 自动推导不出来
auto iter = m.begin(); // 自动推导为map内部的迭代器类型
auto f = bind1st(std::less<int>(), 2); // 自动推导出类型，具体是啥不知道

class X final
{
  auto a = 10; // 错误，类里不能使用auto推导类型
};
```

decltype 的形式很像函数，后面的圆括号里就是可用于计算类型的表达式

```C++
int x = 0; // 整型变量
decltype(x) x1; // 推导为int，x1是int
decltype(x)& x2 = x; // 推导为int，x2是int&，引用必须赋值
decltype(x)* x3; // 推导为int，x3是int*
decltype(&x) x4; // 推导为int*，x4是int*
decltype(&x)* x5; // 推导为int*，x5是int**
decltype(x2) x6 = x2; // 推导为int&，x6是int&，引用必须赋值

int x = 0; // 整型变量
decltype(auto) x1 = (x); // 推导为int&，因为(expr)是引用类型
decltype(auto) x2 = &x; // 推导为int*
decltype(auto) x3 = x1; // 推导为int&

using int_ptr = decltype(&x); // int *
```

最佳实践

```C++
vector<int> v = {2,3,5,7,11}; // vector顺序容器

for(const auto& i : v) { // 常引用方式访问元素，避免拷贝代价
  cout << i << ","; // 常引用不会改变元素的值
}

for(auto& i : v) { // 引用方式访问元素
    i++; // 可以改变元素的值
    cout << i << ",";
}

auto get_a_set() // auto作为函数返回值的占位符
{
  std::set<int> s = {1,2,3};
  return s;
}

// UNIX信号函数的原型，看着就让人晕，你能手写出函数指针吗？
void (*signal(int signo, void (*func)(int)))(int)
// 使用decltype可以轻松得到函数指针类型
using sig_func_ptr_t = decltype(&signal);

class DemoClass final
{
  public:
    using set_type = std::set<int>; // 集合类型别名
  private:
    set_type m_set; // 使用别名定义成员变量
    // 使用decltype计算表达式的类型，定义别名
    using iter_type = decltype(m_set.begin());
    iter_type m_pos; // 类型别名定义成员变量  
}
```

## 指针

当声明一个变量时，底层会分配出一定大小的内存来存储变量的信息，而分配内存的多少，则是在编译时确定的，在程序运行阶段，不能改变分配的这块内存的大小。

创建数据数组来使用大量的变量，其实质就是一大串内存。
但是，数组不能够存储超过写程序时就已指定好的元素数量。

为了能够访问（几乎）无限量的内存，需要一种类型的变量，它能够直接引用存储着变量值的内存，这种变量就叫做指针。
指针可以指：

1. 内存地址本身；
2. 存储内存地址的变量。

当一个变量存储了另一变量的地址，我们就会说，它指向了那个变量。

### 所有权

所有权这个概念是函数及其使用者之间接口的一部分，它在编程语言中没有显式出现。
当写一个函数，它接受一个指针，应该说明该函数是否占用了内存的所有权。
C++不会为你追踪内存的所有权。

只要程序正在运行，C++就永远不会帮你释放已经显式分配了的内存，除非你显式要求释放。

只有某些代码应使用某些内存，这就是为什么不能随便取一些内存地址来使用；

如果只是生成一个随机数，然后把它当做内存地址来用，后果会怎样呢？
技术上来讲可以这样做，但这是一个糟糕的想法。
你不知道谁被分配了这块内存，甚至有可能是栈本身，如果你修改了内存里存储的值，就会破坏正在使用中的数据！
为了帮助发现这类问题，操作系统会将尚未分配给你使用的内存保护起来——该内存对你来说是非法的，访问非法内存将导致程序崩溃。

### 指针声明语法

以下是声明一个指针变量的语法:

```C++
<type> *<ptr_name>;

// 示例
int *p_points_to_integer;
// p_pointer1 是指针, nonpointer1 是普通的 int 变量
int *p_pointer1, nonpointer1; 
// p_pointer1 和 p_pointer2 都是指针
int *p_pointer1, *p_pointer2; 
```

指针既可以直接指向新分配的内存，也可以指向一个已经存在的变量。
为了获得变量地址（即变量在内存中的位置），要把符号 & 放在变量名前。
& 称为取地址操作符，它能返回变量的内存地址。

```C++
int x; 
int *p_x = &x; 
*p_x = 2; // initialize x to 2 
```

指针存储的是地址。
因此，直接使用 “裸” 指针得到的就是地址。
要获得或调整存储在该地址中的值，必须添加额外的*。

变量存储的是数据值。
因此，直接使用变量得到的就是数据值。
而要获得变量的地址，就必须额外添加 &

```C++
#include <iostream> 
using namespace std;

int main () 
{ 
 int x; //x为普通变量
 int *p_int; // p_int 为指向一个整型数的指针 
 
 p_int = &x; // 将 x 的地址赋值给 p_int 
 cout << "Please enter a number: "; 
 cin >> x; // 读入一个值并赋值给变量 x，这里的 x 也可以用 *p_int 来代替
 cout << *p_int << '\n'; // 使用 * 来获得指针所指向的变量的值 

 *p_int = 10; 
 cout << x; // 再次输出10！ 
} 
```

### 引用

引用是指一个变量引用了另外一个变量，它们背后共享着相同的内存。
引用变量的使用方法跟普通变量类似，可以将它想象成一个简化版的指针, 与指针不同的是，引用必须始终指向有效内存。
声明一个引用，需要使用&符号：

```C++
int x = 5; 
int &ref = x; // 注意，不需要在x的前面加上*号！

// 给函数传递结构体时可以使用引用，而无需传递整个结构体，也不用担心空指针问题。
struct myBigStruct 
{ 
 int x[ 100 ]; // 占用了大量内存的大结构体！
}; 
 
void takeStruct (myBigStruct& my_struct) 
{ 
 my_struct.x[0] ="foo"; 
} 
```

### 引用与指针的区别

当需要通过多个名称来使用同一个变量时，我们可以使用引用来替代指针。
比如，你想要将参数传递给一个函数而不用复制它们，又或者希望函数对参数的修改能够对调用者可见。
引用并不像指针那般灵活，因为引用必须总是有效的。

引用和指针还有一个区别：一旦一个引用被初始化，你便不能改变它指向的内存地址。
引用永远指向相同的变量，这也限制了它们构建复杂的数据结构的灵活性。

### 函数指针

函数指针是一种特殊的指针类型，它指向一个函数。
可以使用函数指针来调用函数，存储函数地址，或者将函数作为参数传递给其他函数。
声明函数指针的基本语法如下：

```C++
int add(int a, int b) {
    return a + b;
}
// 返回类型 (*指针名称)（参数列表）;
// 函数指针指向 add 函数
int (*functionPtr)(int, int) = add; 
// 使用函数指针调用函数：
int result = functionPtr(3, 5); 
```

### 动态内存分配

动态分配是指，在程序运行时请求所需要的内存大小。
你的程序将计算出它所需的内存数量，而不是只能处理一组特定大小的固定的变量。

```C++
int *p_int = new int; // 申请内存
delete p_int; // 释放内存
```

### 数组和指针

```C++
int numbers[ 8 ]; 
int* p_numbers = numbers; 

for (int i = 0; i < 8; ++i) 
{ 
  p_numbers[ i ] = i; 
}
```

数组 numbers 被赋给指针时，仿佛它本身就是一个指针一样。
重要的是理解清楚，数组不是指针，但数组可以被赋值给指针。
C++ 编译器知道怎样将一个数组转换为一个指针，这个指针会指向数组的第一个元素。
这种转换在 C++ 中经常发生。
例如，你可以将一个 char 类型的变量赋给一个 int 类型的变量。char 不是 int，但编译器知道如何进行转换。

### 指针运算

指针代表内存地址，而内存地址归根结底只是个数字。
所以，就像使用数字一样，你可以对指针执行一些数学运算。
例如，指针与一个数相加，或两个指针相减。

```C++
int arr[] = {10, 20, 30, 40, 50};
int *p = arr; // p 指向数组的第一个元素

x[3] = 120;
*(x + 3) = 120;

p++; // p 现在指向数组的第二个元素，即 20
p--; // p 回到指向第一个元素

p += 2; // p 现在指向数组的第三个元素，即 120
p -= 1; // p 现在指向数组的第二个元素，即 20
```

## 结构体

定义一个结构体的语法格式是：

```C++
struct SpaceShip 
{ 
 int x_coordinate; 
 int y_coordinate; 
 string name; 
};
```

### 方法声明和调用的语法

方法是在结构体里面声明的。这些方法应被作为该结构体的基本组成部分来看待。

```C++
enum ChessPiece { EMPTY_SQUARE, WHITE_PAWN /* 及其他 */ }; 
enum PlayerColor { PC_WHITE, PC_BLACK }; 

struct ChessBoard 
{ 
 ChessPiece board[ 8 ][ 8 ]; 
 PlayerColor whose_move; 
 
 ChessPiece getPiece (int x, int y) 
 { 
  return board[ x ][ y ]; 
 } 

 PlayerColor getMove () 
 { 
  return whose_move; 
 } 

 void makeMove (int from_x, int from_y, int to_x, int to_y) 
 { 
  // 通常情况下，我们首先需要写点代码验证移动棋子的合法性
  board[ to_x ][ to_y ] = board[ from_x ][ from_y ]; 
  board[ from_x ][ from_y ] = EMPTY_SQUARE; 
 }

};
```

### 方法的定义和结构体分离

把所有的函数体都包含在结构体中真的会很乱而且让人难以理解。
可以把方法拆分成一个在结构体中的声明和一个放在结构体之外的定义。
例子如下：

```C++
enum ChessPiece { EMPTY_SQUARE, WHITE_PAWN /* 及其他 */ }; 
enum PlayerColor { PC_WHITE, PC_BLACK }; 

struct ChessBoard 
{ 
  ChessPiece board[8][8]; 
  PlayerColor whose_move; 
  // 在结构体中声明方法
  ChessPiece getPiece(int x, int y); 
  PlayerColor getMove(); 
  void makeMove(int from_x, int from_y, int to_x, int to_y); 
};

/*方法的定义需要一些方式回头来把它们自身与结构体联系起来
可以使用一个特殊的“范围”语法来表示该方法是属于某个结构体的*/
ChessPiece ChessBoard::getPiece (int x, int y) 
{ 
 return board[x][y]; 
}

PlayerColor ChessBoard::getMove () 
{ 
 return whose_move; 
}

void ChessBoard::makeMove (int from_x, int from_y, int to_x, int to_y) 
{ 
 // 通常情况下，首先需要写点代码验证移动棋子的合法性
 board[to_x][to_y] = board[from_x][from_y]; 
 board[from_x][from_y] = EMPTY_SQUARE; 
}
```

## 类

类就如同一个结构体，只不过它能够定义哪些方法和数据是属于类内部，哪些方法是为了提供给该类的使用者的

```C++
enum ChessPiece { EMPTY_SQUARE, WHITE_PAWN /* 及其他 */ }; 
enum PlayerColor { PC_WHITE, PC_BLACK }; 

class ChessBoard 
{ 
  public: 
    ChessPiece getPiece(int x, int y); 
    PlayerColor getMove(); 
    void makeMove(int from_x, int from_y, int to_x, int to_y);

  private: 
    ChessPiece _board[8][8]; 
    PlayerColor _whose_move; 
}; 

// 方法的定义和之前完全相同！
ChessPiece ChessBoard::getPiece(int x, int y) 
{ 
  return _board[x][y]; 
} 

PlayerColor ChessBoard::getMove () 
{ 
  return _whose_move; 
} 

void ChessBoard::makeMove(int from_x, int from_y, int to_x, int to_y) 
{ 
  //通常情况下，首先需要写点代码验证移动棋子的合法性
  _board[to_x][to_y] = _board[from_x][ from_y]; 
  _board[from_x][from_y] = EMPTY_SQUARE; 
}

// 声明一个类的实例就如同声明一个结构体的实例一样：
ChessBoard b; 
// 在类上进行方法的调用也是和结构体的一模一样：
b.getMove();
```

类的声明和之前结构体的声明看上去很像，除了使用了两个新的关键字：public 和 private。
任何在 public 关键字之后声明的东西，所有人都可以通过该类的对象来使用（在这里就是 getPiece、getMove 和 makeMove 这些方法）。
任何出现在 private 之后的东西，都只能被 ChessBoard 类自身的方法访问到（_board和_whose_move）。

### 类的生命周期

创建一个类的时候，想让它尽可能地易于使用。
有三个基本的操作可能所有的类都需要支持：

1. 初始化自己；
2. 清理占用的内存或者别的资源;
3. 复制自己;

对于每一个类编译器会自动生成以下方法：

1. 默认构造函数；
2. 默认析构函数；
3. 赋值操作符；
4. 复制构造函数

```C++
/*
如果类中没有写构造函数，那么 C++ 就会很友好地创造一个。
自动创造的这个构造函数不接收参数，但是它会调用你类中所有字段的默认构造函数来初始化它们。
一旦为类声明了一个构造函数，C++就再也不会自动生成默认的构造函数了。
*/
enum ChessPiece { EMPTY_SQUARE, WHITE_PAWN /* and others */ }; 
enum PlayerColor { PC_WHITE, PC_BLACK }; 

class ChessBoard 
{ 
  public:
    /*构造函数,和类名一样，没有返回值*/
    ChessBoard(); 
    /*就像普通函数一样，构造函数可以接收任意数量的参数，并且也可以有多个参数类型不同的重载构造函数*/
    ChessBoard (int board_size); 
    PlayerColor getMove(); 
    ChessPiece getPiece(int x, int y); 
    void makeMove(int from_x, int from_y, int to_x, int to_y); 
  private: 
    ChessPiece _board[8][8]; 
    PlayerColor _whose_move; 
}; 

ChessBoard::ChessBoard()
{ 
  _whose_move = PC_WHITE; 
  // 先把整个棋盘清空，然后再填入棋子
  for ( int i = 0; i < 8; i++ ) 
  { 
    for (int j = 0; j < 8; j++ ) 
    { 
      _board[ i ][ j ] = EMPTY_SQUARE; 
    } 
  } 
  // 其他初始化棋盘的代码
}

ChessBoard::ChessBoard(int size) 
{ 
  // 初始化棋盘的代码
} 
```

### 初始化类的成员

类的每一个成员都需要在构造函数中来完成初始化.

```C++
class ChessBoard 
{ 
  public: 
    ChessBoard(); 
    string getMove(); 
    ChessPiece getPiece(int x, int y); 
    void makeMove(int from_x, int from_y, int to_x, int to_y); 
  private: 
    PlayerColor _board[8][8]; 
    string _whose_move;
    int _move_count; 
}; 

// 简单地给 _whose_move 变量赋值
ChessBoard::ChessBoard () 
{ 
  _whose_move = "white"; 
}

// 初始化列表
ChessBoard::ChessBoard() 
 // 跟在冒号后面的是变量的列表，带着传递给构造函数的参数,初始化列表的成员之间使用逗号分隔开
 : _whose_move( "white" )
 , _move_count( 0 ) 
{ 
 // 代码运行到这里的时候，_whose_move 的构造函数已经被调用了
 // 并且它已经有了值"white" 
}
```

使用初始化列表的好处是：

1. 效率：对于非 POD 类型[如内置数据类型、数组或枚举]，初始化列表可以直接调用构造函数，而不是赋值操作，这可能更加高效。
2. 必要性：对于没有默认构造函数的类成员，使用初始化列表是必要的。
3. 清晰性：初始化列表提供了一种清晰的方式来显示成员变量是如何被初始化的。

用初始化列表初始化常量字段
如果定义了类中的一个字段为常量，那么这个字段就必须在初始化列表中完成初始化工作：

```C++
class ConstHolder 
{ 
  public: 
    ConstHolder (int val); 
  private: 
    const int _val; 
};

ConstHolder::ConstHolder() 
 : _val( val ) 
{}
```

### 解构对象

正如同需要构造函数来初始化一个对象一样，有时也需要有代码来清理那些不再需要使用的对象。
举个例子，如果构造函数申请分配了内存（或者其他的任何资源），然后当你的对象不再使用的时候，这些资源最终需要归还给操作系统。
进行这种清除的操作称为摧毁对象，它是在一个叫做析构方法的特殊的方法内部发生的。
在一个对象不再需要的时候会调用析构方法。

```C++
class LinkedList 
{ 
  public: 
  LinkedList (); // 构造函数
  ~LinkedList (); // 析构函数，注意波浪号(~)
  void insert (int val); // 插入一个节点
  private: 
  LinkedListNode *_p_head; 
};

LinkedList::~LinkedList ()
{
 LinkedListNode *p_itr = _p_head;
 while ( p_itr != NULL )
 {
  LinkedListNode *p_tmp = p_itr->p_next;
  delete p_itr;
  p_itr = p_tmp;
 }
}
```

用解构器有个好处：在对象不再需要的时候它会被自动调用。
那么说一个对象“不再需要了” 它意味着下面三种情况中的一种:

1. 当你删除了一个指向对象的指针；
2. 当这个对象超出了作用域；
3. 当拥有这个对象的类的析构函数被调用了的时候。

```C++
// 调用 delete 很明显地反应了什么时候会调用析构函数
LinkedList *p_list = new LinkedList; 
delete p_list; // p_list的~LinkedList(析构函数)被调用了

// 超出作用域时的解构
if ( 1 ) 
{ 
 LinkedList list; 
} // 链表的析构函数在这里调用

//由其他析构函数导致的解构
//如果有个对象包含在另一个类当中，那个对象的析构函数是在类的析构函数调用之后被调用的。
class NameAndEmail 
{ 
/* 正常情况下这里会有一些方法 */ 
  private: 
    string _name; 
    string _email; 
};
```

### 复制类

在C++中，创建可供复制的新类是经常要做的事

```C++
LinkedList list_one; 
LinkedList list_two; 
list_two = list_one; 
LinkedList list_three = list_two;
```

在 C++ 中，有两个函数可以定义用来确保这些复制操作能正常运行。

1. 赋值操作符
2. 复制构造函数

为什么需要这些函数，不是直接写就可以了吗？
答案是可以直接写，有时候就管用，因为 C++ 会提供默认版本的复制构造函数和赋值操作符。
然而，有些情况下不能依赖默认的版本——有时编译器也不是那么聪明，它可能不知道你的意图。
例如，默认版本的复制构造函数和赋值操作符会执行叫做 **浅层指针复制** 的操作。

#### 赋值操作符

在将一个对象赋值给一个已经存在的对象时赋值操作符会被调用

```C++
list_two = list_one;

class LinkedList 
{ 
  public: 
    LinkedList (); // 构造函数
    ~LinkedList (); // 析构函数，注意波浪线
    LinkedList& operator= (const LinkedList& other);
    void insert (int val); // 插入一个节点
  private: 
    LinkedListNode *_p_head; 
}; 

LinkedList& LinkedList::operator= (const LinkedList& other) 
{ 
  // 确保不是将自己赋值给自己，如果出现这种情况就忽略它
  // 注意，这里使用 'this' 来确保另外一个值和我们的对象不是同一个地址
  if (this == & other) 
  { 
    // 返回this对象来维持赋值链不被破坏
    return *this; 
  } 
  // 在复制过来新的值时，我们需要释放原来的内存，因为它没用了
  delete _p_head; 
  _p_head = NULL; 
  LinkedListNode *p_itr = other._p_head;
  while (p_itr != NULL) 
  { 
    insert(p_itr->val); 
  } 
  return *this; 
}
```

#### 复制构造函数

最后还有一种要考虑的情况，假使想要依照另一个对象来构造一个相同的对象会怎样：

```C++
LinkedList list_one; 
LinkedList list_two(list_one);
```

这只是构造函数使用的一个特殊情况——构造器接收的参数是和正在构造的对象属于同一类型的对象。
这样的构造函数称为 **复制构造函数**。
复制构造函数应当能够使新的对象是原有对象的一个直接复制。

```C++
class LinkedList 
{
public: 
  LinkedList(); // 构造函数
  ~LinkedList(); // 析构函数，注意波浪号 
  LinkedList& operator= (const LinkedList& other); 
  LinkedList(const LinkedList& other); 
  void insert(int val); // 插入一个节点 
private: 
  LinkedListNode *_p_head; 
};

/*复制构造函数*/
LinkedList::LinkedList (const LinkedList& other) 
 : _p_head( NULL ) // 默认是NULL，以防另一个列表是空的
{ 
 // 注意，这段代码和operator=很像
 // 在正式的程序中写个辅助性的方法来做这件事是有意义的
 LinkedListNode *p_itr = other._p_head; 
 while ( p_itr != NULL ) 
 { 
  insert( p_itr->val ); 
 } 
} 
```

#### 彻底禁止复制

有些时候根本不需要复制对象的功能。

```C++
class Player 
{ 
public: 
 Player (); 
 ~Player (); 
private: 
 // 通过声明却不定义这些方法来，然后编译器不会为我们自动生成， 
 // 这样就禁止了复制操作
 operator= (const Player& other); 
 Player (const Player& other); 
 PlayerInformation *_p_player_info; 
}; 
// 没有赋值操作符或者复制构造函数相关的实现
```

应当总是选择下面这些操作中的一个：

1. 同时使用默认的复制构造函数和赋值操作符；
2. 同时创建自己的复制构造函数和赋值操作符；
3. 将复制构造函数和赋值操作符都设为私有，并且不去实现它们。

### 继承和多态

继承的意思是一个类从另一个类那里获得一些特性。

```C++
class Drawable 
{ 
  public: 
    virtual void draw(); //可以使用虚方法（virtual），虚方法是父类的一个组成部分，它可以被不同的子类所重写 
}; 

class Ship : public Drawable 
{
  public: 
    virtual draw(); 
    virtual void draw () = 0; //把函数设为纯虚函数强制子类要有它们自己的实现
}; 

Ship::draw () 
{
  /* 执行绘制的代码 */  
}
```

当继承一个父类的时候，子类的构造函数会调用父类的构造函数——就像它调用类的所有字段的那些构造函数一样。

```C++
#include <iostream> 
using namespace std;

class Foo // Foo在计算机编程中是个常用的占位符
{ 
  public: 
    Foo() { cout << "Foo's constructor" << endl; } 
    ~Foo() { cout << "Foo's destructor" << endl; }
}; 

class Bar : public Foo 
{ 
  public: 
    Bar() { cout << "Bar's constructor" << endl; } 
    ~Bar () { cout << "Bar's destructor" << endl; }
}; 

int main () 
{ 
  Bar bar; 
  // Foo's constructor 
  // Bar's constructor 
  // Bar's destructor 
  // Foo's destructor 
} 

class FooSuperclass 
{ 
  public: 
    FooSuperclass (const string& val); 
}; 

// 父类构造函数的调用在初始化列表中应当出现在该类的字段之前。
class Foo : public FooSuperclass 
{ 
  public: 
    Foo ():FooSuperclass( "arg" ) // 初始化列表示例
    {} 
}; 
```

### 多态对象的销毁

和通常的规则一样，当把父类中的任何方法设为虚的时，就应该把父类的析构函数设为虚的。

```C++
class Drawable 
{ 
public: 
 virtual void draw (); 
 virtual ~Drawable ();
}; 

class MyDrawable : public Drawable 
{ 
 public: 
  virtual void draw (); 
  MyDrawable(); 
  virtual ~MyDrawable ();
 private: 
  int *_my_data; 
};

delete drawable;  
```

### 对象切割问题

```C++
class Superclass 
{}; 

class Subclass : public Superclass 
{ 
 int val; 
};

int main() 
{ 
 Subclass sub; 
 // 程序崩溃，来自子类的 val 字段并没有作为赋值操作的一部分赋给父类
 Superclass super = sub; 
}

/*可以把父类的复制构造函数声明为私有的并且不要去实现它 */
class Superclass 
{ 
  public: 
    // 注意，由于我们声明了复制构造函数，
    // 因此需要提供自己的默认构造函数
    Superclass() {} 
  private:
    // 我们不会定义这个方法，这是被禁止的操作，
    Superclass(const Superclass& other); 
}; 

class Subclass : public Superclass 
{ 
 int val; 
};

int main() 
{ 
  Subclass sub; 
  Superclass super = sub; // 现在这行代码就会导致一个编译错误
} 
```

### 与子类共享代码

如果想要一个父类能够提供子类可以调用的方法，但是在类外却不能被使用。
任何在类的 protected 区域的方法都可以被子类访问，不像 private 方法那样，但是在类之外又是不可访问的。

```C++
class Drawable 
{ 
  public: 
    virtual void draw(); 
    virtual ~Drawable(); 
  protected: 
    void clearRegion(int x1, int y1, int x2, int y2); 
};
```

### 属于类的数据

对一个类你所能做的都是把数据存储在单独的对象实例中。
还有一些情况确实需要存储不仅仅是属于某个特定对象的数据，而是属于整个类的数据。

```C++
class Node 
{ 
  public: 
    Node(); 
  private: 
    static int _getNextSerialNumber (); 
    // 静态的，整个类只有一份
    static int _next_serial_number; 
    // 非静态的，对于每个对象都可用，但是不能被静态方法使用
    int _serial_number; 
}; 

// 不是在类声明里，所以需要使用 Node:: 作为前缀
static int Node::serial_number = 0; 

Node::Node() 
    : _serial_number( _getNextSerialNumber() ) 
{}

int Node::_getNextSerialNumber () 
{ 
  // 使用 ++ 在后面的方式来返回变量中的前一个值
  return _next_serial_number++; 
} 
```

### 如何实现多态

多态的核心思想是在接口上执行函数，而不是在一个具体的子类上，这样对应一行给出的机器码就不需要确切知道要调用哪个函数。

```C++
vector<Drawable*> drawables; 

void drawEverything () 
{ 
 for (int i = 0; i < drawables.size(); i++) 
 { 
  drawables[ i ]->draw(); 
 } 
}
```

对象持有一个虚方法的列表作为它的隐藏字段——在这个例子当中，有一个入口，含有 draw 方法的地址。
接口中的每个方法都被赋予了一个数字（draw 是方法 0）；
当调用一个虚方法的时候，与该方法相关联的数字会被用来作为访问该对象虚方法列表的索引。
对虚方法的调用被编译为一个对虚方法列表的查找，后面紧跟着一个对所查找的方法的调用。
在上面的代码中，对于draw方法的调用会变成在方法表中对方法 0 的一个查找，跟随着对方法 0 的地址的调用。
这里的虚方法列表称为 vtable 。

由于对象持有着它自己使用的方法表，编译器在编译不同的类的时候可以改变表中的地址来提供虚方法的一个指定的实现。
编译器都帮你做了。使用方法表的代码只需要确切地知道表中寻找每个虚方法的索引。

当一个虚方法被调用的时候，这就相当于执行访问虚表并且通过索引找到方法的代码。这样写：

```C++
  drawables[i]->draw(); 
```

编译器会做如下理解:

1. 获取存储在 drawables[i] 中的指针。
2. 通过这个指针找到与 Drawable 类型的接口相关的那组方法所在虚表的地址。
3. 在函数表中找到给定名称（这里就是draw）的函数。函数表在字面上就是存储着每个函数在内存中的地址的集合。
4. 带着相关的参数去调用所找到的函数

### 类的最佳实践

一个通用原则是在设计类的时候尽量少用继承和虚函数。
如果非要用继承不可，那么一定要控制继承的层次，如果继承深度超过三层，就说明有点 “过度设计” 了，需要考虑用组合关系替代继承关系，或者改用模板和泛型。

在设计类接口的时候，让类尽量简单、“短小精悍”，只负责单一的功能。

C++11 新增了一个特殊的标识符 **final** 不是关键字，把它用于类定义，就可以显式地禁用继承，防止其他人有意或者无意地产生派生类。

```C++
class DemoClass final // 禁止任何人继承我
{ 

};
```

在必须使用继承的场合，建议你只使用 public 继承，避免使用 virtual、protected，因为它们会让父类与子类的关系变得难以捉摸，带来很多麻烦。
当到达继承体系底层时，也要及时使用“final”，终止继承关系。

```C++
class Interface // 接口类定义，没有final，可以被继承
{

};

class Implement final : // 实现类，final禁止再被继承
      public Interface // 只用 public 继承
{

}
```

C++ 里类的四大函数: 构造函数、析构函数、拷贝构造函数、拷贝赋值函数。
C++11 因为引入了右值（Rvalue）和转移（Move），又多出了两大函数：转移构造函数和转移赋值函数。
所以，在现代 C++ 里，一个类总是会有六大基本函数：三个构造、两个赋值、一个析构。

C++ 编译器会自动生成这些函数的默认实现，省去重复编写的时间和精力。
对于比较重要的构造函数和析构函数，应该用 ***= default*** 的形式，明确地告诉编译器（和代码阅读者）：
“应该实现这个函数，但我不想自己写。” 这样编译器就得到了明确的指示，可以做更好的优化。

```C++
class DemoClass final
{
  public:
    DemoClass() = default; // 明确告诉编译器，使用默认实现
    ~DemoClass() = default; // 明确告诉编译器，使用默认实现
    DemoClass(const DemoClass&) = delete; // 禁止拷贝构造
    DemoClass& operator=(const DemoClass&) = delete; // 禁止拷贝赋值
};
```

C++ 有隐式构造和隐式转型的规则，如果你的类里有单参数的构造函数，或者是转型操作符函数，为了防止意外的类型转换，保证安全，就要使用 “explicit” 将这些函数标记为 “显式”

```C++
class DemoClass final
{
public:
  explicit DemoClass(const string_type& str) // 显式单参构造函数
  { 
  }
  explicit operator bool() // 显式转型为bool
  {
  }
}
```

委托构造

```C++
class DemoDelegating final
{
private:
  int a; // 成员变量
public:
  DemoDelegating(int x) : a(x) // 基本的构造函数
  {}
  DemoDelegating() : // 无参数的构造函数
      DemoDelegating(0) // 给出默认值，委托给第一个构造函数
  {}
  DemoDelegating(const string& s) : // 字符串参数构造函数
      DemoDelegating(stoi(s)) // 转换成整数，再委托给第一个构造函数
  {}
}
```

成员变量初始化

```C++
class DemoInit final // 有很多成员变量的类
{
  private:
    int a = 0; // 整数成员，赋值初始化
    string s = "hello"; // 字符串成员，赋值初始化
    vector<int> v{1, 2, 3}; // 容器成员，使用花括号的初始化列表
  public:
    DemoInit() = default; // 默认构造函数
    ~DemoInit() = default; // 默认析构函数
  public:
    DemoInit(int x) : a(x) {} // 可以单独初始化成员，其他用默认值
};
```

类型别名

```C++
class DemoClass final
{
  public:
    using this_type = DemoClass; // 给自己也起个别名
    using kafka_conf_type = KafkaConfig; // 外部类起别名
  public:
    using string_type = std::string; // 字符串类型别名
    using uint32_type = uint32_t; // 整数类型别名
    using set_type = std::set<int>; // 集合类型别名
    using vector_type = std::vector<std::string>;// 容器类型别名
  private:
    string_type m_name = "tom"; // 使用类型别名声明变量
    uint32_type m_age = 23; // 使用类型别名声明变量
    set_type m_books; // 使用类型别名声明变量
  private:
    kafka_conf_type m_conf; // 使用类型别名声明变量
};
```

## C++ 模板

模板允许你把数据类型 ***提取出来***。
函数调用者列出要使用的类型，作为交换，编译器会为每个调用者要求的类型生成一个函数。

```C++
// 使用 template 关键字来声明这个函数为模板。
// 接着，在尖括号中列出了模板参数——这些参数是模板的使用者将要给定的值
// 模板的参数应该是一个类型而不是值，所以我们使用 typename 关键字。
// 紧跟着 typename 写了参数的名字 T。
// 下面这行模板声明语句可以简单理解成: 接下来的函数（或者类)是个模板，在它内部，它将会使用字母 T 作为一个类型
template<typename T> 
T triangleArea(T base, T height) 
{ 
 return base * height * .5; 
}

// 当函数的调用者提供了一个模板的参数时，模板会把任何的引用当做这个参数 T 来处理，就如同它正是要处理的类型一样
triangleArea<double>(.5, .5);
```

### 类型推断

在有些情况下，模板函数的调用者甚至都不需要显式地提供模板参数——编译器通常可以根据函数的参数来推断模板参数的值。

```C++
triangleArea( .5, .5);
```

### 模板类

声明一个模板类和声明一个模板函数很像

```C++
template <typename Type> 
class Calc 
{ 
  public: 
    Calc(); 
    Type multiply(Type x, Type y); 
    Type add(Type x, Type y); 
};

template <typename Type> Calc<Type>::Calc () 
{}

template <typename Type> Type Calc<Type>::multiply(Type x, Type y) 
{ 
 return x * y; 
}

template <typename Type> Type Calc<Type>::add(Type x, Type y) 
{ 
 return x + y; 
}

int main () 
{ 
 // 展示如何声明
 Calc<int> c; 
} 
```

## 容器

## 标准库
