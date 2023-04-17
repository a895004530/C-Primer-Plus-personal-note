# C++中的代码重用

#### valarray类

valarray被定义为一个模板类，以便能够处理不同的数据类型。

模板特性意味着声明对象时，必须指定具体的数据类型。因此使用valarray类来声明一个对象时，需要在标识符valarray后面加一对尖括号：

```c++
valarray<int> q_values;
valarry<double> weights;
```

下面是这个类的一些方法：

```c++
operator[]();
size();
sum();
max();
min();
```

书中对Student类的设计如下：用到了string和valarray两个类

表现的是类设计中包含的关系：

```c++
class Student
{
	private:
		string name;
		valarray<double> scores;
    ...
}
```

上述类将数据成员声明为私有的，意味着Student类成员函数可以使用string和valarray类的公有接口来访问和修改name和scores。

与之前的is-a关系不同，has-a关系不获得接口，而是通过对象调用其自己的接口（如通过对象name调用string类的公有方法）。**不继承接口是has-a关系的组成部分，获得接口是is-a关系的组成部分。**

除了创建一个包含其他类对象的类以外，C++还有一种方式实现has-a关系：私有继承



#### 私有继承

使用私有继承，基类的公有成员和保护成员都将成为派生类的私有成员。这意味着基类方法将不会成为派生对象的公有接口的一部分，但可以在派生类的成员函数中使用它们。

记得前面提到的：

- is-a：继承接口
- has-a：不继承接口（成为私有部分了也算）

使用私有继承，类将继承实现。例如，如果从string类中派生出Student类，后者将有一个String类组件，可用于保存字符串，另外Student方法可以使用String方法来访问string组件，但是不能直接访问到string组件，即获得实现，但不获得接口。

示例：

```c++
class Student: private std::string, private std::valarray<double>
{
	public:
		...
}
```

私有继承和创建一个包含其他类对象的类的区别在于：（以上面student类为例）不能使用string的对象来描述student对象了，即原本是这样的一个构造函数

```c++
Student(const char* str, const double *pd,int n): name(str),scores(pd,n){}
```

而如果是私有继承，而不是继承别的类对象，那么就需要直接使用类名而不是对象名，即

```c++
Student(const char* str, const double *pd,int n): std::string(str),std::valarray<double>(pd,n){}
```

同样的，想要**访问基类的方法**：如前面求平均值用到了std::valarray中的sum和size，也不能用对象名而是类名

原版本：

```c++
double Student::Average()const
{
	if(scores.size()>0)
		...
}
```

用私有继承：

```c++
double Student::Average()const
{
	if(std::valarray<double>::size()>0)
		...
}
```

##### 访问基类对象

上面提到了，使用私有继承，没有类string对象没有名称，那么只能通过强制类型转换来访问基类对象了，由于Student类是从string类派生而来的，将Student对象转换为string类对象，结果为继承而来的string对象。

```c++
const string & Student::Name() const
{
	return (const string &) *this;
}
```

上述方法返回的string类引用指向用于调用该方法的student对象中继承来的string对象

#### 访问基类友元函数

因为友元不属于类，但是可以通过显式的转换为基类来调用友元函数（此处student->string)

```c++
ostream& operator<<(ostream & os,const Student &stu)
{
	os<<"..."<<(const String &)stu<<"...";
}
```



##### 使用包含还是私有继承？

通常，应使用包含来建立has-a关系；如果新类需要访问原有类的保护成员，或者需要重新定义虚函数，则应使用私有继承。因为派生类可以重新定义虚函数，而包含类不能，因为他不在派生链上中。使用私有继承，重新定义的函数将只能在该类中使用，而不是公有的。



#### 保护继承

使用保护继承时，基类的公有成员和保护成员都将成为派生类的保护成员。和私有继承一样，基类的接口在派生类中也是可用的，但在继承结构外是不可用的。当从派生类派生出另一个类时，私有继承和保护继承的主要区别就体现出来了。

使用私有继承时，第三代类将不能使用基类的接口这是因为基类的公有方法在派生类（第二代）中变成私有方法，而第二代传到第三代的私有继承仅将第二代的公有成员继承为第三代的私有成员；

使用保护继承时，基类的公有方法在第二类中变成保护成员，因此在整个派生链中都是能访问的。

![image-20230327143429310](C:\Users\89500\AppData\Roaming\Typora\typora-user-images\image-20230327143429310.png)



