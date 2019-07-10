# flutter 第一个demo分析

使用vscode 能自动生成一个计数器demo，分析这个demo来入门吧：

## 试玩
运行后显示如图，
![](assets/第一个demo分析0.png)

在lib\main.dart 文件中做些修改：
这里我已经修改了首页的title

```dart
home: MyHomePage(title: 'Flutter Demo Home hulu2'),
```

修改了主题的颜色。默认是颜色
```dart
theme: ThemeData(
        primarySwatch: Colors.green,
      ),
```



## 分析源码

把源码分为5部分：

![](assets/第一个demo分析1.png)

### 第一部分：
```dart
import 'package:flutter/material.dart';
```
导入包，这个包是material.dart包，用于android的material设计；

### 第二部分
```dart
void main() => runApp(MyApp());
```
入口函数main，用于启动runApp，runApp的参数是我们自己定义的class，MyApp；
这个runApp函数肯定有一个事件循环，用于事件监听和处理的；

### 第三部分
```dart
class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.green,
      ),
      home: MyHomePage(title: 'Flutter Demo Home hulu2'),
    );
  }
}
```
去除了注释，仅分析代码本身：类MyApp继承于StatelessWidget，覆写其build方法。可知，整个app由一个widget构成，build后返回一个MaterialApp，这里设置了widget的title，theme和home；home本身也是一个widget。在第四部分了；

### 第四部分
```dart
class MyHomePage extends StatefulWidget {
  MyHomePage({Key key, this.title}) : super(key: key);
  final String title;
  @override
  _MyHomePageState createState() => _MyHomePageState();
}
```
类 MyHomePage 继承于一个带状态的widget ，其状态产生 由 _MyHomePageState函数生成。
我们可以简单认为Stateful widget 和Stateless widget有两点不同：
1. Stateful widget可以拥有状态，这些状态在widget生命周期中是可以变的，而Stateless widget是不可变的。
2. Stateful widget至少由两个类组成：

- 一个StatefulWidget类。
- 一个 State类； StatefulWidget类本身是不变的，但是 State类中持有的状态在widget生命周期中可能会发生变化。
- _MyHomePageState类是MyHomePage类对应的状态类。看到这里，细心的读者可能已经发现，和MyApp 类不同， MyHomePage类中并没有build方法，取而代之的是，build方法被挪到了_MyHomePageState方法中，至于为什么这么做，先留个疑问，在分析完完整代码后再来解答。

### 第五部分
```dart
class _MyHomePageState extends State<MyHomePage> {
  int _counter = 0;

  void _incrementCounter() {
    setState(() {
      _counter++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text(widget.title),
      ),
      body: Center(
        child: Column(
        
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            Text(
              'You have pushed the button this many times:',
            ),
            Text(
              '$_counter',
              style: Theme.of(context).textTheme.display1,
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: _incrementCounter,
        tooltip: 'Increment',
        child: Icon(Icons.add),
      ), 
    );
  }
}

```

_MyHomePageState 是一个widget，则需要build，

`Scaffold` 是Material库中提供的页面脚手架，是一个class；其定义了3个组件，包含导航栏和Body以及FloatingActionButton（如果需要的话）。 

- appBar  定义了 title: **Text**(widget.title),

- body 使用了**Center**布局 
// Center is a layout widget. It takes a single child and positions it
// in the middle of the parent.
  
  - 其child: **Column**定义为Column布局；
  
   *// Column is also layout widget. It takes a list of children and*

​          *// arranges them vertically. By default, it sizes itself to fit its*

​          *// children horizontally, and tries to be as tall as its parent.*

​	在Column下有布局了两个widget，一个是text：You have pushed the button this many times:',

另一个是显示 '$_counter',

- floatingActionButton 这是一个浮动按钮，onPressed时的响应函数是 **_incrementCounter** 。

`Scaffold` 还有其他组件，可以F12查看之。

**现在，我们将整个流程串起来：当右下角的floatingActionButton按钮被点击之后，会调用`_incrementCounter`，在`_incrementCounter`中，首先会自增`_counter`计数器（状态），然后`setState`会通知Flutter框架状态发生变化，接着，Flutter会调用`build`方法以新的状态重新构建UI，最终显示在设备屏幕上。**

### 为什么要将build方法放在State中，而不是放在StatefulWidget中？

现在，我们回答之前提出的问题，为什么`build()`方法在State（而不是StatefulWidget）中 ？这主要是为了开发的灵活性。如果将`build()`方法在StatefulWidget中则会有两个问题：

- 状态访问不便

  试想一下，如果我们的Stateful widget 有很多状态，而每次状态改变都要调用`build`方法，由于状态是保存在State中的，如果将`build`方法放在StatefulWidget中，那么构建时读取状态将会很不方便，试想一下，如果真的将`build`方法放在StatefulWidget中的话，由于构建用户界面过程需要依赖State，所以`build`方法将必须加一个`State`参数，大概是下面这样：

  ```dart
    Widget build(BuildContext context, State state){
        //state.counter
        ...
    }
  ```

  这样的话就只能将State的所有状态声明为公开的状态，这样才能在State类外部访问状态，但将状态设置为公开后，状态将不再具有私密性，这样依赖，对状态的修改将会变的不可控。将`build()`方法放在State中的话，构建过程则可以直接访问状态，这样会很方便。

- 继承StatefulWidget不便

  例如，Flutter中有一个动画widget的基类`AnimatedWidget`，它继承自`StatefulWidget`类。`AnimatedWidget`中引入了一个抽象方法`build(BuildContext context)`，继承自`AnimatedWidget`的动画widget都要实现这个`build`方法。现在设想一下，如果`StatefulWidget` 类中已经有了一个`build`方法，正如上面所述，此时`build`方法需要接收一个state对象，这就意味着`AnimatedWidget`必须将自己的State对象(记为_animatedWidgetState)提供给其子类，因为子类需要在其`build`方法中调用父类的`build`方法，代码可能如下：

  ```dart
  class MyAnimationWidget extends AnimatedWidget{
      @override
      Widget build(BuildContext context, State state){
        //由于子类要用到AnimatedWidget的状态对象_animatedWidgetState，
        //所以AnimatedWidget必须通过某种方式将其状态对象_animatedWidgetState
        //暴露给其子类   
        super.build(context, _animatedWidgetState)
      }
  }
  ```

  这样很显然是不合理的，因为

  1. `AnimatedWidget`的状态对象是`AnimatedWidget`内部实现细节，不应该暴露给外部。
  2. 如果要将父类状态暴露给子类，那么必须得有一种传递机制，而做这一套传递机制是无意义的，因为父子类之间状态的传递和子类本身逻辑是无关的。

综上所述，可以发现，对于StatefulWidget，将`build`方法放在State中，可以给开发带来很大的灵活性