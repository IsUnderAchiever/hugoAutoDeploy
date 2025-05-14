---
title: Python
description: Python
date: 2025-05-12
slug: Python
image: python-logo.png
categories:
  - Python
---

# Python 开发第三方库

## 前提

项目根目录有`setup.py`文件和`pyproject.toml`

## 打包

```bash
python -m build
```

会在`dist`目录下生成一个`tar.gz`和`whl`文件
比如`cookiecutter-2.6.0.tar.gz`和`cookiecutter-2.6.0-py3-none-any.whl`

## 安装

> cd 到 dist 目录下

```bash
pip install cookiecutter-2.6.0.tar.gz
```
