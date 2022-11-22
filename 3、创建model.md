### 创建 Go Module

本教程将创建两个module，第一个用于其它应用可以引用/导入的库，另一个用来做调用方将调用上一个创建的module。

本节包含内容：

- 1、创建一个module -- 写一个简单的module，使之可以通过其他module来调用；
- 2、在另一个module调用 -- 导入并使用的案例；
- 3、处理Error -- 返回Error或者处理Error是我们开发是最常见的问题了；
- 4、单元测试 -- 使用Go内置的单元测试功能来测试代码；

#### 创建一个可以被其他module调用的项目

1、创建一个项目文件夹并初始化项目

```shell
% mkdir greetings
% cd greetings 
greetings % go mod init examole.com/greetings
go: creating new go.mod: module examole.com/greetings
```

2、创建一个文件 greetings.go 并使用你自己的编辑工具打开，粘贴以下代码进去：

```go
package greetings

import "fmt"

// Hello returns a greeting for the named person.
func Hello(name string) string {
    // Return a greeting that embeds the name in a message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message
}
```

这里实现了一个Hello函数，它会向所有调用者返回一句话："Hi, %v. Welcome!", name，这里的name是调用方传过来的。

在Go中，函数名称以大写字母开头的函数可以由不在同一个包中的函数调用。

fun Hello (name string) string

以上是一个函数在Go中的声明，“Hello”是函数名，括号里的“name”是形参，“string“是指参数的类型，最后一个"string"表示这个函数最后的return类型。

在Go中，”:=“这个运算符是声明和初始化的一个简写，同时你也可以使用原始写法去写，如下：

```go
var message string
message = fmt.Sprintf("Hi, %v. Welcome!", name)
```

另外，在上面的案例中使用了fmt这个包，函数中调用了sprintf函数，括号里面前半部分是格式化，后面是要传的值，最终会把传进来的值替换”%v“。

#### 在另外一个module中调用刚才这个函数

这里我们再创建一个module用来调用上面写的那个函数。

1、创建一个文件夹hello，与上面的greetings平行，创建好之后如下所示：

```shell
<usr>/
|--greetings/
|--hello/
```

<这里的这个hello是前面就创建好的，我直接拿来用了>

2、打开hello下面的main.go文件，并复制以下代码进去：

```go
package main

import (
    "fmt"

    "example.com/greetings"
)

func main() {
    // Get a greeting message and print it.
    message := greetings.Hello("Gladys")
    fmt.Println(message)
}
```

代码中声明了main主包，在Go中，作为应用程序执行的代码必须位于主包中。

上面代码导入了两个包，fmt上面讲过了就不说了（处理输入输出的功能），下面 "example.com/greetings"就是我们上面创建的那个module。

然后在下面通过greetings.Hello("Gladys")这种方式进行Hello函数的调用。

3、初始化依赖让example.com/hello 可以正常的调用example.com/greetings

一个module要想公开使用需要进行发布，目前我们上面写的这个（example.com/greetings）是没有发布的，所以需要手动在本地进行依赖。

4、打开hello文件夹并执行如下代码：

```shell
go mod edit -replace example.com/greetings=../greetings
```

运行该命令后，代码执行时在查找example.com/greetings的时候会自动替换为../greetings，同是hello/go.mod文件中也会加上这一句解释，我们打开看一下：

```shell
<usr> hello % cat go.mod 
module example/hellp

go 1.17

require rsc.io/quote v1.5.2

require (
	golang.org/x/text v0.0.0-20170915032832-14c0d48ead0c // indirect
	rsc.io/sampler v1.3.0 // indirect
)

replace example.com/greetings => ../greetings
```

5、还是在hello/文件夹下面执行：go mod tidy命令，用来加载example.com/hello模块中所需跟踪的依赖项：

```shell
% go mod tidy
go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
```

这是后我们再打开go.mod文件看一下，正常情况如下所示：

```shell
<usr> hello % go mod tidy
go: found example.com/greetings in example.com/greetings v0.0.0-00010101000000-000000000000
fu@FudeMacBook-Pro hello % cat go.mod 
module example/hellp

go 1.17

require example.com/greetings v0.0.0-00010101000000-000000000000//多了这一行就对了

replace example.com/greetings => ../greetings
```

上面的命令执行的意义就是让example.com/hello跟example.com/greetings关联起来了（俗称找到了），后面那一长串数字其实是一个版本号，这里是伪版本号。在正式发布的module中会如下所示：

```go
require example.com/greetings v1.1.0
```

6、接下来我们在hello下面去执行那个main.go

```go
%go run .
Hi, Gladys. Welcome!
```

