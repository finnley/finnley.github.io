+++
title = 'Go配置文件管理Viper'
date = 2025-03-10T10:07:39+08:00
draft = true
categories = [ "Go" ]
tags = [ "go", "golang" ]
+++

# 1 介绍

[spf13/viper](https://github.com/spf13/viper)

Viper是适用于Go应用程序的完整配置解决方案。它被设计用于在应用程序中工作，并且可以处理所有类型的配置需求和格式。它支持以下特性：
  - 设置默认值
  - 从JSON、TOML、YAML、HCL、envfile和Java properties格式的配置文件读取配置信息
  - 实时监控和重新读取配置文件（可选）
  - 从环境变量中读取
  - 从远程配置系统（etcd或Consul）读取并监控配置变化
  - 从命令行参数读取配置
  - 从buffer读取配置
  - 显式配置值

# 2 安装

```shell
go get github.com/spf13/viper
```

# 3 使用

## 3.1 两种指定配置文件方式

**config/config.yaml**

```yaml
name: 'use-web'
port: 8080
list:
  - a
  - b
  - c
mysql:
  host: '127.0.0.1'
  port: 3306
```

**指定文件路径**
```go
func TestConfigFile(t *testing.T) {
	// 1.实例化viper实例
	v := viper.New()

	// 2.设置文件路径 文件路径如何设置?
	v.SetConfigFile("./config/config.yaml") // 这种方式需要切换到文件所在目录
	//v.SetConfigFile("./config/config.yaml") // 这种方式相对项目根目录，可以直接运行该文件；也可以在IDE中配置下工作目录（working directory）

	// 3.读取文件，有可能出错，需要判断
	if err := v.ReadInConfig(); err != nil {
		panic(err)
	}

	// 4.获取数据
	name := v.Get("name")
	fmt.Println("name:", name) // user-web
}
```

**指定文件名与格式**
```go
func initViper() {
	// 设置配置文件名字，但不包含配置文件的后缀
	viper.SetConfigName("dev") // 如配置文件为dev.yaml，此处的参数则为 dev
	// 告诉viper配置文件类型为 yaml，而不是toml, json
	viper.SetConfigType("yaml")
	// 告诉 viper 在哪里找到配置文件，在当前工作目录下的config里面
	// 允许有多个配置目录
	viper.AddConfigPath("./config")
	viper.AddConfigPath("/etc/config")
	// 读取配置到viper，或者可以理解为加载到内存中
	err := viper.ReadInConfig()
	if err != nil {
		panic(err)
	}

	// 如果有多个配置，可以自行创建，可以有多个viper的实例
	otherViper := viper.New()
	viper.SetConfigName("other")
	otherViper.SetConfigType("json")
	otherViper.AddConfigPath("./config")
}
```

## 3.2 配置内容映射为Go Struct

1、首先会读取配置文件，将文件内容转化为Go Struct。

```go
type MysqlConfig struct {
	Host string `mapstructure:"host"`
	Port int    `mapstructure:"port"`
}

// 将配置文件映射成 block
type ServerConfig struct {
	// 将配置文件中的 name 的值方位 ServiceName 中
	ServiceName string      `mapstructure:"name"`
	Port        int         `mapstructure:"port"`
	List        []string    `mapstructure:"list"`
	MysqlInfo   MysqlConfig `mapstructure:"mysql"`
}

type List map[string]interface{}

// 配置文件内容映射成 block
func TestConfig2Struct(t *testing.T) {
	v := viper.New()
	v.SetConfigFile("./config/config.yaml")
	if err := v.ReadInConfig(); err != nil {
		panic(err)
	}

	serverConfig := ServerConfig{}
	if err := v.Unmarshal(&serverConfig); err != nil {
		panic(err)
	}

	fmt.Println("server-config:", serverConfig) // {use-web 8080 [a b c] {127.0.0.1 3306}}
	fmt.Printf("name: %s\n", v.Get("name"))
	fmt.Println(serverConfig.List)

	m := make(map[string]struct{})
	list := serverConfig.List
	for i, v := range list {
		fmt.Println(i, v)
		m[v] = struct{}{}
	}

	for k := range m {
		fmt.Println(k)
	}
}
```

## 3.3 隔离开发、生产环境配置隔离

```go
func GetEnvInfo(env string) bool {
	viper.AutomaticEnv()
	return viper.GetBool(env)
}

// =================================== 配置文件环境隔离 ===================================
func TestEnvironmentalIsolation(t *testing.T) {
	// 读取环境变量 设置环境变量如果想要生效，需要重启 goland
	fmt.Println(GetEnvInfo("MXSHOP_DEBUG"))  // true
	fmt.Println(GetEnvInfo("CAT_DEBUG"))     // false
	fmt.Println(GetEnvInfo("EINSCAT_DEBUG")) // true
	debug := GetEnvInfo("MXSHOP_DEBUG")

	configFilePrefix := "config"
	configFileName := fmt.Sprintf("./config/%s-pro.yaml", configFilePrefix)
	if debug {
		configFileName = fmt.Sprintf("./config/%s-debug.yaml", configFilePrefix)
	}
	fmt.Println(configFileName) // config-debug.yam

	v := viper.New()
	v.SetConfigFile(configFileName)
	if err := v.ReadInConfig(); err != nil {
		panic(err)
	}
	serverConfig := ServerConfig{}
	if err := v.Unmarshal(&serverConfig); err != nil {
		panic(err)
	}
	fmt.Println(serverConfig)               //{use-web2 {127.0.0.1 3306}}
	fmt.Printf("name: %v\n", v.Get("name")) // use-web2

	// =================================== 动态监控配置文件变化 ===================================
	// 运行文件 修改配置文件，自动输出
	v.WatchConfig()
	// 如何知道变化的值 这种不会堵塞住，所以为了防止退出需要设置time.Sleep(time.Second * 300)
	v.OnConfigChange(func(e fsnotify.Event) {
		fmt.Println("config file changed:", e.Name)
		_ = v.ReadInConfig()
		_ = v.Unmarshal(&serverConfig)
		fmt.Println(serverConfig)
	})
	time.Sleep(time.Second * 300)
}
```