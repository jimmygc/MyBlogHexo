---
layout: post
title: 关于最近学习内核驱动中使用ETW的记录
published: true
comment: true
---



**ETW**（Event Tracing for Windows）是微软的日志记录系统。与**WPP**不同的是，ETW多用于系统管理而WPP用于开发人员开发过程中的Debug。:smile:

使用ETW需要生成XML格式的描述事件的XML或MAN声明文件，由声明文件用Windows kit自带的mc（message compiler）程序编译生成资源文件rc，头文件以及包含message信息的bin文件。
MSDN官方文章链接：[Click ME](https://msdn.microsoft.com/en-us/library/windows/hardware/ff541236%28v=vs.85%29.aspx)

 - 生成MAN
 
MAN文件可以手写也可以用Windwos Kit里的ECManGen程序生成。

一个MAN文件中至少要有一个**Provider**，一个**template**，一个**event**，一个**channel**，其他项都是可选的，用于管理中方便事件的归类整理。下面提供一个最小化的MAN文件例子。

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes'?>
<instrumentationManifest
    xmlns="http://schemas.microsoft.com/win/2004/08/events"
    xmlns:win="http://manifests.microsoft.com/win/2004/08/windows/events"
    xmlns:xs="http://www.w3.org/2001/XMLSchema"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://schemas.microsoft.com/win/2004/08/events eventman.xsd"
    >
  <instrumentation>
    <events>
      <provider
          guid="{b5a0bda9-50fe-4d0e-a83d-bae3f58c94d6}"
          messageFileName="%SystemDrive%\ETWDriverSample\Eventdrv.sys"
          name="Sample Driver"
          resourceFileName="%SystemDrive%\ETWDriverSample\Eventdrv.sys"
          symbol="DriverControlGuid"
          >
        <channels>
          <importChannel
              chid="SYSTEM"
              name="System"
              />
        </channels>
        <templates>
          <template tid="debug_template">
            <data
                inType="win:UnicodeString"
                name="name"
                outType="xs:string"
                />
          </template>
        </templates>
        <events>
          <event
              channel="SYSTEM"
              level="win:Informational"
              message="$(string.EventMessage)"
              opcode="win:none"
              symbol="DebugEvent"
              template="debug_template"
              value="1"
              />
        </events>
      </provider>
    </events>
  </instrumentation>
  <localization xmlns="http://schemas.microsoft.com/win/2004/08/events">
    <resources culture="en-US">
      <stringTable>
        <string
            id="EventMessage"
            value="Debug String"
            />
      </stringTable>
    </resources>
  </localization>
</instrumentationManifest>
```
 - 编译MAN
 
vs可以自动编译man文件并将生成的文件加入到项目中可实现自动在vs中加入资源及相关宏。根据msdn的介绍方法如下

将man文档加入驱动项目的resourses文件夹中

对man文件的属性进行设置

```
Generate Kernel Mode Logging Macros	Yes (-km)  \\生成宏
Use Base Name of Input	Yes (-b)               \\自动生成文件的前缀
Generate header file for containing counter	Yes \\生成头文件
Header File Path	$(IntDir)           \\生成头文件的位置，建议放在项目下，不要放在生成目录
Generated RC and Binary Message Files Path	Yes \\生成rc文件和二进制message文件
RC File Path	$(IntDir)                        \\生成rc和bin文件的位置
Generated Files Base Name	$(Filename)          \\生成文件的前缀定义，默认是man文件的文件名
```

正常编译一次生成头文件，将头文件include到需要记录的地方就可以使用里面的宏。

- 驱动中使用事件记录宏

在需要记录事件的地方include刚刚生成的头文件，在驱动载入函数（比如driverentry）中调用注册event provider的宏（**EventRegisterSample_Driver**），在驱动卸载函数中（unload函数）中调用反注册函数（**EventUnregisterSample_Driver**）。
在需要生成事件记录的地方调用写事件函数（**EventWriteDebugEvent**），写入事件。

- 系统自动日志记录的配置
![event-tracing-1](http://7xkc1v.com1.z0.glb.clouddn.com/images/event-tracing-1.png)
TO BE Continued
