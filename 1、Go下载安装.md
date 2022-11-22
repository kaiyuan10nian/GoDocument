### Go下载安装

按照以下方式可快速下载并安装Go：

#### 1、下载Go

https://go.dev/dl/（需科学上网）

作者资源：

链接: https://pan.baidu.com/s/1nN2i1LRvX_Dy9NgofN-b5A?pwd=f1ud 提取码: f1ud

若嫌弃百度网盘下载慢的话可直接联系作者索要资源。

#### 2、安装（区分Windows，Mac，Linux）

##### Linux安装

1）删除以前安装的Go版本，然后将刚下载的文件解压至/usr/local中并创建一个Go文件夹：

```shell
$ rm -rf /usr/local/go && tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz
```

可能需要root权限或者通过sudo运行该命令。

注：/usr/local/go最好是新建的，不要用原来的，不然可能出错。

2）添加环境变量，打开你的$HOME/.profile 或者 /etc/profile添加以下代码：

```shell
export PATH=$PATH:/usr/local/go/bin
```

保存后在你重启计算机之前不会立即生效，若想立即生效请执行：`source $HOME/.profile`.

3）验证安装情况，在命令行执行以下命令：

```shell
$ go version
```

若安装成功则会打印go的版本号。

##### Mac安装

1）打开下载的安装包，按照提示安装。默认安装位置为：/usr/local/go，同时会默认添加环境变量。

2）验证安装情况，新打开一个终端命令框，输入：$ go version，显示版本号即表示安装成功。

##### Windows安装

1）打开下载的MSI文件，按照提示进行安装。默认安装在program或者program(x86)中，当然你也可以根据自己喜好来更改安装位置。

2）验证安装情况，打开新的命令提示符，输入：$ go version，显示版本号即表示安装成功。



