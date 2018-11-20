---
title: 使用pipenv管理你的项目
date: '2018-07-05 23:35:44'
tag: 
  - python
meta:
  -
    name: python pipenv 管理工具
    content: python pipenv 包管理工具
---

pipenv 新款Python虚拟环境工具
<!-- more -->

## 技术背景

---

pipenv 新款Python虚拟环境工具 pipenv是requests库作者Kenneth Reitz编写的一个用于创建和管理Python虚拟环境的工具。 Pipenv是Python官方机构推荐的Python包工具。 我们知道，为了管理Python虚拟环境，通常用得比较多的是virtualenv和pyenv。
如果你熟悉nodejs的npm那么使用pipenv那就很容易理解了，pipenv 会自动在我们当前项目下创建一个Pipfile文件，类似于npm下面的package.json文件。


## 技术功能

---

1. 根据 Pipfile 自动寻找项目根目录。
2. 如果不存在，可以自动生成 Pipfile 和 Pipfile.lock。
3. 自动在项目目录的 .venv 目录创建虚拟环境。
4. 自动管理 Pipfile 新安装和删除的包。
5. 自动更新 pip。

## 使用步骤

---

### 安装
          pip install pipenv （全局安装pipenv）

### 使用
    md test （新建一个测试文件夹）
    cd test && pipenv install （运行pipenv install命令）

### 完成
    此时你的项目中会多出一个pipfile文件，此时运行pipenv shell 激活你的虚拟环境。

## 常用命令
```python
# 安装依赖包生成虚拟环境
 pipenv install

# 激活当前项目的虚拟环境
 pipenv shell

# 安装开发依赖包
 pipenv install pytest --dev

# 图形显示包依赖关系
 pipenv graph

# 生成lockfile
 pipenv lock

# 删除所有的安装包
 pipenv uninstall --all
```

## 指定python安装源

---

在国内安装包的速度有可能过慢，因此我们需要使用国内的镜像源，那么在pipenv中我们如何设置镜像源呢。打开项目中的pipfile文件。修改其中的url指向国内镜像源。

```bash
[[source]]
url = "http://mirrors.aliyun.com/pypi/simple/"
verify_ssl = true
name = "pypi"
```

## 国内比较常用的镜像源

[http://pypi.douban.com/simple/](http://pypi.douban.com/simple/) 豆瓣
[http://mirrors.aliyun.com/pypi/simple/](http://mirrors.aliyun.com/pypi/simple/) 阿里
[http://pypi.hustunique.com/simple/](http://pypi.hustunique.com/simple/) 华中理工大学
[http://pypi.sdutlinux.org/simple/](http://pypi.sdutlinux.org/simple/) 山东理工大学
[http://pypi.mirrors.ustc.edu.cn/simple/](http://pypi.mirrors.ustc.edu.cn/simple/) 中国科学技术大学
