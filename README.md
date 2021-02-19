# MoreEffectiveCPlusPlus
Note of MoreEffectiveCPlusPlus
### Item M1：指针与引用的区别

**指针与引用看上去完全不同（指针用操作符“*”和“->”，引用使用操作符“. ”），但是它们似乎有相同的功能。指针与引用都是让你间接引用其他对象。**

在任何情况下都不能使用指向空值的引用。一个引用必须总是指向某些对象。因此如果你使用一个变量并让它指向一个对象，但是该变量在某些时候也可能不指向
任何对象，这时你应该把变量声明为指针，因为这样你可以赋空值给该变量。相反，如果变量肯定指向一个对象，例如你的设计不允许变量为空，这时你就可以把变量声明为引用。

```C++
char *pc = 0; // 设置指针为空值
char& rc = *pc; // 让引用指向空值,结果将是不确定的
```

**不存在指向空值的引用这个事实意味着使用引用的代码效率比使用指针的要高。因为在
使用引用之前不需要测试它的合法性。**
```C++
void printDouble(const double& rd)
{
 cout << rd; // 不需要测试 rd,它肯定指向一个 double 值
} 

相反，指针则应该总是被测试，防止其为空：
void printDouble(const double *pd)
{
 if (pd) { // 检查是否为 NULL
 cout << *pd;
 }
}
```

**指针与引用的另一个重要的不同是指针可以被重新赋值以指向另一个不同的对象。但是
引用则总是指向在初始化时被指定的对象，以后不能改变。**
```C++
string s1("Nancy");
string s2("Clancy");
string& rs = s1; // rs 引用 s1
string *ps = &s1; // ps 指向 s1
rs = s2; // rs 仍旧引用 s1,但是 s1 的值现在是"Clancy"
ps = &s2; // ps 现在指向 s2,s1 没有改变 
```

**在以下情况下你应该使用指针，一是你考虑到存在不指向任何对象的可能（在这种情况下，你能够设置指针为空），二是你需要能够在不同的时刻指向不同的对象（在这
种情况下，你能改变指针的指向）。如果总是指向一个对象并且一旦指向一个对象后就不会改变指向，那么你应该使用引用。**
```C++
vector<int> v(10); // 建立整形向量（vector），大小为 10;
v[5] = 10; // 这个被赋值的目标对象就是操作符[]返回的值
 
 //如果操作符[]返回一个指针，那么后一个语句就得这样写：
*v[5] = 10;
但是这样会使得 v 看上去象是一个向量指针。因此你会选择让操作符返回一个引用。
```

**当你知道你必须指向一个对象并且不想改变其指向时，或者在重载操作符并为防止不必要的语义误解时，你不应该使用指针。而在除此之外的其他情况下，则应使用指针。**

### Item M2：尽量使用C++风格的类型转换

C++通过引进四个新的类型转换操作符克服了 C 风格类型转换的缺点，这四个操作符是static_cast, const_cast, dynamic_cast, 和 reinterpret_cast。

```C++
//语法：static_cast<type>(expression)
//把一个 int 转换成 double，以便让包含 int 类型变量的表达式产生出浮点数值的结果。
int firstNumber, secondNumber;
double result = static_cast<double>(firstNumber)/secondNumber; 
```
  
**有功能上限制。例如，你不能用 static_cast 象用 C 风格的类型转换一样把 struct 转换成 int 类型或者把 double 类型转换成指针类型，另外，static_cast 不能从表达式中去除 const 属性**

const_cast 用于类型转换掉表达式的 const 或 volatileness 属性。

```C++
class Widget { ... };
class SpecialWidget: public Widget { ... };
void update(SpecialWidget *psw);
SpecialWidget sw; // sw 是一个非 const 对象。
const SpecialWidget& csw = sw; // csw 是 sw 的一个引用,它是一个 const 对象
update(&csw); // 错误!不能传递一个 const SpecialWidget* 变量给一个处理 SpecialWidget*类型变量的函数
update(const_cast<SpecialWidget*>(&csw)); // 正确，csw 的 const 被显示地转换掉（csw 和 sw 两个变量值在 update函数中能被更新）

Widget *pw = new SpecialWidget;
update(pw); // 错误！pw 的类型是 Widget*，但是update 函数处理的是 SpecialWidget*类型
update(const_cast<SpecialWidget*>(pw));// 错误！const_cast 仅能被用在影响constness or volatileness 的地方上,不能用在向继承子类进行类型转换。 
```
**到目前为止，const_cast 最普通的用途就是转换掉对象的 const 属性。**

第二种特殊的类型转换符是 dynamic_cast，它被用于安全地沿着类的继承关系向下进行类型转换。这就是说，你能用 dynamic_cast 把指向基类的指针或引用转换成指向其派生
类或其兄弟类的指针或引用，而且你能知道转换是否成功。失败的转换将返回空指针（当对指针进行类型转换时）或者抛出异常（当对引用进行类型转换时）：

```C++
Widget *pw;
...
update(dynamic_cast<SpecialWidget*>(pw)); // 正确，传递给 update 函数一个指针是指向变量类型为 SpecialWidget的pw的指针,如果pw确实指向一个对象,否则传递过去的将使空指针。
void updateViaRef(SpecialWidget& rsw);
updateViaRef(dynamic_cast<SpecialWidget&>(*pw));//正确。 传递给 updateViaRef 函数SpecialWidget pw 指针，如果 pw确实指向了某个对象,否则将抛出异常
```

dynamic_casts 在帮助你浏览继承层次上是有限制的。它不能被用于缺乏虚函数的类型上，也不能用它来转换掉 constness：
int firstNumber, secondNumber;
...
double result = dynamic_cast<double>(firstNumber)/secondNumber;// 错误！没有继承关系
const SpecialWidget sw;
...
update(dynamic_cast<SpecialWidget*>(&sw));// 错误! dynamic_cast 不能转换掉const。
  
reinterpret_cast。使用这个操作符的类型转换，其 的 转 换 结 果 几 乎 都 是 执 行 期 定 义 （ implementation-defined ）。 因 此,使 用reinterpret_casts 的代码很难移植。 
reinterpret_casts 的最普通的用途就是在函数指针类型之间进行转换。
