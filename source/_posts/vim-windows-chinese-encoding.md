title: 【转】Vim在Windows控制台乱码问题解决
tags: 
---

###Vim中有四个与编码有关的选项

- encoding

encoding是vim内部使用的字符编码，当我们设置了encoding之后，vim内部所有的buffer,寄存器，脚本中的字符串等，全都使用这个编码。
vim在工作时，如果编码方式与它的内部编码不一致，它会先把编码转换成内部编码，这样会出现有些字符无法转换，造成字符丢失。而utf8的范围是非常大的，很适合。但在windows这样非utf8字符系统下，菜单和系统提示会乱码，所以需要添加：

```bash
set encoding=utf-8
set langmenu=zh_CN.UTF-8
language message zh_CN.UTF-8
```

- termencoding

这个是vim用于屏幕的编码，在显示的时候，vim把内部编码转成屏幕编码，再用于输出，如果转换失败显示，但不影响对它的编辑动作。

```bash
set termencoding=cp936
```

这样就搞定了。图形界面的gvim不依赖于终端，所以会忽略termencoding，这就是在windows虚拟DOS窗口下运行vim乱码的原因。

- fileencoding

当vim从磁盘上读取文件时，会对文件的编码进行探测。如果文件的编码方式和vim的内部编码方式不同，vim就会对编码进行转换。完成后，vim将fileencoding选项设置为文件的编码。当vim存盘时，如果encoding与fileencoding不一样，则会进行转换之后保存。

- fileencodings

fileencodings，注意是复数，编码的自动识别通过设置fileencodings实现的，他会根据设置的依次检测编码类型，建议设置：

```bash
set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1
```
