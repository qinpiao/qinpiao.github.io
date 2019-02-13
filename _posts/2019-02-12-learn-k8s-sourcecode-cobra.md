---
layout: post
title:  "Go 命令行库cobra的学习"
date:   2019-02-12 21:27:35
categories: Go
tags: Go Kubernetes 
---

* content
{:toc}


[Cobra](https://github.com/spf13/cobra)是spf13写的一个编写/生成交互式命令程序的框架，有很多知名的开源项目都使用了这个框架：

* Kubernetes
* Hugo
* rkt
* etcd
* Docker (distribution)
* OpenShift
* Delve
* GopherJS
* CockroachDB
* Bleve
* ProjectAtomic (enterprise)
* Parse (CLI)
* GiantSwarm's swarm
* Nanobox/Nanopack


了解了这个框架，再去看这些Kubernetes、etcd等开源项目的代码时，也就大概知道如何去看了，这也是我学习cobra的目的





## 安装

cobra的安装由于墙的缘故会导致有些包无法顺利下载下来，这里我们使用git clone的形式绕过

这里需要的包是sys与text 我们需要将这两个包放在 `$GOPATH$/src/golang.org/x/` 目录下

如果你的 `$GOPATH` 下没有这个目录，可以先手动创建好 `$GOPATH$/src/golang.org/x/` 目录

1. 到 `$GOPATH$/src/golang.org/x/` 目录下执行
    ```
    git clone https://github.com/golang/sys.git
    git clone https://github.com/golang/text.git
    ```
2. 再执行 `go get -u github.com/spf13/cobra/cobra` 就能安装好cobra了


## 入门

一般基于cobra的程序的代码结构如下所示:
```
▾ appName/
    ▾ cmd/
        add.go
        your.go
        commands.go
        here.go
      main.go
```
`main.go`的内容非常的简单,里面一般只是初始化cobra,比如下面就是一个比较典型的代码:
```go
package main

import (
	"fmt"
	"os"
	"(pathToYourApp)/cmd"
)

func main() {
	if err := cmd.RootCmd.Excute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```
cobra还提供了一个命令行工具,我们安装完cobra后, 在`$GOBIN`目录下会生成一个可执行的cobra文件,可以使用这个工具来自动生成应用框架以及一些命令支持.

`cobra init [youApp]`

这个命令可以帮我们生成一个代码框架, 生成的代码框架结构如下:
```
.
├── cmd
│   └── root.go
├── LICENSE
└── main.go
```
我们可以看到生成的代码包含一个`main.go`, `LICENSE`以及`cmd`目录,`cmd`目录内只有一个`root.go`文件
`root.go`的内容如下:
```go
package cmd

import (
    "fmt"
    "os"

    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

var cfgFile string

// RootCmd represents the base command when called without any subcommands
var RootCmd = &cobra.Command{
    Use:   "cobra_exp1",
    Short: "A brief description of your application",
    Long: `A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.`,
// Uncomment the following line if your bare application
// has an action associated with it:
//    Run: func(cmd *cobra.Command, args []string) { },
}

// Execute adds all child commands to the root command sets flags appropriately.
// This is called by main.main(). It only needs to happen once to the rootCmd.
func Execute() {
    if err := RootCmd.Execute(); err != nil {
        fmt.Println(err)
        os.Exit(-1)
    }
}

func init() {
    cobra.OnInitialize(initConfig)

    // Here you will define your flags and configuration settings.
    // Cobra supports Persistent Flags, which, if defined here,
    // will be global for your application.

    RootCmd.PersistentFlags().StringVar(&cfgFile, "config", "", "config file (default is $HOME/.cobra_exp1.yaml)")
    // Cobra also supports local flags, which will only run
    // when this action is called directly.
    RootCmd.Flags().BoolP("toggle", "t", false, "Help message for toggle")
}

// initConfig reads in config file and ENV variables if set.
func initConfig() {
    if cfgFile != "" { // enable ability to specify config file via flag
        viper.SetConfigFile(cfgFile)
    }

    viper.SetConfigName(".cobra_exp1") // name of config file (without extension)
    viper.AddConfigPath("$HOME")  // adding home directory as first search path
    viper.AutomaticEnv()          // read in environment variables that match

    // If a config file is found, read it in.
    if err := viper.ReadInConfig(); err == nil {
        fmt.Println("Using config file:", viper.ConfigFileUsed())
    }
}
```
我们暂时只需要关注`RootCmd`这个变量的结构体内容以及`Excute`函数,再看`main.go`的内容:
```go
// Copyright © 2019 NAME HERE <EMAIL ADDRESS>
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

package main

import "cobra_exp1/cmd"

func main() {
	cmd.Execute()
}

```
可以看出,`main`和前面介绍的一样,只是执行了初始化的语句, 运行下程序:
```
➜ cobra_exp1 go run main.go
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:
Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

```

目前我们的程序还只是一个空的框架,什么子命令和标志都没有

我们可以通过`cobra add`来增加我们自己的子命令, 比如我们需要添加下面三条命令:
* cobra_exp1 serve
* cobra_exp1 config
* cobra_exp1 config create

我们在工程目录下依次执行如下命令:
```
➜  cobra_exp1 cobra add serve
Using config file: /Users/Allan/.cobra.yaml
serve created at /Users/Allan/workspace/gopath/src/cobra_exp1/cmd/serve.go
➜  cobra_exp1 cobra add config
Using config file: /Users/Allan/.cobra.yaml
config created at /Users/Allan/workspace/gopath/src/cobra_exp1/cmd/config.go
➜  cobra_exp1 cobra add create -p 'configCmd'
Using config file: /Users/Allan/.cobra.yaml
create created at /Users/Allan/workspace/gopath/src/cobra_exp1/cmd/create.go

# 增加完后的文件结构
.
├── cmd
|   ├── config.go
|   ├── create.go
│   ├── root.go
|   └──serve.go
├── LICENSE
└── main.go
```
然后再运行程序:
```
➜  cobra_exp1 go build
➜  cobra_exp1 ./cobra_exp1
A longer description that spans multiple lines and likely contains
examples and usage of using your application. For example:

Cobra is a CLI library for Go that empowers applications.
This application is a tool to generate the needed files
to quickly create a Cobra application.

Usage:
  cobra_exp1 [command]

Available Commands:
  config      A brief description of your command
  serve       A brief description of your command

Flags:
      --config string   config file (default is $HOME/.cobra_exp1.yaml)

Use "cobra_exp1 [command] --help" for more information about a command.
➜  cobra_exp1 ./cobra_exp1 serve
serve called
➜  cobra_exp1 ./cobra_exp1 config
config called
➜  cobra_exp1 ./cobra_exp1 config create
create called
```
我们添加的命令已经可以正常使用了，后续我们需要做的就是添加命令的动作了。 

接着看如何使用标志(Flag)

cobra提供了两种flag，一种是全局的，一种是局部的。
所谓全局的就是如果A命令定义了一个flag，那A命令下的所有命令都可以使用这个flag。
局部的flag当然就是只能被某个特定的命令使用了。

对于全局的flag, 我们可以直接加到root下面:
```go
RootCmd.PersistentFlags().BoolVarP(&Verbose, "verbose", "v", false, "verbose output")
```
如下面的例子所示:
```go
package main

import (
    "fmt"
    "strings"

    "github.com/spf13/cobra"
)

func main() {
    var echoTimes int

    var cmdPrint = &cobra.Command{
        Use:   "print [string to print]",
        Short: "Print anything to the screen",
        Long: `print is for printing anything back to the screen.
            For many years people have printed back to the screen.
            `,
        Run: func(cmd *cobra.Command, args []string) {
            fmt.Println("Print: " + strings.Join(args, " "))
        },
    }

    var cmdEcho = &cobra.Command{
        Use:   "echo [string to echo]",
        Short: "Echo anything to the screen",
        Long: `echo is for echoing anything back.
            Echo works a lot like print, except it has a child command.
            `,
        Run: func(cmd *cobra.Command, args []string) {
            fmt.Println("Print: " + strings.Join(args, " "))
        },
    }

    var cmdTimes = &cobra.Command{
        Use:   "times [# times] [string to echo]",
        Short: "Echo anything to the screen more times",
        Long: `echo things multiple times back to the user by providing
            a count and a string.`,
        Run: func(cmd *cobra.Command, args []string) {
            for i := 0; i < echoTimes; i++ {
                fmt.Println("Echo: " + strings.Join(args, " "))
            }
        },
    }

    cmdTimes.Flags().IntVarP(&echoTimes, "times", "t", 1, "times to echo the input")

    var rootCmd = &cobra.Command{Use: "app"}
    rootCmd.AddCommand(cmdPrint, cmdEcho)
    cmdEcho.AddCommand(cmdTimes)

    rootCmd.Execute()
}
```
在这个例子中,我们定义了两个顶级命令`echo`和`print`以及一个`echo`的字命令`times`, 测试下看看:
```
# 编译
➜  cobra_exp1 go build -o app main.go

# 打印应用帮助信息
➜  cobra_exp1 ./app
Usage:
  app [command]

Available Commands:
  echo        Echo anything to the screen
  print       Print anything to the screen

Use "app [command] --help" for more information about a command.

# 打印print命令帮助信息
➜  cobra_exp1 ./app print -h
print is for printing anything back to the screen.
            For many years people have printed back to the screen.

Usage:
  app print [string to print] [flags]

# 打印echo命令帮助信息
➜  cobra_exp1 ./app echo --help
echo is for echoing anything back.
            Echo works a lot like print, except it has a child command.

Usage:
  app echo [string to echo] [flags]
  app echo [command]

Available Commands:
  times       Echo anything to the screen more times

Use "app echo [command] --help" for more information about a command.

# 打印echo的子命令times的帮助信息
➜  cobra_exp1 ./app echo times -h
echo things multiple times back to the user by providing
            a count and a string.

Usage:
  app echo times [# times] [string to echo] [flags]

Flags:
  -t, --times int   times to echo the input (default 1)

# 使用
➜  cobra_exp1 ./app print just a test
Print: just a test
➜  cobra_exp1 ./app echo just a test
Print: just a test
➜  cobra_exp1 ./app echo times just a test -t 5
Echo: just a test
Echo: just a test
Echo: just a test
Echo: just a test
Echo: just a test
➜  cobra_exp1 ./app echo times just a test --times=5
Echo: just a test
Echo: just a test
Echo: just a test
Echo: just a test
Echo: just a test
➜  cobra_exp1 ./app echo times just a test --times 5
Echo: just a test
Echo: just a test
Echo: just a test
Echo: just a test
Echo: just a test
```

## 其它特性

cobra还有非常多的其他特性，这里我们只简单说明一下

* 自定义帮助。之前我们已经看到了，我们新增的命令都有help功能，这个是cobra内置的，当然我们可以自定义成我们自己想要的样子。
* 钩子函数。前面我们介绍了命令执行时会去执行命令Run字段定义的回调函数，cobra还提供了四个函数：PersistentPreRun、PreRun、PostRun、PersistentPostRun，可以在执行这个回调函数之前和之后执行。它们的执行顺序依次是：PersistentPreRun、PreRun、Run、PostRun、PersistentPostRun。而且对于PersistentPreRun和PersistentPostRun，子命令是继承的，也就是说子命令如果没有自定义自己的PersistentPreRun和PersistentPostRun，那它就会执行父命令的这两个函数。
* 可选的错误处理函数。
* 智能提示。比如下面的：
```
➜  cobra_exp1 ./app eoch
Error: unknown command "eoch" for "app"

Did you mean this?
 echo

Run 'app --help' for usage.
```

