# dart 学习记录

按极客大佬说的；编程语言无非需要解决两个问题

- 如何表示数据
- 如何处理数据

## Important concepts

As you learn about the Dart language, keep these facts and concepts in mind:

- Everything you can place in a variable is an _object_, and every object is an instance of a _class_. Even numbers, functions, and `null` are objects. All objects inherit from the [Object](https://api.dart.dev/stable/dart-core/Object-class.html) class.

- > 所有的变量都是类，继承于 Object

- Although Dart is strongly typed, type annotations are optional because Dart can infer types. In the code above, `number`is inferred to be of type `int`. When you want to explicitly say that no type is expected, [use the special type `dynamic`](https://dart.dev/guides/language/effective-dart/design#do-annotate-with-object-instead-of-dynamic-to-indicate-any-object-is-allowed).

- > 类型自推导

- Dart supports generic types, like `List<int>` (a list of integers) or `List<dynamic>` (a list of objects of any type).

- > 泛型支持

- Dart supports top-level functions (such as `main()`), as well as functions tied to a class or object (_static_ and _instance methods_, respectively). You can also create functions within functions (_nested_ or _local functions_).

- > 高阶函数

- Similarly, Dart supports top-level _variables_, as well as variables tied to a class or object (static and instance variables). Instance variables are sometimes known as fields or properties.

- 高阶变量

- Unlike Java, Dart doesn’t have the keywords `public`, `protected`, and `private`. If an identifier starts with an underscore (\_), it’s private to its library. For details, see [Libraries and visibility](https://dart.dev/guides/language/language-tour#libraries-and-visibility).

- > 使用\_ 起头表示一个私有成员变量，

- _Identifiers_ can start with a letter or underscore (\_), followed by any combination of those characters plus digits.

- Dart has both _expressions_ (which have runtime values) and _statements_ (which don’t). For example, the [conditional expression](https://dart.dev/guides/language/language-tour#conditional-expressions) `condition ? expr1 : expr2` has a value of `expr1` or `expr2`. Compare that to an [if-else statement](https://dart.dev/guides/language/language-tour#if-and-else), which has no value. A statement often contains one or more expressions, but an expression can’t directly contain a statement.

- Dart tools can report two kinds of problems: _warnings_ and _errors_. Warnings are just indications that your code might not work, but they don’t prevent your program from executing. Errors can be either compile-time or run-time. A compile-time error prevents the code from executing at all; a run-time error results in an [exception](https://dart.dev/guides/language/language-tour#exceptions) being raised while the code executes。

- > 警告或者错误，

## Keywords

> 关键字

The following table lists the words that the Dart language treats specially.

| [abstract](https://dart.dev/guides/language/language-tour#abstract-classes) 2         | [dynamic](https://dart.dev/guides/language/language-tour#important-concepts) 2             | [implements](https://dart.dev/guides/language/language-tour#implicit-interfaces) 2                      | [show](https://dart.dev/guides/language/language-tour#importing-only-part-of-a-library) 1 |
| ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| [as](https://dart.dev/guides/language/language-tour#type-test-operators) 2            | [else](https://dart.dev/guides/language/language-tour#if-and-else)                         | [import](https://dart.dev/guides/language/language-tour#using-libraries) 2                              | [static](https://dart.dev/guides/language/language-tour#class-variables-and-methods) 2    |
| [assert](https://dart.dev/guides/language/language-tour#assert)                       | [enum](https://dart.dev/guides/language/language-tour#enumerated-types)                    | [in](https://dart.dev/guides/language/language-tour#for-loops)                                          | [super](https://dart.dev/guides/language/language-tour#extending-a-class)                 |
| [async](https://dart.dev/guides/language/language-tour#asynchrony-support) 1          | [export](https://dart.dev/guides/libraries/create-library-packages) 2                      | [interface](https://stackoverflow.com/questions/28595501/was-the-interface-keyword-removed-from-dart) 2 | [switch](https://dart.dev/guides/language/language-tour#switch-and-case)                  |
| [await](https://dart.dev/guides/language/language-tour#asynchrony-support) 3          | [extends](https://dart.dev/guides/language/language-tour#extending-a-class)                | [is](https://dart.dev/guides/language/language-tour#type-test-operators)                                | [sync](https://dart.dev/guides/language/language-tour#generators) 1                       |
| [break](https://dart.dev/guides/language/language-tour#break-and-continue)            | [external](https://stackoverflow.com/questions/24929659/what-does-external-mean-in-dart) 2 | [library](https://dart.dev/guides/language/language-tour#libraries-and-visibility) 2                    | [this](https://dart.dev/guides/language/language-tour#constructors)                       |
| [case](https://dart.dev/guides/language/language-tour#switch-and-case)                | [factory](https://dart.dev/guides/language/language-tour#factory-constructors) 2           | [mixin](https://dart.dev/guides/language/language-tour#adding-features-to-a-class-mixins) 2             | [throw](https://dart.dev/guides/language/language-tour#throw)                             |
| [catch](https://dart.dev/guides/language/language-tour#catch)                         | [false](https://dart.dev/guides/language/language-tour#booleans)                           | [new](https://dart.dev/guides/language/language-tour#using-constructors)                                | [true](https://dart.dev/guides/language/language-tour#booleans)                           |
| [class](https://dart.dev/guides/language/language-tour#instance-variables)            | [final](https://dart.dev/guides/language/language-tour#final-and-const)                    | [null](https://dart.dev/guides/language/language-tour#default-value)                                    | [try](https://dart.dev/guides/language/language-tour#catch)                               |
| [const](https://dart.dev/guides/language/language-tour#final-and-const)               | [finally](https://dart.dev/guides/language/language-tour#finally)                          | [on](https://dart.dev/guides/language/language-tour#catch) 1                                            | [typedef](https://dart.dev/guides/language/language-tour#typedefs) 2                      |
| [continue](https://dart.dev/guides/language/language-tour#break-and-continue)         | [for](https://dart.dev/guides/language/language-tour#for-loops)                            | [operator](https://dart.dev/guides/language/language-tour#overridable-operators) 2                      | [var](https://dart.dev/guides/language/language-tour#variables)                           |
| [covariant](https://dart.dev/guides/language/sound-problems#the-covariant-keyword) 2  | [Function](https://dart.dev/guides/language/language-tour#functions) 2                     | [part](https://dart.dev/guides/libraries/create-library-packages#organizing-a-library-package) 2        | [void](https://medium.com/dartlang/dart-2-legacy-of-the-void-e7afb5f44df0)                |
| [default](https://dart.dev/guides/language/language-tour#switch-and-case)             | [get](https://dart.dev/guides/language/language-tour#getters-and-setters) 2                | [rethrow](https://dart.dev/guides/language/language-tour#catch)                                         | [while](https://dart.dev/guides/language/language-tour#while-and-do-while)                |
| [deferred](https://dart.dev/guides/language/language-tour#lazily-loading-a-library) 2 | [hide](https://dart.dev/guides/language/language-tour#importing-only-part-of-a-library) 1  | [return](https://dart.dev/guides/language/language-tour#functions)                                      | [with](https://dart.dev/guides/language/language-tour#adding-features-to-a-class-mixins)  |
| [do](https://dart.dev/guides/language/language-tour#while-and-do-while)               | [if](https://dart.dev/guides/language/language-tour#if-and-else)                           | [set](https://api.dart.dev/stable/dart-core/Set-class.html) 2                                           | [yield](https://dart.dev/guides/language/language-tour#generators) 3                      |

## Built-in types

The Dart language has special support for the following types:

- numbers
- strings 16bits utf-16
- booleans
- lists (also known as _arrays_)
- sets
- maps
- runes (for expressing Unicode characters in a string) 32bits
- symbols

When defining a function, use `{*param1*, *param2*, …}` to specify named parameters:

您可以使用@required 在任何 Dart 代码（不仅仅是 Flutter）中注释命名参数，以指示它是必需参数。

Wrapping a set of function parameters in `[]` marks them as optional positional parameters:

### Type test operators

The `as`, `is`, and `is!` operators are handy for checking types at runtime.

| Operator | Meaning                                                                                                                        |
| -------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `as`     | Typecast (also used to specify [library prefixes](https://dart.dev/guides/language/language-tour#specifying-a-library-prefix)) |
| `is`     | True if the object has the specified type                                                                                      |
| `is!`    | False if the object has the specified type                                                                                     |

### Cascade notation (..)

> 级联符号

Cascades (`..`) allow you to make a sequence of operations on the same object. In addition to function calls, you can also access fields on that same object. This often saves you the step of creating a temporary variable and allows you to write more fluid code.

Consider the following code:

```dart
querySelector('#confirm') // Get an object.
  ..text = 'Confirm' // Use its members.
  ..classes.add('important')
  ..onClick.listen((e) => window.alert('Confirmed!'));
```

The first method call, `querySelector()`, returns a selector object. The code that follows the cascade notation operates on this selector object, ignoring any subsequent values that might be returned.

The previous example is equivalent to:

```dart
var button = querySelector('#confirm');
button.text = 'Confirm';
button.classes.add('important');
button.onClick.listen((e) => window.alert('Confirmed!'));
```

You can also nest your cascades. For example:

```dart
final addressBook = (AddressBookBuilder()
      ..name = 'jenny'
      ..email = 'jenny@example.com'
      ..phone = (PhoneNumberBuilder()
            ..number = '415-555-0100'
            ..label = 'home')
          .build())
    .build();
```

Be careful to construct your cascade on a function that returns an actual object. For example, the following code fails:

```dart
var sb = StringBuffer();
sb.write('foo')
  ..write('bar'); // Error: method 'write' isn't defined for 'void'.
```

The `sb.write()` call returns void, and you can’t construct a cascade on `void`.

## Control flow statements

You can control the flow of your Dart code using any of the following:

- `if` and `else`
- `for` loops
- `while` and `do`-`while` loops
- `break` and `continue`
- `switch` and `case`
- `assert`

## 一个例子

```dart

class Meta{
  double price;
  String name;
  Meta(this.name,this.price);//构造函数语法糖
}

class Item extends Meta{
  Item(name,price) : super(name,price);//委托父类构造
  Item operator+(Item item) => Item(name +' ' + item.name,price + item.price);//+运算符重载？不过dart里是没有重载的说法的，不过可以和c++的运算符重载对应理解
}

//打印的抽象接口类，使用了abstract，恩，和java的interface一样；
abstract class PrintHelper{  
  PrintInfo() => print(getInfo());
  getInfo();
}
//使用with mixin 接口，类似c++里的多重继承，不过应该是有限制的，可能只能使用with连接抽象接口？
//我不确定，
class ShoppingCart extends Meta with PrintHelper{
  DateTime date;
  String code;
  List<Item> bookings;

  double get price => bookings.reduce((value,elem) => value + elem).price;
	
  //这里可以理解为构造函数的重载，withCode可以随意命名，将参数使用{}包起来，表示某些参数可以不写，直接使用，这里是核心语法，需要重点关注！！
  ShoppingCart({name}) : this.withCode(name:name,code:null);
  ShoppingCart.withCode({name,this.code}):date = DateTime.now(),super(name,0);

  //这里覆写了PrintHelper的getInfo函数，在调用PrintInfo时调用这个函数.
  @override
  getInfo()  =>'''
  购物车信息：
  -------------------------
  用户名： $name 
  优惠码：${code??" 没有 "}  //这里也很关键，表示3重运算的简写，等同于 code？code :"没有"；
  总价： $price
  日期： $date
  -------------------------
  ''';
}

void main(){
  ShoppingCart.withCode(name:'张三',code: '123456')
  ..bookings = [Item('苹果',10.0),Item('鸭梨',20.0)]
  ..PrintInfo();

  ShoppingCart.withCode(name:'李四')
  ..bookings = [Item('苹果',15.0),Item('西瓜',20.0)]
  ..PrintInfo();
}
```

输出：

```
[Running] dart "f:\02-dart\shoppingmarket6.dart"
  购物车信息：
  -------------------------
  用户名： 张三 
  优惠码：123456 
  总价： 30.0
  日期： 2019-07-22 15:02:45.600262
  -------------------------
  
  购物车信息：
  -------------------------
  用户名： 李四 
  优惠码： 没有  
  总价： 35.0
  日期： 2019-07-22 15:02:45.606254
  -------------------------
```





喜欢dart没有商量，属于C++系的语法糖，我很喜欢吃！