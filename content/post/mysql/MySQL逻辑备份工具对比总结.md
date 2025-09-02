+++
title = 'MySQL逻辑备份工具对比总结'
date = 2024-09-12T13:23:44+08:00
draft = true
categories = [ "MySQL" ]
tags = [ "mysql" ]
+++

# 一、优缺点对比

--            | mysqldump                                                       | mysql shell utilities
---           |---                                                              |---
优点           |1. MySQL 自带，无需集成 <br> 2. 轻量 <br> 3. 成熟度高、受众广 <br> 4. SQL 格式的备份文件，易于阅读查看和编辑  |1. 多线程，并行处理，速度快 <br> 2. 功能丰富 <br> 3. 官方极力推荐
缺点           |1. 单线程，速度慢                                                  |  1. 版本兼容性不完善 <br> 2. 新发布、更新频繁，不稳定 <br> 3. 学习时间较高 <br> 4. 需单独集成或安装


# 二、系统依赖对比

--        | mysqldump               | mysql shell utilities
-------   |---                      |---
操作系统   |目前大多数主流操作系统均支持   |目前大多数主流操作系统均支持
架构       |1. x86 32-bit <br> 2. x86 64-bit <br> 3. arm 64-bit         |1. x86 32-bit <br> 2. x86 64-bit <br> 3. arm 64-bit
依赖       |MySQL 自带，故依赖于 MySQL                       | glic 2.16、glibc 2.17、glibc 2.28 ，解压即可使用


# 三、功能对比

--        | mysqldump               | mysql shell utilities
-------   |---                      |---
备份所有库   |目前大多数主流操作系统均支持   |目前大多数主流操作系统均支持
备份指定库       |                         |
备份所有表       |                         |
备份指定表     |                         |


# 四、总结

1、如果生产环境数据量规模不大，建议选择 mysqldmp，如果数据量规模较大，建议选择 mysql-shell；
2、如果用户对备份/恢复时间、效率有要求，建议选择 mysql shell，如果用户无时间、效率等方面的要求，可选择 mysqldump；
3、对产品端，如果与urman-agent组集成大小有要求，建议选择 mysqldump，若无要求可选择 mysql-shell；
4、如果用户端、产品端想追求新的工具，建议选择 mysql shell，如果追求稳定，则建议选择 mysqldump；

综上，研发选择偏向保守，建议选择 mysqldump 。

