
## C++基础

### 指针和引⽤的区别
#### 对比

| 特性     | 指针 (Pointer)             | 引用 (Reference)                      |
|--------|--------------------------|-------------------------------------|
| 本质     | 存储内存地址的变量                | 已存在变量的别名                            |
| 初始化要求  | 可不初始化（危险，可能野指针）          | 必须初始化，且绑定后不可改变                      |
| 可为空值   | 可设为 nullptr              | 不可为空，必须绑定有效对象                       |
| 访问方式   | 需显式解引用 (*ptr)            | 直接使用别名 (ref)                        |
| 内存占用   | 占用独立内存空间（通常 4/8 字节）      | 无独立内存（编译器实现透明）                      |
| 重新绑定   | 可更改指向对象 (ptr = &new_var) | 不可重新绑定（终身绑定初始对象）                    |
| 多级间接访问 | 支持多级指针 (int** pp)        | 不支持引用的引用（C++11 右值引用除外）              |
| 地址操作   | 可获取指针自身地址 (&ptr)         | 不可获取引用自身地址 (&ref 返回原变量地址)           |
| 算术运算   | 支持 (ptr++, ptr + n 等)    | 不支持                                 |
| 数组支持   | 支持数组遍历 (int* p = arr)    | 不可直接绑定数组（需用 int (&ref)[N] = arr 语法） |
| 函数参数传递 | 需显式取地址和解引用               | 语法更简洁直观                             |
| 安全性    | 可能空指针/野指针访问              | 更安全（始终关联有效对象）                       |
#### 示例
```
int main() {
    int a = 10, b = 20;
    
    // 指针特性演示
    int* ptr = nullptr;  // 允许空指针
    ptr = &a;            // 指向a
    *ptr = 15;           // 解引用修改a
    ptr = &b;            // 重新指向b（允许重新绑定）
    int** pp = &ptr;     // 支持多级指针

    // 引用特性演示
    int& ref = a;        // 必须初始化（绑定a）
    ref = 25;            // 直接修改a（无需解引用）
    // int& ref2;         // 错误：未初始化
    // ref = b;           // 实际是 a = b（不是重新绑定）
    // int&& rref = ref;  // 错误：不能创建引用的引用
}
```
#### 理解：
> **指针**：指针就像一把钥匙，它能够打开存储数据的房间（内存地址）。这把钥匙（指针）可以复制给其他人（复制指针） ，也可以改变打开的房间（重新赋值指针） 。但是，如果不小心弄丢了这把钥匙，或者用它打开了错误的房间，就可能遇到麻烦（野指针或内存泄漏）。
> **引用**： 引用则像是一条秘密通道，一旦建立，就永远指向一个特定的房间（变量） ，并且不能改变方向。这个通道非常安全，因为它确保了你总是访问正确的房间。但是，它必须在建造时就知道要通往哪里，且一旦建成，就不能改变路径（在定义时必须初始化且不能重新赋值）



### 值传递、引用传递、指针传递（函数参数）区别

| 特性       | 值传递 (void func(T x)) | 指针传递 (void func(T* ptr)) | 引用传递 (void func(T& ref)) |
|----------|----------------------|--------------------------|--------------------------|
| 拷贝开销     | 产生完整对象拷贝（昂贵）         | 仅拷贝地址（高效）                | 仅传递别名（高效）                |
| 修改原对象能力  | ❌ 无法修改原对象            | ✅ 通过解引用修改                | ✅ 直接修改（无需解引用）            |
| 空值/野指针风险 | 无风险                  | 需检查nullptr（易出错）          | 无风险（必须绑定有效对象）            |
| 语法简洁性    | 简单但效率低               | 需&取址和*解引用（冗长）            | 直接操作（最简洁）                |
| const正确性 | 自动保护原对象              | 需显式const T*保护            | const T&明确表达只读意图         |
| 多态支持     | 切片问题（丢失派生类信息）        | 支持动态多态                   | 支持动态多态                   |
| 临时对象支持   | ✅ 可接受临时对象            | ❌ 不能绑定临时对象               | ✅ const T&可绑定临时对象        |
### 引用作为函数参数在有什么好处
#### 1、避免拷贝开销（高性能）
```
struct HeavyData { /* 包含大量成员 */ };

// 值传递：产生完整拷贝（性能灾难）
void processByValue(HeavyData data) { /*...*/ }

// 引用传递：零拷贝（仅传递别名）
void processByRef(const HeavyData& data) { /*...*/ } // const引用确保只读

void demo() {
    HeavyData hd;
    processByValue(hd);  // 触发完整拷贝
    processByRef(hd);    // 无拷贝，直接操作原数据
}
```
#### 2、直接修改原对象（安全直观）
```
// 指针版本：需要繁琐的解引用和空指针检查
void scalePtr(int* ptr, float factor) {
    if (ptr) *ptr *= factor;  // 必须检查空指针
}

// 引用版本：直接操作且无空值风险
void scaleRef(int& ref, float factor) {
    ref *= factor;  // 直接修改原变量
}

void demo() {
    int val = 10;
    scalePtr(&val, 1.5);  // 需手动取地址（&）
    scaleRef(val, 1.5);   // 直接传递变量
}
```
#### 3、支持`const`正确性（接口清晰）
```
// const引用明确表达"只读"意图
void printData(const std::string& data) {
    std::cout << data;
    // data[0] = 'A';  // 错误！编译器阻止修改
}

// 对比非const引用（可能意外修改）
void dangerousModify(std::string& data) {
    data[0] = 'X';  // 调用者数据被意外修改！
}
```

#### 4、支持多态（面向对象编程）
```
class Animal {
public:
    virtual void speak() const { std::cout << "???"; }
};

class Dog : public Animal {
public:
    void speak() const override { std::cout << "Woof!"; }
};

// 引用传递支持运行时多态
void animalSound(const Animal& animal) {
    animal.speak();  // 动态绑定到实际类型
}

void demo() {
    Dog d;
    animalSound(d);  // 输出 "Woof!"（正确调用派生类方法）
}
```
#### **5、绑定临时对象（资源优化）**
```
// const引用可延长临时对象生命周期
void processTemp(const std::string& str) {
    std::cout << str;
}

void demo() {
    // 避免创建临时string变量
    processTemp("Hello " + std::to_string(42));  // 直接绑定临时对象
}
```
#### 最佳实践指南
1. **默认选择`const T&`** 对于只读参数（>4字节类型），优先使用`const T&`
2. **输出参数用非const引用** 替代输出型指针：`void getResults(T& out)`比`void getResults(T* out)`安全
3. **小型内置类型传值** 对`int`/`float`等基本类型（≤寄存器大小），值传递可能更优
4. **需要重绑定时用指针** 当函数内部需要改变指向时（如链表操作），仍需使用指针


### 关键字

#### const

#### static

#### define

#### typedef

#### inline
##### 如何正确使用内联函数

#### new

#### malloc

#### constexpr

#### volatile

#### extern

#### std::atomic

#### 前置++和后置++


### 常量指针和指针常量
| 特性      | 常量指针 (const T*)             | 指针常量 (T* const)        | 双重常量 (const T* const)      |
|---------|-----------------------------|------------------------|----------------------------|
| 声明语法    | const T* ptr 或 T const* ptr | T* const ptr = &var;   | const T* const ptr = &var; |
| 指针指向的值  | ❌ 不可修改 (*ptr = ... 非法)      | ✅ 可以修改 (*ptr = ... 合法) | ❌ 不可修改 (*ptr = ... 非法)     |
| 指针自身的值  | ✅ 可修改 (ptr = ... 合法)        | ❌ 不可修改 (ptr = ... 非法)  | ❌ 不可修改 (ptr = ... 非法)      |
| 是否必须初始化 | 可选                          | ✔️ 必须初始化               | ✔️ 必须初始化                   |
| 别称      | Pointer to const            | Const pointer          | Const pointer to const     |
| 内存图解    | ptr → [const data]          | const ptr → [data]     | const ptr → [const data]   |
| 常见用例    | 函数参数中保护数据不被修改               | 固定地址的硬件寄存器访问           | ROM数据访问                    |
| 类比      | 只读望远镜（可换观测对象，但不能修改对象）       | 固定位置的望远镜（不能换位置，但可修改对象） | 固定位置的只读望远镜                 |
#### 代码示例详解

##### 1. 常量指针 (`const T*`)

```
int a = 10, b = 20;
const int* ptr = &a;  // 指向const int的指针

// 允许的操作：
ptr = &b;      // 改变指针指向的对象 ✅
int c = *ptr;  // 读取指向的值 ✅

// 禁止的操作：
// *ptr = 30;   // 错误：不能修改指向的值 ❌

// 特殊特性：
a = 40;        // 允许：原始变量仍可通过其他方式修改
std::cout << *ptr;  // 输出40（值已改变）
```
##### 2. 指针常量 (`T* const`)
```
int x = 10, y = 20;
int* const ptr = &x;  // 指向int的常量指针（必须初始化）

// 允许的操作：
*ptr = 30;      // 修改指向的值 ✅
int z = *ptr;   // 读取指向的值 ✅

// 禁止的操作：
// ptr = &y;    // 错误：不能改变指针指向 ❌

// 特殊特性：
const int* const_ptr = ptr;  // 可转为常量指针
```
##### 3. 双重常量 (`const T* const`)
```
const int value = 42;
const int* const ptr = &value;  // 指向const int的常量指针

// 禁止所有修改操作：
// *ptr = 50;   // 错误 ❌
// ptr = ...;   // 错误 ❌

// 唯一允许的操作：
int read = *ptr;  // 只读访问 ✅
```

### 函数指针和指针函数的区别
| 特性     | 函数指针 (Function Pointer)    | 指针函数 (Pointer Function)        |
|--------|----------------------------|--------------------------------|
| 本质     | 指针变量 - 存储函数地址              | 函数的本质 - 返回指针类型                 |
| 声明语法   | 返回类型 (*指针名)(参数列表)          | 返回类型* 函数名(参数列表)                |
| 核心用途   | 1. 回调函数机制2. 动态函数调用3. 函数表实现 | 1. 返回动态分配内存2. 返回数组首地址3. 返回对象指针 |
| 内存占用   | 占用指针空间（4/8字节）              | 不额外占用内存（代码段存储）                 |
| 赋值操作   | 可指向不同函数（灵活）                | 函数名不可赋值（固定实现）                  |
| 调用方式   | 指针名(参数) 或 (*指针名)(参数)       | 直接调用：函数名(参数)                   |
| 类型安全   | 依赖函数签名匹配                   | 依赖返回类型检查                       |
| 典型应用场景 | 事件处理、策略模式、插件系统             | 工厂模式、资源创建、数据容器                 |
##### 1. 函数指针（指向函数的指针）
**定义：** 存储函数内存地址的指针变量
```
#include <iostream>

// 函数原型
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }

int main() {
    // 声明函数指针（匹配参数和返回类型）
    int (*funcPtr)(int, int);
    
    // 指向add函数
    funcPtr = &add;  // 等价于 funcPtr = add;
    std::cout << "Add: " << funcPtr(3, 2) << std::endl;  // 输出: 5
    
    // 指向sub函数
    funcPtr = &sub;
    std::cout << "Sub: " << funcPtr(3, 2) << std::endl;  // 输出: 1
    
    // 通过typedef简化
    using MathFunc = int(*)(int, int);
    MathFunc fn = add;
    std::cout << "Typedef: " << fn(5, 3) << std::endl;  // 输出: 8
}
```
##### 2. 指针函数（返回指针的函数）
**定义：** 返回指针类型（地址）的函数
```
#include <iostream>
#include <cstring>

// 返回动态字符串的指针函数
char* createGreeting(const char* name) {
    char* greeting = new char[100];  // 动态分配内存
    strcpy(greeting, "Hello, ");
    strcat(greeting, name);
    strcat(greeting, "!");
    return greeting;  // 返回指针
}

// 返回数组元素的指针函数
int* findValue(int* arr, int size, int target) {
    for (int i = 0; i < size; ++i) {
        if (arr[i] == target) 
            return &arr[i];  // 返回地址
    }
    return nullptr;
}

int main() {
    // 调用指针函数获取动态内存
    char* msg = createGreeting("Alice");
    std::cout << msg << std::endl;  // 输出: Hello, Alice!
    delete[] msg;  // 必须手动释放内存
    
    // 返回数组元素的指针函数
    int data[] = {10, 20, 30, 40};
    int* ptr = findValue(data, 4, 30);
    if (ptr) {
        std::cout << "Found: " << *ptr << std::endl;  // 输出: 30
        *ptr = 35;  // 通过指针修改原数组
        std::cout << "Modified: " << data[2] << std::endl;  // 输出: 35
    }
}
```


