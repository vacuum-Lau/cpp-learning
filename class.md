### 一、 访问控制：为什么要分 public 和 private？

在 C 语言中，结构体的变量谁都能改。但在 C++ 中，我们希望保护数据，防止被改成“不合逻辑”的值（比如把线段长度设为 -10）。

*   **private (私有)**：只有类内部的函数能碰，外面的人看不见也摸不着。
*   **public (公有)**：类对外留的“窗口”，外面的人通过这些函数来操作数据。

```cpp
class Line {
private:
    double length; // 私有的，外面不能直接 line.length = -10;

public:
    // 公有的，外面通过这个函数改长度，我们可以加一个判断逻辑
    void setLength(double len) {
        if (len > 0) {
            length = len;
        } else {
            length = 0; // 如果给负数，强制设为0，这就是保护数据
        }
    }
};
```

---

### 二、 构造函数：对象出生的“出厂设置”

构造函数是一个**名字和类名完全一样**的特殊函数。当你写 `Line myLine;` 的那一刻，这个函数会自动运行。

#### 1. 无参构造函数（默认设置）
如果没有给参数，对象怎么初始化？
```cpp
class Line {
public:
    Line() { 
        length = 0; // 默认线段长度为0
        cout << "线段已创建，默认长度为0" << endl;
    }
private:
    double length;
};

// 使用：
Line a; // 自动打印“默认长度为0”
```

#### 2. 带参数的构造函数（自定义设置）
允许你在创建对象时直接传值。
```cpp
class Line {
public:
    Line(double len) { 
        length = len; 
    }
private:
    double length;
};

// 使用：
Line a(15.5); // 创建时直接把长度设为15.5
```

---

### 三、 析构函数：对象消失前的“遗言”

析构函数是在类名前加个波浪号 `~`。当对象的作用域结束（比如函数运行完了，对象要被销毁了），它会自动执行，通常用来关文件、释放内存。

```cpp
class Line {
public:
    ~Line() {
        cout << "这个线段对象现在被销毁了，释放了它占用的资源" << endl;
    }
};

// 析构函数不能带参数，也不能重载，一个类只能有一个。
```

---

### 四、 this 指针：解决“重名”尴尬

这是初学者最容易晕的地方。其实 `this` 就是**对象自己的地址**。最常用的场景是：当你的**参数名字**和**成员变量名字**一模一样时，用来区分谁是谁。

```cpp
class Box {
private:
    double length;

public:
    // 这里参数叫 length，成员变量也叫 length
    void setLength(double length) {
        // length = length; // 编译器会晕，它以为你在把参数赋给参数
        this->length = length; 
        // this->length 指的是类里面的那个私有变量
        // 等号右边的 length 指的是函数传进来的那个参数
    }
};
```

---

### 五、 静态成员 (static)：全班共享的一块黑板

普通变量是每个对象人手一份（你有一支笔，我有一支笔）。
`static` 变量是**全类共享一份**（全班只有一块黑板，你写了字，我也能看到）。

#### 1. 静态数据成员
它不属于某个对象，属于整个类。
```cpp
class Date {
public:
    static int count; // 声明：我们想统计一共创建了多少个日期对象
    Date() { count++; } // 每次有人出生，计数器就加1
};

// 注意！！静态变量必须在类外面单独初始化一次
int Date::count = 0; 

// 使用：
Date d1, d2, d3;
cout << Date::count; // 输出3，不管通过哪个对象访问，大家看到的都是同一个 count
```

#### 2. 静态成员函数
静态函数专门用来处理静态变量。因为它不属于某个具体对象，所以它里面**没有 this 指针**，也**不能访问非静态变量**。

```cpp
class Date {
private:
    static int count;
public:
    static void showCount() {
        cout << "当前对象总数：" << count << endl;
        // year = 2000; // 错误！静态函数不能访问普通变量 year
    }
};

// 使用：
Date::showCount(); // 直接用类名::就能调用，不需要创建对象也能运行
```

---

### 六、 举一反三：函数重载 (Overloading)

如果我想让一个函数能处理整数，又能处理小数，不用起两个名字，只要参数类型不同即可。

```cpp
// 交换两个整数
void Swap(int &a, int &b) {
    int temp = a; a = b; b = temp;
}

// 交换两个小数（函数名一模一样，但参数类型不同）
void Swap(double &a, double &b) {
    double temp = a; a = b; b = temp;
}

// 使用时：
int x=1, y=2;
Swap(x, y); // 编译器自动去找 int 版本的那个函数
```

---

**总结：**
1.  **封装**：用 `private` 把数据藏起来，用 `public` 函数当接口。
2.  **构造/析构**：对象“出生”和“死亡”时的自动操作。
3.  **this**：代表对象自己，常用于区分同名变量。
4.  **static**：全类共享，不属于个人，属于集体。