#### 使用using重新定义访问权限

使用保护派生或私有派生时，基类的公共成员会变成派生类的保护成员或私有成员。**假设要让基类的方法在派生类外面可用**，方法之一是定义一个使用该基类方法的派生类方法。例如：想要让Student类能使用valarray类的sum方法，可以在Student类声明中声明一个sum方法

```c++
double Student::sum()const
{
	return std::valarray<double>::sum();
}
```

这样Student对象就能够调用Student::sum()，然后进而调用valarray<double>::sum()方法应用于valarray对象。



另一种方法则是将函数调用包装在另一个函数调用中，即：使用using声明

例如：同上使用sum方法

```c++
class Student : private std::string, private std::valarray<double>
{
	...
	public:
		using std::valarray<double>::sum;
		using std::valarray<double>::min;
    	using std::valarray<double>::max;
    	using std::valarray<double>::operator[]();
}
...
    cout<<"Highest score: "<<ada[i].max()<<endl;
```

上述声明使这些valarray方法可用于valarray对象，就像它们是Student类的公有方法一样。其中ada是valarray对象数组

需要注意的是:

- using声明只使用成员名，没有圆括号、函数特征标、返回类型。
- using声明只适用于继承，而不适用于包含。



#### 多重继承

多重继承（MI）可能会给程序员带来很多新问题。其中两个主要的问题是：

1. 从两个不同的基类继承同名方法；
2. 从两个或更多相关基类那里继承同一个类的多个实例。

例：先定义一个抽象基类Worker，并使用它派生出Waiter类和Singer类。然后，便可以使用MI从Waiter类和Singer类派生出SingingWaiter类。

正常来说，可以将派生类对象的地址赋给基类指针，通常这种赋值将把基类指针设置为派生对象中基类对象的地址。

```c++
Waiter ed;
Worker *pt = &ed;
```

但是，由于SingingWaiter类由Waiter类和Singer类多重派生，因此这样直接操作将存在二义性：

```c++
SingingWaiter ed;
Worker * pt = &ed;
```

为了避免二义性，可以使用类型转换来指定对象

```c++
Worker * pw1 = (Waiter *) &ed; // the Worker in Waiter
Worker * pw2 = (Singer *) &ed; // the Worker in Singer
```

这使得使用基类指针来引用不同对象（多态性）变得复杂，并且包含两个基类对象拷贝还会导致其他的问题。而真正的问题是：为什么需要两个基类对象的拷贝？因此C++引入多重继承的同时，也引入了一种新技术——虚基类。

##### 虚基类

通过在类声明中使用关键词virtual，可以使Worker被用作Singer和Waiter的虚基类，virtual和public顺序无所谓。

```c++
class Singer : virtual public Worker {...};
class Waiter : public virtual Worker {...};
class SingingWaiter: public Singer, public Waiter {...};
```

为了使虚基类能够正常工作，**如果类有间接虚基类，则除非只需使用该虚基类的默认构造函数，否则必须显式地调用该虚基类的某个构造函数**。

C++在基类是虚的时，禁止信息通过中间类自动传递给基类。原因如下：自动传递信息时，将通过两条不同的途径将信息传递给Worker对象。

例如：

```c++
SingingWaiter(const Worker & wk, int p = 0, int v = Singer::other) :
Waiter(wk,p), Singer(wk,v) {}
```

自动传递信息存在的问题是：从两条不同的途径（Waiter，Singer）将wk传递给worker。因此上述构造函数将初始化p和v，但不会把wk自动传递给子对象。

所以如果不希望默认构造函数来构造虚基类对象，构造函数应是这样：

```c++
SingingWaiter(const Worker & wk, int p = 0, int v = Singer::other) :
Worker(wk), Waiter(wk,p), Singer(wk,v) {}
```

显式的调用基类构造函数，对于虚基类必须这样做，但对于非虚基类，这样是非法的（等于重复构造了基类对象）。

**多重继承调用哪个基类的方法**：可以采用**模块化**的方式，并使用**作用域解析运算符**来澄清编程者的意图。（这里可以看书P559，不是很复杂。



**混合使用虚基类和非虚基类**：

1. 如果基类是虚基类，派生类将包含基类的一个子对象；
2. 如果基类不是虚基类，派生类将包含多个子对象。



**使用虚基类将改变C++解析二义性的方式：**

