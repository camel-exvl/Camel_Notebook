---
title: C++
status: new
---
# C++

??? abstract "TODO-List"

    咕咕咕 有空再看在写吧 细节上的话边用边写吧

    - [ ] STL
    - [x] 智能指针
    - [ ] 泛型
    - [ ] 移动语义
    - [ ] 匿名函数
    - [x] `constexpr`
    - [x] 引用限定符
    - [ ] 并发
    - [x] 编程习惯

## STL

### 比较函数

`greater<>`实现`x > y`，`less<>`实现`x < y`

注意`sort`和`priority_queue`的比较函数的区别。

```cpp
sort(v.begin(), v.end(), greater<int>());    // 降序排列
priority_queue<int, vector<int>, greater<int>> pq; // 小根堆
```

### Algorithm

#### `upper_bound`和`lower_bound`

二分查找，默认比较函数为`<`，要求数组排序方式和比较函数一致。

`upper_bound(first, last, value(, comp))`

!!! note "自定义比较函数"

    ```cpp
    // for upper_bound
    bool comp(const int &value, const int &element) { // 第一个参数始终为value
        return a > b;
    }
    // for lower_bound
    bool comp(const int &element, const int &value) { // 第二个参数始终为value
        return a > b;
    }
    ```

`upper_bound`返回第一个迭代器使得`value < *it`或`comp(value, *it) == true`或`*it == end`

`lower_bound`返回第一个迭代器使得`value <= *it `或`comp(value, *it) == false`或`*it == end`

```cpp
vector<int> v{1, 2, 3, 4, 5, 6, 7, 8, 9};   // 升序
int i = upper_bound(v.begin(), v.end(), 5) - v.begin(); // i = 5 第一个大于5的数的下标
int j = lower_bound(v.begin(), v.end(), 5) - v.begin(); // j = 4 第一个大于等于5的数的下标
// 自定义比较函数
vector<int> v2{9, 8, 7, 6, 5, 4, 3, 2, 1};   // 降序
int i2 = upper_bound(v2.begin(), v2.end(), 5, greater<int>()) - v2.begin(); // i2 = 4 第一个小于5的数的下标
int j2 = lower_bound(v2.begin(), v2.end(), 5, greater<int>()) - v2.begin(); // j2 = 5 第一个小于等于5的数的下标
```

### `string`

#### `string`转`char*`

`string.c_str()`返回`const char*`，可用`const_cast<char*>(string.c_str())`转换为`char*`，或利用`strcpy`/`std::copy`等函数复制到`char*`数组中。

#### 数字字符串

`atoi`可将`const char*`转换为`int`，`stoi`可将`string`转换为`int`。其中`atoi`不做边界检查，`stoi`会抛出异常。

#### 字符串分割

c++没有提供`split`之类的函数，可以使用`stringstream`或`strtok`实现。

```cpp
// stringstream
vector<string> split(const string &s, char delim) {
    vector<string> elems;
    stringstream ss(s);
    string item;
    while (getline(ss, item, delim)) {
        elems.push_back(item);
    }
    return elems;
}

// strtok
vector<string> split(const string &s, char delim) {
    vector<string> elems;
    char *p = strtok(const_cast<char*>(s.c_str()), delim);
    while (p) {
        elems.push_back(p);
        p = strtok(nullptr, delim);
    }
    return elems;
}
```

## 智能指针

### `std::unique_ptr`

`std::unique_ptr`是一个独占指针，不能被复制，但可以被移动。其大小等同于原始指针，大多数操作与原始指针相同。

### `std::shared_ptr`

`std::shared_ptr`是一个共享指针，其内部维护了一个引用计数，当引用计数为0时，会自动释放内存。这意味着性能问题：`std::shared_ptr`大小通常是原始指针的两倍。

### `std::weak_ptr`

`std::weak_ptr`是一个弱指针，协助`std::shared_ptr`工作，只能从`std::shared_ptr`或`std::weak_ptr`构造，不会引起引用计数的变化。

`std::weak_ptr`能够判断指针是否悬空，但无法解引用。

### 构造智能指针

使用`std::make_shared`和`std::make_unique`构造智能指针，相比`new`而言能够减少代码重复、提升异常安全性，有着一定的优势。

## 泛型

## 移动语义

## 匿名函数

