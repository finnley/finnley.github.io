+++
title = 'jq'
date = 2024-10-26T16:29:42+08:00 
draft = false
categories = [ "Linux" ]
tags = [ "jq" ]
+++

# 介绍

一个灵活的轻量级命令行JSON处理器。

## 功能

- 处理JSON输入，可以将给定的过滤器应用与JSON文本输入并在标准输出上将过滤器的结果生成为JSON

## 安装

```bash
# Debian，如 Ubuntu
sudo apt-get install -y jq

# RedHat, 如 CentOS
yum install -y jq
```

## 语法

```bash
jq [options] <jq filter> [file...]
jq [options] --args <jq filter> [strings...]
jq [options] --jsonargs <jq filter> [JSON_TEXTS...]
```

## 选项

```bash
-c               紧凑而不是漂亮的输出;
-n               使用`null`作为单个输入值;
-e               根据输出设置退出状态代码;
-s               将所有输入读取（吸取）到数组中；应用过滤器;
-r               输出原始字符串，而不是JSON文本;
-R               读取原始字符串，而不是JSON文本;
-C               为JSON着色;
-M               单色（不要为JSON着色）;
-S               在输出上排序对象的键;
--tab            使用制表符进行缩进;
--arg a v        将变量$a设置为value<v>;
--argjson a v    将变量$a设置为JSON value<v>;
--slurpfile a f  将变量$a设置为从<f>读取的JSON文本数组;
--rawfile a f    将变量$a设置为包含<f>内容的字符串;
--args           其余参数是字符串参数，而不是文件;
--jsonargs       其余的参数是JSON参数，而不是文件;
--               终止参数处理;
```

# 练习

**以漂亮格式输出**

```bash
# echo '{"foo": {"bar": {"baz": 123}}}' | jq '.'
{
  "foo": {
    "bar": {
      "baz": 123
    }
  }
}
```

- `echo '{"foo": {"bar": {"baz": 123}}}'`

  首先使用 `echo` 输出一段 JOSN 格式的字符串

- `| jq '.'`

  使用管道符 `|` 将 `echo` 的输出结果传到给 `jq` 处理器。 `jq` 是一个强大的命令行 JSON 处理起，`'.'` 是一个简单的额 jq 表达式，表示原样格式化输出接收的JSON数据。 



**获取键的值**

```bash
# echo '{"foo": {"bar": {"baz": 123}}}' | jq '.foo'
{
  "bar": {
    "baz": 123
  }
}
# echo '{"foo": {"bar": {"baz": 123}}}' | jq '.foo.bar'
{
  "baz": 123
}
# echo '{"foo": {"bar": {"baz": 123}}}' | jq '.foo.bar.baz'
123
```

- `jq ".foo"`

  获取JSON字符串中key是 “foo” 的值。

- `jq '.foo.bar'`

  获取JSON字符串中key是 “bar” 的值。


- `jq '.foo.bar.baz'`

  获取JSON字符串中key是 “baz” 的值。

**数组运算**

```bash
# echo '[{"name": "JSON", "good": true}, {"name": "XML", "good": false}]' | jq '.[1]'
{
  "name": "XML",
  "good": false
}
```

- `jq '.[1]'`

  输出JSON数组的下标为1的元素。

**计算值的长度**

```bash
# echo '[{"name": "JSON", "good": true}, {"name": "XML", "good": false}]' | jq '.[] | length'
2
2
# echo '[[1,2], "string", {"a":2}, null]' | jq '.[] | length'
2
6
1
0
# echo '{"foo": {"bar": {"baz": 123}}}' | jq '.[] | length'
1
```

- `| jq '.[] | length'`

  其中 `'.[]'` 表示遍历数组中的每个元素；`length` 表示返回每个元素的长度；
  
  `[{"name": "JSON", "good": true}, {"name": "XML", "good": false}]` 这一JSON数组包含了两个对象，分别是 `name` 和 `good`，由于每个对象都有两个属性，所以输出为2（每个对象的属性数量）

  `[[1,2], "string", {"a":2}, null]` 这一JSON包含四个元素，同样 `'.[]'` 是遍历数组中的每个元素，对于每个元素调用 `length`，故第一个元素 `[1,2]` 的长度是 2，`"string"` 的长度是6（字符串中有6个字符），`{"a":2}` 的长度是1（对象中有1个属性），`null` 的长度是 0（null 没有长度）

  `{"foo": {"bar": {"baz": 123}}}` 只有一个对象故长度是 1


**获取数组的键**

```bash
# echo '{"abc": 1, "abcd": 2, "Foo": 3}' | jq 'keys'
[
  "Foo",
  "abc",
  "abcd"
]
```

**构造一个数组/对象**

```bash
# echo '{"user":"stedolan","titles":["JQ Primer", "More JQ"]}' | jq '{user, title: .titles[]}'
{
  "user": "stedolan",
  "title": "JQ Primer"
}
{
  "user": "stedolan",
  "title": "More JQ"
}
```

**使用多个过滤器**

```bash
# echo '{ "foo": 42, "bar": "something else", "baz": true}' | jq '.foo, .bar'
42
"something else"
```

**通过管道将一个过滤器的输出当做下一个过滤器的输入**

```bash
# echo '[{"name":"JSON", "good":true}, {"name":"XML", "good":false}]' | jq '.[] | .name'
"JSON"
"XML"
```

**条件判断**

```bash
# echo '[1, 5, 3, 0, 7]' | jq 'map(select(. >= 2))'
[
  5,
  3,
  7
]
```

- `'map(select(. >= 2))'`
   
   这是 jq 的一个表达式；`map(...)` 会对数组中的每个元素应用给定的表达式，并返回一个新的数组

- `select(. >= 2)`

  这是一个过滤条件，只选择哪些大于或等于 2 的元素

```bash
# echo '2' | jq 'if . == 0 then "zero" elif . == 1 then "one" else "many" end'
"many"
```

