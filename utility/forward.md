#转发（std::forward）

定义于头文件`<utility>`中：

```C++
template< typename T >
T&& forward( typename std::remove_reference<T>::type& t ) noexcept ;              (1)     (C++11 - C++14)
```

```C++
template< typename T >
constexpr T&& forward( typename std::remove_reference<T>::type& t ) noexcept ;    (1)     (C++14 - )
```

```C++
template< typename T >
T&& forward( typename std::remove_reference<T>::type&& t ) noexcept ;             (2)     (C++11 - C++14)
```

```C++
template< typename T >
constexpr T&& forward( typename std::remove_reference<T>::type&& t ) noexcept ;   (2)     (C++14 - )
```

在函数模板中按下面的方式使用`forward`调用函数时，转发传递过来的实参及其**值类型**（[value category](../language/value_category.md)）给另外一个函数。

比如，如果`wrapper`函数代码如下，该模板将按下面描述的那样实例化：

```C++
template<class T>
void wrapper(T&& arg)
{
    foo(std::forward<T>(arg)); // Forward a single argument.
}
```

- 当传递给`wrapper`的是`string`类型的右值时，`T`会被推演为`string`（而不是`string&`，`const string&`，或`string&&`），同时`forward`会确保传递给函数`foo`的是一个右值。
- 当传递给`wrapper`的是`const string`类型左值时，`T`会被推演为`const string&`，同时`forward`会确保传递给函数`foo`的是一个常量左值引用。
- 当传递给`wrapper`的是`string`类型左值时，`T`会被推演为`string&`，同时`forward`会确保传递给函数`foo`的是一个非常量左值引用。

##注意

试图把右值当作左值转发，将会产生一个编译时错误，如用左值引用类型`T`实例化声明（2）就是如此。

`T&&`作为函数形参的特殊规则 —— C++11新引入了“引用折叠”（`reference collapsing`），请参考[模板实参推演](templates/template_argument_deduction.md)。

##参数

`t` -   待转发的对象

##返回值

`static_cast<T&&>(t)`

##异常

`noexcept`指定： `noexcept`

即不允许跑出异常。

##例子

下面是一个将参数完美转发给`T`类型的构造函数实参的例子。同时也展示了可变参数的完美转发。

```C++
#include <iostream>
#include <memory>
#include <utility>
#include <array>

struct A {
    A(int&& n) { std::cout << "rvalue overload, n=" << n << "\n"; }
    A(int& n)  { std::cout << "lvalue overload, n=" << n << "\n"; }
};

class B {
public:
    template<class T1, class T2, class T3>
    B(T1&& t1, T2&& t2, T3&& t3) :
        a1_{std::forward<T1>(t1)},
        a2_{std::forward<T2>(t2)},
        a3_{std::forward<T3>(t3)}
    {
    }

private:
    A a1_, a2_, a3_;
};

template<class T, class U>
std::unique_ptr<T> make_unique1(U&& u)
{
    return std::unique_ptr<T>(new T(std::forward<U>(u)));
}

template<class T, class... U>
std::unique_ptr<T> make_unique(U&&... u)
{
    return std::unique_ptr<T>(new T(std::forward<U>(u)...));
}

int main()
{
    auto p1 = make_unique1<A>(2); // rvalue
    int i = 1;
    auto p2 = make_unique1<A>(i); // lvalue

    std::cout << "B\n";
    auto t = make_unique<B>(2, i, 3);

    return 0;
}
```

输出为：

```C++
rvalue overload, n=2
lvalue overload, n=1
B
rvalue overload, n=2
lvalue overload, n=1
rvalue overload, n=3
```

##复杂度

常量级

##另见

- [move](move.md)（C++11）                            强制转换为右值引用（函数模板）
- [move_if_noexcept](move_if_noexcept.md)（C++11）    当移动构造函数未抛出异常时强制转换为右值引用（函数模板）
