# 消息机制
## 消息泵
在 总览 中,我们分析了程序启动时 会启动一个UI线程;
UI线程执行一个消息循环:
我们知道,windows的消息驱动 主要就是两个函数:
::TranslateMessage(&msg);
::DispatchMessage(&msg);
的while循环;
整合过程在`GlobalManager::MessageLoop()`中;
可是,这里,我突然发现NIMUI并没有使用windows的这套传统的东西;

而是使用了base库的消息pump机制;这里我们将进入base库的线程框架下了:
根据启动时的线程类型 nbase::MessageLoop::kUIMessageLoop
----->FrameworkThread::Run()
我们可以知道UI线程的实现类为UIMessageLoop  调用RunWithDispatcher ---> RunInternal

```c++
//RunInternal
if (state_->dispatcher && type() == kUIMessageLoop)
	{
		static_cast<WinUIMessagePump *>(pump_.get())->
			RunWithDispatcher(this, state_->dispatcher);
		return;
	}
```
MessagePump

UI线程时,将调用 WinUIMessagePump 的RunWithDispatcher;
WinUIMessagePump 在构造时 会 void WinUIMessagePump::InitMessageWnd()新建一个用于接收自定义
消息的隐藏窗口`message_hwnd_ = ::CreateWindowW(kWndClass, 0, 0, 0, 0, 0, 0, HWND_MESSAGE, 0, hinst, 0);`
其窗口过程 为`WndProcThunk`.
主要分为kMsgHaveWork和WM_TIMER消息,其中kMsgHaveWork为 WM_USER + 1
```c++
case kMsgHaveWork:
		reinterpret_cast<WinUIMessagePump*>(wparam)->HandleWorkMessage();
		break;
case WM_TIMER:
    reinterpret_cast<WinUIMessagePump*>(wparam)->HandleTimerMessage();
    break;
```
WinUIMessagePump继承于WinMessagePump
----> WinMessagePump::RunWithDispatcher

`WinMessagePump::RunWithDispatcher`  ---> `DoRunLoop();`

void WinUIMessagePump::DoRunLoop()-->
void DefaultMessagePump::Run(Delegate* delegate)

----> ProcessNextWindowsMessage

----> 最后调用`ProcessMessageHelper`

```c++
if (state_->dispatcher)
       {
              if (!state_->dispatcher->Dispatch(msg))

                      state_->should_quit = true;
       }
      else
       {
              TranslateMessage(&msg);
              DispatchMessage(&msg);
       }


```

UI消息泵如果使用消息派发器，那么将不再使用经典的TranslateMessage/DispatchMessage模式而改由
Dispatcher来完成.
如果配发器为空,则依然调用windwows的 TranslateMessage(&msg)和DispatchMessage(&msg);
bool MessageLoop::DoWork()

每个线程 都有三个工作状态 工作DoWork,等待DoIdleWork,延时DoDelayedWork

是的,目前消息转发还是使用的是windows经典的机制,然后,我们看看窗口的处理过程:

## 窗口过程
在HWND Window::Create时,注册窗口过程函数`__WndProc`,内部使用CreateWindowEx创建window.

__WndProc作为系统的消息处理回调函数,处理接收的系统消息,窗口创建完毕后
调用HandleMessage 处理具体的消息,否则调用DefWindowProc

HandleMessage  调用`DoHandlMessage`处理
在DoHandlMessage 中,先遍历消息过滤器的m_aMessageFilters,先由m_aMessageFilters调用
MessageHandler处理一次,根据handled的值决定是否继续处理该消息,m_aMessageFilters是由
接口类IUIMessageFilter的纯虚函数,派生类必须重写本MessageHandler方法,该方法一般将handled
置为false,表示还需要DoHandlMessage 继续处理.可以理解成一个拦路虎,先抢掠一番,但还是放消息过去,
当然,也有不放消息过去的情况.这个m_aMessageFilters可以理解为一个自定义的窗口子类.

一般言,继承Window类 就可以做出一个具体的windows窗口了,不过消息的预处理还差强人意了,本着开闭原则,
将实现的接口预留给类的调用者以实现扩展,这是个不错的思路,
作为windows桌面软件,最重要的就是消息的处理了,所以在类window中,预留了两个消息处理的过滤器
m_aPreMessageFilters和m_aMessageFilters,通过AddPreMessageFilter 和AddMessageFilter添加它们;

m_aMessageFilters对应的处理消息函数是DoHandlMessage 
而m_aPreMessageFilters对应PreMessageHandler,执行派发消前的过滤器,我们将在全局的窗口管理类中看到它.

