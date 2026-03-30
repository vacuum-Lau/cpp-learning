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
