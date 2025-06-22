### 语法
![[lambda语法.png]]
1. **捕获列表**。在C ++规范中也称为Lambda导入器， 捕获列表总是出现在Lambda函数的开始处。实际上，[]是Lambda引出符。编译器根据该引出符判断接下来的代码是否是Lambda函数，捕获列表能够捕捉上下文中的变量以供Lambda函数使用。
2. **参数列表**。与普通函数的参数列表一致。如果不需要参数传递，则可以连同括号“()”一起省略。
3. 可变规格*。mutable修饰符， 默认情况下Lambda函数总是一个const函数，mutable可以取消其常量性。在使用该修饰符时，参数列表不可省略（即使参数为空）。
4. 异常说明。用于Lamdba表达式内部函数抛出异常。
5. **返回类型**。 追踪返回类型形式声明函数的返回类型。我们可以在不需要返回值的时候也可以连同符号”->”一起省略。此外，在返回类型明确的情况下，也可以省略该部分，让编译器对返回类型进行推导。  
6. **lambda函数体**。内容与普通函数一样，不过除了可以使用参数之外，还可以使用所有捕获的变量。

### 参数详解

#### Lambda捕获列表

Lambda表达式与普通函数最大的区别是，除了可以使用参数以外，Lambda函数还可以通过捕获列表访问一些上下文中的数据。具体地，捕捉列表描述了上下文中哪些数据可以被Lambda使用，以及使用方式（以值传递的方式或引用传递的方式）。语法上，在“[]”包括起来的是捕获列表，捕获列表由多个捕获项组成，并以逗号分隔。

- []表示不捕获任何变量
```
auto function = ([]{
        std::cout << "Hello World!" << std::endl;
    }
);
 
function();
```
- [var]表示值传递方式捕获变量var
```
int num = 100;
auto function = ([num]{
        std::cout << num << std::endl;
    }
);
 
function();
```
- [=]表示值传递方式捕获所有父作用域的变量（包括this）
```
int index = 1;
int num = 100;
auto function = ([=]{
            std::cout << "index: "<< index << ", "
                << "num: "<< num << std::endl;
    }
);
 
function();
```
- [&var]表示引用传递捕捉变量var
```
int num = 100;
auto function = ([&num]{
        num = 1000;
        std::cout << "num: " << num << std::endl;
    }
);
```
- [&]表示引用传递方式捕捉所有父作用域的变量（包括this）
```
int index = 1;
int num = 100;
auto function = ([&]{
        num = 1000;
        index = 2;
        std::cout << "index: "<< index << ", "
            << "num: "<< num << std::endl;
    }
);
 
function();
```
- [this]表示值传递方式捕捉当前的this指针
```
#include <iostream>
using namespace std;
  
class Lambda
{
public:
    void sayHello() {
        std::cout << "Hello" << std::endl;
    };
 
    void lambda() {
        auto function = [this]{
            this->sayHello();
        };
 
        function();
    }
};
  
int main()
{
    Lambda demo;
    demo.lambda();
}
```
- [=, &] 拷贝与引用混合
	- [=,&a,&b]表示以引用传递的方式捕捉变量a和b，以值传递方式捕捉其它所有变量。
```
	int index = 1;
int num = 100;
auto function = ([=, &index, &num]{
        num = 1000;
        index = 2;
        std::cout << "index: "<< index << ", "
            << "num: "<< num << std::endl;
    }
);
 
function();
```
- [&,a,this]表示以值传递的方式捕捉变量a和this，引用传递方式捕捉其它所有变量。
不过值得注意的是，捕捉列表不允许变量重复传递。下面一些例子就是典型的重复，会导致编译时期的错误。例如：
- [=,a]这里已经以值传递方式捕捉了所有变量，但是重复捕捉a了，会报错的;
- [&,&this]这里&已经以引用传递方式捕捉了所有变量，再捕捉this也是一种重复。

如果lambda主体`total`通过引用访问外部变量，并`factor`通过值访问外部变量，则以下捕获子句是等效的：
```
[&total, factor]
[factor, &total]
[&, factor]
[factor, &]
[=, &total]
[&total, =]
```
