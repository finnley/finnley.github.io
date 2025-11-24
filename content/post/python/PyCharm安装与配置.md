+++
title = 'PyCharm安装与配置'
date = 2020-01-03T14:11:11+08:00
draft = true
categories = [ "Python" ]
tags = [ "python", "pycharm" ]
+++

# 1 安装

略

# 2 配置

- 虚拟环境配置

- Settings

  Tools -> Action on Save:

    - Reformat code
    - Optimize imports

  Editor -> Font:
    - Size: 18.0
    - Line height: 1.2

  File and Code Templates —> Python Script:

  ```
  # !/usr/bin/env python
  # -*- coding: utf-8 -*-
  """
  @Time    : ${DATE} ${TIME}
  @Author  : finnley@einscat.com
  @File    : ${NAME}.py
  """
  ```

  Plugins:

    - env
    - CodeGlance