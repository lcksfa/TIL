# 网易UI库资源管理

这里我们先聊聊资源管理,
要管理资源,我们需要先获取资源的路径,一般来说,我们在debug时使用文件形式的资源,而在release时,则把资源打包为zip文件.
这里就是如此

```c++
// 获取资源路径，初始化全局参数
	std::wstring theme_dir = QPath::GetAppPath();
#ifdef _DEBUG
	// Debug 模式下使用本地文件夹作为资源
	// 默认皮肤使用 resources\\themes\\default
	// 默认语言使用 resources\\lang\\zh_CN
	// 如需修改请指定 Startup 最后两个参数
	ui::GlobalManager::Startup(theme_dir + L"resources\\", ui::CreateControlCallback(), false);
#else
	// Release 模式下使用资源中的压缩包作为资源
	// 资源被导入到资源列表分类为 THEME，资源名称为 IDR_THEME
	// 如果资源使用的是本地的 zip 文件而非资源中的 zip 压缩包
	// 可以使用 OpenResZip 另一个重载函数打开本地的资源压缩包
	ui::GlobalManager::OpenResZip(MAKEINTRESOURCE(IDR_THEME), L"THEME", "");
	// ui::GlobalManager::OpenResZip(L"resources.zip", "");
	ui::GlobalManager::Startup(L"resources\\", ui::CreateControlCallback(), false);
#endif
```

通过 ui::GlobalManager 管理和解析资源,Startup函数用于开启资源大门,
其原型说明:

```c++
static void Startup(const std::wstring& strResourcePath, const CreateControlCallback& callback, bool bAdaptDpi, const std::wstring& theme = L"themes\\default", const std::wstring& language = L"lang\\zh_CN")

// - 参数：  
//    - `strResourcePath` 资源路径位置
//    - `callback` 创建自定义控件时的全局回调函数
//    - `bAdaptDpi` 是否启用 DPI 适配
//    - `theme` 主题目录名，默认为 themes\\default
 //   - `language` 使用语言，默认为 lang\\zh_CN
 //- 返回值：无
```

通过Startup实现源码,其流程如下

- 资源路径设置
- dpi配置
- 解析全局资源global.xml
- 加载多语言
- 开启gdiplus
- InitCommonControls

由此可知,NIMUI使用了Gdiplus绘制接口 ,支持高DPI,多语言等

以下重点说说全局资源global.xml,这个算是NIMUI里面的独创,在global.xml中定义了全局的样式,避免在多个不同的 XML 中出现相同的描述而产生冗余的代码和消耗开发人员在界面设上的时间。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Global>
    ....
