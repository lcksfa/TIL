# 现代 C++中的 weak_ptr

> label : c++,smart pointer

## 什么是 weak_ptr

官方的解释如下：

std::weak_ptr is a smart pointer that holds a non-owning ("weak") reference to an object that is managed by std::shared_ptr. It must be converted to std::shared_ptr in order to access the referenced object.

std::weak_ptr models temporary ownership: when an object needs to be accessed only if it exists, and it may be deleted at any time by someone else, std::weak_ptr is used to track the object, and it is converted to std::shared_ptr to assume temporary ownership. If the original std::shared_ptr is destroyed at this time, the object's lifetime is extended until the temporary std::shared_ptr is destroyed as well.

Another use for std::weak_ptr is to break reference cycles formed by objects managed by std::shared_ptr. If such cycle is orphaned (i,e. there are no outside shared pointers into the cycle), the shared_ptr reference counts cannot reach zero and the memory is leaked. To prevent this, one of the pointers in the cycle can be made weak.

我的“智能”理解：

std::weak_ptr 是智能指针的一种，它是对被 std::shared_ptr 管理的对象的一种不具有所有权的“弱”引用；它必须转换为 std::shared_ptr 才能访问被引用的对象；

临时所有权是指：一个对象可以被访问的前提是它必须存在，可是它可能被其他人在任何时间 delete 掉，std::weak_ptr 就是用于追踪这个对象，被转换为 std::shared_ptr 后设定临时所有权。即便原始的 std::shared_ptr 被销毁了，这个对象的生命周期将被延长，直到 std::weak_ptr 转换的临时的 std::shared_ptr 也被销毁为止。

另一个使用 std::weak_ptr 的地方是 它可以打破 std::shared_ptr 的**引用循环**，如果这个循环发生了，这个 std::shared_ptr 引用计数就无法变成 0 从而引发内存泄漏。为了防止这个问题，其中一个环内的指针可以是 weak 的。

## 它可以解决什么问题

官方的解释说的清楚明白：

1. 用于追踪对象的生存周期，判断这个对象是否被销毁
2. 打破 std::shared_ptr 的引用循环

## 如何使用

为了说明 std::weak_ptr 的使用，我们使用代码来解释；

### 使用场景 1

现在是个**异步回调**漫天飞舞的世界了，我们看看这样的代码；

```c++
class MyClass;
void MyClass::callback(){}
void MyClass::called(){
    auto bindCallback = std::bind(&MyClass::callback,this);
    CallBackMe(bindCallback);
}
```

这段伪代码的意思是，我们在 called 中 bind 了一个回调函数 bindCallback，CallBackMe 则设置这个回调函数，当 CallBackMe 执行到某个设定的位置时，它会调用这个回调函数从而调用 MyClass 的 callback 函数。只是，这里有个问题，如果当 bindCallback 执行时，MyClass 被释放了呢？

这样的话，一旦 MyClass 的对象继续被访问，势必 引发程序崩溃。

为了解决这个问题，我们需要使用 std::weak_ptr ；

因为 this 指针的特殊性，为了不引发 this 的 double delete 问题，我们需要将其继承 enable_shared_from_this<>。

```c++
class MyClass : public std::enable_shared_from_this<MyClass>;

using OnCallBackFunc = function<void(void)>;

//expired() Equivalent to use_count() == 0. The destructor for the managed object may not yet have been called, but this object's destruction is imminent (or may have already happened).

//lock() ,if Creates a new std::shared_ptr that shares ownership of the managed object. If there is no managed object, i.e. *this is empty, then the returned shared_ptr also is empty.
void MyClass::callback(std::weak_ptr<MyClass> weakptr){
	if(weakptr.expired()) return;
    if(auto ptr = weakptr.lock()){
        ptr->dosomethig();
	}

}

OnCallBackFunc GetWeakCallback(){
    std::weak_ptr<MyClass> thisWeakPtr(shared_from_this());
    return OnCallBackFunc(std::bind(&MyClass::callback,this,thisWeakPtr));
}

void MyClass::called(){

    CallBackMe(GetWeakCallback());
}
```

也就是说，当我们使用 this 时，先创建一个 weak 指针，只有确认这个指针没有被销毁时，我们才继续调用后面的函数。特别注意的有三点：

1. 我们必须让 MyClass 继续于 enable_shared_from_this，这样我们才能使用 shared_from_this()创建 weakptr，
2. 在使用 MyClass 时一定不能使用 Raw point（MyClass\*），而必须使用 shared_ptr<MyClass>，否则，会报 bad weakptr 错误。
3. weakptr 不能直接访问对象，必须先转为 shared_ptr，转换 api 为 lock(),lock 会创建一个新的原 std::shared_ptr 的引用，所有具有延长对象生命周期的能力。

