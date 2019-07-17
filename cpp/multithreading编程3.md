## 异步

[参考](https://putridparrot.com/blog/threads-promises-futures-async-c/)

## Future

## Promise

## async



code 1:

``` c++
#include <iostream>
#include <future>

using namespace std;

int factorial(int N) {
    int res = 1;
    for (int i = N; i > 1; i--) {
        res *= i;
    }

    cout << "Result is " << res << endl;
    return res;
}

int main() {
    std::thread t1(factorial, 4);
    t1.join();
    return 0;
}
```

如果我们想要将参数设为结果，可以使用`std::ref(x)`将参数射入；

code 2:

```c++
#include <iostream>
#include <future>
#include <mutex>

std::mutex mu;
std::condition_variable cond;
using namespace std;

int factorial(int N) {
    int res = 1;
    for (int i = N; i > 1; i--) {
        res *= i;
    }

    cout << "Result is " << res << endl;
    return res;
}

int main() {
    int x = 0;
    //std::future<int> fu = std::async(factorial, 4);
    std::future<int> fu = std::async(std::launch::async, factorial, 4);

    x = fu.get();

    cout << "x = " << x << endl;
    return 0;
}
```







