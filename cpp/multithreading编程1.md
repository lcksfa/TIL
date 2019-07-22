# C++11的多线程开发

## 引入线程库头文件

`#include <thread>`

## 在`std::thread`的构造函数接收

1. 函数指针
2. 函数对象
3. lambda表达式

`std::thread thObj(<CallBack>)`



任何一个线程会等待另一个退出,通过调用`join`函数,
### 使用函数指针

```c++
#include <iostream>
#include <thread>

//use function pointer
void thread_function()
{
    for(int i = 0; i < 10000; i++);
        std::cout<<"thread function Executing"<<std::endl;
}

int main()  
{
std::thread threadObj(thread_function);
for(int i = 0; i < 10000; i++);
    std::cout<<"Display From MainThread"<<std::endl;
threadObj.join();    
std::cout<<"Exit of Main function"<<std::endl;
return 0;
}
```

### 使用函数对象:
```c++
#include <iostream>
#include <thread>
class DisplayThread
{
public:
    void operator()()     
    {
        for(int i = 0; i < 10000; i++)
            std::cout<<"Display Thread Executing"<<std::endl;
    }
};
 
int main()  
{
    std::thread threadObj( (DisplayThread()) );
    for(int i = 0; i < 10000; i++)
        std::cout<<"Display From Main Thread "<<std::endl;
    std::cout<<"Waiting For Thread to complete"<<std::endl;
    threadObj.join();
    std::cout<<"Exiting from Main Thread"<<std::endl;
    return 0;
}
```

### 使用lambda表达式
```c++
#include <thread>
int main()  
{
    int x = 9;
    std::thread threadObj([]{
            for(int i = 0; i < 10000; i++)
                std::cout<<"Display Thread Executing"<<std::endl;
            });
            
    for(int i = 0; i < 10000; i++)
        std::cout<<"Display From Main Thread"<<std::endl;
        
    threadObj.join();
    std::cout<<"Exiting from Main Thread"<<std::endl;
    return 0;
}
```

## 线程区分

每个线程都有ID,`std::thread::get_id()`

获取当前线程的id,使用`std::this_thread::get_id()`





## join 和detach

### join 

等待线程完成

### detach

将线程分离

### 注意点

1. **永远不要在没有关联的执行线程的std :: thread对象上调用join（）或detach（）**

当在一个线程对象上调用join（）函数时，那么当这个连接时（返回0,那个std :: thread对象没有与之关联的线程。如果在这样的对象上再次调用join（）函数，那么它将导致终止程序。detach与之同理.

因此,我们在调用join或者detach前,需要判断线程 是否可以被join或者detach;

使用` joinable`判断.

2. **永远不要忘记在具有相关执行线程的std :: thread对象上调用join或detach**

如果使用具有关联执行线程的std :: thread对象调用join或detach，则在该对象的析构期间 - 或者它将终止程序。因为在析构内部 - 或者它检查线程是否仍然可加入，然后终止程序，即

```c++
#include <iostream>
#include <thread>
#include <algorithm>
class WorkerThread
{
public:
    void operator()()     
    {
        std::cout<<"Worker Thread "<<std::endl;
    }
};
int main()  
{
    std::thread threadObj( (WorkerThread()) );
    // Program will terminate as we have't called either join or detach with the std::thread object.
    // Hence std::thread's object destructor will terminate the program
    return 0;
}
```

使用RAII解决这个问题:

```c++
#include <iostream>
#include <thread>
class ThreadRAII
{
    std::thread & m_thread;
    public:
        ThreadRAII(std::thread  & threadObj) : m_thread(threadObj)
        {
            
        }
        ~ThreadRAII()
        {
            // Check if thread is joinable then detach the thread
            if(m_thread.joinable())
            {
                m_thread.detach();
            }
        }
};
void thread_function()
{
    for(int i = 0; i < 10000; i++);
        std::cout<<"thread_function Executing"<<std::endl;
}
 
int main()  
{
    std::thread threadObj(thread_function);
    
    // If we comment this Line, then program will crash
    ThreadRAII wrapperObj(threadObj);
    return 0;
}
```



## Source

[C++11 Multithreading – Part 1 : Three Different ways to Create Threads](https://thispointer.com/c-11-multithreading-part-1-three-different-ways-to-create-threads/)

[C++11 Multithreading – Part 2: Joining and Detaching Threads][https://thispointer.com//c11-multithreading-part-2-joining-and-detaching-threads/]