### 一、 程序内存布局：对象存哪儿？

在 C++ 中，对象存放的位置决定了它的生命周期：

1.  **栈（Stack）**：局部变量、函数参数。自动分配和释放。
2.  **堆（Heap）**：用 `new` 申请的空间。必须手动释放，否则会内存泄漏。
3.  **静态/全局区**：程序开始时运行，结束时销毁。
4.  **代码区（正文）**：存放函数代码和常量字符串，不可修改。

```cpp
int a = 10;        // 局部变量，在“栈”上
char *s = "Hello"; // 指针 s 在栈上，但字符串 "Hello" 在“代码区”，不可修改
int *p = new int;  // 指针 p 在栈上，指向的空间在“堆”上
```

---

### 二、 C++ 的动态内存：new 和 delete

虽然 C 语言用 `malloc`，但 C++ 必须用 `new`。**区别在于：`new` 会自动调用构造函数，`delete` 会自动调用析构函数。**

#### 1. 申请和释放单个对象
```cpp
// 申请一个 int 空间并初始化为 7
int *p = new int(7); 

// 申请一个对象，会自动运行 Text 类的构造函数
Text *ptr = new Text("Hello");

// 释放空间，会自动运行 Text 类的析构函数
delete ptr; 
```

#### 2. 申请和释放数组（重点：方括号必须配对）
如果申请的是数组，释放时必须加 `[]`，否则会导致部分对象没被销毁。
```cpp
// 申请 3 个 Text 对象的数组
Text *arr = new Text[3]; 

// 正确释放数组：必须带 []
delete []arr; 

// 错误示范：delete arr; -> 只会销毁第一个对象，造成内存泄漏
```

---

### 三、 内存安全：野指针与内存泄漏

这是初学者最常犯的两个致命错误：

#### 1. 内存泄漏 (Memory Leak)
你申请了堆空间，但把指向它的指针弄丢了，这块空间就永远无法回收。
```cpp
void func() {
    int *p = new int(10);
    // 函数结束了，p 消失了，但 new 出来的 10 还在堆里，没人能找到它了
} 
```

#### 2. 野指针 (Dangling Pointer)
你已经 `delete` 释放了空间，但还试图去读写那个指针。
```cpp
int *p = new int(5);
delete p;       // 空间还给系统了
// *p = 10;     // 恐怖！这是非法访问，程序随时崩溃
p = nullptr;    // 养成好习惯：delete 完立即置空
```

---

### 四、 现代 C++ 特性：=default 和 =delete (C++11)

#### 1. =default (显式默认)
如果你写了带参数的构造函数，编译器就不会再自动生成无参构造函数了。用 `=default` 可以强制让编译器帮你把那个默认的补回来。
```cpp
class A {
public:
    A(int x) {}    // 有了它，默认构造函数就没了
    A() = default; // 强制编译器生成默认的 A()
};
```

#### 2. =delete (禁用函数)
如果你不希望别人复制你的类（比如为了安全），可以禁用拷贝构造函数。
```cpp
class Secret {
public:
    Secret(const Secret&) = delete; // 禁用拷贝，别人写 Secret s2 = s1; 会报错
};
```

---

### 五、 成员初始化列表：高效初始化的秘密

在构造函数的大括号里赋值（`this->x = x`）其实是“先创建再赋值”，效率低。**初始化列表**（冒号语法）是“创建时直接初始化”，效率更高。

#### 1. 基本语法
```cpp
struct Point {
    int x, y;
    // 冒号开头，直接把参数传给成员
    Point(int a, int b) : x(a), y(b) {} 
};
```

#### 2. 必须使用初始化列表的三种情况
有些变量由于天生特性，不能先创建后赋值，必须在“出生”那一刻定好。
```cpp
class Test {
private:
    const int id;      // 常量：一旦出生不能更改
    int &ref;          // 引用：必须在定义时绑定
    Text subObj;       // 没用无参构造函数的子对象

public:
    // 这三种情况，只能写在冒号后面，写在大括号里会报错
    Test(int i, int &r) : id(i), ref(r), subObj("init") {}
};
```

#### 3. 初始化顺序的陷阱
**成员变量初始化的顺序是它们在类中定义的顺序，而不是你在冒号后面写的顺序！**
```cpp
class Foo {
    int i;
    int j;
public:
    // 即使你把 j 写在前面，编译器也会先初始化 i，再初始化 j
    Foo(int x) : j(x), i(j) {} // 危险！i(j) 时，j 还没初始化，i 会变成乱码
};
```

---

### 总结建议：
1.  **看到 `new` 就要想到 `delete`**：就像借书一定要还。
2.  **数组配套**：`new[]` 必须对应 `delete[]`。
3.  **首选初始化列表**：除了性能好，还能处理 `const` 和 `引用`。
4.  **置空指针**：`delete` 之后随手 `p = nullptr;` 能救命。
