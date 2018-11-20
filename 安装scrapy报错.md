---
title: 安装scrapy报错
date: '2018-09-21 23:35:44'
tag: 
  - scrapy
meta:
  -
    name: python scrapy安装报错
    content: python scrapy
---

在安装scrapy会出现Twisted报错，解决办法是...
<!-- more -->



## 错误描述

```powershell
Failed building wheel for Twisted
Command "c:\users\administrator\.virtualenvs\splider_lp-zfbukvro\scripts\python.exe -u -c "import setuptools, tokenize;__file__='C:\\Users\\ADMINI~1\\AppData\\Local\\Temp\\pip-install-9huedjbo\\Twisted\\setup.py';f=getattr(tokenize, 'open', open)(__file__);code=f.read().replace('\r\n', '\n');f.close();exec(compile(code, __file__, 'exec'))" install --record C:\Users\ADMINI~1\AppData\Local\Temp\pip-record-k5z5pez6\install-record.txt --single-version-externally-managed --compile --install-headers c:\users\administrator\.virtualenvs\splider_lp-zfbukvro\include\site\python3.6\Twisted" failed with error code 1 in C:\Users\ADMINI~1\AppData\Local\Temp\pip-install-9huedjbo\Twisted\
```

当我们安装scrapy框架的时候有可能会出现以上报错，这个报错是说，Twisted没有安装成功，Twusted是我们scrapy核心。

## 解决办法

[https://www.lfd.uci.edu/~gohlke/pythonlibs/](https://www.lfd.uci.edu/~gohlke/pythonlibs/)  打开网址，找到Twisted相应的版本，下载。我这里找取的是
[Twisted‑18.7.0‑cp36‑cp36m‑win\_amd64.whl](javascript:; "[3.0&nbsp;MB] [Jul 13, 2018]") 这个版本。打开命令行在我们当前项目中运行

```plain
pip install D:/tools/Twisted-18.7.0-cp36-cp36m-win_amd64
pip install scrapy
```

再一次运行安装scrapy，成功。