以上是消息的处理相应,NIM_DUI将其余的复杂的消息全部干掉了,化繁为简,是一个好的风格,毕竟,我实在受不了各种混杂的
消息pump和收发方式了,类MFC的更是让人头大.

## 消息的分发
### 1.发起
窗口建立后,需要有一个loop驱动消息的接收和转发,这个过程在GlobalManager下:

在使用NIMUI时,我们会第一时间创建一个UI线程,派生于FrameworkThread ,,
该UI线程执行一个消息循环RunOnCurrentThreadWithLoop,驱动整
个程序的进行, Run时 根据不同的loop类型创建不同的消息loop类.
UI loop开始 RunWithDispatcher---> RunInternal--->RunWithDispatcher(消息泵--)---->
DoRunLoop

UI消息泵如果使用消息派发器，那么将不再使用经典的TranslateMessage/DispatchMessage模式而改由Dispatcher来完成.

可以这样理解,传统的windows消息先使用 窗口过程函数来处理,处理时会发送UI自定义的消息去分发给控件.控件再来做出具体的响应.
以下,我们分析下具体的过程:

window类的窗口过程的Windows消息处理 在` Window::DoHandlMessage`里,
我们从WM_LBUTTONDOWN,鼠标按下这个消息开始解析起:
```c++
case WM_LBUTTONDOWN:
	{
		if (m_pEventTouch != NULL || m_pEventPointer != NULL)
			break;

		// We alway set focus back to our app (this helps
		// when Win32 child windows are placed on the dialog
		// and we need to remove them on focus change).
		::SetFocus(m_hWnd);
		POINT pt = { GET_X_LPARAM(lParam), GET_Y_LPARAM(lParam) };
		m_ptLastMousePos = pt;
		Control* pControl = FindControl(pt);
		if (pControl == NULL) break;
		if (pControl->GetWindow() != this) break;
		m_pEventClick = pControl;
		pControl->SetFocus();
		SetCapture();

		pControl->HandleMessageTemplate(kEventMouseButtonDown, wParam, lParam, 0, pt);
	}
```

其过程大体是:

- 根据鼠标坐标FindControl控件
- 将控件设置焦点
- 捕获
- 使用Control的HandleMessageTemplate 分发kEventMouseButtonDown消息

### 2.消息转化
HandleMessageTemplate 先将传统 Windows 消息转换为自定义格式的消息EventArgs,

```c++
void Control::HandleMessageTemplate(EventType eventType, WPARAM wParam, LPARAM lParam, TCHAR tChar, CPoint mousePos, FLOAT pressure)
{
	EventArgs msg;
	msg.pSender = this;
	msg.Type = eventType;
	msg.chKey = tChar;
	msg.wParam = wParam;
	msg.lParam = lParam;
	msg.pressure = pressure;
	if (0 == mousePos.x == mousePos.y)
		msg.ptMouse = m_pWindow->GetLastMousePos();
	else
		msg.ptMouse = mousePos;
	msg.dwTimestamp = ::GetTickCount();

	HandleMessageTemplate(msg);
}
```
再调用同名的HandleMessageTemplate函数处理该消息;

### 3.OnEvent
在Control的处理中,引入了两个成员的 EventMap,分别是 OnXmlEvent和OnEvent;
这两个EventMap 算得上NUMUI的精华所在了.

void Control::HandleMessageTemplate(EventArgs& msg):
```c++
auto callback = OnEvent.find(msg.Type);
if (callback != OnEvent.end()) {
    bRet = callback->second(&msg);
}
```

```c++
auto callback = OnXmlEvent.find(msg.Type);
if (callback != OnXmlEvent.end()) {
    bRet = callback->second(&msg);
}
```
二者的使用方式是一样的,使用两个的原因应该是为了做区分使用 场景

EventMap的原型:
`typedef std::map<EventType, CEventSource> EventMap;

}`

其中 EventType是UI库自定义的消息类型,而CEventSource的定义如下:
```c++
typedef std::function<bool(ui::EventArgs*)> EventCallback;

class CEventSource : public std::vector<EventCallback>
```
从本质看,CEventSource其实是一组`bool(ui::EventArgs*)`类型的回调函数,
` bRet = callback->second(&msg);`调用既是把EventCallback类型的函数一一执行了;
这里有明确的()操作符的重载:
```c++
//class CEventSource : 
bool operator() (ui::EventArgs* param) const 
{
    for (auto it = this->begin(); it != this->end(); it++) {
        if(!(*it)(param)) return false;
    }
    return true;
}

```

