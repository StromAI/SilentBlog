title: Ubuntu下Sublime Text 2的安装Fix
date: 2014-10-12 08:06:09
tags: [Linux,Sublime]
categories: Linux
---

貌似在很久以前写过一篇ST安装的文档，然后在最后的启动器设置的时候失败了，之后我在使用的时候解决了这个问题，在这里简单说一下。

---

当然,之前的启动器如果可以用的话可能涉及到系统神马的问题？这里不是很懂，下面是我最后成功时的配置:

与之前一样，在**/usr/share/applications**目录下新建文档，命名为**sublime.desktop**

之后编辑内容如下：
```
[Desktop Entry]
Version=1.0
Type=Application
Name=Sublime Text
GenericName=Text Editor
Comment=Sophisticated text editor for code, markup and prose
Exec=sublime %F
Icon=/usr/lib/Sublime Text 2/Icon/sublime.png
Terminal=false
MimeType=text/plain;
Categories=TextEditor;Development;
StartupNotify=true
Actions=Window;Document;
 
[Desktop Action Window]
Name=New Window
Exec=sublime -n
OnlyShowIn=Unity;
 
[Desktop Action Document]
Name=New File
Exec=sublime --command new_file
OnlyShowIn=Unity;
```

具体的细节我就不作说明了，需要注意的几点是
```
Name:代表该应用的名字
Icon:代表图标所在路径(当然我们可以根据这点来改变图标，搜索相关内容既可获取一些更加好看图标以更换)
Exec:内容为应用的路径以及启动参数(这里由于我已经将sublime的路径加入环境变量，所以可以直接使用)
```
关于更加具体的使用方法网络上有可获取的资源供大家参考，但是当时忘记保存网址了所以……这里没办法给出链接了
关于更换图标的问题：由于网络上获取的资源可能有格式的问题，这里提供给大家一个图片格式在线转换的[网站](http://iconverticons.com/online/)

当然关于sublime的设置还有很多可以说的地方，这里我就不一一列举了，推荐给大家一个不错的[链接](http://lucifr.com/2011/08/31/sublime-text-2-tricks-and-tips/)里面的东西较为详细。包括主题和快捷操作什么的。

这是我配置完成的效果：
![image](http://stromaiblog.qiniudn.com/Screenshot%20from%202014-09-12%2021_40_10.png)

一激动手滑敲错了点东西也没管就截图了，啊，大家就当没看到= =


最后附上Sublime Text的一些快捷操作：
```
Ctrl+Shift+P：打开命令面板
Ctrl+P：搜索项目中的文件
Ctrl+G：跳转到第几行
Ctrl+W：关闭当前打开文件
Ctrl+Shift+W：关闭所有打开文件
Ctrl+Shift+V：粘贴并格式化
Ctrl+D：选择单词，重复可增加选择下一个相同的单词
Ctrl+L：选择行，重复可依次增加选择下一行
Ctrl+Shift+L：选择多行
Ctrl+Shift+Enter：在当前行前插入新行
Ctrl+X：删除当前行
Ctrl+M：跳转到对应括号
Ctrl+U：软撤销，撤销光标位置
Ctrl+J：选择标签内容
Ctrl+F：查找内容
Ctrl+Shift+F：查找并替换
Ctrl+H：替换
Ctrl+R：前往 method
Ctrl+N：新建窗口
Ctrl+K+B：开关侧栏
Ctrl+Shift+M：选中当前括号内容，重复可选着括号本身
Ctrl+F2：设置/删除标记
Ctrl+/：注释当前行
Ctrl+Shift+/：当前位置插入注释
Ctrl+Alt+/：块注释，并Focus到首行，写注释说明用的
Ctrl+Shift+A：选择当前标签前后，修改标签用的
F11：全屏
Shift+F11：全屏免打扰模式，只编辑当前文件
Alt+F3：选择所有相同的词
Alt+.：闭合标签
Alt+Shift+数字：分屏显示
Alt+数字：切换打开第N个文件
Shift+右键拖动：光标多不，用来更改或插入列内容
鼠标的前进后退键可切换Tab文件
按Ctrl，依次点击或选取，可需要编辑的多个位置
按Ctrl+Shift+上下键，可替换行
```

就这样