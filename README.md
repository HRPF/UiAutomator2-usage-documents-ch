# UiAutomator2 入门

## 关于

[UiAutomator](https://developer.android.com/training/testing/ui-automator.html)是Google提供的用来做安卓自动化测试的一个Java库，基于Accessibility服务。功能很强，可以对第三方App进行测试，获取屏幕上任意一个APP的任意一个控件属性，并对其进行任意操作，但有两个缺点：

- 测试脚本只能使用Java语言  
- 测试脚本要打包成jar或者apk包上传并安装到设备上才能运行  

为简化操作步骤并实现联机操作，因此有了Python uiautomator库，原理是在手机上运行了一个http rpc服务，将uiautomator中的功能开放出来，然后再将这些http接口封装成库。  

UiAutomator2库从`xiaocong/uiautomator`进化升级而来，对原有的库的bug进行了修复，还增加了很多新的Feature。主要有以下部分：

- 设备和开发机可以脱离数据线，通过WiFi互联（基于[atx-agent](https://github.com/openatx/atx-agent)）

- 集成了[openstf/minicap](https://github.com/openstf/minicap)达到实时屏幕投频，以及实时截图

- 集成了[openstf/minitouch](https://github.com/openstf/minitouch)达到精确实时控制设备

- 修复了[xiaocong/uiautomator](https://github.com/xiaocong/uiautomator)经常性退出的问题

- 代码进行了重构和精简，方便维护

- 实现了一个设备管理平台(也支持iOS) [atxserver2](https://github.com/openatx/atxserver2)

- 扩充了toast获取和展示的功能

> uiautomator2 不支持iOS测试，需要iOS自动化测试，可以转到这个库 [openatx/facebook-wda](https://github.com/openatx/facebook-wda)。

> 库 ~~<https://github.com/NeteaseGame/ATX>~~ 已不再维护  

## 安装

### Requirements

- Android 4.4+
- Python 3.6+
- UiAutomator2

> 如果用Python2的pip安装，会安装本库的老版本0.2.3；如果用Python3.5的pip安装，会安装本库的老版本0.3.3；两者均已经不再维护  

### 安装方法

[Python官方下载地址](https://www.python.org/downloads/)  

> Windows平台可在Powershell中输入`python`启动Microsoft Store一键安装  

在使用pip安装UiAutomator2时若网速较慢，建议将pip安装源更换为如下地址，更换方式可参考[PIP 更换国内安装源](https://blog.csdn.net/yuzaipiaofei/article/details/80891108)：
- 阿里云 http://mirrors.aliyun.com/pypi/simple/ 
- 中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/ 
- 豆瓣 http://pypi.douban.com/simple/ 
- 清华大学 https://pypi.tuna.tsinghua.edu.cn/simple/ 
- 中国科学技术大学 http://pypi.mirrors.ustc.edu.cn/simple/

### 完成

准备1台开启了`开发者选项`的安卓手机，连接上电脑，确保执行`adb devices`可以看到连接上的设备。

运行`pip install -U uiautomator2` 安装uiautomator2

运行`python -m uiautomator2 init`安装包含httprpc服务的apk到手机`atx-agent, minicap, minitouch` （在过去的版本中，这一步是必须执行的，但是从1.3.0之后的版本，当运行python代码`u2.connect()`时就会自动安装了）

命令行运行`python`打开python交互窗口。然后将下面的命令输入到窗口中。

```python
import uiautomator2 as u2

d = u2.connect() # connect to device
print(d.info)
```

这时看到类似下面的输出，就说明已经成功安装所需的库了。

```
{'currentPackageName': 'net.oneplus.launcher', 'displayHeight': 1920, 'displayRotation': 0, 'displaySizeDpX': 411, 'displaySizeDpY': 731, 'displayWidth': 1080, 'productName': 'OnePlus5', '
screenOn': True, 'sdkInt': 27, 'naturalOrientation': True}
```

### 辅助工具

- uiautomatorviewer

  这是Android SDK自带的工具，用于获取AndroidUI中的控件属性。在使用UiAutomator时，由于安卓端的uiautomator是独占资源，所以两者无法同时使用，推荐使用下面的工具

- weditor

  基于Python，使用pip安装，可与UiAutomator更好融合。

  安装方法：

  ```bash
  pip install -U weditor
  ```

  安装好之后，就可以在命令行运行`weditor --help` 确认是否安装成功了。

  > Windows系统可以使用命令在桌面创建一个快捷方式 `weditor --shortcut`，如果使用的是conda环境则快捷方式无效

  命令行直接输入 `weditor` 会自动打开浏览器，输入设备的ip或者序列号，点击Connect即可。

  >  参考文章：[浅谈自动化测试工具python-uiautomator2](https://testerhome.com/topics/11357)



## 使用方法详解

> UiAutomator2可搭配使用Python unittest单元测试框架使用，使用方法可参考[unittest --- 单元测试框架 — Python 3.11.1 文档](https://docs.python.org/zh-cn/3/library/unittest.html)



### 与设备连接

1. USB线缆

   假设设备序列号是 `123456f` (通过 `adb devices` 命令查看)

   ```python
   import uiautomator2 as u2
   
   d = u2.connect('123456f') # alias for u2.connect_usb('123456f')
   print(d.info)
   ```

2. Wi-Fi

   假设设备IP是 `10.0.0.1` 并与计算机在同一局域网下

   ```python
   import uiautomator2 as u2
   
   d = u2.connect('10.0.0.1') # alias for u2.connect_wifi('10.0.0.1')
   print(d.info)
   ```

3. Wi-Fi ADB

   需要在开发者选项中打开网络调试，此处假设设备IP是10.0.0.1，adb调试端口是5555

   ```python
   import uiautomator2 as u2
   
   d = u2.connect_adb_wifi("10.0.0.1:5555")
   
   # 相当于
   # + Shell: adb connect 10.0.0.1:5555
   # + Python: u2.connect_usb("10.0.0.1:5555")
   ```
   > 若不添加参数地调用u2.connect()，UiAutomator2会尝试从环境变量`ANDROID_DEVICE_IP` 或 `ANDROID_SERIAL` 中获取设备序列号。
   > If this environment variable is empty, uiautomator will fall back to `connect_usb` and you need to make sure that there is only one device connected to the computer.



### 熟悉Android常用UI控件



### 通过坐标的手势交互

| 手势         | 函数调用           | 必选参数                                 | 可选参数   | 示例                                                     | 补充                                  |
| ------------ | ------------------ | ---------------------------------------- | ---------- | -------------------------------------------------------- | ------------------------------------- |
| 点击         | `d.click()`        | `x, y`（px）                             |            | `d.click(50, 100)`                                       | 物理像素点定位                        |
|              |                    | `%x, %y` （%）                           |            | `d.click(0.1, 0.2)`                                      | 百分比定位，基于屏幕大小              |
| 双击         | `d.double_click()` | `x,y` (px/%)                             | `duration` | `d.double_click(50, 100, 0.2)`                           | 默认点击间隔为100ms                   |
| 长按         | `d.long_click()`   | `x,y` (px/%)                             | `duration` | `d.long_click(50, 100, duration=0.5)`                    | 默认持续时间为500ms                   |
| 滑动         | `d.swipe()`        | `startX,startY,endX,endY`                | `duration` | 同上                                                     | 默认持续时间为500ms                   |
| 滑动扩展功能 | `d.swipe_ext()`    | `"right"` / `"up"` / `"left"` / `"down"` |            | `d.swipe_ext("up")`                                      | 上划                                  |
|              |                    |                                          | `scale`    | `d.swipe_ext("up", scale=0.9)`                           | 默认滑动距离为屏幕高度（宽度）的90%   |
|              |                    |                                          | `box`      | `d.swipe_ext("up", box=(0,0,100,100))`                   | 在 (0,0) -> (100, 100) 这个区域做滑动 |
| 按住并拖动   | `d.drag()`         | `startX,startY,endX,endY`                | `duration` | `d.drag(0,0,100,100,duration=0.5)``d.drag(0,0,100,100)`  | 默认执行时长为0.5秒                   |
| 九宫格解锁   | `d.swipe_points()` | 由点坐标（元组/列表，px/%）组成的列表    | `duration` | `d.swipe_points([(0, 0), (100, 100), (200, 200)], 0.5))` | 默认执行时长为0.5秒                   |



### 控件定位

#### 支持的选择器

ui2支持 android 中 UiSelector 类中的所有定位方式，详细可以在这个网址查看https://developer.android.com/reference/android/support/test/uiautomator/UiSelector

整体内容如下,所有的属性可以通过weditor查看到。

| 名称                  | 描述                                            |
| --------------------- | ----------------------------------------------- |
| text                  | text是指定文本的元素                            |
| textContains          | text中包含有指定文本的元素                      |
| textMatches           | text符合指定正则的元素                          |
| textStartsWith        | text以指定文本开头的元素                        |
| className             | className是指定类名的元素                       |
| classNameMatches      | className类名符合指定正则的元素                 |
| description           | description是指定文本的元素                     |
| descriptionContains   | description中包含有指定文本的元素               |
| descriptionMatches    | description符合指定正则的元素                   |
| descriptionStartsWith | description以指定文本开头的元素                 |
| checkable             | 可检查的元素，参数为True，False                 |
| checked               | 已选中的元素，通常用于复选框，参数为True，False |
| clickable             | 可点击的元素，参数为True，False                 |
| longClickable         | 可长按的元素，参数为True，False                 |
| scrollable            | 可滚动的元素，参数为True，False                 |
| enabled               | 已激活的元素，参数为True，False                 |
| focusable             | 可聚焦的元素，参数为True，False                 |
| focused               | 获得了焦点的元素，参数为True，False             |
| selected              | 当前选中的元素，参数为True，False               |
| packageName           | packageName为指定包名的元素                     |
| packageNameMatches    | packageName为符合正则的元素                     |
| resourceId            | resourceId为指定内容的元素                      |
| resourceIdMatches     | resourceId为符合指定正则的元素                  |

#### 选择器定位示例

```element = d(上述列表中的属性=字符串值)```

例：```element = d(resourceId="com.google.android.dialer:id/dialpad_fab")```

> 返回类型为 `<class 'uiautomator2._selector.UiObject'>`
>
> 例如`<uiautomator2._selector.UiObject object at 0x000001D938F596D0>` (即使找不到元素也返回值)

#### 判断控件是否存在

```d(上述列表中的属性=字符串值).exists()```

> 返回类型为bool型



### 高级控件定位

#### 子元素定位

`d(上述列表中的属性=字符串值).child()`

`d(上述列表中的属性=字符串值).child_by_text()`

`d(上述列表中的属性=字符串值).child_by_description()`

`d(上述列表中的属性=字符串值).child_by_instance()`

例：

```python
# 查找类名为android.widget.ListView下的Bluetooth元素
d(className="android.widget.ListView").child(text="Bluetooth")
```

> `child_by_description` is to find children whose grandchildren have
> the specified description, other parameters being similar to `child_by_text`.

> `child_by_instance` is to find children with has a child UI element anywhere
> within its sub hierarchy that is at the instance specified. It is performed
> on visible views **without scrolling**.



#### 兄弟元素定位

`d(上述列表中的属性=字符串值).sibling()`

```python
#查找与google同一级别，类名为android.widget.ImageView的元素
d(text="Google").sibling(className="android.widget.ImageView")
```



#### 相对位置定位

相对定位支持`left, right, top, bottom`即在某个元素的前后左右

```python
d(A).left(B)  # 选择A左边的B
d(A).right(B)  # 选择A右边的B
d(A).up(B)  # 选择A上边的B
d(A).down(B)  # 选择A下边的B

# 选择 WIFI 右边的开关按钮
d(text='Wi‑Fi').right(resourceId='android:id/widget_frame')
```



#### 多属性定位

在定位控件时添加多个属性参数即可

```python
d(className="android.widget.ImageView", resourceId="xxxxxxxx")
```



#### 链式定位

```python
d(className="android.widget.ListView", resourceId="android:id/list") \
  .child_by_text("Wi‑Fi", className="android.widget.LinearLayout") \
  .child(className="android.widget.Switch") \
  .click()
```



#### 多个相同属性控件定位

有时屏幕可能包含多个具有相同属性的控件，例如text属性，那么必须使用选择器中的 "instance"属性来选择一个合格的实例，如下所示。

```python
d(text="Add new", instance=0)  # which means the first instance with text "Add new"
```

此外，uiautomator2提供了一个类似列表的API（类似于jQuery）：

```python
# get the count of views with text "Add new" on current screen
d(text="Add new").count

# same as count property
len(d(text="Add new"))

# get the instance via index
d(text="Add new")[0]
d(text="Add new")[1]
...

# iterator
for view in d(text="Add new"):
    view.info
```

>  **Notes**: when using selectors in a code block that walk through the result list, you must ensure that the UI elements on the screen keep unchanged. Otherwise, when Element-Not-Found error could occur when iterating through the list.



#### XPATH定位

Java uiautoamtor中默认是不支持xpath的，所以这里属于扩展的一个功能。速度不是这么的快。

详情见[XPATH.md](XPATH.md)

For example: 其中一个节点的内容

```xml
<android.widget.TextView
  index="2"
  text="05:19"
  resource-id="com.netease.cloudmusic:id/qf"
  package="com.netease.cloudmusic"
  content-desc=""
  checkable="false" checked="false" clickable="false" enabled="true" focusable="false" focused="false"
  scrollable="false" long-clickable="false" password="false" selected="false" visible-to-user="true"
  bounds="[957,1602][1020,1636]" />
```

xpath定位和使用方法

有些属性的名字有修改需要注意

```
description -> content-desc
resourceId -> resource-id
```

常见用法

```python
# wait exists 10s
d.xpath("//android.widget.TextView").wait(10.0)

# find and click
d.xpath("//*[@content-desc='分享']").click()

# check exists
if d.xpath("//android.widget.TextView[contains(@text, 'Se')]").exists:
    print("exists")
    
# get all text-view text, attrib and center point
for elem in d.xpath("//android.widget.TextView").all():
    print("Text:", elem.text)
    # Dictionary eg: 
    # {'index': '1', 'text': '999+', 'resource-id': 'com.netease.cloudmusic:id/qb', ......}
    print("Attrib:", elem.attrib)
    # Coordinate eg: (100, 200)
    print("Position:", elem.center())
```



### 控件交互

#### 获取控件信息

| 方法             | 描述                 | 返回类型 | 备注             | 示例                                                         |
| ---------------- | -------------------- | -------- | ---------------- | ------------------------------------------------------------ |
| exists()         | 判断元素是否存在     | bool     | @property        | `e_ex = d(text="abc").exists`                                |
| info()           | 返回元素的所有信息   | dict     | @property        | `e_info = d(text="abc").info`                                |
| get_text()       | 返回元素文本         | str      |                  | `e_text = d(text="abc").get_text()`                          |
| set_text("text") | 设置元素文本         | None     |                  | `d(text="abc").set_text("text")`                             |
| clear_text()     | 清空元素文本         | None     |                  | `d(text="abc").clear_text()`                                 |
| center()         | 返回元素的中心点位置 | (x,y)    | 基于整个屏幕的点 | `e_center = d(text="abc").center()`                          |
| screenshot()     | 获取控件的屏幕截图   |          |                  | `im = d(text="Settings").screenshot()`<br>`im.save("settings.jpg")` |

> @property表示将方法视为属性，可直接获取，例如`d(text="abc").exists` 而非 `d(text="abc").exists()`



#### 点击控件



#### 控件的手势交互



### Watcher