### 静态局部变量\全局变量\局部变量的区别和使⽤场景
| 特性    | 局部变量           | 静态局部变量               | 全局变量              |
|-------|----------------|----------------------|-------------------|
| 声明位置  | 函数/代码块内部       | 函数/代码块内部（带static关键字） | 所有函数外部            |
| 生命周期  | 函数/代码块执行期间     | 程序整个运行期              | 程序整个运行期           |
| 作用域   | 声明所在的代码块内      | 声明所在的代码块内            | 整个程序（跨文件需extern）  |
| 存储位置  | 栈内存            | 静态存储区                | 静态存储区             |
| 初始化   | 默认随机值（必须显式初始化） | 默认初始化为0（仅初始化一次）      | 默认初始化为0（main前初始化） |
| 重复调用  | 每次调用重新创建       | 保持上次调用的值             | 始终可访问             |
| 多线程安全 | 线程安全（栈隔离）      | 需手动同步                | 需手动同步             |
| 内存分配  | 自动分配/释放        | 程序启动分配，结束时释放         | 程序启动分配，结束时释放      |
| 使用场景  | 函数内部临时数据       | 函数调用间保持状态            | 程序全局共享数据          |
##### 1. 局部变量 (Local Variable)
```
void processOrder() {
    int itemCount = 0;  // 局部变量
    itemCount++;
    std::cout << "Items in this order: " << itemCount << "\n";
}

int main() {
    processOrder();  // 输出: Items in this order: 1
    processOrder();  // 输出: Items in this order: 1 (每次调用重新创建)
}
```

**特点**：

- 生命周期：函数执行期间存在
- 作用域：仅限于声明块内
- 存储位置：栈内存
- 使用场景：临时计算结果、循环计数器、函数参数

##### 2. 静态局部变量 (Static Local Variable)
```
void processOrder() {
    static int totalItems = 0;  // 静态局部变量
    totalItems++;
    std::cout << "Total items processed: " << totalItems << "\n";
}

int main() {
    processOrder();  // 输出: Total items processed: 1
    processOrder();  // 输出: Total items processed: 2 (保持状态)
    processOrder();  // 输出: Total items processed: 3
}
```
**特点**：
- 🚀 **首次初始化后保持值**
- ⏳ 生命周期持续到程序结束
- 🔒 作用域仍限制在函数内
- 💾 存储在静态存储区
- 使用场景：
    - 函数调用计数器
    - 缓存机制（首次加载资源）
    - 单例模式实现

##### 3. 全局变量 (Global Variable)
```
#include <iostream>

int globalCounter = 0;  // 全局变量

void increment() {
    globalCounter++;
}

void decrement() {
    globalCounter--;
}

int main() {
    increment();
    increment();
    decrement();
    std::cout << "Global counter: " << globalCounter;  // 输出: 1
}
```
**特点**：
- 🌍 **整个程序可访问**（跨文件需`extern`）
- ⏳ 生命周期持续到程序结束
- 📂 存储在静态存储区
- 🔓 默认外部链接（可用`static`限制为文件内）
- 使用场景：
    - 程序全局配置
    - 跨模块共享数据
    - 资源管理器句柄

### 强制类型转换
| 转换类型   | 语法                     | 安全性   | 主要用途                   | 适用场景                | 风险提示        |
|--------|------------------------|-------|------------------------|---------------------|-------------|
| 静态转换   | static_cast<类型>()      | ★★★☆☆ | 基础类型转换、类层级转换           | 数值类型转换、向上转型、void*转换 | 向下转型不安全     |
| 动态转换   | dynamic_cast<类型>()     | ★★★★★ | 多态类型安全向下转型             | 基类到派生类的安全转换         | 需要RTTI支持    |
| 常量转换   | const_cast<类型>()       | ★★☆☆☆ | 添加/移除const/volatile限定符 | 修改常量对象、兼容旧API       | 可能导致未定义行为   |
| 重新解释转换 | reinterpret_cast<类型>() | ☆☆☆☆☆ | 低级别位模式重新解释             | 指针类型转换、指针/整数互转      | 高度平台依赖，极易出错 |
##### 1. `static_cast` - 静态转换

**最常用的基础类型转换**
```
// 基础类型转换
double d = 3.14;
int i = static_cast<int>(d); // 3（截断小数）

// 类层级转换（向上转型安全）
class Base {};
class Derived : public Base {};
Derived* pd = new Derived();
Base* pb = static_cast<Base*>(pd); // 安全向上转型

// void*转换
void* pv = malloc(sizeof(int));
int* pi = static_cast<int*>(pv); // void* → int*
```
**适用场景**：
- 数值类型转换（double→int）
- 类指针向上转型（派生类→基类）
- void*与其他指针互转
**风险**：
- 向下转型不安全（编译通过但运行时可能出错）
```
Base* pb = new Base(); Derived* pd = static_cast<Derived*>(pb); // 危险！可能访问无效内存
```

##### 2. `dynamic_cast` - 动态转换

**运行时类型安全的向下转型**

```
class Animal {
public:
    virtual ~Animal() {} // 必须包含虚函数
};
class Dog : public Animal {};
class Cat : public Animal {};

Animal* animal = new Dog();

// 安全向下转型
Dog* dog = dynamic_cast<Dog*>(animal);
if(dog) { // 转换成功检查
    cout << "This is a Dog" << endl;
}

// 引用转换（失败时抛bad_cast异常）
try {
    Dog& dogRef = dynamic_cast<Dog&>(*animal);
} catch(const std::bad_cast& e) {
    cerr << "Bad cast: " << e.what() << endl;
}
```
**适用场景**：
- 多态类型安全向下转型（基类→派生类）
- 交叉类型转换（多重继承）
- 运行时类型识别（RTTI）
**限制**：
- 需要虚函数表支持（至少一个虚函数）
- 性能开销较大（运行时类型检查）
- 只能用于指针/引用类型
##### 3. `const_cast` - 常量转换
**常量属性修改**
```
// 移除const限定符
const int ci = 10;
int* modifiable = const_cast<int*>(&ci);
*modifiable = 20; // 未定义行为！实际内存可能只读

// 合法使用场景：兼容旧API
void legacyAPI(char* str) {
    // 旧函数需要非常量参数
}

const char* input = "Hello";
legacyAPI(const_cast<char*>(input)); // 安全：函数不会修改字符串

// 添加const限定符（较少使用）
int value = 42;
const int* constPtr = const_cast<const int*>(&value);
```
**适用场景**：
- 兼容需要非const参数的旧API
- 修改mutable成员（在const成员函数中）
- 临时绕过const限制（需确保内存可写）
**风险**：
- 修改真正的常量对象导致未定义行为
- 破坏设计意图（应优先考虑重构代码）
##### 4. `reinterpret_cast` - 重新解释转换
**底层二进制重新解释**
```
// 指针类型转换
int value = 0x12345678;
int* pi = &value;
char* pc = reinterpret_cast<char*>(pi); 
cout << hex << (int)pc[0]; // 输出取决于字节序（小端：78）

// 指针/整数互转
uintptr_t address = reinterpret_cast<uintptr_t>(pi);
int* pi2 = reinterpret_cast<int*>(address);

// 函数指针转换
void (*func)() = reinterpret_cast<void(*)()>(0x1000); 
// func(); // 危险调用！

// 不相关类指针转换（危险！）
class A { int x; };
class B { double y; };
A a;
B* pb = reinterpret_cast<B*>(&a); // 语法允许，但访问成员危险
```
**适用场景**：
- 序列化/反序列化操作
- 底层硬件编程（寄存器访问）
- 函数指针特殊处理
- 极端优化场景
**风险**：
- 高度平台依赖（字节序、对齐要求）
- 极易引发未定义行为
- 代码可移植性差
##### 转换类型选择指南

|转换需求|推荐转换|示例|
|---|---|---|
|基础数值类型转换|static_cast|double → int|
|类指针向上转型|static_cast|Derived* → Base*|
|类指针向下转型（安全）|dynamic_cast|Base* → Derived*|
|移除const（兼容旧API）|const_cast|const char* → char*|
|函数指针类型转换|reinterpret_cast|void(_)() → int(_)()|
|指针/整数互转|reinterpret_cast|void* → uintptr_t|
|不相关指针类型转换|**避免使用**|-|

### C++中 struct 和 class 的区别？

| 特性     | struct                      | class                       | 备注说明         |
| ------ | --------------------------- | --------------------------- | ------------ |
| 默认访问权限 | public                      | private                     | 最核心区别        |
| 默认继承权限 | public                      | private                     | 继承时的默认访问控制   |
| 语义侧重   | 数据聚合                        | 对象封装                        | 约定俗成的编程风格    |
| 模板元编程  | 更常用                         | 较少用                         | 特化时struct更简洁 |
| 构造函数   | 可定义                         | 可定义                         | 无区别          |
| 析构函数   | 可定义                         | 可定义                         | 无区别          |
| 成员函数   | 可定义                         | 可定义                         | 无区别          |
| 继承支持   | 完整支持                        | 完整支持                        | 无区别          |
| 多态支持   | 完整支持                        | 完整支持                        | 无区别          |
| 访问控制   | 支持 public/protected/private | 支持 public/protected/private | 需要显式指定       |
在 C++中， struct 和 class 非常相似，它们都可以用来定义包含数据和函数的类类型。 主要区
别在于默认的访问修饰符和继承类型。 struct 默认的成员访问权限和继承都是公开的，这使得它更
适合用于那些主要用于数据存储的简单数据结构。相反， class 默认的成员访问和继承权限是私有
的，这鼓励了更加严格的封装和抽象，使其成为定义复杂行为和数据交互的理想选择
##### 代码
```
// struct 默认 public
struct PersonStruct {
    std::string name;  // 默认为 public
    int age;           // 默认为 public
};

// class 默认 private
class PersonClass {
    std::string name;  // 默认为 private
    int age;           // 默认为 private
};

void demoAccess() {
    PersonStruct ps;
    ps.name = "Alice";  // ✅ 直接访问
    
    PersonClass pc;
    // pc.name = "Bob"; // ❌ 编译错误：无法访问 private 成员
}
struct BaseStruct {
    int publicVar;
};
---------------------------------------------------------------------------------
class BaseClass {
public:
    int publicVar;
};

// struct 继承：默认 public
struct DerivedStruct : BaseStruct {
    void use() {
        publicVar = 42;  // ✅ 可以访问基类成员
    }
};

// class 继承：默认 private
class DerivedClass : BaseClass {
    void use() {
        // publicVar = 42; // ❌ 编译错误：BaseClass::publicVar 不可访问
    }
};
----------------------------------------------------------------------------------
// 适合存储纯数据
struct Vec3 {
    float x, y, z;  // 公有成员
    std::string toString() const {
        return std::to_string(x) + "," + std::to_string(y);
    }
};

// POD类型（Plain Old Data）
struct SensorData {
    uint32_t timestamp;
    double temperature;
    double humidity;
};
-----------------------------------------------------------------------------------
// 适合封装复杂行为
class BankAccount {
private:
    double balance;
    std::string owner;

public:
    BankAccount(const std::string& owner) : owner(owner), balance(0) {}
    
    void deposit(double amount) {
        if(amount > 0) balance += amount;
    }
    
    bool withdraw(double amount) {
        if(amount <= balance) {
            balance -= amount;
            return true;
        }
        return false;
    }
};
--------------------------------------------------------------------------------
// struct 更常用于模板特化
template <typename T>
struct TypeTraits {
    static const bool isPointer = false;
};

template <typename T>
struct TypeTraits<T*> {
    static const bool isPointer = true;
};

// class 也可用但较少见
template <typename T>
class TypeChecker {
    // ...
};

static_assert(TypeTraits<int*>::isPointer, "Should be pointer");
----------------------------------------------------------------------------------
// 可与C语言兼容的数据结构
extern "C" struct CCompatStruct {
    int id;
    char name[50];
};

// 在C++代码中使用
void processCData(CCompatStruct* data) {
    // ...
}

// 在C代码中可同样使用
/* C代码：
struct CCompatStruct {
    int id;
    char name[50];
};
*/
```