</Global>
```
可以在里面 添加 的样式 有字体Font,颜色TextColor,以及通用样式Class,要注意的是，`class` 属性必须在所有属性最前面，当你需要覆盖一个通用样式中指定过的属性时，只需要在 `class` 属性之后再重新定义这个属性就可以了

那么global.xml如何解析的 呢?
```c++
void GlobalManager::LoadGlobalResource()
{
	ui::WindowBuilder dialog_builder;
	ui::Window paint_manager;
	dialog_builder.Create(L"global.xml", CreateControlCallback(), &paint_manager);
}
```
使用 WindowBuilder类 Create处理的.

一下是解析Global的代码

```c++
if( strClass == _T("Global") )
		{
			int nAttributes = root.GetAttributeCount();
			for( int i = 0; i < nAttributes; i++ ) {
				strName = root.GetAttributeName(i);
				strValue = root.GetAttributeValue(i);
				if( strName == _T("disabledfontcolor") ) {
					GlobalManager::SetDefaultDisabledTextColor(strValue);
				} 
				else if( strName == _T("defaultfontcolor") ) {	
					GlobalManager::SetDefaultTextColor(strValue);
				}
				else if( strName == _T("linkfontcolor") ) {
					DWORD clrColor = GlobalManager::GetTextColor(strValue);
					GlobalManager::SetDefaultLinkFontColor(clrColor);
				} 
				else if( strName == _T("linkhoverfontcolor") ) {
					DWORD clrColor = GlobalManager::GetTextColor(strValue);
					GlobalManager::SetDefaultLinkHoverFontColor(clrColor);
				} 
				else if( strName == _T("selectedcolor") ) {
					DWORD clrColor = GlobalManager::GetTextColor(strValue);
					GlobalManager::SetDefaultSelectedBkColor(clrColor);
				}
			}
		}
        
        if( strClass == _T("Global") ) {
			for( CMarkupNode node = root.GetChild() ; node.IsValid(); node = node.GetSibling() ) {
				strClass = node.GetName();
				if( strClass == _T("Image") ) {
					ASSERT(FALSE);	//废弃
				}
				else if( strClass == _T("Font") ) {
					nAttributes = node.GetAttributeCount();
					std::wstring strFontName;
					int size = 12;
					bool bold = false;
					bool underline = false;
					bool italic = false;
					for( int i = 0; i < nAttributes; i++ ) {
						strName = node.GetAttributeName(i);
						strValue = node.GetAttributeValue(i);
						if( strName == _T("name") ) {
							strFontName = strValue;
						}
						else if( strName == _T("size") ) {
							size = _tcstol(strValue.c_str(), &pstr, 10);
						}
						else if( strName == _T("bold") ) {
							bold = (strValue == _T("true"));
						}
						else if( strName == _T("underline") ) {
							underline = (strValue == _T("true"));
						}
						else if( strName == _T("italic") ) {
							italic = (strValue == _T("true"));
						}
						else if( strName == _T("default") ) {
							ASSERT(FALSE);//废弃
						}
					}
					if( !strFontName.empty() ) {
						GlobalManager::AddFont(strFontName, size, bold, underline, italic);
					}
				}
				else if( strClass == _T("Class") ) {
					nAttributes = node.GetAttributeCount();
					std::wstring strClassName;
					std::wstring strAttribute;
					for( int i = 0; i < nAttributes; i++ ) {
						strName = node.GetAttributeName(i);
						strValue = node.GetAttributeValue(i);
						if( strName == _T("name") ) {
							strClassName = strValue;
						}
						else if( strName == _T("value") ) {
							strAttribute = strValue;
						}
					}
					if( !strClassName.empty() ) {
						GlobalManager::AddClass(strClassName, strAttribute);
					}
				}
				else if( strClass == _T("TextColor") ) {
					nAttributes = node.GetAttributeCount();
					std::wstring strColorName;
					std::wstring strColor;
					for( int i = 0; i < nAttributes; i++ ) {
						strName = node.GetAttributeName(i);
						strValue = node.GetAttributeValue(i);
						if( strName == _T("name") ) {
							strColorName = strValue;
						}
						else if( strName == _T("value") ) {
							strColor = strValue;
						}
					}
					if( !strColorName.empty()) {
						GlobalManager::AddTextColor(strColorName, strColor);
					}
				}
			}
		}

```

也就是说,通过这种方式,NIMUI将设置在Global.xml中的各种样式 第一时间注入到了全局内存中,通过类GlobalManager保存起来了.
后面的xml将可以直接使用!

说完了全局的xml的加载,再来看看主窗口的资源管理和解析

以basic例子 说明 ,里面在UI线程 create了一个BasicForm的窗口,该窗口类继承于WindowImplBase类,有3个接口 是必须覆写的,GetSkinFolder和GetSkinFile分别是获取资目录(路径)和资源(xml)的名称,GetWindowClassName则是设置这个窗口的唯一的名称.

接着,我们需要知道这个窗口的创建过程了:
因为其调用Create函数创建窗口,
WindowImplBase : public Window, public IUIMessageFilter
这个Create函数应该追溯到window基类了,
create的过程如下:
RegisterWindowClass 注册窗口 ,其窗口过程处理函数为__WndProc
CreateWindowEx 调用windows api创建窗口

我们知道windows 会把消息都让该窗口的过程函数回调处理
---->在该回调函数中 ,会调用 HandleMessage 处理正常消息
-----> 其具体调用 DoHandlMessage 
----> 先遍历 m_aMessageFilters 使用HandleMessage处理所有消息,本质是IUIMessageFilter* 类型的消息过滤器向量组
接着处理WM 消息,对于 大部分消息,都是通过鼠标位置FindControl,之后调用控件的HandleMessageTemplate处理具体的控件消息,

具体 在
[网易UI库源码分析(三)-消息机制](https://app.yinxiang.com/shard/s66/nl/14816126/e05d34da-0a6f-4478-b799-59cbc48f518e)
记录.

回到资源管理上,我们知道,在window创建时,WindowImplBase 重写了HandleMessage,它会预处理WM_CREATE 消息,对应OnCreate函数,
这个函数加载了对应的窗口的xml
````c++
    WindowBuilder builder;
	std::wstring strSkinFile = GetWindowResourcePath() + GetSkinFile();

	auto callback = nbase::Bind(&WindowImplBase::CreateControl, this, std::placeholders::_1);
	auto pRoot = (Box*)builder.Create(strSkinFile.c_str(), callback, this);
```

WindowBuilder 将解析所有的xml node元素,按node创建对应的控件,pRoot为所以控件的根容器节点指针.

pRoot = m_shadow.AttachShadow(pRoot);  添加阴影.

AttachDialog(pRoot); 添加对话控件绑定窗口的顶层容器