### 使用场景 2

第二个场景就是 shared_ptr 的引用环了；

实例代码：

```c++
#include <iostream>
#include <memory>
#include <string>

class Dog
{
private:
    /* data */
    std::shared_ptr<Dog> _friend;
    std::string _name;
public:
    Dog(const std::string& name);
    ~Dog();
    std::string GetName(){return _name;}
    void bark(){std::cout << "a dog bark"<<"\n";}
    void makeFriend(std::shared_ptr<Dog> f){
        _friend = f;
        std::cout << "Dog: " << _name << "make friend with Dog:" << _friend->GetName() << std::endl;
    }
};

Dog::Dog(const std::string& name):_name(name)
{
    std::cout << "a dog named:" << _name <<" created!"<< std::endl;
}

Dog::~Dog()
{
    std::cout << "dog "<< _name << "was destoryed!" << std::endl;
}


int main(int argc, char const *argv[])
{
    /* code */
    std::shared_ptr<Dog> pd1 = std::make_shared<Dog>("Puggy");
    std::shared_ptr<Dog> pd2 = std::make_shared<Dog>("Alex");

    pd1->makeFriend(pd2);
    pd2->makeFriend(pd1);
    return 0;
}

```

运行结果：

```c++
a dog named:Puggy created!
a dog named:Alex created!
Dog: Puggymake friend with Dog:Alex
Dog: Alexmake friend with Dog:Puggy
```

#### 问题在哪里？

问题在于析构函数没有调用啊！也就是说，这里内存泄漏了！

#### 如何解决这个问题呢？

将`std::shared_ptr<Dog> _friend;` 改为`std::weak_ptr<Dog> _friend;` 。因为 weak_ptr 不能直接访问对象成员，故还将使用`_friend`的地方 添加其生成 shared_ptr 的 api ：lock();

修改后的运行结果：

```c++
a dog named:Puggy created!
a dog named:Alex created!
Dog: Puggymake friend with Dog:Alex
Dog: Alexmake friend with Dog:Puggy
dog Alexwas destoryed!
dog Puggywas destoryed!
```

如此：两个 Dog 对象在出作用域时正常析构了；

#### 为什么？

要解答这个问题，得从 weak_ptr 和 share_ptr 的原理出发。

我们已知道，shared_ptr 要析构其管理的内存，必须等到其引用计数为 0。

让我们继续分析下；

- 当\_friend 的类型是 shared_ptr 时：

```c++
int main(int argc, char const *argv[])
{
    /* code */
    std::shared_ptr<Dog> pd1 = std::make_shared<Dog>("Puggy");
    std::shared_ptr<Dog> pd2 = std::make_shared<Dog>("Alex");

    pd1->makeFriend(pd2);
    std::cout << "pd1 use cout "<< pd1.use_count() <<std::endl;  //1
    std::cout << "pd2 use cout "<< pd2.use_count() <<std::endl;  //1

    pd2->makeFriend(pd1);
    std::cout << "pd2 use cout "<< pd2.use_count() <<std::endl; //2
    std::cout << "pd1 use cout "<< pd1.use_count() <<std::endl;  //2
    return 0;
}
```

也就是说，在出作用域前，两个 shared_ptr 的引用计数都是 2 了，这当然不会引发析构。

- 当\_friend 的类型是 weak_ptr 时：

```c++
int main(int argc, char const *argv[])
{
    /* code */
    std::shared_ptr<Dog> pd1 = std::make_shared<Dog>("Puggy");
    std::shared_ptr<Dog> pd2 = std::make_shared<Dog>("Alex");

    pd1->makeFriend(pd2);
    std::cout << "pd1 use cout "<< pd1.use_count() <<std::endl; //1
    std::cout << "pd2 use cout "<< pd2.use_count() <<std::endl; //1

    pd2->makeFriend(pd1);
    std::cout << "pd2 use cout "<< pd2.use_count() <<std::endl; //1
    std::cout << "pd1 use cout "<< pd1.use_count() <<std::endl; //1
    return 0;
}
```

可见使用 weak_ptr 时，Dog 对象的引用计数不会因赋值等操作而增加，从而保证出作用域时计数归 0 析构。

## 总结

写了这个多，总结 weak_ptr 的使用场景：

1. 当我们需要判断 shard_ptr 管理的内存对象是否还可用
2. 当我们的类中的成员需要环状引用时。
