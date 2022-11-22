教程：Go入门

本教程内容：

1）安装Go

2）写一个“hello,world”

3）执行你的代码（命令行）

4）在你的代码中引入包

5）调用外部模块的函数

准备工作：

1）具备一定的编程经验（懂最基本的计算机知识即可）

2）编码工具，VSCode（免费）GoLand（收费-推荐）Vim（免费）或者其他可以编辑的工具，记事本也可以。

1、安装Go(本教程采用MAC系统演示)

请参看[上一节](1、Go下载安装.md)

2、写一个“hello,world”

1)打开命令行，创建一个文件夹hello:

```shell
% mkdir hello
```

2)进入hello文件夹下：

```shell
cd hello
```

3)给代码启用依赖跟踪

当你的代码导入其他模块中包含的包时，可以通过依赖跟踪去管理这些依赖，Go提供了mod文件来专门进行依赖管理。在实际开发中模块路径通常指保存源代码的存储库位置，例如：github.com/yourmodule,本教程基于演示目的采用example/hello。

```shell
hello % go mod init example/hellp
go: creating new go.mod: module example/hellp        //（手误导致写成了hellp 0.0）
```

4、打开你的文本编辑工具，创建hello.go并开始编码。

5、复制以下代码进你的hello.go并保存（记住保存路径是/hello）

```go
package main//声明一个主包

import "fmt"//带入一个比较常用的包，作用主要包含格式化文本和打印输出至控制台，该包时Go安装时就带的标准包
//实现一个主函数，运行主包时默认执行主函数（一个应用应该只有一个main）
func main() {
    fmt.Println("Hello, World!")//输出“Hello, World!”至控制台
}
```

6、执行hello.go

```go
go run hello.go
或者
go run .

输出结果：
hello % go run hello.go 
Hello, World!
```

引入外部包

Go有很多外部包可用，极大的方便了开发者重复造轮子的问题。（可以前往pkg.go.dev查看）

在这里我们引入quote来做演示，这个包就是使得打印出来的信息更加有趣~

1、在代码中import该包，然后在输出的时候调用即可，代码如下：

```go
package main

import "fmt"

import "rsc.io/quote"//引入包

func main() {
    fmt.Println(quote.Go())//使用包内函数
}
```

2、在/hello下面刷新go.mod文件，进行依赖关联管理：

```go
go mod tidy
输出以下内容表述管理成功：
go: finding module for package rsc.io/quote
go: downloading rsc.io/quote v1.5.2
go: found rsc.io/quote in rsc.io/quote v1.5.2
go: downloading rsc.io/sampler v1.3.0
go: downloading golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c
```

3、执行hello.go

```go
go run .
输出：
Don't communicate by sharing memory, share memory by communicating.
```

ok ，大功告成~ 本节结束

