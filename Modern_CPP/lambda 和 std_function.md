现代 C++ 最重要的特性之一，Lambda 提供类似匿名函数的特性。匿名函数的使用场景是需要一个函数，但是又不想费力去命名一个函数的情况。

函数可以作为另一个函数的参数，且这个作为参数的函数也可以有参数例如 `void func_2(void func_1(int))`，函数类型还可以作为模板参数 `template <class Func>`，但是要在全局添加函数

# lambda表达式(C++11)

本质是语法糖。

## 语法

lambda表达式可以在函数体内创建一个函数，不需要在全局添加函数，基本语法为：

```cpp
[捕获列表](参数列表) mutable(可选) 异常属性 -> 返回类型 {  
	// 函数体 
}
```

- 捕获列表：可以理解为参数的一种类型，Lambda 表达式内部函数体在默认情况下是不能够使用函数体外部的变量的， 这时候捕获列表可以起到传递外部数据的作用。捕获列表分为值捕获，引用捕获，隐式捕获，表达式捕获
- 参数列表：类似函数的参数列表
- 返回类型：在参数列表后 `-> 类型` 添加返回类型，不指定时和 `-> auto` 等价，没有 `return` 时和 `-> void` 等价。

```cpp
template<class Func>
void call_twice(Func func) {
  std::cout << func(0) << '\n';
  std::cout << func(1) << '\n';
}
int main() {
  auto twice = [](int n) -> int{
    return n * 2;
  };
  call_twice(twice);
  return 0;
}
```

## 值捕获

与参数传值类似，值捕获的前提是变量可以拷贝，不同之处则在于，被捕获的变量在 Lambda 表达式被创建时拷贝， 而非调用时才拷贝

```cpp
void lambda_value_capture() {  
    int value = 1;  
    auto copy_value = [value] {  
        return value;  
    };  
    value = 100;  
    auto stored_value = copy_value();  
    std::cout << "stored_value = " << stored_value << std::endl;  
    // 这时, stored_value == 1, 而 value == 100.  
    // 因为 copy_value 在创建时就保存了一份 value 的拷贝  
}
```
## 引用捕获

与引用传参类似，引用捕获保存的是引用，值会发生变化。

```cpp
void lambda_reference_capture() {  
    int value = 1;  
    auto copy_value = [&value] {  
        return value;  
    };  
    value = 100;  
    auto stored_value = copy_value();  
    std::cout << "stored_value = " << stored_value << std::endl;  
    // 这时, stored_value == 100, value == 100.  
    // 因为 copy_value 保存的是引用  
}
```

## 隐式捕获

手写捕获列表很复杂，可以直接在捕获列表中写一个 `[&]` 或 `[=]` 向编译器声明采用引用捕获或者值捕获
### `[&]` 捕获引用变量

lambda函数体中可以使用定义他的 `main` 函数中的变量，把 `[]` 改成 `[&]`，与引用传参类似，引用捕获保存的是引用。

```cpp
#include <iostream>

template <class Func> void call_twice(Func func) {
  std::cout << func(0) << '\n';
  std::cout << func(1) << '\n';
}
int main() {
  int fac = 2;
  auto twice = [&](int n) -> int { return n * fac; }; // fac 是 main 函数定义的变量
  call_twice(twice);
  return 0;
}
```

`[&]` 还可以写入 `main()` 中的变量

```cpp
int main() {
  int fac = 2;
  int cnt = 0;
  auto twice = [&](int n) -> int { 
    cnt++;
    return n * fac;
  };
  call_twice(twice);
}
```

#### 传入常引用避免拷贝开销

将模板参数声明为 `const&` 避免不必要的拷贝，传参时只传入内存地址，不用重新创建一个对象

```cpp
template<class Func>
void call_twice(Func const& func) {
   ...
}
```

#### 函数作为返回值

```cpp
auto make_twice(int fac) {
  return [](int n) {
    return n * 2;
  };
}
int main() {
  auto twice = make_twice();
  call_twice(twice);
  return 0;
}
```

然而使用 `return [&](int n) {return n * fac;};` 捕获定义域 `make_twice` 的 `fac` 时出错，因为 `[&]` 捕获的是引用即 `fac` 的地址，但是 `make_twice` 已经返回了，导致 `fac` 的引用变成了一块已经失效的地址，因此要保证lambda对象生命周期不超过他捕获的所有引用的寿命

###  `[=]` 值捕获变量

`[=]` 捕获的是值而不是引用，给每个引用的变量做一份拷贝，性能可能不如 `[&]`，上面的例子改成 `return [=](int n) {return n * fac;};` 即可

### `[]` 捕获变量

Lambda 表达式的本质是一个和函数对象类型相似的类类型（称为闭包类型）的对象（称为闭包对象）， 当 Lambda 表达式的捕获列表为空时，闭包对象还能够转换为函数指针值进行传递。

> 闭包(*closure*)
>
> 函数式编程的特性，指的是函数可以引用定义位置所有的变量


- `[]` 空捕获列表
- `[name1, name2, ...]` 捕获一系列变量
- `[&]` 引用捕获, 从函数体内的使用确定引用捕获列表
- `[=]` 值捕获, 从函数体内的使用确定值捕获列表

# `std::function`：避免用模板参数 (C++11)

`std::function` 是一种通用、多态的函数封装， 它的实例可以对任何可以调用的目标实体进行存储、复制和调用操作， 它也是对 C++ 中现有的可调用实体的一种类型安全的包裹（相对来说，函数指针的调用不是类型安全的）， 换句话说，就是函数的容器。当我们有了函数的容器之后便能够更加方便的将函数、函数指针作为对象进行处理。

采用了**类型擦除技术**，无需写明仿函数类的具体类型，能容纳任何仿函数或函数指针。只需在模板参数中写明函数的参数和返回值类型即可，所有具有同样参数和返回值类型的仿函数或函数指针都可以传入。

```cpp
#include <functional>
#include <iostream>

int foo(int para) {
    return para;
}

int main() {
    // std::function 包装了一个返回值为 int, 参数为 int 的函数
    std::function<int(int)> func = foo;
    
    int important = 10;
    std::function<int(int)> func2 = [&](int value) -> int {
        return 1+value+important;
    };
    std::cout << func(10) << std::endl;
    std::cout << func2(10) << std::endl;
}
```
## 使用案例

`<class Func>` 可以让编译器对每个不同的lambda生成一次，有助于优化，但有时候我们希望通过头文件分离声明和实现，这时不能用 `template class` 作为参数，为了灵活性可以用 `std::function`，尖括号内写 `<返回类型(参数列表)>` 

```cpp
#include <functional>
#include <iostream>

void call_twice(std::function<int(int)> const &func) {
    std::cout << func(0) << '\n';
    std::cout << func(1) << '\n';
}

std::function<int(int)> make_twice(int fac) {
    return [=] (int n) {
        return n * fac;
    };
}

int main() {
    auto twice = make_twice(2);
    call_twice(twice);
    return 0;
}
```

## 无捕获的lambda传为函数指针

lambda不捕获局部变量，为 `[]`，那么只需要用函数指针类型 `int(int)` 即可，最大的好处是用于一些只接收函数指针的C语言API比如 `pthread`

## 类型擦除

只要提供一个 `operator()` 就可以实现调用各种类型的函数，`std::array` 也是一个类型擦除的容器

# Media Link

`std::function` 的原理：
https://www.bilibili.com/video/BV1yH4y1d74e/?spm_id_from=333.337.search-card.all.click&vd_source=9d28f0e4734f1bde4c84c3169e3a429d

