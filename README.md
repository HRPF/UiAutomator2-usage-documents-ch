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
- 豆瓣(douban) http://pypi.douban.com/simple/ 
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
2. Wi-Fi
3. Wi-Fi ADB