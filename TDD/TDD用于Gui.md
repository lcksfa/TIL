# 在Gui程序中使用TDD



It depends on what your code is doing, i.e. what is the actual logic you need to test.

I presume that your application uses an API in order to draw objects such as circles, lines, etc. In this case, your goal is not to test whether the image/screen corresponds to what was expected—if you requested the API to draw a circle at a given position, it belongs to that API to reflect the image/screen accordingly. Instead, what you need to test is that when a caller invoked a specific method of *your* code, then your code called a method to draw this or that.

You do this by using fakes or mocks. In other words, when testing a specific class, you won't inject the real API which draws objects on an image/screen, but instead a substitute that you've written specifically for the tests, and which will do no real drawing. Instead, the substitute will just, for instance, record that you called a given method with specific parameters. Later on, your test will request this substitute to tell if the method was called, and succeed or fail depending on that.

---



The answer which has evolved over the last few years is, *you don't apply TDD to the GUI, you design the GUI in such as way that there's a layer just underneath you can develop with TDD.* The Gui is reduced to a trivial mapping of controls to the ViewModel, often with framework bindings, and so is ignored for TDD.

This is known variously as the [Presentation Model](http://martinfowler.com/eaaDev/PresentationModel.html) (Fowler) the [Model-View-ViewModel](http://blogs.msdn.com/johngossman/archive/2005/10/08/478683.aspx) and [DataModel-View-ViewModel](http://robbbloggg.blogspot.com/2008/04/datamodel-view-viewmodel-in-wpf-part-1.html) architecture.

This approach removes the GUI layer from TDD and unit testing. It does not mean the GUI is never tested but just acknowledges that it is not cost effective to pursue automated GUI testing, particularly as part of TDD. Integration and user testing should cover the GUI.

Josh Smith's [2009 WPF article](https://msdn.microsoft.com/en-us/magazine/dd419663.aspx) is a detailed explanation of MVVM with some testing.

More recently, Houssem Dellai's [2016 video Creating Unit Tests for Xamarin Forms Apps](https://channel9.msdn.com/Blogs/MVP-Windows-Dev/Creating-Unit-Tests-for-Xamarin-Forms-Apps) shows a XAML UI with bound ViewModel and walks through creating a unit test project

>  在过去几年中已经发展的答案是，您没有将TDD应用于GUI，您可以设计GUI，以便您可以使用TDD开发层。Gui被简化为对ViewModel的控件的简单映射，通常使用框架绑定，因此对于TDD将被忽略。这被称为Presentation Model（Fowler），Model-View-ViewModel和DataModel-View-ViewModel架构。此方法从TDD和单元测试中删除GUI层。这并不意味着GUI从未经过测试，只是承认追求自动化GUI测试并不符合成本效益，特别是作为TDD的一部分。集成和用户测试应涵盖GUI。Josh Smith的2009 WPF文章详细解释了MVVM的一些测试。最近，Houssem Dellai 2016年视频创建单元测试Xamarin Forms Apps显示了一个带有绑定ViewModel的XAML UI，并逐步创建了一个单元测试项目

所以，最终，我的思路是把UI和逻辑分离，使用的模式是MVVM

[模式 - 使用Model-View-ViewModel设计模式的WPF应用程序](https://msdn.microsoft.com/en-us/magazine/dd419663.aspx)

## what is MVVM

MVVM中的视图和视图模型使用方法，属性和事件进行通信。视图和视图模型之间的双向数据绑定或双向数据绑定可确保视图模型中的模型和属性与视图同步。MVVM设计模式非常适合需要支持双向数据绑定的应用程序。



**If a Model changes state the ViewModel listens and translates that state and that's it.The View is then listening to the ViewModel for state change and it also updates based on the translation from the ViewModel.**

**Model**

The Model represents a set of classes that describes the business logic and data. It also defines business rules for data means how the data can be changed and manipulated.

**View**

The View represents the UI components like CSS, jQuery, html etc. It is only responsible for displaying the data that is received from the controller as the result. This also transforms the model(s) into UI.

**View Model**

The View Model is responsible for exposing methods, commands, and other properties that helps to maintain the state of the view, manipulate the model as the result of actions on the view, and trigger events in the view itself.

> 如果Model 更改状态，ViewModel将侦听并转换该状态。然后，View正在侦听ViewModel以进行状态更改，并且它还会根据ViewModel的转换进行更新。
>
> Model 
>
> Model表示一组描述业务逻辑和数据的类。它还定义了数据的业务规则，意味着如何更改和操作数据。
>
> View
>
> View表示CSS组件，如CSS，jQuery，html等。它仅负责显示从控制器接收的数据作为结果。这也将模型转换为UI。
>
> View Model
>
> 查看模型视图模型负责公开方法，命令和其他属性，这些属性有助于维护视图的状态，操作模型作为视图上的操作的结果，并触发视图本身中的事件。

**Key Points about MVVM Pattern:**

1. User interacts with the View.
2. There is many-to-one relationship between View and ViewModel means many View can be mapped to one ViewModel.
3. View has a reference to ViewModel but View Model has no information about the View.
4. Supports two-way data binding between View and ViewModel.

> 关于MVVM模式的要点：
>
> 1. 用户与视图交互。
> 2. View和ViewModel之间存在多对一关系意味着可以将许多View映射到一个ViewModel。
> 3. View具有对ViewModel的引用，但View Model没有关于View的信息。
> 4. 支持View和ViewModel之间的双向数据绑定。

## MVP pattern looks like

```java
interface Presenter {
    void onSendClicked();
}

interface View {
    String getInput();
    void showProgress();
    void hideProgress();
}

class PresenterImpl implements Presenter {
    // ...ignore other implementations
    void onSendClicked() {
        String input = view.getInput();
        view.showProgress();
        repository.store(input);
        view.hideProgress();
    }
}

class ViewImpl implements View {
    // ...ignore other implementations
    void onButtonClicked() {
        presenter.onSendClicked();
    }

    String getInput() {
        return textBox.getInput();
    }

    void showProgress() {
        progressBar.show();
    }

    void hideProgress() {
        progressBar.hide();
    }
}
```

在MVP中，Presenter包含View的UI业务逻辑。View中的所有调用都直接委托给Presenter。Presenter也直接与View分离，并通过界面与之对话。这是为了允许在单元测试中模拟View。MVP的一个常见属性是必须进行大量的双向调度。例如，当有人单击“保存”按钮时，事件处理程序将委托给Presenter的“OnSave”方法。保存完成后，Presenter将通过其界面回调View，以便View可以显示保存已完成。MVP往往是在Web窗体中实现单独呈现的非常自然的模式。



## 最终结论

使用MVP模式 用于TDD开发，把model 和 presenter层做厚，把view分离，方便测试开发；



## source

[How to apply Test Driven development for GUI application(VC MFC)?](https://stackoverflow.com/questions/382946/how-to-apply-test-driven-development-for-gui-applicationvc-mfc)

[DD for Graphics application](https://softwareengineering.stackexchange.com/questions/376412/tdd-for-graphics-application)

[What is the MVVM architectural pattern and when do you use the MVVM architectural pattern?](https://www.quora.com/What-is-the-MVVM-architectural-pattern-and-when-do-you-use-the-MVVM-architectural-pattern)