### 结构体 struct 和联合体 union 的区别
| 内存分配方式 | 每个成员拥有独立内存空间      | 所有成员共享同一块内存空间            |
|--------|-------------------|--------------------------|
| 内存占用大小 | ≥ 所有成员大小之和（含对齐填充） | = 最大成员的大小（含对齐填充）         |
| 成员访问   | 所有成员可同时访问         | 同一时间只能激活一个成员             |
| 数据存储   | 存储所有成员的值          | 只能存储一个成员的值               |
| 初始化    | 可同时初始化所有成员        | 只能初始化第一个成员（C++11起可指定初始化） |
| 赋值影响   | 修改一个成员不影响其他成员     | 修改一个成员会覆盖其他成员            |
| 默认访问控制 | public            | public                   |
| 用途     | 存储相关数据集合          | 节省内存的场景/类型转换             |
| 构造函数   | 支持                | C++11起支持                 |
| 析构函数   | 支持                | C++11起支持                 |
| 继承     | 支持                | 不支持                      |
| 成员类型限制 | 无限制               | 不能包含引用或非平凡类型（C++11前）     |
##### 1. 内存布局（核心区别）
```
struct Point {
    float x; // 4字节
    float y; // 4字节
    char label[10]; // 10字节
}; // 总大小 ≈ 18字节（实际可能20-24字节含填充）

void demoStruct() {
    Point p{1.5f, 2.8f, "center"};
    // 内存示意图：
    // [x:4][y:4][label:10][填充:2-6]
}
-----------------------------------
union Data {
    int i;       // 4字节
    float f;     // 4字节
    char str[8]; // 8字节
}; // 总大小 = 8字节（最大成员）

void demoUnion() {
    Data d;
    d.i = 42;    // 激活整形成员
    // d.f = 3.14; // 激活此成员会覆盖整数值
    
    // 内存示意图：
    // [共享内存区:8字节]
    // 激活i： [0x0000002A]
    // 激活f： [浮点表示]
}
```
##### 2. 成员访问特性
```
//结构体同时访问
struct Rect {
    Point topLeft;
    Point bottomRight;
};

void useRect() {
    Rect r{{0,0,"TL"}, {100,100,"BR"}};
    cout << r.topLeft.x; // 访问topLeft的x
    cout << r.bottomRight.y; // 同时访问bottomRight的y
}
-------------------------------------------------------
//联合体互斥访问
union Converter {
    uint32_t raw;
    float value;
    struct { // 匿名结构体（C++11）
        uint16_t low;
        uint16_t high;
    } parts;
};

void useConverter() {
    Converter c;
    c.value = 3.14159f; // 激活浮点成员
    
    // 访问不同成员（互斥）
    cout << "Float: " << c.value;
    cout << "Hex: 0x" << hex << c.raw;
    cout << "Parts: " << c.parts.low << "-" << c.parts.high;
    
    // 注意：同时访问不同成员会导致数据解释错误
}
```
##### 3. 初始化差异
```
//结构体完整初始化
struct Config {
    int timeout;
    bool logging;
    char mode;
};

Config cfg{500, true, 'A'}; // 初始化所有成员
---------------------------------------------
//联合体受限初始化
// C++11前：只能初始化第一个成员
union Token {
    int num;
    char symbol;
    char* text;
};

Token t1 = {42}; // 只能初始化第一个成员num

// C++11起：指定初始化
Token t2{.symbol = 'X'}; // 指定初始化symbol成员
```
##### 4.典型应用场景
```
//结构体
// 1. 坐标系统
struct Vec3 {
    float x, y, z;
};

// 2. 数据库记录
struct UserRecord {
    int id;
    std::string name;
    time_t created_at;
};

// 3. 链表节点
struct ListNode {
    int data;
    ListNode* next;
};
------------------------------
//联合体
// 1. 类型转换
union FloatConvert {
    float f;
    uint32_t u;
};

float fastInverseSqrt(float x) {
    FloatConvert conv;
    conv.f = x;
    conv.u = 0x5f3759df - (conv.u >> 1); // 魔法常数
    return conv.f;
}

// 2. 协议解析
union IPAddress {
    uint32_t address;
    uint8_t octets[4];
};

// 3. 节省内存的变体
union Variant {
    int int_val;
    double dbl_val;
    bool bool_val;
};

struct TaggedVariant {
    enum Type { INT, DOUBLE, BOOL } type;
    Variant value;
};
```

### C++函数调用的压栈过程
![[函数压栈的过程.png]]
函数调用时从右到左将参数压入栈中，随后压入返回地址，跳转到函数代码执行，处理返回值并清理栈，最后返回到调用位置

| 步骤       | 操作            | 寄存器变化      | 栈内容变化             | 说明                       |
|----------|---------------|------------|-------------------|--------------------------|
| 1. 参数压栈  | 从右向左压入参数      | ESP 递减     | [参数N] → [参数1]     | 参数大小影响ESP移动距离            |
| 2. 返回地址  | call 指令压入返回地址 | ESP -= 4   | [返回地址]            | 保存函数返回后继续执行的地址           |
| 3. 保存帧指针 | push ebp      | ESP -= 4   | [旧EBP]            | 保存调用者的基址指针               |
| 4. 建立新帧  | mov ebp, esp  | EBP = ESP  | -                 | 设置当前函数的栈帧基址              |
| 5. 局部变量  | sub esp, N    | ESP -= N   | [局部变量1] → [局部变量N] | 为局部变量分配空间                |
| 6. 保存寄存器 | push reg      | ESP -= 4   | [寄存器值]            | 保存需要保护的寄存器(Callee-saved) |
| 7. 函数执行  | 执行函数体         | EAX 存储返回值  | -                 | 访问参数：[EBP+8+4*(n-1)]     |
| 8. 恢复现场  | pop reg       | ESP += 4   | -                 | 恢复寄存器值                   |
| 9. 释放局部  | mov esp, ebp  | ESP = EBP  | 清除局部变量            | 释放局部变量空间                 |
| 10. 恢复帧  | pop ebp       | EBP = 旧值   | ESP += 4          | 恢复调用者栈帧                  |
| 11. 返回   | ret           | EIP = 返回地址 | ESP += 4          | 跳回调用位置                   |
| 12. 清参数  | add esp, M    | ESP += M   | 清除参数              | 调用者清理参数(cdecl)           |
```
#include <iostream>

// 被调用函数
int calculate(int a, int b) {
    int result = a * b;      // 局部变量
    int temp = result + 5;   // 另一个局部变量
    return temp;
}

int main() {
    int x = 10, y = 5;
    int sum = calculate(x, y); // 函数调用
    std::cout << "Result: " << sum;
    return 0;
}
-----------------------------------
main:
    ; 初始化局部变量
    mov     dword [ebp-4], 10    ; x = 10
    mov     dword [ebp-8], 5     ; y = 5
    
    ; 1. 参数压栈（从右向左）
    push    dword [ebp-8]        ; 压入 y(5)  [ESP -= 4]
    push    dword [ebp-4]        ; 压入 x(10) [ESP -= 4]
    
    ; 2. 调用函数（压入返回地址）
    call    calculate            ; [ESP -= 4] 压入返回地址
    ; 返回地址 = main+0x?? (call下条指令地址)
    
    ; 12. 调用者清理参数（cdecl约定）
    add     esp, 8               ; [ESP += 8] 清理参数x,y
    
    ; 存储返回值
    mov     [ebp-12], eax        ; sum = EAX
    
calculate:
    ; 3. 保存调用者帧指针
    push    ebp                 ; [ESP -= 4] 保存main的EBP
    
    ; 4. 建立新栈帧
    mov     ebp, esp            ; EBP = ESP (新基址)
    
    ; 5. 分配局部变量空间
    sub     esp, 8              ; [ESP -= 8] 分配两个int空间
    
    ; 6. 保存需要保护的寄存器（可选）
    ; push    esi                ; 如果需要使用ESI
    
    ; 7. 函数体执行
    mov     eax, [ebp+8]        ; EAX = a(10) [EBP+8]
    imul    eax, [ebp+12]       ; EAX *= b(5) [EBP+12]
    mov     [ebp-4], eax        ; result = EAX [EBP-4]
    
    mov     ecx, [ebp-4]        ; ECX = result
    add     ecx, 5              ; ECX += 5
    mov     [ebp-8], ecx        ; temp = ECX [EBP-8]
    
    mov     eax, [ebp-8]        ; EAX = temp (返回值)
    
    ; 8. 恢复寄存器（如有保存）
    ; pop     esi                 ; 恢复ESI
    
    ; 9. 释放局部变量空间
    mov     esp, ebp            ; ESP = EBP (释放局部变量)
    
    ; 10. 恢复调用者帧指针
    pop     ebp                 ; [ESP += 4] 恢复main的EBP
    
    ; 11. 返回指令
    ret                         ; [ESP += 4] 弹出返回地址到EIP
    --------------------------------------------------------------------------
    高地址
+-----------------+ 
|                 | 
|      ...        | 
+-----------------+ 
|       y(5)      |  ← EBP + 12 
+-----------------+ 
|       x(10)     |  ← EBP + 8 
+-----------------+ 
|   返回地址      |  ← EBP + 4 
+-----------------+ 
|   旧EBP         |  ← EBP (当前基址) 
+-----------------+ 
|    result       |  ← EBP - 4 
+-----------------+ 
|      temp       |  ← EBP - 8  ← ESP (栈顶指针)
+-----------------+ 
低地址
```

### 什么是回调函数，回调函数的使用场景
理解参见：[[回调函数]]
#### 定义
回调函数（Callback Function）是一种**编程模式**，它允许一个函数（称为"高阶函数"或"调用函数"）在特定时刻调用另一个函数（称为"回调函数"）。回调的本质是**将函数作为参数传递给另一个函数**，并在需要时由接收方执行该函数。
#### 场景
![[回调函数应用场景.png]]


#### 隐式转换，如何消除隐式转换
#### 定义
编译器自动将一种类型转换为另一种类型
#### 详细类型转换

| 概念     | 定义                  | 示例                                                        | 消除方法                                                              |
|--------|---------------------|-----------------------------------------------------------|-------------------------------------------------------------------|
| 算术类型转换 | 数值类型自动提升或降级         | int i = 3.14; // double→int                               | 使用static_cast显式转换：int i = static_cast<int>(3.14);                 |
| 指针转换   | 派生类指针→基类指针          | Derived* d = new Derived(); Base* b = d;                  | 无需消除（安全转换）                                                        |
| 类类型转换  | 通过单参数构造函数或转换运算符自动转换 | class String{ String(const char*); }; String s = "hello"; | 构造函数添加explicit：explicit String(const char*);                      |
| 数组退化   | 数组名自动转为指针           | int arr; int* p = arr;                                    | 使用std::array：std::array<int,5> arr; auto p = arr.data();          |
| 布尔转换   | 指针/数值→bool          | if (ptr) {...} // 非空检查                                    | 使用explicit operator bool()：explicit operator bool() const { ... } |
| 常量转换   | 非const→const        | const int* cp = &var;                                     | 无需消除（安全转换）                                                        |

