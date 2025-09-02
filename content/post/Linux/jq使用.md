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

  使用管道符 `|` 将 `echo` 的输出结果传到给 `jq` 处理器。 `jq` 是一个强大的命令行 JSON 处理起，`'.'` 是一个简单的 jq 表达式，表示原样格式化输出接收的JSON数据。 



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

# 综合练习

config.json 配置文件内容如下：
```json
{
  "centos7_x64": {
    "enable": false,
    "count": 5,
    "ports": [
      "13306:3306",
      "18080:18080"
    ],
    "volumes": [
      "~/workspace:/root/workspace"
    ]
  },
  "centos7_altarch": {
    "enable": true,
    "count": 5,
    "ports":{
      "chaos-1": [
        "10000:10000",
        "18080:18080"
      ],
      "chaos-2": [
        "10001:10000",
        "18081:18080"
      ]
    },
    "volumes": {
        "chaos-1": [
            "~/workspace:/root/workspace"
        ],
        "chaos-2": [
            "~/workspace:/root/workspace"
        ]
    }
  }
}
```

jq处理如下：
```bash
jq -c 'to_entries[]' ${CONFIG_FILE} | while read -r entry; do
  key=$(echo "${entry}" | jq -r '.key')
  value=$(echo "${entry}" | jq -r '.value')
  enable=$(echo "${value}" | jq -r '.enable')
  template="agent_$(to_lowercase ${key})"

  if [[ "${enable}" == "true" ]]; then
    # Add template to the file if not already added
    if ! contains_element "${template}" "${templates[@]}"; then
      echo "  ${template}:" >> ${COMPOSE_FILE}
      yq eval ".x-templates.${template}" ${TEMPLATES_FILE} | sed 's/^/    /' >> ${COMPOSE_FILE}
      templates+=("${template}")
    fi
  fi
done
```

**输出**

- `jq -c 'to_entries[]' ${CONFIG_FILE}`

  `jq -c` 表示以紧凑格式（即不换行）输出数据。比如：
  ```bash
  # cat config.json | jq -c '.'
  {"centos7_x64":{"enable":false,"count":5,"ports":["13306:3306","18080:18080"],"volumes":["~/workspace:/root/workspace"]},"centos7_altarch":{"enable":true,"count":5,"ports":{"chaos-1":["10000:10000","18080:18080"],"chaos-2":["10001:10000","18081:18080"]},"volumes":{"chaos-1":["~/workspace:/root/workspace"],"chaos-2":["~/workspace:/root/workspace"]}}}
  ```
  
  `to_entries` 是 jq 中一个内置函数，用于将JSON对象转换为一个键值对数组。即它将对象的每个键值对转化为一个包含key和value的对象，形成一个数组。好处就是可以使便利对象的每个属性变得更加简单。比如:
  ```bash
  # echo '{"name": "John", "age": 30, "is_student": false}' | jq 'to_entries'
  [
    {
      "key": "name",
      "value": "John"
    },
    {
      "key": "age",
      "value": 30
    },
    {
      "key": "is_student",
      "value": false
    }
  ]
  ```

  看下查询的结果示例：
  ```bash
  # jq -c 'to_entries[]' config.json | jq '.'
  {
    "key": "centos7_x64",
    "value": {
      "enable": false,
      "count": 5,
      "ports": [
        "13306:3306",
        "18080:18080"
      ],
      "volumes": [
        "~/workspace:/root/workspace"
      ]
    }
  }
  {
    "key": "centos7_altarch",
    "value": {
      "enable": true,
      "count": 5,
      "ports": {
        "chaos-1": [
          "10000:10000",
          "18080:18080"
        ],
        "chaos-2": [
          "10001:10000",
          "18081:18080"
        ]
      },
      "volumes": {
        "chaos-1": [
          "~/workspace:/root/workspace"
        ],
        "chaos-2": [
          "~/workspace:/root/workspace"
        ]
      }
    }
  }
  ```

- `| while read -r entry; do`

  通过管道将 jq 的输出传递到 while 循环中，一次读取一个键值对，存储在 entry 变量中。

- `key=$(echo "${entry}" | jq -r '.key')`

  使用 jq -r 从 entry 中提取 key 值，-r 表示以原始格式输出，不加引号。

- `value=$(echo "${entry}" | jq -r '.value')`

  从 entry 中提取 value 值。

- `enable=$(echo "${value}" | jq -r '.enable')`

  从 value 中提取 enable 字段，判断该功能是否启用。

