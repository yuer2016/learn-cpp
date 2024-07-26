# C++

- [C++](#c)
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
  - [C++ 模板](#c-模板)
    - [类型推断](#类型推断)
    - [模板类](#模板类)

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
    ChessPiece getPiece (int x, int y); 
    PlayerColor getMove (); 
    void makeMove (int from_x, int from_y, int to_x, int to_y);

  private: 
    ChessPiece _board[ 8 ][ 8 ]; 
    PlayerColor _whose_move; 
}; 

// 方法的定义和之前完全相同！
ChessPiece ChessBoard::getPiece (int x, int y) 
{ 
  return _board[ x ][ y ]; 
} 

PlayerColor ChessBoard::getMove () 
{ 
  return _whose_move; 
} 

void ChessBoard::makeMove (int from_x, int from_y, int to_x, int to_y) 
{ 
  //通常情况下，首先需要写点代码验证移动棋子的合法性
  _board[ to_x ][ to_y ] = _board[ from_x ][ from_y ]; 
  _board[ from_x ][ from_y ] = EMPTY_SQUARE; 
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

template <typename Type> Type Calc<Type>::multiply (Type x, Type y) 
{ 
 return x * y; 
}

template <typename Type> Type Calc<Type>::add (Type x, Type y) 
{ 
 return x + y; 
}

int main () 
{ 
 // 展示如何声明
 Calc<int> c; 
} 
```