输出正确~，至此怎么创建module，怎么调用module我们就都掌握了，你也可以去多写几个试试。

#### 返回并处理Error

处理Error是我们编程时不得不考虑的重要事项之一，所以下面我们就看下在Go中是怎么处理的。

1、以上面提到的greetings module为例

greetings中有一个Hello函数公开给大家调用，调用时需传入一个参数，然后返回的时候才会正常。试想如果我们传入的参数是空值，那么会返回什么内容呢？我们修改下代码然后直接运行一下试试看：

```go
1.在hello文件夹下面打开hello.go
hello % vi hello.go 
2.修改代码
 message := greetings.Hello("Gladys")
 修改为：
 message := greetings.Hello("")
3.执行
hello % go run .
4.返回结果如下：
Hi, . Welcome!
```

我们改造一下greetings，如果传入参数是空，则返回一个Error，修改如下：

```Go
package greetings

import (
    "errors"
    "fmt"
)

// Hello returns a greeting for the named person.
func Hello(name string) (string, error) {
    // If no name was given, return an error with a message.
    if name == "" {
        return "", errors.New("empty name")
    }

    // If a name was received, return a value that embeds the name
    // in a greeting message.
    message := fmt.Sprintf("Hi, %v. Welcome!", name)
    return message, nil
}
```

在Go里面，任何函数都可以返回多个值。

2、在调用的时候也做下处理，看看返回给我们的error是什么，打开hello/hello.go:

```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // Request a greeting message.
    message, err := greetings.Hello("")
    // If an error was returned, print it to the console and
    // exit the program.
    if err != nil {
        log.Fatal(err)
    }

    // If no error was returned, print the returned message
    // to the console.
    fmt.Println(message)
}
```

log包是Go里面专门用于处理日志的，在另一个开源项目（[logrus详解—Go项目实战之日志必备篇](https://www.xiaoyin.live/?p=96)）里面有详细介绍过。

3、最后我们看一下运行效果：

```
go run . //依旧是在hello文件夹下

greetings: empty name
exit status 1
```

#### 随机返回一个值

有没有发现上面的greetings很单调，面对所有人都是说：欢迎光临~，为了好玩我们再升级一下greetings,来客了就随机从既定话术里面调一句。

本节涉及Go里面的切片概念（slice），类似于数组，属于可变类型。

1、打开greetings/greetings.go，之家做如下修改：

```Go
package greetings

import (
    "errors"
    "fmt"
    "math/rand"
    "time"
)

// Hello 函数 随机返回一条欢迎语
func Hello(name string) (string, error) {
    // 如果name为空则返回一个error
    if name == "" {
        return name, errors.New("empty name")
    }
    // 创建一个随机的话术
    message := fmt.Sprintf(randomFormat(), name)
    return message, nil
}

// init sets initial values for variables used in the function.
func init() {
    rand.Seed(time.Now().UnixNano())
}

// 随机话术函数
func randomFormat() string {
    // 一个话术库切片
    formats := []string{
        "Hi, %v. Welcome!",
        "Great to see you, %v!",
        "Hail, %v! Well met!",
    }

    // 随机返回
    return formats[rand.Intn(len(formats))]
}
```

- 添加了一个randomFormat函数，用来随机生成话术库，小写字母开头表示只能在本包中使用；
- 上面这种声明格式是省略了大小值([1]),表示内容是可以随时改变的；
- math/rand是用来生成随机数的；
- init是初始化函数，Go在程序启东市自动执行init。这里用于做rand包的种子；

2、完成上述内容后，去hello/hello.go中执行一下看看（上面我们把参数置为空，要先修改回来）：

```Go
<usr> hello % go run .
Great to see you, Gladys!
<usr> hello % go run .
Great to see you, Gladys!
<usr> hello % go run .
Great to see you, Gladys!
<usr> hello % go run .
Hi, Gladys. Welcome!
<usr> hello % go run .
Hail, Gladys! Well met!
```

0.0 我前三次随机的也太整齐了~

#### 多参传入以及匹配多参输出

本节需要掌握的就是参数以数组形式传入多个，然后再以map形式返回与之对应个一个个值，这种写法在日常开发任务中非常适用。

1、打开greetings/greetings.go在里面新添加一个函数，如下：

```Go
// 给每一个传入的name匹配一条对应的问候语
func Hellos(names []string) (map[string]string, error) {
    // 初始化一个map
    messages := make(map[string]string)
    // 循环入参切片（数组）只要不为空就匹配一条欢迎语
    for _, name := range names {//不需要索引，这里以“_”省略
        message, err := Hello(name)//直接调用原先的Hello避免代码重复的同时还保留了原功能
        if err != nil {
            return nil, err
        }
        // 把欢迎语跟name关联起来
        messages[name] = message
    }
    return messages, nil
}
```

2、在hello/hello.go中修改一下，传入一个切片类参数：

```go
package main

import (
    "fmt"
    "log"

    "example.com/greetings"
)

func main() {
    // Set properties of the predefined Logger, including
    // the log entry prefix and a flag to disable printing
    // the time, source file, and line number.
    log.SetPrefix("greetings: ")
    log.SetFlags(0)

    // 初始化一个切片
    names := []string{"Gladys", "Samantha", "Darrin"}

    // 传入切片参数
    messages, err := greetings.Hellos(names)
    if err != nil {
        log.Fatal(err)
    }
    // If no error was returned, print the returned map of
    // messages to the console.
    fmt.Println(messages)
}
```

3、看下运行效果：

```go
<usr> hello % go run .
map[Darrin:Great to see you, Darrin! Gladys:Hi, Gladys. Welcome! Samantha:Great to see you, Samantha!]
```

#### 单元测试

单元测试是开发一个高质量项目必不可少的一步，但是日常开发中我们往往会忽略。

1、在greetings文件夹下面创建一个greetings_test.go。在Go中文件名以“_test.go”结尾的标识本文件包含了测试模块

2、打开greetings_test.go并把一下内容复制过去：

```Go
package greetings

import (
    "testing"
    "regexp"
)

// TestHelloName 调用Hello检查其返回值
func TestHelloName(t *testing.T) {
    name := "Gladys"
    want := regexp.MustCompile(`\b`+name+`\b`)
    msg, err := Hello("Gladys")
    if !want.MatchString(msg) || err != nil {
        t.Fatalf(`Hello("Gladys") = %q, %v, want match for %#q, nil`, msg, err, want)
    }
}

