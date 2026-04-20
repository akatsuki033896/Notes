- 具体类
- 抽象类
- RAII和虚函数的基础知识

# 具体类

定义：具体类的成员变量是其定义的一部分，例如复数类 `complex`

```c++
class complex {
    double real, imag; // 成员变量
public:
    complex() : real{0}, imag{0} {} // 默认构造函数
    complex(complex z) : real{z.real}, imag{z.imag} {} // 拷贝构造函数
    complex(double r, double i) : real{r}, imag{i} {} // 用实部和虚部构造该复数
    complex(double r) : real{r} {} // 用实部构造该复数
    
    // 获取成员变量
    double get_real() const {	return real;	}
    double get_imag() const {	return imag;	}
    
    // 给成员变量赋值
    void get_real(double d)	{	real = d;	}
    void get_imag(double d)	{	imag = d;	}
    
    // 复数操作 运算符重载
    complex& operator+=(complex	z) {
        real += z.real;
        imag += z.imag;
        return *this;
    }
}
```

## 成员权限

### 私有 `private`

类不声明，默认都是私有

类的成员变量可以限定为 `private`，即只能通过成员函数访问，一旦发生明显改动就需要重新编译整个程序。

想要提高灵活性可以将成员变量的主要部分放在动态内存或堆中，然后通过存储在类对象内部的成员访问他们，例如 `vector` 和 `string`，相当于有精致接口的资源管理器

### 公有 `public`

结构体不声明默认公有

## 资源管理

> 构造函数和析构函数构成数据句柄模型，管理一些在对象生命周期大小会发生变化的数据
>
> 资源获取即初始化(RAII)：在构造函数获取资源，在析构函数释放资源，避免裸 `new` 操作即在代码中分配内存（同样要避免裸 `delete` 操作，防止资源泄漏）

以容器 `vector` 为例

```cpp
class Vector {
    double* elem;
    int sz;
public:
    Vector(int s) : elem{ new doube[s] }, sz{s} {
        for (int i = 0; i < s; i++) {
            elem[i] = 0;
        }
    } // 初始化
    ~Vector() {	delete[] elem;	} // 释放资源
}
```

### 构造函数

构造函数：为元素分配空间并正确的初始化类成员

默认情况下拷贝赋值函数和拷贝构造函数会由编译器隐式生成

默认构造函数：不需要实参就可以调用的构造函数，可以防止该类型的对象未被初始化

### 析构函数

析构函数确保构造函数分配的内存一定会被销毁，回收使用 `new` 分配的资源，是资源管理的基础

单独 `delete` 释放一个独立对象，`delete[]` 释放一个数组

## 内联函数

定义在类内部的函数默认是内联的，也可以在函数声明前面加上 `inline` 来显式指定内联

## `const` 成员函数

 `const` 成员函数表示函数不会修改所调用的对象，可以被 `const` 和非 `const` 对象调用

非 `const` 成员函数只能被非 `const` 对象调用

```cpp
complex z = {1, 0};
const complex cz = {1, 3};
cz = z; // 非const不能赋值给const
```

## 运算符重载

使用运算符重载的时候要尊重常规使用习惯，编译器不允许改变一个操作符操作内置类型的含义，例如不能重载 `+` 使得执行减法。(6.4)

# 抽象类

抽象类型把使用者和类的实现细节隔离，将接口和实现分开，实现上完全放弃纯局部变量，由于对抽象类的实现一无所知，所以必须从自由存储中为对象分配存储空间，再通过指针和引用访问对象。

例如设计一个容器类，对于之后定义的具体的容器而言，这个抽象类的用途只能作为接口

```cpp
class Container {
public:
    virtual double& operator[](int) = 0;
    virtual int size() const = 0;
    virtual ~Container() {}
}
```

## 虚函数

使用 `virtual` 定义，意味着这个函数可能在之后的派生类中被重新定义

`= 0` 说明是纯虚函数，意味着派生类**必须定义这个函数**，根据范例，我们不能单纯地定义 `Container c`，必须指定大小例如  `Container *p = new Vector_container(10)`，只有纯虚函数的类就是抽象类

抽象类的用途只是接口，它的派生类负责具体实现成员函数 `operator[]()` 和 `size()`

>很多设计模式都和虚函数有关

## 资源管理和虚析构

因为不需要初始化数据，所以抽象类没有构造函数

但是抽象类要通过指针和引用来操作，通过指针销毁抽象类的对象的时候，我们不知道实现部分持有哪些资源，因此需要一个虚析构函数

## 抽象类的派生和继承

对 `Container` 类派生一个 `Vector_container` 类，成员存放定义好的 `Vector` 容器类

```cpp
class Vector_container : public Container {
	Vector v;
public:
    Vector_container(int s) : v(s) { }
    ~Vector_container() {}
    double& operator[](int i) override {
        return v[i];
    }
    int size() const override {
        return v.size();
    }
}
```

`:public` 意味着这个类是派生出来的，`Vector_container` 派生自 `Container`，而 `Container` 是 `Vector_container` 的基类，派生类从子类继承了成员

`override` 可用可不用，显式指定允许编译器捕捉错误，例如虚函数名称和类型声明错误，在大型的类层次中很有用

派生类的析构函数会覆盖基类的析构函数

## 多态类型和抽象类的使用

例如输出容器中元素

```cpp
void use(Container& c) {
    const int sz = c.size();
    for (int i = 0; i < sz; i++) {
        std::cout << c[i] << '\n';
    }
}
```

可见该函数在完全忽视类型的情况下使用了 `Container()` 作为接口，一个为其他类型提供接口的类就是**多态类型**

## 虚函数表(vtbl)

基类 `Container` 的对象包含一些有助于在运行时选择正确函数的信息

虚函数表是一种存放函数指针的表，编译器将虚函数的名字转换成表中对应的索引值，每个含虚函数的类都有自己一个虚函数表，用来辨识虚函数。即使调用函数不知道对象的大小和布局，虚函数表中的函数也能确保对象被正确使用，调用函数的实现只要知道 `Container` 中 `vtbl` 指针的位置和虚函数对应的索引

- 时间效率接近普通函数调用机制
- 空间开销对于一个类是一个虚函数表和调用每个对象时的指针（如果包含虚函数）

![virtual_function.png](https://s2.loli.net/2026/01/09/WfarZURThsYAN5j.png)
