---
title: pytorch dlopen image not found mac
date: 2020-11-25 17:54:00
tags: python
comments: false
---
一个安装问题
<!--more-->
```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/Users/username/Library/Python/3.8/lib/python3.8/site-packages/torch/__init__.py", line 84, in <module>
    from torch._C import *
ImportError: dlopen(/Users/username/Library/Python/3.8/lib/python3.8/site-packages/torch/_C.cpython-38-darwin.so, 9): Library not loaded: @rpath/libc++.1.dylib
  Referenced from: /Users/username/Library/Python/3.8/lib/python3.8/site-packages/torch/_C.cpython-38-darwin.so
  Reason: image not found
```
安装运行pytorch之后报了一个上面的错，找了很多方法，最后找到结果，参考：
[https://discuss.pytorch.org/t/installation-problem-library-not-loaded-rpath-libc-1-dylib/36802](https://discuss.pytorch.org/t/installation-problem-library-not-loaded-rpath-libc-1-dylib/36802)

以下的so换成你自己的路径

```bash
sudo install_name_tool -add_rpath /usr/lib /Users/username/Library/Python/3.8/lib/python3.8/site-packages/torch/_C.cpython-38-darwin.so
```

对我是有效的。
