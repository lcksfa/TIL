# 异步编程

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


[参考](https://putridparrot.com/blog/threads-promises-futures-async-c/)



### **std::future and std::promise**

Futures and promises are two sides of the same coin, so to speak. A promise allows us to return state from a thread. Whereas a future is for reading that returned state.

In other words, let’s think of it in this, fairly simple way – we create a promise that really says that at some time there will be a state change or return value from thread. The promise can be used to get a std::future, which is the object that gives us this return result.

So we’ve got the equivalent of a simple producer/consumer pattern

```c++
`auto` `promise = std::promise<std::string>();``auto` `producer = std::``thread``([&]``{``   ``// simulate some long-ish running task``   ``std::this_thread::sleep_for(std::chrono::seconds(5));``   ``promise.set_value(``"Some Message"``);``});` `auto` `future = promise.get_future();``auto` `consumer = std::``thread``([&]``{``   ``std::cout << future.get().c_str();``});` `// for testing, we'll block the current thread``// until these have completed``producer.join();``consumer.join();`
```

*Note: In the example above I’m simply using a lambda and passing reference to all variables in scope via the capture, using [&], as a capture list is required to access the promise and future outside of the lambdas.*

In this example code we simply create a promise and then within the first thread we’re simulating a long-ish running task with the *sleep_for* method. Once the task is complete we set_value on the promise which causes the future.get() to unblock, but we’ve jumped ahead of ourselves, so…

Next we get the future (get_future) from the promise and to simulate a a producer/consumer we spin up another thread to handle any return from the promise via the future.

In the consumer thread, the call to future.get() will block that thread until the promise has been fulfilled.

Finally, the code calls producer.join() and consumer.join() to block the main thread until both producer and consumer threads have completed.



**std::async**

The example for the promise/future combination to create a producer consumer can be simplified further using the std::async which basically deals with creating a thread and creating a future for us. So let’s see the code for the producer/consumer now with the std::async

```c++
`std::future<std::string> future = std::async([&]``{``   ``std::this_thread::sleep_for(std::chrono::seconds(5));``   ``return` `std::string(``"Some Message"``);``});` `auto` `consumer = std::``thread``([&]``{``   ``std::cout << future.get().c_str();``});` `consumer.join();`
```

In the above we’ve done away with the promise and the first thread which have both been replaced with the higher level construct, std::async. We’ve still got a future back and hence still call future.get() on it, but notice how we’ve also switch to returning the std::string and the future using the template argument to match the returned type.

We’re not actually sure if the code within the async lambda is run on a new thread or run on the calling thread, as the [async documentation](http://en.cppreference.com/w/cpp/thread/async) states…

*“The template function async runs the function f asynchronously (potentially in a separate thread which may be part of a thread pool) and returns a std::future that will eventually hold the result of that function call.”*

So the async method may run the code on a thread or deferred. Deferred meaning the code is run when we call the future’s get method in a “lazy” manner.

We can stipulate whether to run on a thread or as deferred using an std::async overload

```
`std::future<std::string> future = ``   ``std::async(std::launch::deferred, [&]``   ``{``   ``});`
```

In the above we’ve deferred the async call and hence it’ll be run on the calling thread, alternatively we could stipulate std::launch::async, see [std::launch](http://en.cppreference.com/w/cpp/thread/launch).

**What about exceptions?**

If an exception occurs in async method or the promise set_exception is called in our producer/consumer using a promise. Then we need to catch the exception in the future (call to get()). Hence here’s an example of the async code with an exception occuring

```
`std::future<std::string> future = std::async([&]``{``   ``throw` `std::exception(``"Some Exception"``);``   ``return` `std::string(``"Some Message"``);``});` `auto` `consumer = std::``thread``([&]``{``   ``try``   ``{``      ``std::cout << future.get().c_str();``   ``}``   ``catch``(std::exception e)``   ``{``      ``std::cout << e.what();``   ``}``});` `consumer.join();`
```

Here we throw an exception in the async lambda and when future.get is called, that exception is propagated through to the consumer thread. We need to catch it here and handle it otherwise it will leak through to the application and potentially crash the application if not handled elsewhere.





more code:



```c++
#include <iostream>
#include <future>
#include <thread>
 
int main()
{
    // future from a packaged_task
    std::packaged_task<int()> task([]{ return 7; }); // wrap the function
    std::future<int> f1 = task.get_future();  // get a future
    std::thread t(std::move(task)); // launch on a thread
 
    // future from an async()
    std::future<int> f2 = std::async(std::launch::async, []{ return 8; });
 
    // future from a promise
    std::promise<int> p;
    std::future<int> f3 = p.get_future();
    std::thread( [&p]{ p.set_value_at_thread_exit(9); }).detach();
 
    std::cout << "Waiting..." << std::flush;
    f1.wait();
    f2.wait();
    f3.wait();
    std::cout << "Done!\nResults are: "
              << f1.get() << ' ' << f2.get() << ' ' << f3.get() << '\n';
    t.join();
}
```

In the words of [futures.state] a `std::future` is an *asynchronous return object* ("an object that reads results from a shared state") and a `std::promise` is an *asynchronous provider* ("an object that provides a result to a shared state") i.e. a promise is the thing that you *set* a result on, so that you can *get* it from the associated future.

### The asynchronous provider is what initially creates the shared state that a future refers to. 

`std::promise` is one type of asynchronous provider, `std::packaged_task` is another, and the internal detail of `std::async` is another. Each of those can create a shared state and give you a `std::future` that shares that state, and can make the state ready.

`std::async` is a higher-level convenience utility that gives you an asynchronous result object and internally takes care of creating the asynchronous provider and making the shared state ready when the task completes. You could emulate it with a `std::packaged_task` (or `std::bind` and a `std::promise`) and a `std::thread` but it's safer and easier to use `std::async`.

`std::promise` is a bit lower-level, for when you want to pass an asynchronous result to the future, but the code that makes the result ready cannot be wrapped up in a single function suitable for passing to `std::async`. For example, you might have an array of several `promise`s and associated `future`s and have a single thread which does several calculations and sets a result on each promise. `async` would only allow you to return a single result, to return several you would need to call `async` several times, which might waste resources.





### Future and Promise are the two separate sides of an asynchronous operation.

`std::promise` is used by the "producer/writer" of the asynchronous operation.

`std::future` is used by the "consumer/reader" of the asynchronous operation.

The reason it is separated into these two separate "interfaces" is to **hide** the "write/set" functionality from the "consumer/reader".

```cpp
auto promise = std::promise<std::string>();

auto producer = std::thread([&]
{
    promise.set_value("Hello World");
});

auto future = promise.get_future();

auto consumer = std::thread([&]
{
    std::cout << future.get();
});

producer.join();
consumer.join();
```

One (incomplete) way to implement std::async using std::promise could be:

```cpp
template<typename F>
auto async(F&& func) -> std::future<decltype(func())>
{
    typedef decltype(func()) result_type;

    auto promise = std::promise<result_type>();
    auto future  = promise.get_future();

    std::thread(std::bind([=](std::promise<result_type>& promise)
    {
        try
        {
            promise.set_value(func()); // Note: Will not work with std::promise<void>. Needs some meta-template programming which is out of scope for this question.
        }
        catch(...)
        {
            promise.set_exception(std::current_exception());
        }
    }, std::move(promise))).detach();

    return std::move(future);
}
```

Using `std::packaged_task` which is a helper (i.e. it basically does what we were doing above) around `std::promise` you could do the following which is more complete and possibly faster:

```cpp
template<typename F>
auto async(F&& func) -> std::future<decltype(func())>
{
    auto task   = std::packaged_task<decltype(func())()>(std::forward<F>(func));
    auto future = task.get_future();

    std::thread(std::move(task)).detach();

    return std::move(future);
}
```

Note that this is slightly different from `std::async` where the returned `std::future` will when destructed actually block until the thread is finished.

