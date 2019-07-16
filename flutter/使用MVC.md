# use mvc_pattern

> 借助 Flutter 的 MVC [三方库](https://pub.dev/packages/mvc_pattern)，实现 MVC 的架构。
> 我们知道，MVC 是一种架构思维方式，所以也不应该生搬硬套似的使用这个框架，要实在的理解这个惯用法，以及细细体会它的好处。

## 目标：使用 MVC 改造之前的 calculatorapp

原始的代码[在这里](./独立的使用VScode.md)

### 改造第 1 步

添加三方库，
在 pubspec.yaml 中添加依赖`mvc_pattern: ^3.4.1`

### 改造第 2 步

there's now a separation of ‘the Interface’ and ‘the data’ as intended with the MVC architecture. The build() function serves as 'the View.' It's concerned solely with the ‘look and feel’ of the app’s interface—‘how’ things are displayed. While it is the Controller that determines 'what’ is displayed. The Controller is also concerned with 'how' the app interacts with the user, and so it's involved in the app's event handling.

### 第一个版本

```dart
//mian.dart
import 'package:flutter/material.dart';
import 'package:mvc_pattern/mvc_pattern.dart';
import 'package:calculatorapp/home_page.dart';

void main() => runApp(MyApp());

class MyApp extends AppMVC {
  MyApp({Key key}) : super(key: key);
  static final String title = 'Calculator App';
  final MyHomePage home = MyHomePage(title);

  ControllerMVC get controller => home.controller;
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: title,
      theme: ThemeData(
        primarySwatch: Colors.red,
      ),
      home: home,
    );
  }
}

```

```dart
//home_page.dart
import 'package:flutter/material.dart';
import 'package:mvc_pattern/mvc_pattern.dart';

class MyHomePage extends StatefulWidget {
  MyHomePage(this.title, {Key key}) : super(key: key);
  final String title;
  final _MyHomePageState state = _MyHomePageState();

  Controller get controller => state.controller;
  @override
  State createState() => state;
}

class _MyHomePageState extends StateMVC<MyHomePage> {
  final TextEditingController t1 = TextEditingController(text: "0");
  final TextEditingController t2 = TextEditingController(text: "0");

  _MyHomePageState() : super(Controller()) {
    _con = controller;
  }
  Controller _con;

  @override
  Widget build(BuildContext context) {
    var materialButtonAdd = MaterialButton(
      child: Text("+"),
      color: Colors.greenAccent,
      onPressed: () {
        _con.setData(int.parse(t1.text), int.parse(t2.text));
        setState(_con.doAdd);
      },
    );
    var materialButtonSub = MaterialButton(
      color: Colors.greenAccent,
      child: Text("-"),
      onPressed: () {
        _con.setData(int.parse(t1.text), int.parse(t2.text));
        setState(_con.dosub);
      },
    );

    var children2 = <Widget>[materialButtonAdd, materialButtonSub];

    var materialButtonMul = MaterialButton(
      color: Colors.greenAccent,
      child: Text("*"),
      onPressed: () {
        _con.setData(int.parse(t1.text), int.parse(t2.text));
        setState(_con.doMul);
      },
    );
    var materialButtonDiv = MaterialButton(
      color: Colors.greenAccent,
      child: Text("/"),
      onPressed: () {
        _con.setData(int.parse(t1.text), int.parse(t2.text));
        setState(_con.doDiv);
      },
    );

    var child2 = Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text("Output : ${_con.result}",
            style: TextStyle(
                fontSize: 20.0,
                color: Colors.purple,
                fontWeight: FontWeight.bold)),
        TextField(
          keyboardType: TextInputType.number,
          decoration: InputDecoration(hintText: "Enter Number 1"),
          controller: t1,
        ),
        TextField(
          keyboardType: TextInputType.number,
          decoration: InputDecoration(hintText: "Enter Number 2"),
          controller: t2,
        ),
        Padding(
          padding: const EdgeInsets.only(top: 20.0),
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: children2,
        ),
        Padding(
          padding: const EdgeInsets.only(top: 20.0),
        ),
        Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: <Widget>[
              materialButtonMul,
              materialButtonDiv,
            ])
      ],
    );
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Container(
        padding: const EdgeInsets.all(40.0),
        child: child2,
      ),
    );
  }
}

class Controller extends ControllerMVC {
  int get result => _result;
  int _num1 = 0, _num2 = 0, _result = 0;
  void setData(int num1, int num2) {
    _num1 = num1;
    _num2 = num2;
  }

  void doAdd() {
    _result = _num1 + _num2;
  }

  void dosub() {
    _result = _num1 - _num2;
  }

  void doMul() {
    _result = _num1 * _num2;
  }

  void doDiv() {
    _result = _num1 ~/ _num2;
  }
}

```

这个版本把运算的逻辑放到了 controller 中，从而和 UI 的逻辑分离开来；
不过这里只有 View 和 controller ，没有 model，
同时将按钮的 UI 配置改写成了变量，这样方便单独维护，嵌套的括号真是不好看。
这个版本的 press 函数 有代码重复，code smell not good，而且数据的获取显得很生硬；

### 继续改进 1

功能上已经将 UI 和逻辑实现分离了，但是代码上还有很大的重构可能；现在继续实现下；

使用静态类方法处理类配置过多的问题：

```dart
//main.dart
import 'package:flutter/material.dart';
import 'package:mvc_pattern/mvc_pattern.dart';
import 'package:calculatorapp/home_page.dart';

void main() => runApp(MyApp());

class MyApp extends AppMVC {
  MyApp({Key key}) : super(key: key);
  static final String title = 'Calculator App';

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'CalculorApp',
      theme: ThemeData(
        primarySwatch: Colors.red,
      ),
      home: MyHomePage(),
    );
  }
}

```

```dart
//home_page.dart
import 'package:calculatorapp/main.dart';
import 'package:flutter/material.dart';
import 'package:mvc_pattern/mvc_pattern.dart';

class MyHomePage extends StatefulWidget {
  MyHomePage({Key key}) : super(key: key);
  @protected
  @override
  State createState() => _MyHomePageState();
}

class _MyHomePageState extends StateMVC<MyHomePage> {
  final TextEditingController t1 = TextEditingController(text: "0");
  final TextEditingController t2 = TextEditingController(text: "0");

  _MyHomePageState() : super(Controller());
  @override
  Widget build(BuildContext context) {
    var materialButtonAdd = MaterialButton(
      child: Text("+"),
      color: Colors.greenAccent,
      onPressed: () {
        Controller.setData(int.parse(t1.text), int.parse(t2.text));
        setState(Controller.doAdd);
      },
    );
    var materialButtonSub = MaterialButton(
      color: Colors.greenAccent,
      child: Text("-"),
      onPressed: () {
        Controller.setData(int.parse(t1.text), int.parse(t2.text));
        setState(Controller.dosub);
      },
    );

    var children2 = <Widget>[materialButtonAdd, materialButtonSub];

    var materialButtonMul = MaterialButton(
      color: Colors.greenAccent,
      child: Text("*"),
      onPressed: () {
        Controller.setData(int.parse(t1.text), int.parse(t2.text));
        setState(Controller.doMul);
      },
    );
    var materialButtonDiv = MaterialButton(
      color: Colors.greenAccent,
      child: Text("/"),
      onPressed: () {
        Controller.setData(int.parse(t1.text), int.parse(t2.text));
        setState(Controller.doDiv);
      },
    );

    var child2 = Column(
      mainAxisAlignment: MainAxisAlignment.center,
      children: <Widget>[
        Text("Output : ${Controller.result}",
            style: TextStyle(
                fontSize: 20.0,
                color: Colors.purple,
                fontWeight: FontWeight.bold)),
        TextField(
          keyboardType: TextInputType.number,
          decoration: InputDecoration(hintText: "Enter Number 1"),
          controller: t1,
        ),
        TextField(
          keyboardType: TextInputType.number,
          decoration: InputDecoration(hintText: "Enter Number 2"),
          controller: t2,
        ),
        Padding(
          padding: const EdgeInsets.only(top: 20.0),
        ),
        Row(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: children2,
        ),
        Padding(
          padding: const EdgeInsets.only(top: 20.0),
        ),
        Row(
            mainAxisAlignment: MainAxisAlignment.spaceEvenly,
            children: <Widget>[
              materialButtonMul,
              materialButtonDiv,
            ])
      ],
    );
    return Scaffold(
      appBar: AppBar(
        title: Text(MyApp.title),
      ),
      body: Container(
        padding: const EdgeInsets.all(40.0),
        child: child2,
      ),
    );
  }
}

class Controller extends ControllerMVC {
  static int get result => _result;
  static int _num1 = 0, _num2 = 0;
  static int _result = 0;
  static void setData(int num1, int num2) {
    _num1 = num1;
    _num2 = num2;
  }

  static void doAdd() {
    _result = _num1 + _num2;
  }

  static void dosub() {
    _result = _num1 - _num2;
  }

  static void doMul() {
    _result = _num1 * _num2;
  }

  static void doDiv() {
    _result = _num1 ~/ _num2;
  }
}

```

改造后的代码最关键的是把 controller 变成了静态的，可以直接通过类名访问它的静态成员函数。不过，这样也仅仅只是减少了 V 和 C 之间的类配置问题，并没有解决处理函数代码有些重复的问题。

### 继续改进 2

Mixin
DescriptionIn object-oriented programming languages, a mixin is a class that contains methods for use by other classes without having to be the parent class of those other classes. How those other classes gain access to the mixin's methods depends on the language.

[bloc](https://felangel.github.io/bloc/#/gettingstarted?id=overview)
