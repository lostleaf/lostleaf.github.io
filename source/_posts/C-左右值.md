---
title: C++左右值与右值引用
date: 2019-10-08 13:08:56
tags:
- C++
---
总结左右值的定义，和 C++11 中右值引用的用途
<!-- More -->

## 左值与右值

C++ 中，左右值的简化定义:
- 左值：占用了一定内存，且拥有可辨认的地址的对象
- 右值：左值以外的所有对象

典型的左值，C++中绝大部分的变量都是左值
```C++
int i = 1;   // i 是左值
int *p = &i; // i 的地址是可辨认的
i = 2; // i 在内存中的值可以改变

class A;
A a; // 用户自定义类实例化的对象也是左值
```

典型的右值
```C++
int i = 2; // 2 是右值
int x = i + 2; // (i + 2) 是右值
int *p = &(i + 2); // 错误，不能对右值取地址
i + 2 = 4; // 错误，不能对右值赋值

class A;
A a = A(); // A() 是右值

int sum(int a, int b) {return a + b;}
int i = sum(1, 2) // sum(1, 2) 是右值
```

引用（左值引用）
```C++
int i;
int &r = i;
int &r = 5; // 错误，不能左值引用绑定右值

// const 引用是例外
const int &r = 5; // 可以认为指向一个临时左值的引用，临时左值为5
```

对于函数，类似地有
```C++
int square(int& a) {return a * a;}

int i = 2;
square(i); // 正确
square(2); // 报错，左值引用绑定右值

// 变通一下
int square(const int& a) {return a * a;} // 可以调用 square(2)
```

左值可以创造右值, 右值也可创造左值
```C++
int i = 2;
int x = i + 2; // i + 2 是右值

int v[3];
*(v + 1) = 2; // 指针解引用可以把右值转化为左值
```

一些注意事项
```C++
// 1. 函数也可以返回左值
int myglobal ;
int& foo() {return myglobal;}
foo() = 50;

array[3] = 50; // 一个更常见的例子，[]操作符几乎总是返回左值

// 2. 左值并不总是可修改的
const int i = 1;  // i是左值, 但不可修改

// 3. 右值有时是可修改的
class dog;
dog().bark();  // bark() 可能会修改 dog() 对象
```

## 右值引用
右值引用的两大用途：
- 移动语意
- 完美转发

### 移动语意
移动语意(std::move)，可以将左值转化为右值引用
```C++
int a = 1; // 左值
int &b = a; // 左值引用
int &&c = std::move(a); // 右值引用，移动语意

void printInt(int& i) { cout << "lvalue reference: " << i << endl; }
void printInt(int&& i) { cout << "rvalue reference: " << i << endl; } 

int main() {
   int i = 1;
   printInt(i);   // 调用 printInt(int&), i是左值
   printInt(6);   // 调用 printInt(int&&), 6是右值
   printInt(std::move(i));   // 调用 printInt(int&&)，移动语意
}
```

由于编译器调用时无法区分 `printInt(int) 与 printInt(int&)` 以及 `printInt(int) 与 printInt(int&&)`，如果再定义 printInt(int) 函数，会报错

为什么需要移动语意
```C++
class myVector {
    int size;
    double* array;
public:
	myVector(const myVector& rhs) {  // 复制构造函数
        std::cout << "Copy Constructor\n";
        size = rhs.size; 
        array = new double[size];
        for (int i=0; i<size; i++) { array[i] = rhs.array[i]; }
    }

    myVector(int n) {
        size = n;
        array = new double[n];
    }
};
void foo(myVector v) {/* Do something */}
myVector createmyVector();  // Creates a myVector

int main() {
    myVector reusable = createmyVector();
    // 这里会调用 myVector 的复制构造函数，如果我们不希望 foo 乱动 reusable，这种情况下是 ok 的
    foo(reusable); 
    /* Do something else with reusable */

    // 这里 createmyVector 会返回一个临时的右值，传参过程中会多余地被复制一次
    // 虽然大部分情况下会被编译器优化掉
    foo(createmyVector());
}
```

解决方法, 添加一个移动构造函数
```C++
myVector(myVector&& rhs) {  // 移动构造函数
    std::cout << "Move Constructor\n";
    size = rhs.size; 
    array = rhs.array;
    rhs.size = 0;
    rhs.array = nullptr;
}
```

在 C++03 中，可能需要定义两个 foo 函数来解决这种问题：`foo_by_value(myVector)` 和 `foo_by_ref(myVector&)`

但是，假如我们在调用 foo 之后，reusable 就没用了, 可以使用移动语意
```C++
int main () {
    myVector reusable = createmyVector();
    // 这里会调用 myVector 的移动构造函数
    foo(std::move(reusable));
    /* No use of reusable anymore */
}
```

另一个有用之处在于重载赋值运算符
```C++
X& X::operator=(X const & rhs); 
X& X::operator=(X&& rhs);
```

自从 C++11，所有的 STL 都实现了移动构造