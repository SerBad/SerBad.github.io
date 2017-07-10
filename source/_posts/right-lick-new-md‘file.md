---
title: 右键新建md文件
date: 2017-07-10 14:10:28
tags: 操作
comments: false
---
因为经常使用markdown，所以会遇到新建``.md`` 文件的时候，会很麻烦。记录下Windows下右键新建``.md`` 文件的方法：
<!--more-->
1. 打开注册表 ``regedit`` ，在目录 ``HKEY_CLASSES_ROOT`` 下新建``.md`` 项，然后把默认值修改为``.md``。
![这里写图片描述](http://img.blog.csdn.net/20170509131503696?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU2VyX0JhZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2.  然后在``.md`` 项下再新建项``ShellNew`` ，然后在该项下面新建字符串值``FileName`` ，该字符值选择为，你新建的模板地址，比如我存放模板的位置是``C:\Users\user\AppData\Local\atom\markdown.md``
![这里写图片描述](http://img.blog.csdn.net/20170509131552978?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU2VyX0JhZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

至于Mac党，推荐一个应用[Easy New File](https://itunes.apple.com/cn/app/easy-new-file/id1162194131?mt=12)，也不贵蛮好用的，下面是设置界面。
![这里写图片描述](http://img.blog.csdn.net/20170710141632960?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvU2VyX0JhZA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