- 如果类从不同的类那里继承了两个或更多的同名成员（数据或方法），则使用该成员名时，如果没有用类名进行限定，将导致二义性。

- 如果使用的是虚基类，则这样做不一定会导致二义性。在这种情况下，如果某个名称**优先于**（dominates）其他所有名称，则使用它时，即便不使用限定符，也不会导致二义性。

- 一个成员名**如何优先于**另一个成员名呢？ 派生类中的名称优先于直接或间接祖先类中的相同名称。
  例如：

  ```c++
  class B{
  public:
  		short q();
  		...
  };
  
  class C:virtual public B{
  public:
  		long q();
  		int omg()
  		...
  };
  
  class D:public C{
  		...
  };
  
  class E:virtual public B{
  private:
  		int omg()
  		...
  };
  
  class F:public D,public E{
  		...
  };
  
  ```

  在这个例子中，类C继承了类B中的q（），因此在F中可以直接使用q()来表示C::q()

  然而，因为类C与类E之间不存在派生关系，所以不存在优先关系，直接使用非限定omg()将导致二义性。

**虚二义性规则和访问规则无关：**

即使E::omg()是私有的，不能在F类中直接访问，但使用非限定的omg()仍将导致二义性。
即使C::q()是**私有**的，它将优先于D::q()(这里的D::q()由前提看来是从基类B继承而来)。在这种情况下，可以在类F中调用B::q(),如果不限定q()，则意味着要调用**不可访问**的C::q()

#### 类模板

C++模板提供参数化（parameterised）类型：能够让类型名作为参数传递给接收方来建立类或函数。

例如前面用到的valarray,vector,array

而16章会提到C++标准模板库（STL）



##### 定义类模板

模板类的代码定义：

```c++
template <typename Type> 
template <class Type> 
```

- `template` 告诉编译器，将要定义一个模板，尖括号中的内容相当于函数的参数列表。

- 关键字 `class/typename` 看作变量的类型名，变量接受类型作为其值，把Type看作是该变量的名称。
- Type 表示一个通用的类型说明符。在使用模板时，使用实际的类型替换它。

因此对于Stack类来说，应将声明中所有的typedef标识转换为Type

```c++
Item items[MAX];
//改为
Type items[MAX];
```

使用模版成员函数替换原有类的方法。**每个函数头**都将以**相同的模版声明**打头:并且限定符也需要变化

```c++
bool Stock::push(const Item &item){
		...
}

//改为

template <class Type>
bool Stack<Type>::push(const Type &item){
  	...
}

```

需要注意的是，**类模板本身不是类也不是成员函数**，因此它们不能单独编译。**模板必须与特定的模板实例化一起使用**，为此，最简单的方法是把用到的模板放在一个头文件里，然后include。

根据上面要求把Stack类变成Stack类模板，下一步就是声明模板类对象，**必须**显式的提供所需的类型（即Type）

```c++
Stack<int> kernels; // 存储int类型的栈类
Stack<string> colonels; //存储string类型的栈类
```

当然，使用的**算法**必须与**类型**一致。例如：Stack类假设可以将一个对象赋给另一个对象。这种假设对基本类型、结构和类来说是成立的（除非将赋值运算符设成私有），但对于数组不成立。

模版类的类型参数(type parameter)**必须显式声明**（因此在上面例子kernels声明中Type的值为int）。这就是与常规的函数模版有所不同的地方，常规的函数模版由**编译器**根据函数的参数类型来确定要生成哪种函数：

函数模板例：

```c++
template <class T>
void simple(T t){cout << t << '\n';}
...
simple(2);			//generate void simple(int)
simple("two");	//generate void simple(const char *)
```



##### 深入探讨模板类

可以将内置类型或类对象用作类模板Stack<Type>类型。那么指针可以吗？例如，14.14中用（书上有代码）char指针替代string对象。

答案是可以的，但如果不对程序做重大修改，无法很好的完成工作。下面介绍几个例子

1.

```c++
Stack<char *> st;
char * po;//原本是string
```

这以为着，程序将用char指针而不是string对象来接收cin的输入，所以这种方法很明显不行。

2.

```c++
Stack<char *> st;
char po[40];//原本是string
```

这个方法为字符串分配了空间，然而，与方法pop()相冲突。

```c++
template <class Type>
bool Stack<Type>::pop(Type & item)
{
	if(top>0)
	{
		item = items[--top];
		return true;
	}
	else
		return false;
}
```