---> __FindControlFromNameHash 会将本窗口添加到 m_mNameHash 表中

InitWindow;用于重载 的窗口初始化函数,一般子类的 控件指针 在这里获取.

---

继续分析  WindowBuilder::Create 的过程:

```c++
Box* WindowBuilder::Create(STRINGorID xml, CreateControlCallback pCallback, 
	Window* pManager, Box* pParent, Box* pUserDefinedBox)
```
使用 CMarkup 类 解析xml文件 m_xml

xml资源 有两种形式,一种是文件形式,另一种是 zip包,当然,也可以是VS资源包的形式,不过在这里都是 zip形式的,需要从内存中加载.
其过程 分别对应了使用CMarkup 的Load 或者 LoadFromFile 方法,

- 先是把文件中的数据拷贝到内存中
```c++
  m_pstrXML = static_cast<LPTSTR>(malloc(cchLen * sizeof(TCHAR)));
    ::CopyMemory(m_pstrXML, pstrXML, cchLen * sizeof(TCHAR));
```
m_pstrXML 就是后面需要解析的资源字符串 它根据文件的大小,使用CopyMemory完成拷贝.

- 调用_Parse完成分析已加载的xml的过程(todo,没有弄明白具体 过程)

m_xml 把文件资源获取后,开始调用`Create(CreateControlCallback pCallback, Window* pManager, Box* pParent, Box* pUserDefinedBox)`真正解析xmlmakeup后的nodes:
整个过程是递归的,

递归时,在WindowBuilder中创建控件,这里调用了WindowBuilder的_Parse函数,其参数是**CMarkup**的每一个node节点.

CreateControlByClass
GlobalManager::CreateControl(strClass)

pControl->SetWindow(pManager);

如此,窗口以及其控件的相关指针和内存就准备好了.

接下来,我们看看,控件是如何绘制的;

这里涉及到 整个图形渲染层,

整个图形采用了gdiplus绘制,
在绘制工具的Factory中,创建了各式各样的工具.
其中 CreateRenderContext 设备上下文 比较关键

void Window::Paint()  窗口的绘制消息响应

```c++
// 绘制
AutoClip rectClip(m_renderContext.get(), rcPaint, true);
CPoint ptOldWindOrg = m_renderContext->OffsetWindowOrg(m_renderOffset);
m_pRoot->Paint(m_renderContext.get(), rcPaint);
m_pRoot->PaintChild(m_renderContext.get(), rcPaint);
m_renderContext->SetWindowOrg(ptOldWindOrg);
```
这里的m_pRoot类型是Box 控件的,分别调用自己的Paint和PaintChild函数,而这两个函数都是可以重载的.

class UILIB_API Box : public Control .使用的就是Control的Paint,

PaintChild函数

```c++
void Control::Paint(IRenderContext* pRender, const UiRect& rcPaint)
{
    if( !::IntersectRect(&m_rcPaint, &rcPaint, &m_rcItem) ) return;

	PaintBkColor(pRender);
	PaintBkImage(pRender);
	PaintStatusColor(pRender);
	PaintStatusImage(pRender);
	PaintText(pRender);
	PaintBorder(pRender);
}
```
```c++
void Box::PaintChild(IRenderContext* pRender, const UiRect& rcPaint)
{
	UiRect rcTemp;
	if( !::IntersectRect(&rcTemp, &rcPaint, &m_rcItem) ) return;

	for (auto it = m_items.begin(); it != m_items.end(); it++) {
		Control* pControl = *it;
		if( !pControl->IsVisible() ) continue;
		pControl->AlphaPaint(pRender, rcPaint);
	}
}
```
AlphaPaint:


alpha渲染绘制 子控件
>
> 最高位的一个byte表示alpha值，它并不代表实际的颜色，它是控制颜色的百分比。范围是0x00 - 0xff，也就是十进制的0-255。它可以控制256个级别的透明程度，0表示完全透明，就是什么也看不见了。255表示完全不透明。随后的三个字节分别表示红绿蓝三原色。那么alpha值是如何工作的呢？如果开启了alpha混合，假设指定了颜色为0x80ff0000，可知alpha值为0x80（十进制128），红色=0xff（十进制255），为满色，绿色和蓝色都为0。于是最终的颜色就是（128 / 255）* 255 = 128，也就是说，透明程度是原来红色的50%。

经过这些过程,控件实际就绘制好了,整个过程里的控件有别于MFC的控件,是没有句柄的,只是在一个窗口里使用各种画笔绘制的点线矩形,以及填色,而已.

而,这些控件是通过用户的点击时鼠标的位置消息,找到这些个控件,通过发送自定义消息(UI消息),解析后实现响应的.

这个过程 ,我在消息机制里 记录.