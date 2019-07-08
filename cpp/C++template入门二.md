# C++template入门二

> be As a C++ enthusiast

在[C++template入门一](D:\nutCloud\sourceMd\TyporaMD\C++template入门一.md)中，我们主要关注了模板函数，这一次，我们开始关注**类模板**；

## 为何使用类模板

与函数模板一样，您也可以为泛型类操作创建类模板。 有时，您需要一个只使用了不同的数据类型的，对所有类都有相同的类实现。 通常，您需要为每种数据类型创建不同的类，或者在单个类中创建不同的成员变量和函数。 这将不必要地代码膨胀并且使得代码难以维护，因为发生的更改是应该对所有类/函数执行。 但是，类模板可以轻松地为所有数据类型重用相同的代码。

## 如何使用

### 类模板的代码	

```c++
template <class T> class templateCls {
public:
    ....
    T var;
    ...
};
```

`T`就是模板参数的占位符；

### 如何创建一个类模板的对象呢？

```c++
templateCls<int> inttempCls;
```

举例：

测试用例：

```c++
TEST(TESTTEMPLATECLASS, CALCULATOR_int) {
    Calculator<int> intCalc(2, 1);
    EXPECT_EQ(3, intCalc.add());
    EXPECT_EQ(1, intCalc.subtract());
    EXPECT_EQ(2, intCalc.multiply());
    EXPECT_EQ(2, intCalc.divide());
}

TEST(TESTTEMPLATECLASS, CALCULATOR_float) {

    Calculator<float> floatCalc(2.4, 1.2);
    EXPECT_FLOAT_EQ(3.6, floatCalc.add());
    EXPECT_FLOAT_EQ(1.2, floatCalc.subtract());
    EXPECT_FLOAT_EQ(2.68, floatCalc.multiply()); //failed！
    EXPECT_FLOAT_EQ(2.0, floatCalc.divide());
}
```

定义一个类模板，这个类实现类计算类，用于两个数据的加减乘除；

```c++
template <class T> class Calculator {
  private:
    T num1, num2;

  public:
    Calculator(T n1, T n2) {
        num1 = n1;
        num2 = n2;
    }

    T add() { return num1 + num2; }

    T subtract() { return num1 - num2; }

    T multiply() { return num1 * num2; }

    T divide() { return num1 / num2; }
};
```

测试用例运行：

int类型时测试成功；

float数据类型时，测试失败