### strlen 和 sizeof 区别
| 特性    | strlen函数                    | sizeof运算符          |
|-------|-----------------------------|--------------------|
| 本质    | C标准库函数 (#include <cstring>) | C/C++语言内置运算符       |
| 作用对象  | 仅用于以\0结尾的字符串                | 适用于任何类型（变量、类型、表达式） |
| 计算内容  | 字符串中\0前的字符个数                | 对象/类型在内存中占用的字节数    |
| 参数类型  | const char* (字符指针)          | 类型名 或 表达式          |
| 返回值类型 | size_t (字符数量)               | size_t (字节数量)      |
| 编译时行为 | 运行时计算                       | 编译时确定结果（除C99变长数组外） |
| 空字符处理 | 遇到\0停止计数（不包含\0）             | 包含整个数组（包括\0和其他元素）  |
| 常量表达式 | 不能用于常量表达式                   | 可以用于常量表达式          |
strlen 和 sizeof 虽然都用于测量大小，但它们的用途截然不同。
Strlen： Strlen（） 专门用于计算 C 风格字符串的长度，即字符数组直到第一个 null 字符的距离。它在运行时执行，因此对于动态内容很有用。
Sizeof： Sizeof（） 是一个编译时运算符，用于确定数据类型或结构的内存占用量，非常适合在编译时需要确定数据布局大小的场景。在编程中正确选择和使用这两者对于优化内存使用和保证程序的正确性至关重要。


### 头文件如何避免出现重定义的问题

| 解决方案           | 实现方式                                            | 优点           | 缺点              | 适用场景          |
|----------------|-------------------------------------------------|--------------|-----------------|---------------|
| `Include Guards` | `#ifndef UNIQUE_NAME#define UNIQUE_NAME...#endif` | 跨平台标准化广泛支持   | 需手动确保唯一标识符      | 所有C/C++项目     |
| `#pragma once`   | `#pragma once`                                    | 简洁明了编译器自动处理  | 非标准特性（但主流编译器支持） | 非严格标准要求的项目    |
| `extern声明`       | `头文件：extern int globalVar;源文件：int globalVar = 0;` | 避免全局变量重定义    | 增加维护成本          | 需在多文件共享的全局变量  |
| `static限制作用域`    | `static int localVar;`                            | 作用域限制到单个编译单元 | 每个文件有独立副本       | 文件内部使用的全局变量   |
| `匿名命名空间`         | `namespace {int internalVar;}`                    | C++标准方式限制作用域 | 仅限C++           | C++文件内部私有实现   |
| `内联函数`           | `inline void func() {...}`                        | 允许多次定义       | 可能增加代码体积        | 头文件中需要定义的小型函数 |
| `const隐式static`  | `const int MAX_SIZE = 100;`                       | 自动内部链接       | 仅限基本类型常量        | 头文件中的常量定义     |
| `类定义保护`          | `类声明放在头文件成员定义放在源文件`                               | 避免成员函数重定义    | 增加文件数量          | C++类实现        |

### 临时对象在什么时候会产生
| 场景类别      | 具体情形         | 示例代码                                | 生命周期           |
| --------- | ------------ | ----------------------------------- | -------------- |
| 函数返回非引用   | 返回非引用类型对象    | `string func() { return "tmp"; }`   | 表达式结束时销毁       |
| 类型转换      | 隐式构造转换       | `process(42); // 形参是string`         | 函数调用结束时销毁      |
| 传值调用      | 值传递对象        | `void log(string s); log("data");`  | 函数调用结束时销毁      |
| 算术表达式     | 中间运算结果       | `double d = 1.2 + 3.4 * 5.6;`       | 表达式结束时销毁       |
| 显式构造      | 匿名对象创建       | `Point p = Point(10,20);`           | 表达式结束时销毁       |
| 类型转换运算符   | 自定义类型转换      | `operator int() {...} int i = obj;` | 表达式结束时销毁       |
| 绑定常量引用    | 绑定到 const 引用 | `const string& rs = "hello";`       | 引用作用域结束时销毁     |
| Lambda 捕获 | 按值捕获表达式结果    | `[v = getValue()]{...}`             | Lambda 对象销毁时销毁 |
| 抛出异常      | 抛异常对象        | `throw MyError("msg");`             | 被异常处理捕获后销毁     |


### C++的返回值优化
| 优化类型 | 触发条件        | 优化效果     | C++标准状态               | 代码示例                     |
|------|-------------|----------|-----------------------|--------------------------|
| RVO  | 返回匿名临时对象    | 消除临时对象拷贝 | 允许(C++98) → 强制(C++17) | return MyClass();        |
| NRVO | 返回命名局部对象    | 消除局部对象拷贝 | 允许(非强制)               | MyClass obj; return obj; |
| 无优化  | 复杂返回路径/禁用优化 | 完整拷贝构造   | -                     | 分支返回不同对象                 |
#### 优化机制详解

##### 1. 未优化时的拷贝过程
```
MyClass createObject() {
    MyClass temp; // 1. 构造局部对象
    return temp;  // 2. 拷贝到返回值存储区 → 3. 拷贝到调用方对象
}

MyClass obj = createObject(); // 两次拷贝构造
```
##### 2. RVO 优化流程
```
MyClass createObject() {
    // 直接在调用方预留的内存构造对象
    return MyClass(); // 零拷贝
}

MyClass obj = createObject(); // 直接构造
```
##### 3. NRVO 优化流程
```
MyClass createObject() {
    MyClass temp; // 直接在目标地址构造
    
    // 编译器重写为在obj内存操作
    return temp; // 零拷贝
}

MyClass obj = createObject(); // 直接构造
```

| 优化类型 | 输出结果                          | 说明          |
|------|-------------------------------|-------------|
| RVO  | 构造 → 析构                       | 直接构造 → 最终析构 |
| NRVO | 构造 → 析构                       | 直接构造 → 最终析构 |
| 无优化  | 构造 → 构造 → 拷贝构造 → 析构 → 析构 → 析构 | 完整拷贝过程      |

### 局部 Static 变量为什么线程安全
在 C++11 及之后的标准中，**局部 static 变量的初始化是线程安全的**，这是通过编译器实现的底层同步机制保证的。以下是关键原理：
#### 线程安全机制实现原理
```
void initResource() {
    static Resource* res = new Resource(); // 线程安全初始化
}
//编译器生成的等效代码：
void initResource() {
    static bool initialized = false;     // 首次标记
    static Resource* res = nullptr;       // 实际变量
    static std::mutex init_mutex;        // 隐式互斥锁（概念示意）
    
    if (!initialized) {
        std::lock_guard<std::mutex> lock(init_mutex); // 编译器生成的锁
        
        if (!initialized) {              // 双检查锁定
            res = new Resource();
            initialized = true;
        }
    }
    return res;
}
```
#### 核心安全机制
1. **首次调用屏障**：
    - 编译器插入原子操作检查初始化状态
    - 使用 `std::atomic_flag` 或等效机制
2. **双检查锁定**：
3. **内存屏障**：
    - 初始化完成后插入内存屏障指令
    - 保证其他线程看到完整初始化结果

(1)初始化
Static 变量在程序启动时（全局 static 变量）或第一次使用时（局部 static 变量）初始化。初始化过程由编译器和运行时环境控制，确保在多线程环境下只进行一次初始化。
(2)固定内存位置
Static 变量存储在全局/静态存储区，该区域在程序的整个生命周期内存在，且位置固定。这意味着不同线程访问 static 变量时，访问的是同一个内存位置。
(3)单例模式
Static 变量的单次实例化特性类似于单例模式，在多线程环境下，通过一次性实例化避免竞争条件。编译器和运行时确保 static 变量只实例化一次。
(4)具体原因和机制
C++11 标准规定： 根据 C++11 标准，第 6.7.4 节中规定，局部静态变量的初始化将在第一个进入该变量所在的作用域的线程完成，并且是线程安全的。这意味着在多个线程同时访问一个局部静态变量时，只有一个线程会进行初始化，而其他线程会被阻塞，直到初始化完成。
编译器的实现： 编译器在生成代码时，会在局部静态变量的初始化代码周围插入同步机制， 确保只有一个线程能执行初始化代码，而其他线程会等待该初始化过程完成。
实现方式：
加锁（Mutex）： 在变量初始化代码周围加锁，确保同一时刻只有一个线程能执行初始化。原子操作： 在某些平台上，使用原子操作来确保变量只初始化一次。
双重检查锁定（Double-Checked Locking）： 减少不必要的锁开销，先检查变量是否已经初始化，如果没有，再加锁进行初始化。
**特别注意： 全局静态变量和命名空间静态变量和类的静态成员变量都是没有内置的线程安全保证**


## C++内存管理

### 堆和栈
#### 🧠 堆（Heap）与栈（Stack）深度对比（C++视角）
#### 📊 核心区别对比表

| **特性**     | **栈（Stack）**        | **堆（Heap）**                     |
| ---------- | ------------------- | ------------------------------- |
| **管理方式**   | 编译器自动管理（LIFO）       | 程序员手动管理（`new`/`delete`）         |
| **分配速度**   | ⚡️ 极快（移动栈指针）        | ⏱ 较慢（搜索空闲内存块）                   |
| **释放方式**   | 自动（作用域结束）           | 手动（需显式释放）                       |
| **内存大小**   | 固定（通常 1-8 MB/线程）    | 动态（受系统内存限制）                     |
| **内存碎片**   | ❌ 无                 | ✅ 可能产生                          |
| **访问方式**   | 直接（通过变量名）           | 间接（通过指针）                        |
| **生命周期**   | 与作用域绑定              | 显式控制（可跨作用域）                     |
| **线程安全**   | ✅ 线程私有              | ❌ 全局共享（需同步）                     |
| **典型用例**   | 局部变量/函数参数           | 大型对象/动态数据结构                     |
| **溢出风险**   | 栈溢出（Stack Overflow） | 堆耗尽（Out of Memory）              |
| **分配失败结果** | ❗ 程序崩溃（无法捕获）        | 🕳 返回 `nullptr` 或抛出 `bad_alloc` |
#### 🧱 技术细节与代码示例

##### 1. **栈内存分配（自动管理）**
```
void stackExample() {
    int a = 10;          // 基本类型在栈上
    double arr[100];      // 小数组在栈上
    Point p {1.0, 2.0};  // 小对象在栈上（~几十字节）

    // 函数结束时自动释放
} 
```
##### 2. **堆内存分配（手动管理）**
```
void heapExample() {
    // 基本类型
    int* num = new int(42); 
    
    // 大数组（1MB）
    float* bigArray = new float[250000]; 
    
    // 大对象
    auto* bigObj = new LargeObject(/* params */); 

    // 必须手动释放！
    delete num;
    delete[] bigArray;
    delete bigObj;
}
```
##### 3. **生命周期对比**
```
int* createHeapInt() {
    int local = 5;          // ❌ 错误：返回栈地址
    return &local;          // 悬空指针！

    int* heapVal = new int(10);  // ✅ 正确
    return heapVal;         // 需调用者delete
}

void lifecycleDemo() {
    int* ptr = createHeapInt();
    std::cout << *ptr;      // 有效（堆内存存活）
    delete ptr;             // 必须释放
}
```
#### ⚠️ 关键风险与解决方案

| **风险类型**  | **栈**      | **堆**    | **解决方案**                        |
| --------- | ---------- | -------- | ------------------------------- |
| **内存泄漏**  | 不可能        | 高频风险     | 智能指针（`unique_ptr`/`shared_ptr`） |
| **悬空指针**  | 常见（返回局部变量） | 释放后访问    | RAII + 作用域管理                    |
| **空间不足**  | 栈溢出（崩溃）    | 分配失败（异常） | 检查分配结果 + 异常处理                   |
| **碎片化**   | 无          | 频繁分配释放导致 | 内存池 + 对象复用                      |
| **多线程冲突** | 无（线程私有）    | 需同步访问    | 互斥锁（`mutex`）                    |

#### 🛡️ 最佳实践指南
##### 优先使用栈的场景：
```
// ✅ 推荐栈分配
void processData() {
    std::array<int, 100> buffer; // 栈上固定数组
    TransformMatrix mat;         // 小矩阵（<1KB）
    TemporaryCalculator calc;    // 临时工具对象
} // 自动释放无负担
```
##### 必须使用堆的场景：
```
// ✅ 安全堆分配实践
auto loadLargeResource() {
    // 智能指针自动管理
    auto image = std::make_unique<Image>(4096, 4096);
    
    // 超大容器（堆分配内部存储）
    auto dataset = std::make_shared<std::vector<DataPoint>>();
    dataset->resize(1'000'000);
    
    return std::make_pair(std::move(image), dataset);
}
```

#### 🔍 底层原理剖析
```
┌──────────────┐       ┌──────────────┐
│    Stack     │       │     Heap     │
├──────────────┤       ├──────────────┤
│ 高地址开始   │       │ 低地址开始   │
│ 向下增长     │       │ 向上增长     │
│              │       │              │
│ 局部变量     │       │ 动态分配对象 │
│ 函数参数     │       │              │
│ 返回地址     │       ├──────┬───────┤
│              │       │ 空闲│ 已分配 │
└──────┬───────┘       └──────┴───────┘
       │ 自动管理               │
       │ (push/pop)            │ 手动管理
       ▼                       ▼
```
#### 💡 决策流程图：堆 vs 栈
1. 对象 > 1KB
2. 需要跨作用域存活
3. 容器元素数量 > 1000
4. 需要多线程共享访问
![[堆和栈.png|375]]



#### C++栈溢出问题/如何防止栈溢出
##### 什么是栈溢出？
当程序**超出线程栈内存限制**（通常 1-8 MB）时发生，常见原因：
1. **过深的递归调用**
2. **过大的局部变量**
3. **无限递归**
4. **大型对象在栈上分配**

##### 预防栈溢出的方法

| **方法**        | **实现方式**                            | **示例**                                          |
| ------------- | ----------------------------------- | ----------------------------------------------- |
| **1. 避免大栈分配** | 使用堆内存 (`new`/`malloc`) 替代栈内存        | `std::vector<int> data(1000000); // 堆分配`        |
| **2. 递归改迭代**  | 用循环+显式栈结构替代递归                       | 见下方迭代示例                                         |
| **3. 尾递归优化**  | 确保递归调用是最后操作（编译器自动优化）                | `return recursive_call();`                      |
| **4. 限制递归深度** | 添加深度计数器                             | 见下方深度限制示例                                       |
| **5. 增大栈空间**  | 编译器/链接器选项设置（平台相关）                   | `g++ -Wl,--stack=8388608` (Windows 8MB)         |
| **6. 对象池技术**  | 重用对象减少分配                            | `static thread_local std::vector<Obj> pool;`    |
| **7. 动态数据结构** | 使用 `std::vector`/`std::list` 替代原生数组 | `std::vector<float> matrix(1000*1000);`         |
| **8. 异步任务分解** | 将大任务拆分为小任务异步执行                      | `std::async(std::launch::async, process_chunk,` |

### memcpy 和 strcpy 什么区别
| 特性    | memcpy                                                | strcpy                                      |
| ----- | ----------------------------------------------------- | ------------------------------------------- |
| 所属头文件 | `<cstring>`                                           | `<cstring>`                                 |
| 原型    | `void* memcpy(void* dest, const void* src, size_t n)` | `char* strcpy(char* dest, const char* src)` |
| 复制依据  | 按字节数复制                                                | 按字符串终止符 \0 复制                               |
| 复制长度  | 需显式指定长度参数                                             | 自动计算到 \0 的长度                                |
| 处理字符  | 可处理任意二进制数据                                            | 仅处理 C 风格字符串                                 |
| 安全性   | 可能越界（需手动保证长度）                                         | 可能越界（无长度检查）                                 |
| 返回值   | 返回目标指针                                                | 返回目标指针                                      |
| 重叠处理  | 未定义行为（需用 memmove）                                     | 未定义行为                                       |
| 效率    | 更高（无终止符检查）                                            | 稍低（需检查 \0）                                  |


### C++内存分区
![[C++内存分区.png]]
#### C++ 内存分区概览表

| **内存分区**      | **存储内容**         | **生命周期**      | **管理方式** | **特点**            |
| ------------- | ---------------- | ------------- | -------- | ----------------- |
| **栈区(Stack)** | 局部变量、函数参数、返回值等   | 函数开始到结束       | 编译器自动管理  | 速度快、大小有限、自动释放     |
| **堆区(Heap)**  | 动态分配的内存          | 由程序员控制（分配至释放） | 手动管理     | 速度较慢、容量大、需显式释放    |
| **全局/静态区**    | 全局变量、静态变量（局部和全局） | 程序启动到结束       | 编译器自动管理  | 初始化的数据区与未初始化数据区分开 |
| **常量区**       | 字符串常量和其他常量       | 程序启动到结束       | 只读       | 与代码区可能重叠          |
| **代码区**       | 程序代码（机器指令）       | 程序启动到结束       | 只读       | 通常不可写入            |
#### 详细分区解析
##### 1. 栈区 (Stack)
- **存储内容**:
    - 函数内部定义的局部变量（非`static`）
    - 函数参数
    - 函数返回值地址
    - 函数调用的上下文信息（返回地址、寄存器状态等）
- **生命周期**: 函数调用开始到函数返回
- **特点**:
    - **自动管理**: 由编译器自动分配和释放
    - **速度快**: 栈指针移动即可完成内存分配
    - **容量有限**: 通常较小（Windows默认1MB，Linux默认8MB）
    - **后进先出(LIFO)**: 最后分配的内存最先释放

##### 2. 堆区 (Heap)
- **存储内容**:
    - 动态分配的内存（`malloc`/`free`, `new`/`delete`等）
- **生命周期**: 从分配开始到显式释放为止
- **特点**:
    - **手动管理**: 需要程序员显式分配和释放
    - **容量大**: 受限于系统可用内存
    - **分配速度慢**: 需要寻找合适的内存块
    - **碎片问题**: 频繁分配释放可能产生内存碎片

```
void heapExample() {
    int* ptr = new int[100]; // 堆上分配数组
    delete[] ptr; // 必须显式释放
}
```
##### 3. 全局/静态区
分为两个子区域：
- **BSS段 (Block Started by Symbol)**:
    - 存储**未初始化**的全局变量和静态变量
    - 程序加载时由系统初始化为零值
- **数据段 (Data Segment)**:
    - 存储**已初始化**的全局变量和静态变量
    - 包含显式初始化的值

| **变量类型** | **存储位置** | **初始化值** |
| -------- | -------- | -------- |
| 未初始化全局变量 | BSS段     | 0        |
| 已初始化全局变量 | 数据段      | 指定初始值    |
| 静态局部变量   | 数据段      | 只初始化一次   |
| 静态全局变量   | 数据段      | 指定初始值    |
```
int globalUninit;       // BSS段（未初始化）
int globalInit = 42;    // 数据段（已初始化）

void func() {
    static int count = 0; // 数据段（静态局部变量）
    count++;
}
```

##### 4. 常量区
- **存储内容**:
    - 字符串常量
    - `constexpr`常量
    - 全局`const`变量
- **特点**:
    - **只读属性**: 通常无法修改
    - **共享内存**: 相同常量可能共享同一内存
    - **生命周期**: 程序运行期间始终存在
##### 5. 代码区 (Text Segment)
- **存储内容**:
    - 编译后的机器指令
    - 函数体的二进制代码
- **特点**:
    - **只读属性**: 防止程序意外修改指令
    - **共享内存**: 多个实例可共享同一代码段
    - **优化目标**: 编译器优化的主要区域


### 内存泄漏 
#### 内存泄漏的本质与分类
内存泄漏指程序中**已动态分配的堆内存**由于某种原因**无法被释放或回收**的情况。这种资源未被回收的现象会导致程序内存占用持续增长，最终可能导致系统性能下降或程序崩溃。
#### 内存泄漏分类表

|**类型**|**特征描述**|**常见场景**|
|---|---|---|
|**常发性泄漏**|每次执行特定操作都会泄漏内存|循环体中未释放资源|
|**偶发性泄漏**|仅在特殊情况或特定输入下发生泄漏|异常处理路径中遗漏释放|
|**一次性泄漏**|程序启动时初始化后无法释放|全局变量的初始化资源未释放|
|**隐式泄漏**|程序运行中持有不再需要的资源|缓存未清理、静态集合累积|
|**资源泄漏**|除内存外的其他资源泄漏|文件句柄、网络连接未关闭|
#### 内存泄漏的根本原因

##### 1. 手动内存管理失误
```
void memoryLeakDemo() {
    int* ptr = new int[100];  // 动态分配
    // ...使用ptr...
    // 忘记delete[] ptr;  → 泄漏100*sizeof(int)内存
}
```
##### 2. 异常安全漏洞
```
void unsafeFunction() {
    Resource* res = new Resource();
    
    riskyOperation();  // 可能抛出异常
    
    delete res;  // 异常发生时无法执行
}
```
##### 3. 数据结构未正确清理
```
class ContactList {
    std::map<std::string, Contact*> contacts;
public:
    ~ContactList() {
        // 错误：未释放Contact对象
        // 应添加：for(auto& p : contacts) delete p.second;
    }
};
```
##### 4. 循环引用（智能指针）
```
class Node {
public:
    std::shared_ptr<Node> next;
};

void circularReference() {
    auto node1 = std::make_shared<Node>();
    auto node2 = std::make_shared<Node>();
    
    node1->next = node2;  // node2引用计数+1
    node2->next = node1;  // node1引用计数+1 → 循环引用!
}
```
##### 5. API 使用不当
```
void openFiles() {
    for(int i = 0; i < 1000; i++) {
        FILE* f = fopen("data.txt", "r");  // 打开文件
        // ...操作文件...
        // 忘记fclose(f); → 文件句柄泄漏
    }
}
```
#### C++内存泄露及检测工具
##### 1. 静态代码分析工具

|**工具**|**平台**|**特点**|
|---|---|---|
|**Clang-Tidy**|跨平台|集成于LLVM，支持现代C++规范|
|**Cppcheck**|跨平台|轻量级，低误报率|
|**PVS-Studio**|Windows/Linux|商业级，深度分析|
|**Coverity**|企业级|支持大型代码库|
##### 2. 动态检测工具

|**工具**|**操作系统**|**检测能力**|
|---|---|---|
|**Valgrind**|Linux|内存泄漏、越界访问|
|**AddressSanitizer**|跨平台|快速内存错误检测|
|**Dr. Memory**|Windows|类似Valgrind的Windows方案|
|**Visual Studio诊断工具**|Windows|集成调试器，内存使用分析|
##### 3. 运行时监控技术
```
// 重载new/delete追踪内存分配
void* operator new(size_t size) {
    void* ptr = malloc(size);
    MemoryTracker::recordAlloc(ptr, size);
    return ptr;
}

void operator delete(void* ptr) noexcept {
    MemoryTracker::recordFree(ptr);
    free(ptr);
}

// 内存跟踪器实现
class MemoryTracker {
    static std::map<void*, size_t> allocations;
public:
    static void recordAlloc(void* ptr, size_t size) {
        allocations[ptr] = size;
    }
    
    static void recordFree(void* ptr) {
        allocations.erase(ptr);
    }
    
    static void reportLeaks() {
        for(auto& [ptr, size] : allocations) {
            std::cerr << "泄漏内存: " << ptr << " 大小: " << size << "字节\n";
        }
    }
};
```
#### 内存泄漏防范策略
##### 1. RAII（资源获取即初始化）原则
```
class FileWrapper {
    FILE* file;
public:
    explicit FileWrapper(const char* filename) 
        : file(fopen(filename, "r")) {}
    
    ~FileWrapper() {
        if(file) fclose(file);
    }
    
    // 禁止复制
    FileWrapper(const FileWrapper&) = delete;
    FileWrapper& operator=(const FileWrapper&) = delete;
    
    // 移动语义支持
    FileWrapper(FileWrapper&& other) noexcept 
        : file(other.file) {
        other.file = nullptr;
    }
};

void safeFileOperation() {
    FileWrapper f("data.txt");  // 自动管理资源
    // 无需手动关闭，即使抛出异常
}
```
##### 2. 智能指针的正确使用
```
void smartPointerDemo() {
    // 独占所有权
    auto unique = std::make_unique<Resource>();
    
    // 共享所有权
    auto shared = std::make_shared<Resource>();
    
    // 打破循环引用
    struct SafeNode {
        std::shared_ptr<SafeNode> next;
        std::weak_ptr<SafeNode> prev;  // 使用weak_ptr避免循环
    };
    
    auto node1 = std::make_shared<SafeNode>();
    auto node2 = std::make_shared<SafeNode>();
    
    node1->next = node2;
    node2->prev = node1;  // 安全：weak_ptr不增加引用计数
}
```
##### 3. 容器资源管理
```
// 错误：容器存储原始指针
std::vector<Resource*> resources;

// 正确：使用智能指针容器
std::vector<std::unique_ptr<Resource>> safeResources;

// 正确：对象直接存储
std::vector<Resource> directResources;
```
##### 4. 异常安全保证

| **安全等级** | **要求**        | **实现方法**               |
| -------- | ------------- | ---------------------- |
| **基本保证** | 无资源泄漏         | RAII类封装资源              |
| **强保证**  | 操作要么完成要么保持原状态 | 事务性操作，copy-and-swap惯用法 |
| **不抛保证** | 承诺不抛出异常       | noexcept声明，简单操作        |
```
// 强保证实现示例
class Transaction {
    Resource* res1;
    Resource* res2;
public:
    void execute() {
        auto tmp1 = std::make_unique<Resource>();
        auto tmp2 = std::make_unique<Resource>();
        
        // 所有操作可能失败的部分
        performOperation(tmp1.get());
        performOperation(tmp2.get());
        
        // 全部成功才提交
        std::swap(res1, tmp1);
        std::swap(res2, tmp2);
    }
};
```

### RAII 思想， 他是怎么保证对象没有内存泄漏的？
#### 📌 核心理念

> **"资源生命周期 = 对象生命周期"**  
> 在构造函数中获取资源，在析构函数中释放资源，利用C++对象作用域规则实现自动资源管理

#### 🔧工作原理
![[RAII结构.png]]
#### 🛡️ 防止内存泄漏的机制
1. **自动释放保证**
```
void process() {
    FileHandler f("data.txt");  // 构造函数打开文件
    // ... 使用文件（可能抛出异常）
} // 无论正常返回或异常退出，f的析构函数都会关闭文件
```
2. **异常安全屏障**
```
void highRiskOperation() {
    auto ptr = std::make_unique<Resource>();  // RAII保护
    dangerousStep1();      // 可能抛出异常
    dangerousStep2();      // 可能抛出异常
} // 即使异常发生，ptr仍会释放资源
```
3. **作用域绑定**
```
{
    DatabaseConnection db("localhost"); // 连接数据库
    db.executeQuery("SELECT ...");
} // db离开作用域，自动断开连接
```
#### 🧩 关键技术组件

| **组件** | **作用**         | **标准库示例**                        |
| ------ | -------------- | -------------------------------- |
| 构造函数   | 获取资源（内存/文件/锁等） | `std::fstream::fstream()`        |
| 析构函数   | 释放资源           | `std::unique_ptr::~unique_ptr()` |
| 作用域规则  | 确定对象生命周期       | 局部变量/临时对象                        |
| 栈展开机制  | 异常时自动调用析构函数    | C++异常处理系统                        |
#### 🌟 RAII实践示例

**1. 智能指针（内存管理）**
```
void safeMemory() {
    auto ptr = std::make_unique<int[]>(1000);  // 分配内存
    // 使用内存...
    processData(ptr.get());
} // 自动释放内存，无泄漏
```
**2. 文件句柄管理**
```
class SafeFile {
public:
    SafeFile(const char* path) : handle(fopen(path, "r")) {
        if(!handle) throw std::runtime_error("Open failed");
    }
    
    ~SafeFile() { 
        if(handle) fclose(handle); 
    }
    
    // 禁用拷贝（或实现移动语义）
    SafeFile(const SafeFile&) = delete;
    SafeFile& operator=(const SafeFile&) = delete;

private:
    FILE* handle;
};
```
**3. 互斥锁管理**
```
void threadSafeFunction() {
    std::mutex mtx;
    
    {
        std::lock_guard<std::mutex> lock(mtx);  // 构造时加锁
        // 临界区操作...
    } // 离开作用域自动解锁
}
```
 🆚 与传统资源管理对比、
 ```
 // 危险的传统方式（易泄漏）
void riskyMethod() {
    Resource* res = new Resource();
    if(operationFails()) {
        return;  // 内存泄漏！
    }
    delete res;  // 必须手动释放
}

// RAII安全方式
void safeMethod() {
    auto res = std::make_unique<Resource>();
    if(operationFails()) {
        return;  // 自动释放内存
    }
} // 自动释放
```

##### 📊 RAII优势矩阵

|**特性**|**传统方式**|**RAII方式**|**防泄漏效果**|
|---|---|---|---|
|异常安全|❌ 易泄漏|✅ 自动释放|⭐⭐⭐⭐⭐|
|多退出点支持|❌ 需每个return释放|✅ 统一处理|⭐⭐⭐⭐⭐|
|代码简洁性|❌ 冗余清理代码|✅ 逻辑专注核心功能|⭐⭐⭐⭐|
|资源所有权明确|❌ 容易混淆|✅ 对象绑定清晰|⭐⭐⭐⭐⭐|
|嵌套资源管理|❌ 极易出错|✅ 自动级联释放|⭐⭐⭐⭐⭐|
##### ⚠️ 关键注意事项
1. **禁止拷贝**  
    多数RAII类应禁用拷贝构造/赋值（或实现移动语义）
```
class NonCopyable {
public:
    NonCopyable(const NonCopyable&) = delete;
    NonCopyable& operator=(const NonCopyable&) = delete;
};
```
2. **移动语义支持**
```
class SocketConnection {
public:
    SocketConnection(SocketConnection&& other) 
        : socket_(other.socket_) {
        other.socket_ = INVALID_SOCKET;
    }
private:
    SOCKET socket_;
};
```
3. **多态资源处理**
```
class BaseResource {
public:
    virtual ~BaseResource() = default;  // 必须虚析构！
};

class DerivedResource : public BaseResource {
    ~DerivedResource() override { /* 释放具体资源 */ }
};
```



#### 💡 设计原则

1. **单一资源原则**：每个RAII类只管理一种资源类型
2. **完全封装**：资源句柄不暴露给外部
3. **异常安全**：构造函数要么完全成功，要么抛出异常（无半构造状态）
4. **资源转移**：通过移动语义实现所有权转移
    

> 🌟 **行业实践**：现代C++中超过95%的资源管理应通过RAII实现，直接使用`new`/`delete`或原始资源API的情况应限制在底层封装中。


### 智能指针
手撕智能指针ref-----[[手撕智能指针]]
> **核心使命**：通过RAII自动化内存管理，消除裸指针风险，解决内存泄漏和悬空指针问题
#### 📊 智能指针家族矩阵

|**类型**|**所有权模型**|**复制行为**|**最佳场景**|**性能开销**|
|---|---|---|---|---|
|`unique_ptr`|独占所有权|禁止复制，仅移动|单一所有者资源|零开销（与裸指针相当）|
|`shared_ptr`|共享所有权|引用计数增加|多所有者共享资源|中等（原子计数操作）|
|`weak_ptr`|无所有权|不增加引用计数|解决`shared_ptr`循环引用|低（与`shared_ptr`协同）|
#### 🧩 核心技术剖析

##### 1. `unique_ptr`（独占指针）
**核心特性**：
- 独占资源所有权，不可复制
- 零开销抽象（编译期优化）
- 支持自定义删除器
```
#include <memory>

void uniquePtrDemo() {
    // 创建独占指针（推荐make_unique）
    auto sensor = std::make_unique<Sensor>("A001");
    
    // 转移所有权（移动语义）
    auto newOwner = std::move(sensor);  // sensor现在为nullptr
    
    // 自定义删除器（用于特殊资源）
    auto fileDeleter = [](FILE* f) { 
        if(f) fclose(f); 
    };
    std::unique_ptr<FILE, decltype(fileDeleter)> 
        logFile(fopen("app.log", "w"), fileDeleter);
    
    // 自动释放：离开作用域时调用删除器
}
```
##### 2. `shared_ptr`（共享指针）
**核心机制**：
- 引用计数（原子操作保证线程安全）
- 控制块存储计数器和删除器
- 支持弱引用计数
```
void sharedPtrDemo() {
    // 创建共享资源（推荐make_shared）
    auto server = std::make_shared<WebServer>(8080);
    
    // 共享所有权
    auto admin = server;
    auto monitor = server;
    
    // 引用计数验证
    std::cout << "Use count: " << server.use_count() << "\n";  // 输出3
    
    // 自定义分配器+删除器
    auto customAlloc = [](size_t size) { 
        return memPool.allocate(size); 
    };
    auto customDelete = [](Connection* conn) {
        conn->release();
        memPool.deallocate(conn);
    };
    
    std::shared_ptr<Connection> conn(
        new (customAlloc(sizeof(Connection))) Connection(),
        customDelete,
        customAlloc
    );
}
```
##### 3. `weak_ptr`（弱指针）
**核心价值**：
- 打破`shared_ptr`循环引用
- 不增加引用计数
- 需通过`lock()`转换为临时`shared_ptr`访问资源
```
class DeviceController;

class Sensor {
public:
    void setController(std::shared_ptr<DeviceController> ctrl) {
        controller = ctrl;
    }
private:
    std::weak_ptr<DeviceController> controller;  // 弱引用避免循环
};

class DeviceController {
public:
    void addSensor(std::shared_ptr<Sensor> s) {
        sensors.push_back(s);
    }
private:
    std::vector<std::shared_ptr<Sensor>> sensors;
};

void weakPtrDemo() {
    auto sensor = std::make_shared<Sensor>();
    auto controller = std::make_shared<DeviceController>();
    
    sensor->setController(controller);
    controller->addSensor(sensor);
    
    // 安全访问弱指针
    if(auto ctrl = sensor->controller.lock()) {
        ctrl->adjustSettings();
    } else {
        std::cout << "Controller expired\n";
    }
}
```
#### ✅ 最佳实践
```
// 创建方式
auto p1 = std::make_unique<Widget>();      // 首选（异常安全）
auto p2 = std::make_shared<Resource>();    // 单次内存分配

// 所有权转移
std::unique_ptr<Data> processData() {
    auto data = std::make_unique<Data>();
    // ...处理数据...
    return data;  // 移动语义自动生效
}

// 与弱指针配合
class Cache {
public:
    std::shared_ptr<Image> loadImage(int id) {
        if(auto it = cache.find(id); it != cache.end()) {
            if(auto img = it->second.lock()) {
                return img;  // 缓存命中
            }
        }
        auto img = std::make_shared<Image>(id);
        cache[id] = img;  // 存储弱引用
        return img;
    }
private:
    std::unordered_map<int, std::weak_ptr<Image>> cache;
};
```
#### 🔧 高级技术：自定义删除器
------待补充--------------------------
```
// 1. 管理非内存资源
auto dbConn = std::unique_ptr<sqlite3, decltype(&sqlite3_close)>(
    openDatabase(), 
    sqlite3_close
);

// 2. 数组特殊处理
auto pixelBuffer = std::shared_ptr<uint8_t[]>(
    new uint8_t[4K_BUFFER_SIZE],
    [](uint8_t* p) { delete[] p; }  // 数组删除器
);

// 3. 异步安全删除
auto guiObject = std::shared_ptr<QWidget>(
    new CustomWidget,
    [](QWidget* w) { w->deleteLater(); }  // Qt线程安全删除
);
```

![[智能指针决策树.png]]


#### 智能指针有什么缺点
| 缺点类别    | 底层原因                        | 具体表现                   | 解决方案                   |
|---------|-----------------------------|------------------------|------------------------|
| 性能开销    | 原子操作、虚函数表、控制块管理             | shared_ptr操作比裸指针慢2-10倍 | 改用unique_ptr或裸指针优化热点路径 |
| 循环引用    | shared_ptr双向引用导致引用计数永不归零    | 对象无法释放，内存泄漏            | 使用weak_ptr打破循环         |
| 控制块开销   | shared_ptr需单独分配控制块（16-32字节） | 小对象内存浪费可达300%          | 优先使用make_shared合并分配    |
| 线程安全局限  | 引用计数原子安全，但指向对象非自动安全         | 多线程读写需额外同步             | 结合mutex或使用线程安全对象       |
| 定制删除器陷阱 | 删除器类型影响shared_ptr类型         | 不同类型指针无法放入同一容器         | 使用std::function包装删除器   |
| API兼容性  | C接口和某些库要求裸指针                | 需显式获取原始指针(get())       | 设计适配器包装C接口             |
| 内存碎片    | 控制块分布式分配                    | 长期运行程序出现内存碎片           | 使用内存池或定制分配器            |
| 移动语义局限  | shared_ptr移动仍保留控制块          | 移动后源指针为nullptr但控制块延迟释放 | 热点路径改用unique_ptr       |

##### 代码示例
 1. 性能开销问题
```
//原子操作成本
// 性能测试代码
void sharedPtrPerf() {
    auto start = std::chrono::high_resolution_clock::now();
    
    for(int i = 0; i < 1'000'000; ++i) {
        // 热点路径
        auto p = std::make_shared<int>(i);
        auto q = p;  // 原子递增
    } // 原子递减
    
    auto duration = std::chrono::high_resolution_clock::now() - start;
    std::cout << "shared_ptr耗时: " 
              << duration.count() / 1e6 << "ms\n";
}

// 对比裸指针版本快3-5倍
----------------------------------------------------------
//优化
// 关键性能路径改用unique_ptr
void processFrame() {
    std::unique_ptr<Frame> frame = acquireFrame();
    
    // 无原子操作开销的处理
    transformFrame(*frame); 
    
    // 自动释放
}
```
2. 循环引用陷阱-导致内存泄漏
```
//典型死锁场景：
struct TreeNode {
    std::shared_ptr<TreeNode> parent;
    std::shared_ptr<TreeNode> child;
    
    ~TreeNode() { std::cout << "节点释放\n"; }
};

void circularReference() {
    auto parent = std::make_shared<TreeNode>();
    auto child = std::make_shared<TreeNode>();
    
    parent->child = child;  // 引用计数+1
    child->parent = parent; // 引用计数+1
    
    // 退出作用域后：
    // parent计数=1 (被child引用)
    // child计数=1 (被parent引用)
    // 内存泄漏！
}
--------------------------------------------------------------
//解决方案-使用弱指针
struct SafeNode {
    std::shared_ptr<SafeNode> child;
    std::weak_ptr<SafeNode> parent;  // 弱引用打破循环
    
    void printParent() {
        if(auto p = parent.lock()) {  // 安全访问
            std::cout << "父节点存在\n";
        }
    }
};
```
3. 控制块开销
```
内存布局对比：

 传统堆分配:
  [对象数据]

shared_ptr分配:
  [控制块] -> [引用计数][弱引用计数][删除器][分配器]
  [对象数据]
----------------------------------------------------------
// 坏方案: 两次堆分配
std::shared_ptr<Widget> p(new Widget); 

// 好方案: 单次分配合并控制块和对象
auto p = std::make_shared<Widget>(); 
```
4. 线程安全问题
```
//错误示例
class UnsafeCache {
    std::shared_ptr<Config> config;
public:
    void update() {
        auto newConfig = loadConfig();  // 耗时操作
        config = newConfig;  // 看似原子赋值
    }
    
    void use() {
        if(config) {
            // 可能访问正在被修改的config
            config->apply(); 
        }
    }
};
----------------------------------------------------------
//安全示例
class SafeCache {
    std::shared_ptr<const Config> config;  // 指向const
    mutable std::mutex mtx;
public:
    void update() {
        auto newConfig = loadConfig();
        std::lock_guard lock(mtx);
        config = std::move(newConfig);
    }
    
    std::shared_ptr<const Config> get() const {
        std::lock_guard lock(mtx);
        return config;  // 返回副本
    }
};
```

#### shared_ptr 是不是线程安全的

| 特性        | 描述              |
| --------- | --------------- |
| 引用计数线程安全  | 对引用计数的操作是线程安全的  |
| 对象本身非线程安全 | 被管理对象本身不是线程安全的  |
| 读写操作      | 读写操作需要额外的同步机制   |
| 使用场景      | 适用于多线程共享同一对象的场景 |
shared_ptr 的引用计数操作是线程安全的，确保引用计数的增减不会引发竞争条件。然而，shared_ptr 所管理的对象本身并不是线程安全的， 读写操作需要额外的同步机制来保证线程安全

> **核心结论**：`shared_ptr` **部分线程安全**  
> 其线程安全性需从三个维度分析：**控制块**、**指针实例**和**指向对象**

| **操作对象**             | **线程安全规则**           | **示例**                  |
| -------------------- | -------------------- | ----------------------- |
| **控制块**              | ✅ **完全线程安全**（原子引用计数） | 多线程同时拷贝`shared_ptr`     |
| **同一`shared_ptr`实例** | ❌ **非线程安全**（需外部同步）   | 多线程同时修改同一个`shared_ptr`  |
| **指向的对象**            | ❌ **非线程安全**（需单独保护）   | 多线程访问`shared_ptr->data` |
##### 🔍 详细分析

###### 1. 控制块线程安全（✅ 安全）
```
// 安全：不同线程操作不同shared_ptr实例（指向同一对象）
void threadSafeDemo() {
    auto globalPtr = std::make_shared<int>(42);
    
    auto task = [](std::shared_ptr<int> p) {
        // 每个线程有自己的shared_ptr副本
        auto localCopy = p;  // ✅ 引用计数原子增加
        // 使用localCopy...
    };
    
    std::thread t1(task, globalPtr);
    std::thread t2(task, globalPtr);
    t1.join(); t2.join();
} // 所有副本销毁后对象自动释放
```
**原理**：  
引用计数器使用**原子操作**（如`std::atomic::fetch_add`），保证多线程环境下计数准确。

###### 2. 同一实例线程不安全（❌ 危险）
```
// 危险：多线程操作同一个shared_ptr实例
std::shared_ptr<Resource> sharedInstance;

void unsafeThreadA() {
    sharedInstance = std::make_shared<Resource>(); // 写操作
}

void unsafeThreadB() {
    if(!sharedInstance.expired()) { // 读操作
        sharedInstance->use();     // 可能访问已释放对象
    }
}
```
**风险场景**：
- 线程A执行`reset()`
- 线程B同时调用`get()`
- 导致[[#野指针和悬挂指针|悬挂指针]]或**双重释放**
###### 3. 指向对象非线程安全（❌ 需保护）
```
struct Counter {
    int value = 0;
    void increment() { ++value; } // 非原子操作
};

void unsafeIncrement() {
    auto counter = std::make_shared<Counter>();
    
    std::thread t1([&]{ counter->increment(); });
    std::thread t2([&]{ counter->increment(); });
    // 最终value可能不等于2（数据竞争）
}
```

#### 什么时候用智能指针？什么时候用裸指针
##### 智能指针使用场景

| 场景       | 智能指针类型          | 代码示例                                                                             | 理由            |
|----------|-----------------|----------------------------------------------------------------------------------|---------------|
| 独占资源管理   | `std::unique_ptr` | `auto file = std::make_unique<File>("data.txt");`                                  | 自动释放资源，防止泄漏   |
| 共享资源所有权  | `std::shared_ptr` | `auto config = std::make_shared<Config>(); worker.useConfig(config);`              | 多个组件共享访问权     |
| 打破循环引用   | `std::weak_ptr`   | `parent->child = child; child->parent = parent->weak_from_this();`                 | 解决循环引用导致的泄漏   |
| 工厂模式返回   | `std::unique_ptr` | `static std::unique_ptr<Engine> create() { return std::make_unique<V8Engine>(); }` | 明确所有权转移       |
| 异常安全资源获取 | `任意智能指针`          | `auto conn = std::make_shared<Database>();`                                        | 异常时自动释放资源     |
| 多线程对象共享  | `std::shared_ptr` | `auto globalData = std::make_shared<ThreadSafeData>();`                            | 原子引用计数确保线程安全  |
| 回调生命周期绑定 | `std::shared_ptr` | `object->asyncTask([self = shared_from_this()]{ self->complete(); });`             | 确保对象在回调期间保持活动 |
##### 裸指针使用场景
| 场景         | 代码示例                                                          | 理由                  | 风险控制               |
| ---------- | ------------------------------------------------------------- | ------------------- | ------------------ |
| 性能关键路径     | `for(auto* item : items) total += item->price; // 高频循环内`        | 避免智能指针开销（原子操作、间接访问） | 确保对象生命周期覆盖使用期      |
| 非拥有观察者     | `void printUser(const User* user) { /* 仅读取 */ }`                | 明确表示不参与生命周期管理       | 文档标注指针有效性依赖外部      |
| 第三方/C接口交互  | `legacy_process(resource.get()); // 从智能指针提取`                    | 兼容不支持智能指针的接口        | 使用get()提取并用注释说明所有权 |
| 数据结构内部链接   | `class TreeNode { TreeNode* parent; /* 父节点不拥有子节点 */ };`         | 避免循环引用，简化数据结构       | 确保容器整体管理生命周期       |
| 栈对象/全局对象引用 | `extern Config g_config; void use() { auto* cfg = &g_config; }` | 对象已有确定生命周期          | 避免在智能指针和裸指针间转换     |
| 低级内存操作     | `void* rawMem = malloc(1024); // 特殊分配器场景`                       | 需要直接控制内存布局          | 包装在RAII类中管理        |
| 编译器兼容性要求   | `#pragma pack(1) struct Packet { char* payload; }; // 特定内存布局`   | 智能指针可能破坏POD特性       | 仅在必要模块使用           |
对比

| 特性     | 智能指针                           | 裸指针         |
|--------|--------------------------------|-------------|
| 生命周期管理 | 自动管理                           | 手动管理        |
| 所有权表达  | 明确（unique/shared/weak）         | 隐式          |
| 循环引用风险 | shared_ptr可能需weak_ptr解决        | 无自动解决方案     |
| 性能开销   | 原子操作、控制块开销（shared_ptr约10-30ns） | 零开销（约1-3ns） |
| 线程安全   | shared_ptr引用计数原子安全             | 完全无保护       |
| 代码安全性  | 自动异常安全                         | 易泄漏         |
| 可读性    | 所有权明确                          | 需注释说明       |
| 兼容性    | 需C++11及以上                      | 全平台兼容       |
| 内存占用   | shared_ptr额外16-32字节控制块         | 仅指针大小       |
| 多态支持   | 无缝支持                           | 支持但危险       |

#### shared_ptr 与 weak_ptr 的区别与联系
| 特性       | shared_ptr                | weak_ptr            |
|----------|---------------------------|---------------------|
| 所有权语义    | 强所有权（共享所有权）               | 弱所有权（无所有权）          |
| 影响对象生命周期 | ✔︎ 延长对象生命周期               | ✘ 不影响对象生命周期         |
| 引用计数影响   | 增加强引用计数                   | 增加弱引用计数             |
| 直接访问对象   | 支持（operator->, operator*） | 不支持（需转换为shared_ptr） |
| 空状态构造    | 可直接构造空指针                  | 通常从shared_ptr构造     |
| 典型应用场景   | 共享资源管理                    | 打破循环引用、缓存、观察者模式     |
代码示例
- 所有权与控制权
```
// shared_ptr：拥有所有权
auto shared = std::make_shared<Resource>();
shared->use();  // 直接访问

// weak_ptr：无所有权
std::weak_ptr<Resource> weak = shared;  
// weak->use();  // 错误！不能直接访问

// weak_ptr访问需升级
if(auto temp = weak.lock()) {  // 转换为shared_ptr
    temp->use();  // 安全访问
}
```
- 生命周期影响
```
void testLifecycle() {
    std::weak_ptr<Resource> observer;
    
    {
        auto shared = std::make_shared<Resource>();
        observer = shared;
        
        std::cout << "强引用计数: " << observer.use_count() << "\n"; // 1
    }  // shared离开作用域，Resource销毁
    
    std::cout << "对象是否存活: " 
              << (observer.expired() ? "否" : "是") << "\n"; // 否
}
```
- 引用计数机制
```
auto shared1 = std::make_shared<Resource>(); // 强引用=1, 弱引用=0
std::weak_ptr<Resource> weak = shared1;      // 强引用=1, 弱引用=1

{
    auto shared2 = shared1;                  // 强引用=2, 弱引用=1
}  // shared2析构，强引用=1, 弱引用=1

shared1.reset(); // 强引用=0 → Resource销毁
                 // 弱引用=1 → 控制块保留

std::cout << "控制块是否有效: "
          << (weak.expired() ? "否" : "是") << "\n"; // 是（但对象已销毁）
```


### new 和 malloc 区别
| 特性     | new (C++ 运算符)               | malloc (C 库函数)      |
| ------ | --------------------------- | ------------------- |
| 语言归属   | C++ 原生运算符                   | C 标准库函数 (<cstdlib>) |
| 内存来源   | 堆区                          | 堆区 (Heap)           |
| 返回值类型  | 类型安全指针 (无需转换)               | void* (需显式类型转换)     |
| 内存大小   | 自动计算 (无需sizeof)             | 需手动计算字节大小           |
| 失败行为   | 抛出 std::bad_alloc 异常        | 返回 NULL             |
| 构造/析构  | ✅ 调用构造函数/析构函数               | ❌ 仅分配/释放内存          |
| 初始化支持  | ✅ 支持直接初始化 (e.g. new int(5)) | ❌ 分配后需手动初始化         |
| 重载机制   | ✅ 可重载 (operator new)        | ❌ 不可重载              |
| 数组处理   | ✅ 专用语法 (new[]/delete[])     | ❌ 统一处理 (需手动计算)      |
| 内存不足处理 | 可通过 set_new_handler 定制      | 需检查返回值              |
| 类型安全   | ✅ 编译时类型检查                   | ❌ 类型不安全 (易出错)       |
| 与析构协调  | ✅ 自动生命周期管理                  | ❌ 完全手动管理            |
工作机制
malloc机制
![[malloc工作机制.png|250]]
new机制
![[new工作机制.png|275]]

**黄金法则**：
1. C++中**永远优先使用 `new/delete`**
2. 除非在**与C库交互**或**实现底层分配器**时才考虑`malloc/free`
3. **现代C++应通过智能指针和容器避免直接使用两者**

#### new 和 malloc 申请的是哪里的内存？如何减少内存碎片？
| 分配方式   | 内存来源      | 内存分区 | 特点                                      |
|--------|-----------|------|-----------------------------------------|
| malloc | C 标准库分配器  | 堆区   | 通过 brk/sbrk 或 mmap 系统调用分配虚拟内存           |
| new    | C++ 内存管理器 | 堆区   | 调用 operator new (底层通常使用 malloc)，并执行构造函数 |
##### 🧩 内存碎片类型与成因

|**碎片类型**|**成因**|**示意图**|
|---|---|---|
|**外部碎片**|频繁分配释放不同大小对象导致内存空洞|███ ▭ ███ ▭ ███ (空洞)|
|**内部碎片**|分配器对齐填充或最小块限制造成的空间浪费|[请求30B→分配32B]|
|**堆碎片**|`brk` 分配导致堆空间不连续|[低地址]███[高地址]▭|
##### 🔧 碎片优化策略

###### 1. 内存池技术（固定大小分配）
```
class FixedMemoryPool {
    struct Block { Block* next; };
    Block* freeList = nullptr;
    size_t blockSize;

public:
    FixedMemoryPool(size_t size) : blockSize(std::max(size, sizeof(Block))) {}
    
    void* allocate() {
        if (!freeList) expandPool();
        auto block = freeList;
        freeList = freeList->next;
        return block;
    }

    void deallocate(void* ptr) {
        auto block = static_cast<Block*>(ptr);
        block->next = freeList;
        freeList = block;
    }
};

// 使用示例
FixedMemoryPool pool(sizeof(MyObject));
MyObject* obj = new(pool.allocate()) MyObject();  // 定位new
obj->~MyObject();
pool.deallocate(obj);
```
###### 2. 高级分配器选择
| 分配器      | 特点          | 适用场景   |
|----------|-------------|--------|
| jemalloc | 多线程优化，低碎片   | 高性能服务器 |
| tcmalloc | 线程局部缓存，快速分配 | 多线程应用  |
| mimalloc | 现代设计，极致性能   | 资源受限环境 |
###### 3. 智能分配策略
```
// 对象池模板
template<typename T>
class ObjectPool {
    std::vector<std::unique_ptr<T[]>> chunks;
    std::stack<T*> freeList;
    static constexpr size_t CHUNK_SIZE = 64;

public:
    T* acquire() {
        if (freeList.empty()) allocateChunk();
        auto obj = freeList.top();
        freeList.pop();
        return new(obj) T();  // 原位构造
    }
    
    void release(T* obj) {
        obj->~T();
        freeList.push(obj);
    }
};
```
###### 4. 分配模式优化
| 策略     | 实现方式                          | 效果       |
|--------|-------------------------------|----------|
| 批量预分配  | std::vector.reserve()         | 减少多次小分配  |
| 对象复用   | 对象池模式                         | 避免反复分配释放 |
| 合理选择容器 | std::deque 替代 std::vector     | 减少重新分配   |
| 自定义对齐  | alignas(64) struct Data {...} | 优化缓存利用率  |

#### 实际情况中想用 new 分配内存但又不确定大小怎么做？
| 场景             | 推荐策略              | 示例            |
|----------------|-------------------|---------------|
| 集合大小可在使用前确定    | 两阶段分配法            | 文件加载前获取文件大小   |
| 逐步添加元素的集合      | std::vector 或动态扩容 | 读取网络流数据       |
| 元素数量未知但最大尺寸可预估 | 最大预估分配法           | 图像处理（最大分辨率已知） |
| 需要高性能的内存分配/释放  | 自定义内存池            | 游戏引擎、高频交易系统   |
| 需要异常安全的内存管理    | 智能指针              | 复杂业务逻辑中的资源管理  |
| 内存受限环境         | 精确的两阶段分配          | 嵌入式系统开发       |
##### 1. 两阶段分配法（最常用）
```
// 第一阶段：确定所需大小
size_t calculate_required_size() {
    size_t size = 0;
    // 计算逻辑...
    return size;
}

// 第二阶段：分配内存
void allocate_memory() {
    size_t required_size = calculate_required_size();
    int* dynamic_array = new int[required_size];
    
    // 使用分配的内存...
    
    delete[] dynamic_array; // 释放内存
}
```
##### 2.动态扩容策略
```
//适用于需要逐步增加内存的场景
class DynamicBuffer {
    char* data = nullptr;
    size_t size = 0;
    size_t capacity = 0;

public:
    void append(const char* new_data, size_t len) {
        // 检查是否需要扩容
        if (size + len > capacity) {
            // 计算新容量（常见的1.5倍扩容策略）
            size_t new_capacity = capacity == 0 ? 64 : capacity * 3 / 2;
            while (size + len > new_capacity) {
                new_capacity = new_capacity * 3 / 2;
            }
            
            // 分配新内存
            char* new_data_area = new char[new_capacity];
            
            // 复制旧数据
            if (data) {
                std::memcpy(new_data_area, data, size);
                delete[] data; // 释放旧内存
            }
            
            data = new_data_area;
            capacity = new_capacity;
        }
        
        // 添加新数据
        std::memcpy(data + size, new_data, len);
        size += len;
    }
    
    ~DynamicBuffer() {
        delete[] data;
    }
};
```
##### 3. 使用标准库容器（推荐）
```
//标准库容器自动处理动态内存管理：
#include <vector>
#include <iostream>

int main() {
    std::vector<int> dynamic_array;
    
    // 动态添加元素，无需预先知道大小
    for (int i = 0; i < 100; ++i) {
        dynamic_array.push_back(i * i);
    }
    
    // 访问元素
    for (int num : dynamic_array) {
        std::cout << num << " ";
    }
    
    // 内存自动释放
}
```
##### 4. 最大预估分配法
```
//当可以确定最大可能大小时：
void process_data() {
    const size_t MAX_POSSIBLE_SIZE = 1024 * 1024; // 1MB
    
    // 分配最大可能需要的空间
    char* buffer = new char[MAX_POSSIBLE_SIZE];
    
    size_t actual_used = 0;
    // ...填充缓冲区，记录实际使用量...
    
    // 如果需要，可以缩减分配（但通常不需要）
    // 使用缓冲区...
    
    delete[] buffer;
}
```
##### 5. 使用智能指针管理不确定大小的内存
```
#include <memory>

void smart_allocation() {
    // 不确定大小的分配
    auto calculate_size = []() -> size_t {
        return /* 动态计算大小 */;
    };
    
    // 使用unique_ptr管理动态数组
    std::unique_ptr<int[]> dynamic_array(new int[calculate_size()]);
    
    // 使用动态数组...
    // 内存自动释放
}
```
##### 6. 内存池技术（高性能场景）
```
//对于频繁分配释放的场景：
class MemoryPool {
    struct MemoryChunk {
        MemoryChunk* next;
    };
    
    MemoryChunk* free_list = nullptr;
    size_t chunk_size;
    size_t chunks_per_block = 1000;

public:
    MemoryPool(size_t size) : chunk_size(std::max(size, sizeof(MemoryChunk))) {}
    
    void* allocate() {
        if (!free_list) {
            add_new_block();
        }
        
        MemoryChunk* chunk = free_list;
        free_list = free_list->next;
        return chunk;
    }
    
    void deallocate(void* ptr) {
        MemoryChunk* chunk = static_cast<MemoryChunk*>(ptr);
        chunk->next = free_list;
        free_list = chunk;
    }

private:
    void add_new_block() {
        // 分配大块内存
        char* block = new char[chunk_size * chunks_per_block];
        
        // 将大块分割并加入空闲列表
        for (size_t i = 0; i < chunks_per_block; ++i) {
            MemoryChunk* chunk = reinterpret_cast<MemoryChunk*>(block + i * chunk_size);
            chunk->next = free_list;
            free_list = chunk;
        }
    }
};
```


### delete 和 free 区别
| 特性     | delete                    | free                           |
|--------|---------------------------|--------------------------------|
| 所属语言   | C++ 运算符                   | C 标准库函数（定义在 <cstdlib>）         |
| 配对操作   | 释放 new 分配的内存              | 释放 malloc/calloc/realloc 分配的内存 |
| 析构函数调用 | ✅ 调用对象的析构函数               | ❌ 不调用任何析构函数                    |
| 类型安全   | ✅ 自动计算对象大小（无需手动指定）        | ❌ 需手动计算内存大小（如 sizeof）          |
| 重载支持   | ✅ 可重载类专属的 operator delete | ❌ 不可重载                         |
| 失败行为   | ❌ 失败时行为未定义（通常崩溃）          | ❌ 失败时行为未定义（通常崩溃）               |
| 空指针处理  | ✅ 安全（delete nullptr 无操作）  | ✅ 安全（free(nullptr) 无操作）        |
| 数组处理   | ✅ 需用 delete[] 释放数组        | ❌ 统一用 free（不区分数组/单对象）          |
| 底层实现依赖 | ❌ 不依赖 free（可独立实现）         | ✅ 通常与 malloc 同源实现              |
代码
```
//1. 析构函数调用
class Resource {
public:
    Resource()  { std::cout << "Resource acquired\n"; }
    ~Resource() { std::cout << "Resource released\n"; } // 析构函数
};

// 使用 delete（正确）
Resource* res1 = new Resource();
delete res1;  // 输出: Resource released

// 使用 free（错误）
Resource* res2 = new Resource();
free(res2);    // 无输出！资源泄漏 + 未调用析构函数
--------------------------------------------------------------
//2、类型安全
// delete 自动计算大小
int* p1 = new int(42);
delete p1;  // 无需指定大小

// free 需手动计算大小
int* p2 = (int*)malloc(sizeof(int));
free(p2);   // 需用 sizeof 计算
------------------------------------------------------------------
//数组处理
// 正确释放数组
int* arr1 = new int[10];
delete[] arr1;  // 必须用 delete[]

// 错误释放数组
int* arr2 = (int*)malloc(10 * sizeof(int));
free(arr2);      // free 不区分数组
----------------------------------------------------------------------
//重载机制
class CustomAlloc {
public:
    void* operator new(size_t size) {
        std::cout << "Custom new\n";
        return ::operator new(size);
    }
    void operator delete(void* ptr) {
        std::cout << "Custom delete\n";
        ::operator delete(ptr);
    }
};

CustomAlloc* obj = new CustomAlloc;
delete obj;  // 输出: Custom delete → 可定制释放逻辑
```

### 野指针和悬挂指针
> **核心区别**：  
> **野指针** = 未初始化的随机指针（指向未知区域）  
> **悬挂指针** = 指向已释放内存的无效指针（曾经有效）

| 特性          | 野指针 (Wild Pointer) | 悬挂指针 (Dangling Pointer) |
|-------------|--------------------|-------------------------|
| 产生原因        | 未初始化指针变量           | 指向已释放的内存区域              |
| 指向状态        | 随机内存地址 (可能未分配)     | 曾经有效但已被释放的内存地址          |
| 危险等级        | ⚠️ 高危 (可能立即崩溃)     | ⚠️⚠️⚠️ 极高危 (隐性数据破坏)     |
| 典型场景        | 声明后未赋值的指针          | 1. 释放后未置空               |
| 2. 返回局部变量地址 |
| 检测难度        | 较易 (编译器警告)         | 困难 (可能间歇性崩溃)            |
| 修复方案        | 初始化时置空             | 释放后置空 + 所有权管理           |
**所有指针在创建时必须初始化，在释放后必须置空，在非必要时使用智能指针**

### 内存对齐
> **核心概念**：数据在内存中的起始地址必须是特定值（对齐系数）的整数倍，以优化CPU访问效率

| 关键要素       | 说明                 | 典型值                    |
|------------|--------------------|------------------------|
| 对齐系数       | 地址必须是该值的整数倍        | 1, 2, 4, 8, 16, 32, 64 |
| 自然对齐       | 类型的对齐要求 = 其大小      | int大小4→对齐4             |
| Cache Line | CPU缓存行大小（对齐优化关键）   | 现代CPU通常64字节            |
| SIMD指令要求   | SSE/AVX等指令集的强制对齐要求 | 16/32/64字节边界           |
| 跨平台差异      | 不同架构对齐要求不同         | ARM vs x86差异           |
#### 为什么需要内存对齐？
##### 1. 硬件访问优化
```
// 未对齐访问可能触发多次内存读取
uint64_t* ptr = (uint64_t*)((char*)buffer + 3); // 地址非8倍数
uint64_t value = *ptr;  // x86: 性能损失, ARM: 可能崩溃
```
##### 2. CPU缓存效率
![[CPU缓存效率.png]]
#### C/C++ 中的对齐控制
##### 1. 结构体内存布局
```
struct UnalignedStruct {
    char a;      // 偏移0 (1字节)
    int b;       // 偏移4 (需要4对齐，1-3填充)
    double c;    // 偏移8 (8对齐)
    short d;     // 偏移16 (2字节)
}; // 大小=24 (18+6填充)

struct AlignedStruct {
    double c;    // 偏移0 (8字节)
    int b;       // 偏移8 (4字节)
    short d;     // 偏移12 (2字节)
    char a;      // 偏移14 (1字节)
}; // 大小=16 (15+1填充)，节省33%
```
##### 2. 手动对齐控制
```
// C++11 alignas 说明符
struct alignas(32) CacheAligned {
    int data[8]; // 32字节对齐
};

// GCC/Clang 属性
__attribute__((aligned(64))) char buffer[1024];

// MSVC 指令
__declspec(align(64)) struct PlatformSpecific {};
```
##### 3. 动态内存对齐
```
#include <cstdlib>

// C11/C++17 标准对齐分配
void* aligned_alloc(size_t alignment, size_t size);

// POSIX 扩展
void* memalign(size_t alignment, size_t size);

// Windows API
void* _aligned_malloc(size_t size, size_t alignment);
```

## C++面向对象


### C++ 三大特性

#### 多态的实现
#### 静态多态与动态多态

### 访问修饰符


### 多重继承


### 重载和重写

#### 编译器在底层如何实现重载


### 成员函数/成员变量/静态成员函数/静态成员变量的区别


### 构造函数和析构函数


### 虚函数

#### 哪些函数不能被声明为虚函数

#### 构造函数和析构函数能不能是虚函数，说明原因

#### 没有虚函数的话， C++ 如何实现多态




### 虚函数和虚函数表

#### 虚函数表在哪个阶段被分配的

#### 同一个类的不同对象的虚函数表是同一个吗

#### 基类的虚函数表存放在内存的什么区，虚表指针 vptr的初始化时间？

#### 虚函数表，是对象拥有还是类拥有

#### 虚函数内部调用非虚函数是调用指针类还是对象类




### 虚函数和纯虚函数

#### 纯虚函数的实现原理

#### 纯虚函数的应用场景


### 抽象类和纯虚函数


### 虚析构函数
#### 简述⼀下虚析构函数，什么作⽤

#### 说说为什么要虚析构，为什么不能虚构造


### 虚继承

#### 虚继承解决什么问题


### 深拷⻉和浅拷⻉的区别


### 运算符重载


### 空类能实例化吗



### 当一个类中没有任何成员变量和成员函数 ， 这时sizeof(A)的A值是多少


### 子类不能继承父类的函数有哪些


### C++类构造函数初始化列表执行顺序


### 构造函数的顺序，析构函数的顺序


### 成员变量初始化顺序





### 友元函数


### 友元类


### 在成员函数中调用 delete this 会出现什么问题




## C++ STL

### STL


### 常⻅的STL容器


### pair 容器


### vector容器
#### vector的实现原理

#### vector容器实现与扩充


### list


### deque


### stack&&queue


### heap && priority_queue


### map&&set


### map&&unordered_map


### push_back 和 emplace_back 的区别


### vector 和 list


### 迭代器

#### 迭代器有什么作⽤？什么时候迭代器会失效


## C++ 泛型编程

### C++模板全特化和偏特化


### 模板是在什么时候进行实例化


### 为什么模板类一般都是放在一个 h 文件中





## C++ 新特性

### C++11的新特性有哪些


### 智能指针


### 类型推导


### 右值引用


### nullptr


### 范围for循环


### 列表初始化


### lambda 表达式


### 并发
#### std::thread
#### lock_guard

#### unique_lock



