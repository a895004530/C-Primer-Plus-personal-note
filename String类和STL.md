# String类和STL

#### String类

构造函数：size_type 是一个依赖于实现的整型，是在头文件 string 中定义的。 string 类将 string::npos 定义为字符串的最大长度，通常为 unsigned int 的最大值。另外，表格中使用缩写 NBTS(null-terminated string) 来表示以空字符结束的字符串——传统的 C 字符串。


| 构造函数                                                     | 描述                                                         |
| ------------------------------------------------------------ | :----------------------------------------------------------- |
| string(const char * s)                                       | 将 string 对象初始化为 s 指向的 NBTS                         |
| string(size_type n, char c)                                  | 创建一个包含 n 个元素的 string 对象，其中每个元素都被初始化为字符 c |
| string(const string & str）                                  | 将一个 string 对象初始化为 string 对象 str（复制构造函数）   |
| string()                                                     | 创建一个默认的 string 对象，长度为 0（默认构造函数）         |
| string(const char * s, size type n)                          | 将 string 对象初始化为 s 指向的 NBTS 的前 n 个字符，即使超过了 NBTS 的结尾 |
| template<class Iter> string(Iter begin, Iter end)            | 将string对象初始化为区间 [begin,end) 内的字符，其中 begin 和 end 的行为就像指针，用于指定位置，范围包括 begin 在内，但不包括 end |
| string(const string & str, size_type pos = 0, size_type n = npos | 将一个 string 对象初始化为对象 str 中从位置 pos 开始到结尾的字符，或从位置 pos 开始的 n 个字符 |
| string(string && str) noexcept                               | 这是 C++11 新增的，它将一个 string 对象初始化为 string 对象 str， 并可能修改 str(移动构造函数) |
| string(initializer_list<char> il                             | 这是 C++11 新增的，它将一个 string 对象初始化为初始化列表 il 中的字符 |

使用构造函数时都进行了简化，即隐藏了这样的一个事实：string实际上时模板具体化 basic_string<char> 的一个 typedef

对于string(string && str) noexcept：类似于复制构造函数，导致新创建的string为str的副本。但与复制构造函数不同的是，它不保证将str视作const，这种构造函数被称作移动构造函数（十八章）

对于string(initializer_list<char> il

```c++
std::string list111 = { 'Q','X','Y' };//使得string类可以这样声明合法。
std::cout << list111;//显示：QXY
```



#### string类输入

对于C-风格字符串，有三种输入方式：

```c++
char info[100]
cin >>info;
cin.getline(info,100);
cin.get(info,100);
```

对于string类对象，有两种：

```c++
string stuff;
cin>>stuff;
getline(cin,stuff);
```

两个版本的getline()都有一个可选参数，用于指定使用哪个字符来确定输入的边界：

```c++
cin.getline(info,100,':')//读到':'停止
getline(stuff, ':' );//string版本的可以自动调整目标string对象的大小
```

自动调整大小的功能让 string 版本的 getline() 不需要指定读取多少个字符的数值参数。

```c++
char fname[10];
string lname;
cin >> fname; 	// could be a problem if input size > 9 characters
cin >> lname;	// can read a very, very long word
cin.getline(fname, 10);		// may truncate input
getline(cin, fname);			// no truncation
```

在设计方面的一个区别是，读取 C-风格字符串的函数是 istream 类的方法，而 string 版本是独立的函数。这就是对于 C-风格字符串输入，cin 是调用对象；而对于 **string 对象**输入，cin 是一个**函数参数**的原因。这种规则也适用于 >> 形式，如果使用函数形式来编写代码，这一点将显而易见


```c++
cin.operator>>(fname);		// cin是ostream的类方法，用调用对象cin.operator>>
operator>>(cin, lname);		// 对于string对象的运算符重载，cin是ostream的类方法，因此是作为函数参数。
```

#### 深入探讨string输入函数

对于string的输入函数，两个方法都自动调整string的大小，但也存在一些限制。

第一个限制因素是string类对象的**最大允许长度**，由常量string::npos指定。这通常是最大的unsigned int值，因此对于普通的交互式输入，这不会带来实际的限制，但是如果您试图将整个文件的内容读取到单个string对象中，这可能成为限制因素

第二个限制因素是程序可以使用的**内存量**。



string版本的getline()函数从输入中读取字符，并将其存储到string中，直到发生下列三种情况之一：

- 到达文件尾，在这种情况下，输入流的 eofbit 将被设置，这意味着方法 fail() 和 eof() 都将返回 true;
  - *eofbit*是C++中的一个流状态标志，用于指示输入流已到达其输入源的结尾，当eofbit被设置时，表示后续的读取操作将无法从输入流中读取更多的数据。

- 遇到分界字符（默认为\n），在这种情况下，**将把分界字符从输入流中删除**，**但不存储它**；
- 读取的字符数达到最大允许值（string::nops 和可供分配的内存字节数中较小的一个），在这种情况下，将设置输入流的 failbit，这意味着方法 fail() 将返回 true。



string 版本的 operator>>() 函数的行为与此类似，只是它不断读取，直到遇到空白字符并将其**留在输入队列**中，而不是不断读取，直到遇到分界字符并将其丢弃。空白字符指的是空格、换行符和制表符，更普遍地说，是任何将其作为参数来调用 isspace() 时，该函数返回 true 的字符。

例：

```c++
// strfile.cpp -- read strings from a file

#include<iostream>
#include<fstream>
#include<string>
#include<cstdlib>

int main(){
    using namespace std;

    ifstream fin;
    fin.open("tobuy.txt");

    if (fin.is_open() == false ){
        cerr << "Can't open file. Bye.\n";
        exit(EXIT_FAILURE);
    }
    string item;
    int count = 0;
    getline(fin, item, ':');

    while(fin){
        ++count;
        cout << count << ": "<< item << endl;
        getline(fin, item, ':');
    }
    cout << "Done\n";
    fin.close();
    return 0;
}
```

需要注意的是：将':'作为分界字符后，换行符将被视为常规字符（即不会打断getline的读取）。



#### 字符串的使用

string类对全部六个关系运算符都进行了重载，并且对于每个关系运算符都以三种方式被重载，以便能够将string对象和另一个string对象或c-风格字符串进行比较。（s-s,s-c,c-s三种)

```c++
string snake1("cobra");
string snake2("coral");
char snake3[20] = "anaconda";
if (snake1 < snake2)		// operator<(const string &, const string &)
	...
if (snake1 == snake3)		// operator==(const string &, const char *)
	...
if (snake3 != snake2)		// operator!=(const char * , const string &)
	...
```

可以确定字符串的**长度**，size()和length()成员函数都返回字符串中的字符数。

```c++
if (snake1.length() == snake2.length() )
	cout << "Both strings have the same length.\n";
```



搜索：

可以以多种不同的方式在字符串中搜索给定的子字符串或字符。

string::npos 是字符串可存储的最大字符数

| 方法原型                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| size_type find(const string & str, size_type pos = 0 ) const | 从字符串的 pos 位置开始，查找**子字符串 str**。如果找到，则返回该字符串首次出现时其首字符的索引；否则，返回string::npos |
| size_type fine(const char * s, size_type pos = 0) const      | 从字符串的pos位置开始，查找子字符串s。如果找到，则返回该字符串首次出现时其首字符的索引；否则，**返回 string::npos** |
| size_type find(const char * s, size_type pos = 0, size_type n ) | 从字符串的 pos 位置开始，查找s的**前n个字符**组成的**子字符串**。如果找到，则返回该子字符串首次出现时其首字符的索引；否则，**返回 string::npos** |
| size_type fine(char ch, size_type pos = 0) const             | 从字符串的 pos 位置开始，查找**字符** ch。如果找到，则返回该字符首次出现的位置；否则，**返回 string:npos** |

string库还提供了相关的方法：

| 方法名                         | 作用                                         |
| ------------------------------ | -------------------------------------------- |
| rfind()                        | 查找子字符串或字符最后一次出现的位置         |
| find_first_of()/find_last_of() | 查找参数中任何一个字符首次（最后）出现的位置 |
| find_first_not_of()            | 查找第一个不包含在参数中的字符               |
| find_last_not_of()             | 查找最后一个不包含在参数中的字符             |

具体find实例见代码16.3



#### 智能指针模板类

要创建智能指针对象，必须包含头文件 memory，该文件含有模板定义。然后使用通常的模板语法来实例化所需的类型的指针。例如，模板 auto_ptr 包含如下构造函数：

```c++
template<class X> class auto_ptr {
public:
	explicit auto_ptr(X* p = 0) throw();
	...
}
```

本书前面说过，throw() 意味着构造函数不会引发异常；与 auto_ptr 一样，throw() 也被摒弃。因此，请求 X 类型的 auto_ptr 将获得一个指向 X 类型的 auto_ptr：

```c++
auto_ptr<double> pd(new double); // pd an autp_ptr to double
								 // (use in place of double *pd
autp_ptr<string> ps(new string);// ps an auto_ptr to string
								// (use in place of string * ps)
```

new double 是 new 返回的指针，指向新分配的内存块。它是构造函数 auto_ptr<double> 的参数，即对应于原型中形参 p 的实参。同样，new string 也是构造函数的实参。其他两种智能指针使用同样的语法：

```c++
unique_ptr<double> pdu(new double);	// pdu an unique_ptr to double
shared_ptr<string> pss(new string);	// pss a shared_ptr to string
```

因此要使用智能指针：

- 包含头文件memory

- 将指向对象的指针替换为指向对象的智能指针，即：

  ```c++
  string str = new string;//替换掉
  auto_ptr<string> ps(new string);//智能指针
  ```

- 删除原本用new构造指针的delete 语句

- 注意到智能指针模板位于名称空间 std 中。



所有智能指针都有一个explicit构造函数，即不能隐式构造，该构造函数将指针作为参数，因此不需要自动将指针转换为智能指针对象

```c++
shared_ptr<double> pd;
double * p_reg = new double;
pd = p_reg;				// 不允许隐式使用构造函数生成智能指针。
pd = shared_ptr<double>(p_reg);	// 普通指针则没有这个限制
shared_ptr<double> pshared = p_reg;	// 隐式，不允许
shared_ptr<double> pshared(p_reg);	// 显式调用构造函数，允许
```



对全部三种智能指针都应避免的一点：

```c++
string vacation("I wandered lonely as a Cloud.");
shared_ptr<string> pvac(&vacation);	// NO!
```



#### 有关智能指针的注意事项

为何又三种智能指针呢？实际上有4种，但本书不讨论 weak_ptr。为何摒弃 auto_ptr 呢？

先看下面的赋值语句

```c++
auto_ptr<string> ps (new string("I reigned lonely as cloud." ) );
auto_ptr<string> vocation;
vocation = ps;
```

如果 ps 和 vocation 是常规指针，则两个指针将指向同一个 string 对象。这是不能接受的，因为程序将试图删除同一个对象两次。

要避免这种问题，方法有多种：

- 定义赋值运算符，使之执行深复制。这样两个指针将指向不同的对象，其中的一个对象是另一个对象的副本
- 建立所有权（ownership）概念，对于特定的对象，只能有一个智能指针可拥有它，这样只有拥有对象的智能指针的构造函数会删除该对象。然后，让赋值操作转让所有权。这就是用于 auto_ptr 和 unique_ptr 的策略，但 unique_ptr 更严格。
- 创建智能更高的指针，跟踪引用特定对象的智能指针数。这成为**引用计数**（reference counting）。例如，赋值时，计数将加1，而指针过期时，计数将减1。仅当最后一个指针过期时，才调用 delete。这是 shared_ptr 采用的策略。
  

auto_ptr采用的是所有权策略，他与unique_ptr的区别在于：

```c++
auto_ptr<string> p1(new string("auto"));	// #1
auto_ptr<string> p2;			// #2
p2 = p1;
```

在auto_ptr中 p2会接管p1的所有权，此时如果程序随后试图使用p1，p1此时不再指向有效的数据，因此会报错。

而unique_ptr:

```c++
unique_ptr<string> p3(new string("auto");	// #4
unique_ptr<string> p4;						// #5
p4 = p3;
```

在p4=p3这会被编译器认为非法，避免了p3不再指向有效数据的问题（编译阶段错误比潜在程序崩溃更安全）

那为什么不直接把赋值都认作非法呢？

看如下代码：

```c++
unique_ptr<string> demo(const char *s) {
	unique_ptr<string> temp(new string(s));
	return temp;
}

unique_ptr<string> ps;
ps = demo("Uniquely special");
```

函数demo的参数是字符串，并且在函数内部创建了一个智能指针指向这个字符串，返回给了一个临时指针temp，随后再把这个临时指针赋给ps，再返回完毕之后，由于函数的性质，很快的temp就会被销毁，因此它没有机会来访问无效的数据。



**总之**，程序试图将一个 unique_ptr 赋给另一个时，如果源 unique_ptr 是个**临时右值**，编译器允许这样做；如果源 unique_ptr 将存在一段时间，编译器将禁止这样做

即：

```c++
using namespace std;
unique_ptr<string> pu1(new string "Hi ho!");
unique_ptr<string> pu2;
pu2 = pu1;		// #1 不允许
unique_ptr<string> pu3;
pu3 = unique_ptr<string> (new string "Yo!");	// #2 允许
```

另外，需要**注意**：

使用 new 分配内存时，才能使用 auto_ptr 和 shared_ptr，使用 new[]分配内存时不能使用它们，而要用unique_ptr，因为unique_ptr有使用new[]和delete[]的版本。



### 标准模板库STL

#### 模板类vector

类似array

```c++
#include <vector>
using namespace std;
vector<int> ratings(5);			// a vector of 5 ints
int n;
cin >> n;
vector<double> scores(n);	// a vector of n doubles

ratings[0] = 9;
for (int i = 0; i < n; i++){
	cout << scores[i] << endl;
}
```



分配器：与 string 类相似，各种 STL 容器模板都接受一个可选的模板参数，该参数指定使用哪个分配器对象来管理内存。例如，vector 模板的开头与下面类似：

```c++
template<class T, class Allocator = allocator<T> >
	class vector { ...
```



#### 迭代器

它是一个广义指针，它可以是指针也可以是对其进行类似指针的操作如接触引用、递增

通过将指针广义化为迭代器，让STL能够为不同的容器类提供统一接口。每个容器类都定义了一个合适的迭代器，该迭代器的类型是一个名为iterator的typedef

例如，要为vector的double类型声明一个迭代器：

```c++
vector<double>::iterator pd;

vector<double> scores;

pd = scores.begin();//pd指向scores的第一个元素
*pd=22.3;//解除引用并且把第一个元素指定为22.3
++pd;//指向下一个元素
```

##### begin

自动类型推断在这很有用：

```c++
vector<double>::iterator pd=scores.begin();
//可以直接这样
auto pd = scores.begin();
```

##### end

我们知道begin是容器第一个元素，那么end呢，end是容器最后一个元素后面的元素，和C-风格字符串最后一个字符后面的空字符类似，只是空字符是一个值，而“超过结尾”是指向一个元素的指针（迭代器），即我们可以这样：

```c++
for (pd = scores.begin(),pd !=scores.end(),pd++)
```

来遍历所有元素。



##### push_back()

该方法将元素添加到矢量末尾，这样做时它将负责内存管理，增加矢量的长度，使之能够接纳新的成员。

```c++
vector<double> scores;		// create an empty vector
double temp;
while (cin >> temp && temp >= 0) {
	scores.push_back(temp);
}
cout << "You entered " << scores.size() << " scores.\n";
```

在编写或运行程序时，**无需了解元素的数目**。只要能够取得足够的内存，程序就可以根据需要增加 scores 的长度。



##### erase()

删除矢量给定区间的元素，它接收两个迭代器参数，第一个指开始位置，第二个指结束位置的后一个元素（类似end）。

```c++
scores.erase(scores.begin(),scores.begin()+2);
//这个例子则删除begin(),begin()+1两个元素
```

可以理解成删除[it1,it2)的元素



##### insert()

功能与erase相反，它接收三个迭代器参数：

it1:新元素插入位置，即insert到哪个位置，注意是向前插入，见下面例子。

it2,it3:定义了被插入区间，即insert的部分是什么，该区间一般是另一个容器对象的一部分，例：

```
vector<int> old_v;
vector<int> new_v;
...
old_v.insert(old_v.begin(), new_v.begin()+1, new_v.end());
```

上述例子将new_v从第二个元素开始到最后一个元素，全部插入到old_v的首元素位置**前面**。

同样也可以插在矢量末尾

```c++
old_v.insert(old_v.end(), new_v.begin(), new_v.end());
```



#### 对vector可执行的其他操作

程序员通常要对数组执行很多操作，如搜索、排序、随机排序等。vector 模板类包含了执行这些常见的操作的方法吗？**没有**！STL 从更广泛的角度定义了非成员（non-member）函数来执行这些操作，即不是为每个容器类定义 find() 成员函数，而是定义了一个**适用于所有容器类**的**非成员函数** find()。这种设计理念省去了大量重复的工作。例如，假设由8个容器类，需要支持10种操作。如果每个类都有自己的成员函数，则需要定义 80（8*10）个成员函数。但采用 STL 方式时，只需要定义 10 个非成员函数即可。在定义新的容器类时，只要遵循正确的指导思想，则它也可以使用已有的 10 个非成员函数来执行查找、排序等操作。



另一方面，即使有执行相同任务的非成员函数，STL 有时也会定义一个成员函数。**这是因为对有些操作来说，类特定算法的效率比通用算法高**，因此，vector 的成员函数 swap() 的效率比非成员函数 swap() 高，但非成员函数让您能够交换两个类型不同的容器的内容。

下面来看 3 个具有代表性的 STL 函数：for_each()、random_shuffle() 和 sort()。



##### for_each()

可用于很多容器类，接受三个参数。前两个是定义容器中区间的迭代器，最后一个是指向函数的指针（函数对象）

for_each函数将被指向的函数应用于容器区间中的各个元素。被指向的函数不能修改容器元素的值。可以用for_each来代替for循环，例如：

```c++
vector<Review>::iterator pr;
for (pr = books.begin(); pr != books.end(); pr++){
	ShowReview(*pr);
}
//替换为
for_each(books.begin(),books.end(),ShowReview)//这里第三个对象是函数名
```

这样可以避免显式的使用迭代器变量。



##### random_shuffle()

接受两个指定区间的迭代器参数，并随机排列该区间中的元素。例如，下面的语句随机排列books矢量中的所有元素：

```C++
random_shuffle(books.begin(), books.end() );
```

与可用于任何容器类的 for_each 不同，该函数要求容器类允许随机访问，vector类可以做到这一点。



##### sort()

sort()函数也要求容器支持随机访问，该函数有两个版本，第一个版本接受两个定义区间的迭代器参数，并使用为存储在容器中的类型元素定义的<运算符，对区间中的元素进行操作。

```
vector<int> coolstruff;
...
sort(coolstuff.begin(), coolstuff.end());
```

如果容器元素是用户定义的对象，则要使用 sort()，必须定义能够处理该类型对象的 operator<() 函数。例如，如果为 Review 提供了成员或非成员函数 operator<()，则可以对包含 Review 对象的矢量进行排序。由于 Review 是一个结构，因此其成员是公有的，这样的非成员函数将为：（即需要定义适用于用户定义对象的小于号运算符）

```c++
bool operator<(const Review & r1, const Review & r2) {
	if (r1.title < r2. title){
		return true;
	}
	else if (r1.title == r2.title && r1.rating < r2.rating){
		return true;
	}
	else{
		return false;
	}
}
```

有了这样的函数后，就可以对包含 Review 对象（如 books）的矢量进行排序了：

```c++
sort(books.begin(),books.end());
```

另一个版本的sort接收三个参数，前两个还是表示区间，第三个则是指向要使用的函数指针（函数对象）

比如现在想要仅根据rating来排序

```c++
bool WorseThan(const Review & r1, const Review & r2) {
	if (r1.rating < r2.rating) {
		return true;
	}
	else{
		return false;
	}
}
```

有了这个版本的sort则可以实现仅根据rating来排序

```c++
sort(books.begin(),books.end(),WorseThan);
```



##### 基于范围的 for 循环

基于范围的 for 循环是为用于 STL 而设计的。为复习该循环，下面是第5章的第一个示例：

```c++
double prices[5] = {4.99, 10.99, 6.87, 7.99, 8.49};
for(double x : prices){
	std::cout << x << std::endl;
}
```

在这种 for 循环中，括号内的代码声明一个类型与容器存储的内容相同的变量，然后指出了容器的名称。接下来，循环体使用指定的变量依次访问容器的每个元素。例如，对于下述摘自程序的语句：

```c++
for_each(books.begin(), books.end(), ShowReview);
//可替换为
for(auto x : books ) ShowReview(x);
```

不同于 for_each()，基于范围的 for 循环可修改容器的内容，诀窍是指定一个引用参数。

```c++
void InflateReview(Review &r) {
	r.rating ++;
}
```

结合for循环，可以实现对books每一个元素执行rating加一的操作

```c++
for (auto & x:books) InflateRevie(x);
```



#### 迭代器的基本知识

STL是一种泛型编程，面向对象编程关注的是编程的数据面，而泛型编程关注的是算法。它们之间的共同点是抽象和创建可重用代码，但它们的理念决然不同

泛型编程旨在编写**独立于数据类型的代码**。在 C++ 中，完成通用程序的工具是模板。当然，模板使得能够按泛型定义函数或类，而 STL 通过通用算法更进了一步。模板让这一切成为可能，但必须对元素进行仔细地设计。为解模板和设计是如何协同工作的，来看一看**需要迭代器的原因**。



##### 迭代器类型

STL定义了5种迭代器，

- 输入迭代器
- 输出迭代器
- 正向迭代器
- 双向迭代器
- 随机访问迭代器

例如，find() 的原型与下面类似：

```c++
template<class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& value);
```

这指出，这种算法需要一个输入迭代器。同样，下面的原型指出排序算法需要一个随机访问迭代器：

```c++
template<class RandomAccessIterator>
void sort(RandomAccessIterator first, RandomAccessIterator last);
```



- 输入迭代器:
  术语“输入”是从程序的角度说的，即**来自容器的信息被视为输入**，就像来自键盘的信息对程序来说是输入一样。因此，输入迭代器可被程序用来**读取容器中的信息**。具体地说，对输入迭代器解除引用将使程序能够读取容器中的值，**但不一定能让程序修改值**。因此，**需要输入迭代器的算法将不会修改容器中的值**。

  输入迭代器**必须能够访问容器中所有的值**，这是通过支持++运算符（前缀格式和后缀格式）来实现的。如果将输入迭代器设置为指向容器中的第一个元素，并不断将其递增，直到到达超尾位置，则它将依次指向容器中的每一个元素。顺便说一句，并不能保证输入迭代器第二次遍历容器时，顺序不变。另外，输入迭代器被递增后，也不能保证其先前的值仍然可以被解除引用。基于输入迭代器的任何算法都应当是单通行（single-pass）的，不依赖于前一次遍历时的迭代器值，也不依赖于本次遍历中前面的迭代器值。

  注意，**输入迭代器是单向迭代器，可以递增，但不能倒退**。

- 输出迭代器：

  STL 使用术语“输出”来指用于**将信息从程序传输给容器的迭代器**，因此程序的输出就是容器的输入。输出迭代器与输入迭代器相似，**只是解除引用让程序能够修改容器值，而不能读取**。也许您会感到奇怪，能够写，却不能读。发送到显示器上的输出就是如此，cout 可以修改发送到显示器的字符流，却不能读取屏幕上的内容。STL足够通用，其容器可以表示输出设备，因此容器也可能如此。另外，如果算法不用读取作容器的内容就可以修改它（如通过生成要存储的新值），则没有理由要求它使用能够读取内容的迭代器。
  **简而言之，对于单通行、只读算法，可以使用输入迭代器；而对于单通行、只写算法，则可以使用输出迭代器**。

- 正向迭代器

  与输入迭代器和输出迭代器相似，**正向迭代器只使用++运算符来遍历容器**，所以它每次沿容器向前移动一个元素；然而，与输入和输出迭代器不同的是，它总是按相同的顺序遍历一系列值。另外，将正向迭代器递增后，仍然可以对前面的迭代器值解除引用（如果保存了它），并可以得到相同的值。这些特征使得多次通行算法成为可能。

  正向迭代器**既可以使得能够读取和修改数据，也可以使得只能读取数据**：

  ```
  int *pirw;			// read-write iterator
  const int * pir;		// read-only iterator
  ```

- 双向迭代器

  假设算法需要能够双向遍历容器，情况将如何呢？例如，reverse 函数可以交换第一个元素和最后一个元素、将指向第一个元素的指针加1、将指向第二个元素的指针减1，并重复这种处理过程。**双向迭代器具有正向迭代器的所有特性，同时支持两种（前缀和后缀）递减运算符**

- 随机访问迭代器

  有些算法（如标准排序和二分检索）要求**能够直接跳到容器中的任何一个元素**，这叫做**随机访问**，需要随机访问迭代器。**随机访问迭代器具有双向迭代器的所有特性**，同时添加了支持随机访问的操作（如指针增加运算）和用于对元素进行排序的关系运算符。下表列出了除双向迭代器的操作外，随机访问迭代器还支持的操作。其中，X表示随机迭代器类型，T表示被指向的类型，a 和 b 都是迭代器值，n 为整数，r 为随机迭代器变量或引用。
  | 表达式 | 描述                               |
  | ------ | ---------------------------------- |
  | a + n  | 指向a所指向的元素后的第n个元素     |
  | n + a  | 与 a + n 相同                      |
  | a - n  | 指向 a 所指向的元素前的第 n 个元素 |
  | r += n | 等价于 r = r + n                   |
  | r -= n | 等价于 r = r - n                   |
  | a[n]   | 等价于 *(a+n)                      |
  | b - a  | 结果为这样的 n 值，即 b = a + n    |
  | a < b  | 如果 b - a > 0，则为真             |
  | a > b  | 如果 b < a ，则为真                |
  | a >= b | 如果 !(a<b)，则为真                |
  | a <= b | 如果 !(a>b)，则为真                |

  注意，像a+n这种表达式仅当a和a+n都在容器区间内时才合法


#### 迭代器的层次结构

正向迭代器具有输入、输出迭代器的全部功能。

双向迭代器具有正向迭代器的全部功能。

随机访问迭代器具有双向迭代器的全部功能。



#### 概念、改进和模型

概念可以具有类似继承的关系。例如，双向迭代器继承了正向迭代器的功能。然而，不能将 C++ 继承机制用于迭代器。例如，可以将正向迭代器实现为一个类，而将双向迭代器实现为一个常规指针。因此，对 C++ 而言，这种双向迭代器是一种内置类型，不能从类派生而来。然而，从概念上看，它确实能够继承。有些 STL 文献使用术语改进（refinement）来表示这种概念上的继承，因此，双向迭代器是对正向迭代器概念的一种改进。

概念的具体**实现**被称为**模型**（model）。因此，指向 int 的常规指针是一个随机访问迭代器模型，也是一个正向迭代器模型，因为它满足该概念的所有要求。



现在，假设要将信息复制到显示器上。如果有一个表示输出流的迭代器，则可以使用 copy()。STL 为这种迭代器提供了 **ostream_iterator** 模板。用 STL 的话说，该模板是**输出迭代器概念**的一个模型，它也是一个适配器（adapter）—— 一个类或函数，可以将一些其他接口转换为 STL 使用的接口。可以通过包含头文件 iterator（以前为 iterator.h）并作下面的声明来**创建这种迭代器**：

```c++
#include<iterator>
...
ostream_iterator<int, char> out_iter(cout, " ");
```

out_iter 迭代器现在是一个接口，第一个模板参数（这里为int）指出了被发送给输出流的数据类型，第二个模板参数（这里为char）指出了输出流使用的字符类型。构造函数的第一个参数（这里为 cout ）指出了要使用的输出流，它也可以是用于文件输出的流（参见第17章）；最后一个字符串参数是在发送给输出流的每个数据后显式的分隔符。

这样，我们就可以用这个迭代器：

```c++
*out_iter++ = 15;
```

即把15和上面的分隔符“ ”一同赋给out_iter之后指针+1，其中这组成的字符串会被发送到cout管理的输出流中，并为下一个输出操作做好了准备。可以将copy用于迭代器：

```c++
copy(dice.begin(), dice.end(), out_iter);		// copy vector to output stream
```

也可以不创建命名的迭代器，而直接构建一个匿名迭代器。即可以这样使用适配器：

```c++
copy(dice.begin(),dice.end(),ostream_iterator<int,char> out_iter(cout," "));
```

iterator 头文件还定义了一个 istream_iterator 模板，是 istream 输入可用作迭代器接口。它是一个**输入迭代器概念**的模型，可以使用两个 istream_iterator 对象来定义 copy() 的输入范围：

```c++
copy(istream_iterator<int,char>(cin),istream_iterator<int, char>(), dice.begin() );
```

与 ostream_iterator 相似，istream_iterator 也使用两个模板参数。第一个参数指出要读取的数据类型(int)，第二个参数指出输入流使用的字符类型(char)。使用构造函数参数 cin 意味着使用由 cin 管理的输入流，**省略**构造函数参数表示输入失败，**因此上述代码从输入流中读取，直到文件结尾、类型不匹配或出现其他输入故障为止**。


#### 其他有用的迭代器

reverse_iterator、back_insert_iterator、front_insert_iterator 和 insert_iterator。



##### reverse_iterator

对 reverse_iterator 执行递增操作将导致它被递减。为什么不直接对常规迭代器进行递减呢？主要原因是为了简化对已有的函数的使用。

```c++
ostream_iterator<int, char> out_iter(cout, " ");
copy(dice.begin(), dice.end(), out_iter);		// 正向
```

vector 类有一个名为 rbegin() 的成员函数和一个名为 rend() 的成员函数，前者返回一个指向超尾的反向迭代器，后者返回一个指向第一个元素的反向迭代器。

知道这些，我们可以直接

```c++
copy(dice.rbegin(), dice.rend(), out_iter);		// 反向打印
```

甚至不必声明反向迭代器。

注意：rbegin() 和 end() 返回相同的值（超尾），但类别不同（reverse_iterator 和 iterator ）。同样，rend() 和 begin() 也返回相同的值（指向第一个元素的迭代器），但类型不同。

假设 rp 是一个被初始化的 dice.rbegin() 的反转指针。那么 *rp 是什么呢？因为 rbegin() 返回超尾，因此不能对该地址进行解除引用。同样，如果 rend() 是第一个元素的位置，则 copy() 必须提早一个位置停止，因为区间的结尾处不包括在区间中。

**反向指针通过先递减，再解除引用解决了这两个问题**。即 *rp 将在 *rp 的当前值之前对迭代器执行解除引用。也就是说，如果 rp 指向位置 6，则 *rp 将是位置 5 的值，依次类推。

```c++
vector<int>::reverse_iterator ri;
    for(ri = dice.rbegin(); ri!=dice.rend(); ++ri)
    {        cout << *ri << ' ';}
```



##### back_insert_iterator、front_insert_iterator 和 insert_iterator

三种插入迭代器，也将提高 STL 算法的通用性。前面说过，下面的语句将值复制到从 dice.begin() 开始的位置：

```c++
copy(casts, casts+10, dice.begin());
```

这些值将覆盖 dice 中以前的内容，且该函数假设 dice 有足够的空间，能够容纳这些值，即 copy() **不能自动**根据发送值调整目标容器的长度。

然而，如果预先并不知道 dice 的长度，该如何办呢？或者要将元素添加到 dice 中，而不是覆盖已有的内容，又该如何办呢？

三种插入迭代器通过将复制转换为插入解决了这些问题。插入将添加新的元素，而不会覆盖已有的数据，并使用**自动内存分配**来确保能够容纳新的信息。

back_insert_iterator 将元素插入到容器尾部，而 front_insert_iterator 将元素插入到容器的前端。最后，insert_iterator 将元素插入到 insert_iterator 构造函数的参数指定的位置前面。这三个插入迭代器都是**输出容器概念**的模型。

三种插入迭代器存在一定限制：

- back_insert_iterator 只能用于允许在尾部快速插入的容器（vector满足）（后面会提这个概念）
- front_insert_iterator 只能用于允许在起始位置做时间固定插入的容器类型（vector不满足）
- insert_iterator 没有这些限制，因此可以用它把信息插入到矢量的前端

可以用 insert_iterator 将复制数据的算法转换为插入数据的算法

这些迭代器将容器类型作为模板参数，将**实际的容器标识符作为构造函数参数**。也就是说，要为名为 dice 的 vector<int> 容器创建一个 back_insert_iterator， 可以这样做：

```c++
back_insert_iterator< vector<int> > back_iter(dice);
```

必须声明容器类型的原因是，迭代器必须使用合适的容器方法。

声明 front_insert_iterator 的方式与back的相同。对于 insert_iterator 声明，还需一个指示插入位置的构造函数参数：

```c++
insert_iterator<vector<int> > insert_iter(dice, dice.begin() );
```

这个部分好好看看代码16.11，需要注意的是，因为front_insert不支持vector类容器，因此代码没有演示。



#### 容器种类

STL 具有容器概念和容器类型。概念是具有名称（如容器、序列容器、关联容器等）的通用类别；容器类型是可用于创建具体容器对象的模板。以前的 11 个容器类型分别是 deque、list、queue、priority_queue、stack、vector、map、multimap、set、multiset 和 bitset（本章不讨论 bitset，它是在比特级处理数据的容器）；C++11 新增了 forward_list、unordered_map、unordered_multimap、unordered_set 和 unordered_multiset，且不将 bitset 视为容器，而将其视为一种独立的类别。因为概念对类型进行了分类，下面先讨论它们。

##### 容器概念

概念描述了所有容器类都通用的元素，它是一个概念化的抽象基类——说它概念化，是因为容器类并不真正使用继承机制。换句话说，容器**概念指定了所有STL容器类都必须满足的一系列要求**。

容器是存储其他对象的对象。被存储的对象必须是同一种类型的，不是任何类型的对象都可以存储在容器中，具体地说，**类型必须是可复制构造的和可赋值的**。

表16.5有一些基本的容器特征

其中，复杂度一列描述了执行操作所需的时间。这个表列出了3种可能性，从快到慢依次为：

编译时间->固定时间->线性时间

如果复杂度为**编译时间**，则操作将在编译时执行，执行时间为0。固定复杂度意味着操作发生在运行阶段，但**独立**于对象中的元素数目。**线性复杂度**意味着**时间与元素数目成正比**。即如果a和b都是容器，则 a==b 具有线性复杂度，因为 == 操作必须用于容器中的每个元素。实际上，这是最糟糕的情况。如果两个容器的长度不同，则不需要作任何的单独比较，因为肯定不等。

复杂度要求是 STL 特征，虽然实现细节可以隐藏，但性能规格应公开，以便程序员能够知道完成特定操作的计算成本。



##### 序列容器

两个表P697

介绍7种序列容器类型：

##### vector

数组的一种类表示，它提供了自动管理内存的功能，可以动态地改变vector对象的长度。提供了元素的随机访问，在尾部添加和删除元素的时间是固定的，但是在头部和中间是线性的（要挪位置）

vector还是反转容器，这增加了两个类方法：rbegin()和rend()。就是前面讲的反向显示用到的。

vector是最简单的序列类型，除非其他类型的特殊优点能更好地满足程序需求，否则应该默认使用这种类型。



##### deque

即双端队列（double-ended queue）实现类似vector，支持随机访问。区别在于从 deque 对象的开始位置插入和删除元素的时间是固定的，而不像 vector 中那样是线性时间的。当然，和vector一样，在中间部分也是线性的。



##### list

表示双向链表，除了第一个和最后一个元素外，每个元素都与前后的元素相链接，这意味着可以双向遍历链表。

list 和 vector 之间关键的区别在于，list 在链表中**任一位置**进行插入和删除的时间都是固定的

因此，vector 强调的是通过随机访问进行快速访问，而 list 强调的是元素的快速插入和删除。

与 vector 相似，list 也是可反转容器。与 vector 不同的是，list **不支持**数组表示法和随机访问。



##### forward_list

单链表，每个节点只链接到下一个节点，因此forward_list只需要正向迭代器而不需要双向迭代器。因此，不同于 vector 和 list，forward_list 是**不可反转的容器**。相比于 list，forward_list 更简单、更紧凑，但功能也更少。



##### queue

queue 模板类（在头文件 queue（以前为 queue.h）中声明）是一个适配器类。由前所述，ostream_iterator 模板就是一个适配器，让输出流能够使用迭代器接口。同样，queue 模板让底层类（默认为 deque）展示典型的队列接口。

queue 模板的限制比 deque 更多。它不仅**不允许随机访问**队列元素，甚至**不允许遍历队列**。它把使用限制在定义队列的基本操作上，可以将元素添加到队尾、从队首删除元素、查看队首和队尾的值、检查元素数目和测试队列是否为空。



##### priority_queue

priority_queue 模板类（在 queue 头文件中声明）是另一个适配器类，它支持的操作与 queue 相同。两者之间的主要区别在于，在priority_queue 中，**最大的元素被移到队首**

内部区别在于，默认的底层类是 vector。可以修改用于确定哪个元素放到队首的比较方式，方法是提供一个可选的构造函数参数：

```c++
priority_queue<int>  pq1;		// 默认版本
priority_queue<int>  pq2(greater<int>);		// 用 greater<int>来排序决定谁在队首
```



##### stack

与 queue 相似，stack（在头文件 stack—— 以前为 stack.h——中声明）也是一个适配器类，它给底层类（默认情况下为 vector）提供了典型的**栈接口**。

stack模板的限制比vector更多。它不仅不允许随机访问栈元素，甚至不允许遍历栈。它把使用**限制在定义栈的基本操作**上，即可以将**压入**推到栈顶、从栈顶**弹出**元素、查看**栈顶的值**、检查**元素数目**和测试栈是否为空。



##### array

第 4 章介绍过，模板类 array 是否头文件 array 中定义的，**它并非 STL 容器**，因为其**长度是固定的**。因此，array 没有定义调整容器大小的操作，如 push_back() 和 insert()，但定义了对它来说有意义的成员函数如 operator[]() 和 at()。可将很多标准 STL 算法用于 array 对象，如 copy() 和 for_each()。



#### 关联容器

关联容器（associative container）是对容器概念的另一个改进。关联容器将值与键关联在一起，并使用键来查找值。前面说过，对于容器 X，表达式 X::value_type 通常指出了存储在容器中的值类型。对于关联容器来说，表达式 X::key_type 指出了键的类型。

关联容器的优点在于，它提供了对元素的快速访问。也允许插入新元素，但**不能指定**元素的**插入位置**。原因是关联容器通常有用于确定数据放置位置的算法，以便能够快速检索信息。

关联容器通常是使用某种树实现的。

STL提供了4种关联容器：set、multiset、map 和 multimap。前两种是在头文件 set（以前分别为 set.h 和 multiset.h）种定义的，而后两种是在头文件 map（以前分别为 map.h 和 multimap.h）中定义的。



##### set_union

并集

```c++
set_uniong(A.begin(), A.end(), B.begin(), B.end(), insert_iterator<set<string> >(C. C.begin() );
//或者输出到屏幕
set_uniong(A.begin(), A.end(), B.begin(), B.end(), ostream_iterator<string,char >out(cout," " );
```

set_intersection() 和 set_difference() 分别查找交集和获得两个集合的差，它们的接口与 set_union() 相同。



#### 预定义的函数符

如transform()，有两个版本。第一个版本接收四个参数，前两个是区间迭代器，第三个是结果输出位置的迭代器，最后一个参数是函数符，它被应用于区间中的每一个元素，生成结果中的新元素。

```c++
transform(gr8.begin(),gr8.end(),out,sqrt);
```

这里out是ostream的cout迭代器接口。

也可以这样

```c++
transform(gr8.begin(),gr8.end(),gr8.begin(),sqrt);
```

把out换成begin，那么计算后的数据会覆盖原本的值。

另一个版本，需要用到两个参数的函数，如mean(a,b)求平均值。

此时transform的第三个参数被认为是函数的第二个形参的起点，例如：

```c++
transform(gr8.begin(), gr8.end(), m8.begin(), out, mean);
```

这里总共用到五个参数，从gr8和m8的begin开始相加，每个元素都相加，然后输出到out。


给每一个函数都定义太麻烦，因为不同的数据类型可能需要重新定义，如int和double，更好的方法是定义一个模板。头文件中functional定义了多个模板类函数对象，其中包括plus<>()

见P711的表，基本运算符都涵盖了。



##### binder1st, binder2nd

例如，要讲二元函数multipies(a,b)转换为讲参数乘以2.5的一元函数，可以这样：

```c++
bind1st(multiplies<double>(), 2.5)
```

binder2nd和其类似，只是将常数赋给第二个参数

因此，将 gr8 中的每个元素与 2.5 相乘，并显示结果的代码如下：

```c++
transform(gr8.begin(), gr8.end(), out, bind1st(multiplies<double(), 2.5));
```



#### 算法

##### 算法组

STL将算法库分成四组：

- 非修改式序列操作

  非修改式序列操作对区间中的每个元素进行操作。这些操作**不修改容器的内容**。例如，find() 和 for_each() 就属于这一类。

- 修改式序列操作

  修改式序列操作也对区间中的每个元素进行操作。然而，顾名思义，它们可以**修改容器的内容**。可以修改值，也可以修改值的排列顺序。transform()、random_shuffle() 和 copy() 属于这一类。

- 排序和相关操作

  排序和相关操作包括多个排序函数（包括sort() 和其他各种函数，包括集合操作）。

- 通用数字运算

  数字操作包括将区间的内容累积、计算两个容器的内部乘积、计算小计、计算相邻对象差的函数。通常，这些都是数组的操作特性，因此 vector 是最有可能使用这些操作的容器。



前三组在头文件algorithm中描述，第四组是专门用于数值数据的，有自己的头文件numeric



有些算法有两个版本：**就地版本**和**复制版本**。STL 的约定是，复制版本的名称将以 _copy 结尾。复制版本将接受一个额外的输出迭代器参数，该参数指定结果的放置位置。例如，函数 replace() 的原型如下：

```c++
template<class ForwardIterator, class T>
void replace(ForwardIterator first, ForwardIterator last, const T& old_value, const T& new_value);
//复制版本
template<class InputIterator, class OutputIterator, class T>
OutputIterator replace_copy(InputIterator first, InputIterator last,
					OutputIterator result, const T& old_value, const T& new_value);
```

注意，replace_copy() 的返回类型为 OutputIterator。对于复制算法，统一的约定是：返回一个迭代器，该迭代器指向复制的最后一个值后面的一个位置（超尾）。



#### STL和String类

string 类虽然不是 STL 的组成部分，但设计它时考虑到了 STL。例如，它包含 begin()、end() 、rbegin() 和 rend() 等成员，因此可以使用 STL 接口。

有时可以选择使用 STL 方法或 STL 函数（methods or function)，通常方法是更好的选择。

见例子

尽管方法通常更适合，但非方法**函数更通用**。正如您看到的，可以将它们用于数组、string对象、STL 容器，还可以用它们来处理混合的容器类型，例如，将矢量容器中的数据存储到链表或集合中。