```cpp
[capture] (parameters) mutable exception attribute -> return_type { body }
```
```cpp
[]        // 沒有定义任何变量，但必须列出空的方括号。在Lambda表达式中尝试使用任何外部变量都会导致编译错误。
[x, &y]   // x是按值传入，y是按引用传入。
[&]       // 任何被使用到的外部变量都按引用传入。
[=]       // 任何被使用到的外部变量都按值传入。
[&, x]    // x按值传入。其它变量按引用传入。
[=, &z]   // z按引用传入。其它变量按值传入。
```

## `constexpr`

被`constexpr`修饰的值表明其在编译时就可以确定，可以在编译时优化。`const`对象并不需要在编译时确定初值，只是在运行时不可修改。

简而言之所有`constexpr`对象都是`const`，但不是所有`const`对象都是`constexpr`。

## 引用限定符

在成员函数的后面添加 `&` 或 `&&`，从而限制调用者的类型为左值或右值。

```cpp
class Widget {
public:
    void doWork() &;    // 只有*this为左值的时候才能被调用
    void doWork() &&;   // 只有*this为右值的时候才能被调用
}; 

Widget makeWidget();    // 工厂函数（返回右值）
Widget w;               // 普通对象（左值）

w.doWork();             // 调用被左值引用限定修饰的Widget::doWork版本
makeWidget().doWork();  // 调用被右值引用限定修饰的Widget::doWork版本
```

使用引用限定符修饰const函数时，const需位于引用限定符之前。

```cpp
class Widget {
public:
    void doWork() const &;  // *this既可为左值也可为右值
    void doWork() &&;       // 只有*this为右值的时候才能被调用
};
```

## 并发

## 编程习惯

以下内容来自[《Effective Modern C++》](https://github.com/CnTransGroup/EffectiveModernCppChinese)，具体原因请参考原书，此处仅做简单整理。

### 优先考虑`auto`而非显式类型声明

`auto`变量必须初始化，避免移植和效率问题，重构更方便。

### 优先考虑`nullptr`而非`0`和`NULL`

`nullptr`类型明确，不会被隐式转换，不会被误用。

### 避免重载指针和整型

空指针NULL存在歧义，事实上使用空指针调用会调用整型重载。
    
```cpp
void f(int);
void f(bool);
void f(void*);

f(0);               // 调用f(int)而不是f(void*)
f(NULL);            // 一般来说调用f(int)，但绝对不会调用f(void*)
```

### 优先考虑别名声明而非`typedef`

通常情况下两者等价，但部分场景下别名更方便，且别名支持模板化。
    
```cpp
typedef void (*FP)(int, const std::string&);

using FP = void (*)(int, const std::string&);
```

### 优先考虑scoped `enum`而非unscoped `enum`

1. scoped `enum` 不会导致命名空间污染
    
    === "unscoped"

        ```cpp
        enum Color {black, white, red};
        auto white = false;     // 错误，white已经被定义
        ```
        
    === "scoped"

        ```cpp
        enum class Color {black, white, red};
        auto white = false;     // 正确
        auto c = Color::white;  // 也正确
        ```

2. scoped `enum`中的枚举名是强类型，不会被隐式转换

    === "unscoped"

        ```cpp
        enum Color {black, white, red};
        auto c = red;
        if (c < 14.5) { // 正确，c被隐式转换为double
            // ...
        }
        ```
        
    === "scoped"

        ```cpp
        enum class Color {black, white, red};
        auto c = Color::red;
        if (c < 14.5) { // 错误，不能比较Color和double
                // ...
        }
        ```
        
3. scoped `enum`可以前置声明

    ```cpp
    enum class Color;                   // 前置声明
    enum class Color : std::uint32_t;   // 默认底层类型为int，可以重写
    ```
    
### 优先考虑使用`delete`函数而非未定义的`private`声明

`delete`函数可以在编译时报错，`private`声明只能在链接时报错。

使用时，`delete`函数通常被声明为`public`，否则在非法调用`delete`函数时一些编译器可能只会报`private`错误而忽略`delete`错误。

### 优先考虑`const_iterator`而非`iterator`

### 如果函数不抛出异常请使用`noexcept`

### 尽可能地使用[`constexpr`](#constexpr)

### 让`const`成员函数线程安全

`const`成员函数意味着其为读操作，应当支持并发读取，除非能够**确定**该函数永远不会被并发调用。

## 参考资料

[Effective Modern C++](https://github.com/CnTransGroup/EffectiveModernCppChinese)