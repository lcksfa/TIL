# std::packaged_task 的使用

## 定义

The class template std::packaged_task wraps any Callable target (function, lambda expression, bind expression, or another function object) so that it can be invoked asynchronously. Its return value or exception thrown is stored in a shared state which can be accessed through std::future objects.

>  类模板std :: packaged_task包装任何Callable目标（函数，lambda表达式，绑定表达式或其他函数对象），以便可以异步调用它。它返回的值或抛出的异常存储在一个共享状态，可以通过std :: future对象访问。

## 使用过程



### 异步调用

```c++
#include <iostream>
#include <future>
#include <functional>

using namespace std;

int factorial(int N)
{
	int res = 1;
	for (int i = N; i > 1; i--)
	{
		res *= i;
	}
	cout << "Result is: " << res << endl;
	return res;
}

int main(int argc, char **argv)
{
    auto tbind = std::bind(factorial,6);
	std::packaged_task<int()> t(tbind);
	//make a future from packaged_task
	std::future<int> fu = t.get_future();
	t();
	//future return
	int x = fu.get();
	cout << "x = " <<x << endl;

	return 0;
}
```

以上代码在linux上直接编译会发生崩溃，正确的编译是`g++ packagedtask.cpp -o packagedtask -pthread`

也就是说，必须加上`-pthread`，注意不是`-lpthread`。使用后者 依然崩溃。

`

Result is: 720

x = 720

`

### 线程调用

```c++
#include <iostream>
#include <future>
#include <functional>
#include <deque>
#include <algorithm>
#include <mutex>
#include <condition_variable>

using namespace std;

int factorial(int N)
{
	int res = 1;
	for (int i = N; i > 1; i--)
	{
		res *= i;
	}
	cout << "Result is: " << res << endl;
	return res;
}

std::deque<std::packaged_task<int()>> task_q;
//task_q is use in two threads ,
//wo should use mutex keep thread safe.
std::mutex mu;
std::condition_variable cond;
void thread_1(){
	std::packaged_task<int()> t;
	{
		std::unique_lock<std::mutex> locker(mu);
		cond.wait(locker,[](){
			return !task_q.empty();
		});
		t = std::move(task_q.front());	
		task_q.pop_front();
	}
	
	t();
}
int main(int argc, char **argv)
{
	std::thread t1(thread_1);
	std::packaged_task<int()> t(std::bind(factorial,6));
	std::future<int> fu = t.get_future();
	{
		std::lock_guard<std::mutex> locker(mu);
		task_q.push_back(std::move(t));////we must use move semantic here .
	}
	cond.notify_one();
	cout << "output in main get " << fu.get() <<endl;

	t1.join();
	return 0;
}

```





## Source

[学习视频](https://www.youtube.com/watch?v=FfbZfBk-3rI)

[崩溃的解决思路](https://stackoverflow.com/questions/41790363/c-core-dump-with-packaged-task)