首先&item必须是引用的某种类型的左值而不是数组名。就算可以给item赋值，即使item能引用数组，也不能为数组名赋值（因为数组名作为指向首元素的地址），赋值给地址没有意义，因此这种方法也不行。

3.

```c++
Stack<char *> st;
char * po= new char[40];//原本是string
```

这个看似为字符串分配了内存空间，并且po是变量而不是数组名因此和pop也兼容，然而这里面临的问题是，po是指向动态数组的指针，**因此该变量总是指向相同的内存单元**，即，每次读取字符串时内容也许不一样但每次执行压入操作时，压入的都是同一个地址，因此对栈执行弹出操作时，弹出的地址总是相同的。简而言之，栈没有保存每一个新的字符串，而是在原本的基础上覆盖了旧字符串的内存，但地址没变，因此没有任何用途。

##### 正确使用指针栈

使用指针栈的方法之一是，让程序提供一个指针数组，其中每个指针都指向不同的字符串。注意，创建不同指针是调用程序的职责，而不是栈的职责，栈的任务是管理指针，而不是创建指针。

```c++
Type * items;
```



#### 数组模板

一种方法是在类中使用动态数组和构造函数来提供元素数目，另一种方法是使用模板参数来提供常规数组大小，C++11新增的模板array就是这样做的。

```c++
template <class T, int n>   // T为类型参数，n为int类型(非类型/表达式参数)
class ArrayTP
{
	private:
    	T ar[n];
    public:
    	...
};

template <class T, int n> 
Array<T,n>::ArrayTP(const T & v)
{
    for (int i =0;i <n; i++)
        ar[i]= v;
}
// 定义一个名为ArrayTP<double, 12>的类，并创建一个类型为ArrayTP<double, 12>的eggweight对象。
ArrayTP<double, 12> eggweights; 
```

关键词class（或typename）指出T为类型参数，int指出n的类型为int。这种参数（指定特殊的类型而不是用作泛称）成为非类型或表达式参数。

表达式参数有一些限制，**表达式参数可以是整型、枚举、引用或指针。** 例如 double m是非法的，但是double *rm和double & pm是合法的。

另外，**模板代码不能修改参数的值，也不能使用参数的地址**。即（n++,&n）是不合法的。

最后，**实例化模板时，用作表达式参数的值必须是常量表达式。**例如这里不能只写ArrayTP<double, 12> eggweights而不能是ArrayTP<double, n> eggweights



那么数组模板和Stack中的构造函数方法相比的区别：

- 构造函数方法使用的是通过new和delete管理的堆内存，而表达式参数方法使用的是为自动变量维护的内存栈。

- 表达式参数的优点：执行速度将**更快**，尤其是在使用了很多小型数组时。

- 表达式参数的缺点：每种数组大小都将生成自己的模板。（见下面代码）

- 构造函数方法**更通用**，这是因为数组大小是作为类成员（而不是硬编码）存储在定义中的。这样可以将一种尺寸的数组赋给另一种尺寸的数组，也可以创建允许数组大小可变的类。

  ```c++
  // 表达式参数生成两个独立的类声明
  ArrayTP<double, 12> eggweights;
  ArrayTP<double, 13> donuts;
  // 只生成一个类声明，并将数组大小信息传递给类的构造函数
  Stack<int> eggs(12);
  Stack<int> dunkers(13);
  ```



#### 模板多功能性

```c++
template <typename T> // or <class T>
class Array
{
private:
    T entry;
    ...
};//数组类模板

template <typename Type>
class GrowArray : public Array<Type> {...}; // inheritance

template <typename Tp>
class Stack
{
    Array<Tp> ar; // use an Array<> as a component
    ...
};//栈类模板
...
Array < Stack<int> > asi; // 元素为栈的数组
// 上面一行代码C++98要求>>之间有空格，C++11不要求
```



1. **递归的使用模板**

```c++
ArrayTP<ArrayTP<int,5>,10> twodee;
```

twodee是一个包含是个元素的数组，其中每个元素都是包含五个元素的数组。

即

```c++
int twodee[10][5];
```

在模板语法中，维的顺序与等价的二维数组相反。

2. 使用多个类型参数

   模板可以包含多个类型参数，例如，希望类可以保存两种值。

   ```c++
   template <class T1, class T2>
   class Pair
   {
   private:
       T1 a;
       T2 b;
   public:
       T1 & first();
       T2 & second();
       T1 first() const { return a; }
       T2 second() const { return b; }
       Pair(const T1 & aval, const T2 & bval) : a(aval), b(bval) { }
       Pair() {}
   };
   ...
   ```


