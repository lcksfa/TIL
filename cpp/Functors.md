# c++ Functor

> be As a C++ enthusiast

A **functor** (or function object) is a **C++** class that acts like a function. **Functors** are called using the same old function call syntax. To create a **functor**, we create a object that overloads the operator(). ... Thus, an object a is created that overloads the operator().

- warp a stl such as sort callback function ,as a () operator

```c++
#include <iostream>
#include <cassert>
#include <vector>
#include <algorithm>
using namespace std;

struct Add_X {
    int _x;
    int operator()(int y) { return _x + y; }
    Add_X(int x) : _x(x) {}
};
int main(int argc, char const *argv[]) {
    /* code */
    // usage 1
    Add_X _42Add(42);
    int is50 = _42Add(8);
    assert(50 == is50);
    cout << _42Add(8) << endl;

    // usage2
    std::vector<int> vint{1, 2, 3, 4, 5};
    std::vector<int> vout(vint.size());

    std::transform(vint.begin(), vint.end(), vout.begin(), Add_X(2));
    for (auto &it : vout) {
        cout << it << ","
             << " ";
    }
    return 0;
}
```

这里有两个比函数优秀的地方，其一是它可以记录状态，_42Add对象就是一例，它记录了一个42的成员数据，当我们使用这个对象实例时，调用括号表达式即完成加法运算；

其二就是作为回调函数使用，在transform函数中，使用`Add_X(2)`即完成一个可以被回调的函数，

使用lambda达到同样的效果；需要如下：

```c+
 std::transform(vint.begin(), vint.end(), vout.begin(), [](int n) { return n += 2; });
```

那么继续回头理解这里的Add_X(2)，这里构造了一个Add_X的实例，其成员_x_为2；_当transform回调时，调用方式应该是调用Add_X的括号表达式，由`vint`迭代器传回的每一个int类型数值作为Add_X的y进行加法运算，而后返回加法的结果；

这样看来，functor也是有它的简洁之处的。

- do something functor ,warp the most argument
- use in template while match more data type

```c++

```

