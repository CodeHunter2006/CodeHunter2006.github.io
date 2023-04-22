---
layout: post
title: "Python 多版本安装"
date: 2022-08-16 22:00:00 +0800
tags: Python
---

![pyenv](/assets/images/2022-08-16-Python_ENV_1.jpeg)
记录 python 多版本安装方法

# 单版本安装

- 在安装中遇到问题时，最简单的方法就是在[官网下载](https://www.python.org/downloads/macos/)各种版本的安装包安装

- 在 mac 一般安装时用`brew install python?.?` 就可以，但是对于`3.6`这种已废弃版本就可能无法安装
  - `brew switch python 3.6.5_1` brew 安装多个 python 版本后，可以用 switch 切换`python`命令对应的版本

# pip

pip 用于各种包的安装，随 python 版本变更，可执行程序名称版本也和 python 保持一致。

## 常用命令

- `pip install xxx==x.x.x -i xxx.source`
  从指定的源安装指定包的指定版本。

- `pip uninstall package`
  卸载包

- `pip list`
  查看当前环境安装的所有包及其版本号

- `pip install pip -U`
  升级 pip 自身

- `pip config list`
  显示 pip 的源

- `pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple`
  设置 pip 的源

- `pip freeze > requirements.txt`
  将当前项目的依赖导出

- `pip install -r requirements.txt`
  安装项目依赖

- `pip show requests`
  显示包的基本信息，其中包括哪些包依赖这个包

- `pip index versions requests`
  显示有哪些可用的版本

- 常用 pip 源：
  - douban
    `python3 -m pip install xxx -i http://pypi.douban.com/simple/ --trusted-host pypi.douban.com`
  - tsinghua
    `python3 -m pip install xxx -i https://pypi.tuna.tsinghua.edu.cn/simple`
  - python 官方
    `https://pypi.python.org/simple/`

# pyenv

pyenv 可以同时支持多个 python 版本，通过 python 命令后的版本号区分，如`python3.6`、`python3.7.10`

## pyenv 的安装

- `brew install pyenv`

- 另外，最好同时装下 pyenv 的 pyenv-virtualenv 插件
  `brew install pyenv-virtualenv`

- 然后在`~/.zshrc`中添加

  ```bash
  export PYENV_ROOT="$HOME/.pyenv"
  export PATH=$PYENV_ROOT/shims:$PATH
  eval "$(pyenv init -)"
  eval "$(pyenv virtualenv-init -)"
  ```

- `pyenv-virtualenv` 可以在特定文件夹(工程)内创建虚拟环境，这样在下载依赖包时不会影响到全局环境。

## 常用命令

- `pyenv version`
  查看当前版本

- `pyenv versions`
  查看已安装的所有版本，前面加`*`表示已启用

- `pyenv install --list`
  查看可安装版本

- `pyenv install 3.6.5`
  安装指定版本
- `pyenv rehash`
  安装完毕后 rehash 一下

- `pyenv uninstall 3.6.5`
  删除指定版本

- `pyenv global`
  查看或指定全局可用版本。开启后，可以用版本号直接调用，如`python3.6`。
  在切换版本时，pip 和包仓库会自动切换。

- `pyenv virtualenv 3.6.5 v365env`
  在当前文件夹内创建虚拟 python 环境，环境信息保存在`v365env`文件夹内。
- `pyenv activate v365env`
  激活虚拟环境
- `pyenv deactivate v365env`
  关闭虚拟环境
- `pyenv virtualenvs`
  查看已经存在的虚拟环境
- `pyenv uninstall v365env`
  删除虚拟环境

# virtualenv

virtualenv 和 `pyenv-virtualenv`插件功能相同

- `pip install virtualenv` 安装

- `virtualenv venv` 创建虚拟环境，信息保存到`venv`文件夹
- `virtualenv -p /usr/bin/python2.7 venv` 创建时指定 python 版本
- `source venv/bin/activate` 激活虚拟环境
- `deactivate` 关闭虚拟环境

- 删除虚拟环境时，在虚拟环境关闭状态下直接删除创建的文件夹即可

## 结合 vscode 使用

使用 vscode 时，要设置合适的 python 路径才能正确解析代码。可以利用项目中的`.vscode/settings.json`，添加下面配置：

```json
{
  "python.defaultInterpreterPath": ".../venv/bin/python",
  "python.analysis.extraPaths": [".../venv/lib/python2.7/site-packages"],
  "python.autoComplete.extraPaths": [".../venv/lib/python2.7/site-packages"]
}
```

# 用 python 自身的多版本功能

- `python -m venv ./venv`
  在当前 venv 文件夹下创建虚拟环境

- `source ./venv/bin/activate`
  进入虚拟环境

- `deactivate`
  退出虚拟环境