// TestHelloEmpty 调用 greetings.Hello 携带空字符串,检查返回值和error
func TestHelloEmpty(t *testing.T) {
    msg, err := Hello("")
    if msg != "" || err == nil {
        t.Fatalf(`Hello("") = %q, %v, want "", error`, msg, err)
    }
}
```

testing是Go的测试包；

2、运行一下，看结果：

```go
%go test
PASS
ok  	examole.com/greetings	0.828s
%go test -v
=== RUN   TestHelloName
--- PASS: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
PASS
ok  	examole.com/greetings	0.649s
```

go test命令执行后，会执行当前包下所有名称以_test.go文件中的Test开头的函数。

go test -v 是把所有执行的详细信息都打印出来。

3、为了看一下测试情况，我们在原代码中做一下修改：

把greetings.go中的：

```go
message := fmt.Sprintf(randomFormat(), name)
```

修改为：

```go 
message := fmt.Sprint(randomFormat())
```

Sprint和Sprintf的最大区别就是一个允许使用占位符一个不允许用。

修改完毕后我们再执行以下单元测试，结果如下：

```go
<usr> greetings % go test -v
=== RUN   TestHelloName
    greetings_test.go:15: Hello("Gladys") = "Hail, %v! Well met!", <nil>, want match for `\bGladys\b`, nil
--- FAIL: TestHelloName (0.00s)
=== RUN   TestHelloEmpty
--- PASS: TestHelloEmpty (0.00s)
FAIL
exit status 1
FAIL	examole.com/greetings	1.620s
```

测试不通过咯~

#### 编译及安装Go应用

- go build 命令编译包以及相关依赖，但是并不会安装；
- go install 命令编译且安装；

1、在hello/文件夹下执行编译命令

```go
go build
```

执行后生成：hello可执行文件（我的电脑是mac），如果是Windows会生成hello.exe

2、执行编译后的文件

```go
%./hello
map[Darrin:Great to see you, Darrin! Gladys:Hail, Gladys! Well met! Samantha:Hi, Samantha. Welcome!]
```

3、修改环境变量

如果截止到此，我们每次执行都需要按照：./hello这种格式去执行，如果我们想直接执行hello就需要去配置一下环境变量：

```shell
export PATH=$PATH:/path/to/your/install/directory（mac配置方式）
```

这个路径你可以通过：go list -f '{{.Target}}'这个命令去查看（每个人都不一样），如果你这个路径已经用了那么你可以通过以下命令去更改安装目标路径：

```shell
go env -w GOBIN=/path/to/your/bin
```

4、更新路径后，运行go install命令来编译和安装

```shell
go install
```

ok 以后想运行了，直接打开命令端输入：hello即可

（本节完）