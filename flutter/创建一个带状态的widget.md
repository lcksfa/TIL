# stateful widget
stateless widget 是一成不变的，这意味这它们的属性不能改变，所有的值都是final。

statueful widgets 可以在它的生命周期中改变状态；
实现一个statueful widget 至少需要两个类：
- 一个statueful widget 创建一个实例
- 一个statue 类，

整体代码如下:
```dart
import 'package:english_words/english_words.dart' as prefix0;
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    // final WordPair = prefix0.WordPair.random();
    return MaterialApp(
      title: 'Flutter Hello Demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text('My second demo in flutter'),
        ),
        body: Center(
          child: RandomWords(),
        ),
      ),
    );
  }
}

class RandomWordsState extends State<RandomWords> {
  @override
  Widget build(BuildContext context) {
    final wordPair = prefix0.WordPair.random();
    return Text(wordPair.asPascalCase);
  }
}

class RandomWords extends StatefulWidget {
  @override
  RandomWordsState createState() => RandomWordsState();
}
```

这段代码中，RandomWords是一个statueful的widget ，
，它创建一个state，为RandomWordsState，RandomWordsState 里面重写build方法，用于statue改变时重写生成随机word。

一般而已，我们的程序套路都应该是这样的，即便是最简单的hello word。



这里使用了一个第三方包english_words



要在`pubspec.yaml`中配置下依赖关系：

```
dependencies:
  flutter:
    sdk: flutter

  # The following adds the Cupertino Icons font to your application.
  # Use with the CupertinoIcons class for iOS style icons.
  cupertino_icons: ^0.1.2
  english_words: ^3.1.0
```

配置完成后，VScode会自动下载。



