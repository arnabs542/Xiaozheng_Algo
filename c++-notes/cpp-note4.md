# 细节
_update May 3,2018  3:33_

---

## 1. `emplace_back` vs `push_back`
#### 官方文档
首先看官方文档：[cplusplus.com](http://www.cplusplus.com/reference/vector/vector/emplace_back/)  
摘要：
> Inserts a new element at the end of the vector, right after its current last element. This new element is constructed in place using args as the arguments for its constructor.  
> The element is constructed in-place by calling `allocator_traits::construct` with args forwarded. A similar member function exists, `push_back`, which either copies or moves an existing object into the container.

可以看出，`emplace` 用于直接传入constructor 需要的参数，inplace地构造对象。而 `push_back` 则需要将对象 copy or move 入容器。

[这里](https://stackoverflow.com/questions/10890653/why-would-i-ever-use-push-back-instead-of-emplace-back?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa) 有一个stackoverflow的讨论，感觉讲得不错。
[这篇文章](http://blog.guorongfei.com/2016/03/16/cppx-stdlib-empalce/) 讲了 `emplace` 和 `insert` ，感觉讲的很好。

**Example:**
```cpp
  class Test {
  public:
    int a;
  
    explicit Test(int a) {
      this->a = a;
      cout << "constructor: a = " << a << endl;
    }
  
    Test(const Test& that) {
      cout << "copy constructor, a = " << that.a << endl;
      this->a = that.a;
    }
  
    Test(Test&& that) noexcept {
      cout << "move constructor, a = " << that.a << endl;
      this->a = that.a;
    }
  };
  
  int main() {
    Test t1{3};

    vector<Test> v;
    v.emplace_back(2); // 即使构造函数是 explicit，仍然可以隐式调用，只call一次constructor, no copy，no move
    
    v.emplace_back(Test(2)); // call 一次 constructor，一次 move
    
    v.push_back(2); // 如果构造函数 Test(int a) 声明为 explicit，则报错，因为无法隐式调用
    
    v.push_back(Test(2)); // call 一次 constructor，一次move
    
    v.emplace_back(2.5); // no warning, create object of Test with a == 2
    
    v.push_back(2.5); // 如果构造函数非explicit，会有warning，"implicit conversion from 'double' to 'int'"
                      // 如果构造函数explicit，同样会报错。
    
    return 0;
  }
  
```

从上面的例子可以看出，通过直接传入构造参数的方法调用 `emplace_back()`，只需要调用一次 constructor，不需要调用 move constructor，这是一个优势。但同时，当我们向 int 参数传入 float 的时候，`emplace_back()` 并不会报warning，不如`push_back()` 安全。

<br>

---

## 2. Integer Literal Type
#### 问题产生
在Java中，如果我判断 `if (a < 0x80000000)`，this is equivalent to `if (a < Integer.MIN_VALUE)` and `if (a < -2147483648)`。但在 c++ 中，`0x80000000` 却被当做了 usigned int，等于 `2147483648`，如果需要实现和Java中一样的效果，需要手动转换为int：`(int)0x80000000`。

#### 问题解释
[官方解释](http://en.cppreference.com/w/cpp/language/integer_literal)，摘要如下：

1. 十进制，十六进制，八进制，二进制的 integer literal 所自动fit的type是从int开始顺序匹配的，例如这里出现的问题就是因为 `0x80000000` 被 fit 到了 ul 类型.
2. 在 Integer Literal 中使用 `负号 -` 的时候需要特别注意，可能会带来 implicit type conversions.

#### 解决
使用 `(int)0x80000000` 或者 `int(0x80000000)` 进行 type conversion，这是 c++ 中两种 type conversion 的格式。

<br>

---

