理解了代码,我们就知道了OnEvent的使用方式,也就是通过回调的方式添加某种或多种自定义消息的处理方式;

OnXmlEvent与之同理,
二者的使用场景 一个用于代码中添加处理方式,一个用于在xml中添加,不得不说,考虑得十分周全.

使用方式,比如要添加鼠标点击的处理;
在button控件中,有如下方法:
```c++
//class UILIB_API ButtonTemplate : public LabelTemplate<InheritType>
void AttachClick(const EventCallback& callback)	{ this->OnEvent[kEventClick] += callback;	}
```
在具体的类中可以这样写:
```c++
ui::Button* settings = dynamic_cast<ui::Button*>(FindControl(L"settings"));
settings->AttachClick([this](ui::EventArgs* args) {
    RECT rect = args->pSender->GetPos();
    ui::CPoint point;
    point.x = rect.left - 175;
    point.y = rect.top + 10;
    ClientToScreen(m_hWnd, &point);

    ui::CMenuWnd* pMenu = new ui::CMenuWnd(NULL);
    ui::STRINGorID xml(L"settings_menu.xml");
    pMenu->Init(xml, _T("xml"), point);
    return true;
});
```

通过匿名函数,很好的实现了点击消息处理的闭包.说的更明白些,这里的匿名函数实际就是上面HandleMessageTemplate里的
` bRet = callback->second(&msg);` 的回调函数,即second的调用.
而event的处理 在Contrl的子类中  还会继续拓展.
`	virtual void HandleMessageTemplate(EventArgs& msg) override;`
是一个可以被覆写的虚函数.
比如在Box子类中,又加入了 OnBubbledEvent和OnXmlBubbledEvent事件map;这两个的使用 ,后面补充;;

//todo



event事件消息处理完毕后 会继续使用`Control::HandleMessage(EventArgs& msg)`处理消息,除非bRet为false.


整个流程看完,似乎也就完了;


### 等等,我们似乎漏了什么东西:

在window类的消息处理DoHandlMessage中 有如下过程:
```c++
//Window::DoHandlMessage
// Cycle through listeners
	for (auto it = m_aMessageFilters.begin(); it != m_aMessageFilters.end(); it++)
	{
		BOOL bHandled = false;
		LRESULT lResult = (*it)->MessageHandler(uMsg, wParam, lParam, bHandled);
		if (bHandled && uMsg != WM_MOUSEMOVE) {
			handled = true;
			return lResult;
		}
	}
```
1. AddMessageFilte

这里是指会遍历所有的消息监听者,调用监听者的MessageHandler.貌似是用了观察者模式;

m_aMessageFilters是通过 AddMessageFilter添加的UIMessageFilter类型的消息过滤器,`此时消息已经派发`

这个类型需要实现MessageHandler方法,在这个NIMUI中,我只看到了ShadowWndBase会使用这个AddMessageFilter方法,
由此可见,这个消息处理是用于朱窗口处理之前,而阴影效果的处理恰如其分.

2. AddPreMessageFilter
而另一个类似的则是,添加一个消息被派发到窗口前的消息过滤器AddPreMessageFilter,该方法在窗口WindowImplBase创建时调用了,对应的处理函数是
PreMessageHandler,该函数在GlobalManager::TranslateMessage中调用.
如此,我们引出了另一个loop,即`void GlobalManager::MessageLoop()`,通过PreMessageHandler,我们可以把他们联系起来了;
在消息派发前我们通过`GlobalManager::TranslateMessage(&msg) `先转译预处理下;这里会监控所以的注册window,遍历后,调用window类的PreMessageHandler进行处理.


综合上面的分析,使用UI线程的框架时 AddPreMessageFilter 并没有起到应有的作用.


## 自定义控件消息的响应

控件的继承体系如下:


nbase::SupportWeakCallback  弱引用的回调类,可能是为了防止回调时原控件被销毁导致崩溃,还没有证实


PlaceHolder  占位基类,这个类名取得不错


Control  控件基类
主要有三大类函数,,一个是绘制自己,一个是响应用户的交互消息的处理最后则是自定义消息的关联
Attach* 系列.

- 绘制属性设置和绘制相关的 暂时不提
- Attach* 系列是把消息和处理回调函数进行关联
- HandleMessageTemplate系则是消息的处理

处理和关联 之前已经分析了.见 3.OnEvent 相关

也就是说,当我们想要设置某个控件的消息处理函数时,调用Attack* 系列函数即可.