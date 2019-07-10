# hello world ,flutter

代码如下：

```dart
import 'package:flutter/material.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Hello Demo',
      home: Scaffold(
        appBar: AppBar(
          title: Text('My first demo in flutter'),
        ),
        body: Center(
          child: Text("Hello World Flutter!"),
        ),
      ),
    );
  }
}

```

不得不承认，括号实在有点多了，特别是下括号，但是借助vscode的代码提示和下括号自动注释，我觉得轻松了不少；
vscode下的提示是这样的：

![1562768127645](assets/1562768127645.png)

我也能写安卓程序了哈；

![1562768085290](assets/1562768085290.png)

## 小结

这是2019年7月10日最大的收获，本来不准备做这方面的尝试了的，不过最终还是没有顶住移动端的诱惑入坑了，目前感觉良好，后面会学习github上的[flutter example](https://github.com/flutter/samples/blob/master/INDEX.md)来继续深入的学习。