3. 默认类型的模板参数

   类模板的另一项新特性是，可以为类型参数提供默认值：

   ```c++
   template <class T1,class T2=int> class Topo{...}
   
   Topo<double,double> m1;
   Topo<double> m2;//如果不指定第二个参数，那么就默认为int
   ```



#### 模板的具体化

与函数模板类似，可以有隐式实例化、显式实例化和显式具体化，它们统称为具体化 。模板以泛型的方式描述类，而具体化是使用具体的类型生成类声明。

1. 隐式实例化

   前面用到的，目前来说都是隐式实例化，形如

   ```c++
   ArrayTP<int,100> stuff;
   ```

   需要注意的是，编译器直到需要对象，都不会生成类的隐式实例化。即：

   ```c++
   ArrayTP<int,100> *pt;//仅指针，不会生成类定义
   pt = new ArrayTP<int,100>;//此时在创建一个对象，因此需要根据模板创建类定义
   ```

2. 显式实例化

   当使用关键字template并指出所需类型来声明类时，编译器将生成类声明的显式实例化

   ```c++
   template class ArrayTP<int,100>;//生成一个ArrayTP<int,100>类
   ```

3. 显式具体化

   是特定类型（用于替换泛型）的定义。有时候可能需要为特殊类型实例化时，对模板进行修改，使其行为不同。这时候就用到了显式具体化，例如：

   ```
   template <typename T>
   class SortedArray
   {
   	...//假设这个类的元素在加入时被排序
   }
   ```

   假设模板用>运算符来对值进行比较，如果元素是数字，那么这合理，但如果元素是字符串，这就不合理了。

   因此，可以提供一个显式模板具体化，这将为具体类型定义的模板，而不是为泛型定义的模板。（类型匹配就用具体的，没有匹配的具体化就用泛型版本的模板）

   具体化模板格式和函数类似

   ```c++
   template<> class ClassName<specialized-type-name>{...}
   ```

   所以上面的例子，可以用具体化模板

   ```c++
   template<> class SortedArray<const * char>
   ```

   在这个模板里，可以用strcmp()来进行比较而不是大于小于号。



4. 部分具体化

   即部分限制模板的通用性。例如，部分具体化可以给类型参数之一指定具体的类型。

   ```c++
   //一般模板
   template<class T1,class T2> class Pair{...}
   //部分具体化模板
   template<class T1> class Pair<T1,int> {...}
   //显式具体化
   template<> class Pair<int,int> {...}
   ```

   关键词template后面的<>声明的是没有被具体化的参数，如第二个里面的class T1，若<>里是空的，那么就变成显式具体化了（可以理解成显式具体化是所有参数都具体化的特殊的部分具体化）

   如果有多个模板可供选择，编译器将使用具体化程度最高的模板。

   ```c++
   Pair<double,double> p1;//对应第1个
   Pair<double, int> p2;//2
   Pair<int,int> p3;//3
   ```

   同样，不只是数据类型，指针也可以有具体化的模板

   ```c++
   template<class T>
   class Feeb {...};
   template<class T*>
   class Feeb {...};
   
   Feeb<char> fb1;
   Feeb<char*> fb2;
   ```

   如果没有进行部分具体化，则第二个声明将使用通用模板，如果进行了部分具体化，则第二个声明将使用具体化模板，其中T为char。

   部分具体化特性使得能够设置各种限制，例如：

   ```c++
   //通用模板
   template<class T1,class T2,class T3> class Trio{...};
   //把T3设置为T2同类型的
   template<class T1,class T2> class Trio<T1,T2,T2>{...};
   //把T3和T2都设置成T1*类型
   template<class T1> class Trio<T1,T1*,T1*>{...};
   ```



#### 成员模板

模板可以用作结构、类或模板类的成员。可以看P584的14.20程序清单，这个例子的模板类将另一个模板类和模板函数作为其成员。

在这个例子中，T、V、U用作模板的参数，且存在嵌套关系，那么在定义函数或者定义类的时候，要用到下面的语法：

```c++
template <typename T>
	template<typename V>
		class beta<T>::hold
        {
            private:
            	V val;
            public:
            	hold(V v=0): val(v){}
        }
template <typename T>
	template<typename U>
		U beta<T>::blab(U u, T t)
        {
            return (n.Value()+q.Value())*u/t;
        }
//而不能使用下面的语法
template<typename T,typename V> //不行
```

