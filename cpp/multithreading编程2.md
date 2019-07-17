# C++11的多线程开发2

## 向线程传值

### 线程对象的参数传值默认是复制的。

```c++
#include <iostream>
#include <string>
#include <thread>
void threadCallback(int x, std::string str)
{
    std::cout<<"Passed Number = "<<x<<std::endl;
    std::cout<<"Passed String = "<<str<<std::endl;
}
int main()  
{
    int x = 10;
    std::string str = "Sample String";
    std::thread threadObj(threadCallback, x, str);
    threadObj.join();
    return 0;
}
```

### 不要传递本地堆栈的变量地址给线程函数。

### 同时小心使用堆区的变量传递给线程函数，

可能存在被意外释放的问题

### 传引用

## Source