定义中还必须指出hold和blab是beta<T>类的成员，这是通过作用域解析运算符来完成的（::）。



##### 将模板作为参数

类模板，不仅仅可以把基本类型作为参数，它还能将另一模板类作为参数。

```c++
//一般类模板
template <typename T>
//以类模板作为参数的类模板
template<template <typename T> class Thing>
class Crab
{
    private:
    	Thing<int> s1;
    	Thing<double> s2;
    ...
}
int main
{
    Crab<Stack> nebula;
}
```

这里的Thing就是等价于通用模板中的T（类名称），而此处的参数类型则是template <typename T> class，即模板类。

即，以另一个模板类Thing为参数的一个模板类。

所以此处main函数中声明了一个以Stack（Thing）类为参数的模板类Cra

此时，Crab类中声明的两个私有成员将被替换为

```c++
Stack<int> s1;
Stack<double> s2;
```

同样的，模板参数和常规参数也可以混用

```c++
template<template <typename T> class Thing, typename U,typename V>
class Crab
{
    ...
}
Crab<Stack,int,double> nebula;
```



##### 模板类和友元

模板类声明也可以有友元，友元分三类：

- 非模板友元
- 约束模板友元，即友元的类型取决于类被实例化时的类型
- 非约束模板友元，即友元的所有具体化都是类的每一个具体化友元（见下面解释）

这个部分看看书，代码解释的比较清楚

其中，约束模板友元要比非模板友元多以下3步：

- 首先，在类定义之前声明每个模板函数。

  ```c++
  template <typename T> void counts();
  template <typename T> void report(T &);
  ```

- 在函数中再次将约束模板声明为友元，这些语句根据类模板的参数类型声明具体化。

  ```
  template<typename TT>
  class HasFriendT
  {
  	..
  	friend void count<TT>();
  	friend void report<>(HasFriendT<TT> &)
  }
  ```

  其中，声明中的<>指出这是模板具体化，对于report（），<>可以为空，因为可以通过参数判断模板类型参数T，当然，也可以声明出具体化类型

  ```
  friend void report<HasFriendT<TT> >(HasFriendT<TT> &)
  ```

  但counts（）没有参数，因此必须使用模板参数语法<>，即该counts函数是哪个模板类的友元函数。

  为了更好理解，假设声明了如下的对象：

  ```c++
  HasFriendT<int> squack;
  ```

  编译器将使用int替换TT，生成如下定义

  ```c++
  class HasFriendT<int>
  {
  	..
  	friend void count<int>();
  	friend void report<>(HasFriendT<int> &)
  }
  ```

- 模板类的非约束模板友元函数

  ```c++
  template <typename T>
  class ManyFriend
  {
  	...
  	template< typename C, typename D> friend void show(C&,D);
  	}
  ```

  对于非约束友元，友元模板类型参数与模板类类型参数是不同的，（上面的counts就和模板类类型相同，都是TT，而这里C和D与T不同。

  通过在类内部声明模板，可以创建非约束友元函数（前面的约束友元函数需要在类定义之前声明每个模板函数）

  例：

  ```c++
  void show2<ManyFriend<int>& , ManyFriend<int>&> (ManyFriend<int>& c,ManyFriend<int>& d)//ManyFriend<int>是模板ManyFriend的int类的具体化
  ```

  因为上述函数show2是ManyFriend具体化的友元，所以能访问所有具体化的item成员，因此我们使用ManyFriend<int>使得它仅访问ManyFriend<int>对象。

  解释：如果同时创建了int对象和double对象，这时候我们会有两个ManyFriend类，两个类中存在int item和double item，从根本上讲，show2是允许访问int item 和 double item的，但是这可能不满足我们的要求，因此在show2中可以看到我们使用了模板类型参数语法，进行根据类型的重载。

  同理

  ```c++
  void show2<ManyFriend<double>& , ManyFriend<double>&> (ManyFriend<double>& c,ManyFriend<double>& d)
  ```

  它也是ManyFriend具体化的友元，所以能访问所有具体化的item成员，这里让它访问了ManyFriend<double>对象的item成员。

##### 模板别名

```c++
template<typename T>
	using arrtype = std::array<T,12>;
	
arrtype<double> g;// 等价于 std::array<double,12>
```

