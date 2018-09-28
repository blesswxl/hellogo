[TOC]

## 1 安装 VSCode 编辑器

安装密钥
```
$ sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
```
安装储存库
```
$ sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
```

## 2 安装 golang
### 2.1 安装
使用系统包管理工具安装 golang
```
$ sudo yum install golang
```
查看安装目录
```
$ rpm -ql golang |more
```
测试安装
```
$ go version
```
![](http://p8imgfwl2.bkt.clouddn.com/18-9-28/4531339.jpg)

### 2.2 设置环境变量
1、创建工作空间
Go代码必须放在工作空间内，即为一个目录，包含三个子目录：
+ src 目录包含Go的源文件，它们被组织成包（每个目录都对应一个包），
+ pkg 目录包含包对象，
+ bin 目录包含可执行命令。

go 工具用于构建源码包，并将其生成的二进制文件安装到 pkg 和 bin 目录中。

src 子目录通常包会含多种版本控制的代码仓库（例如Git或Mercurial）， 以此来跟踪一个或多个源码包的开发。
```
$ mkdir $HOME/gowork
```
2、配置的环境变量，对于 CentOS 在 ```~/.profile``` 文件中添加：
```
export GOPATH=$HOME/gowork
export PATH=$PATH:$GOPATH/bin
```
然后执行这些配置
```
$ source $HOME/.profile
```
3、检查配置
```
$ go env
...
GOPATH = ...
...
GOROOT = ...
...
```
![](http://p8imgfwl2.bkt.clouddn.com/18-9-28/1274941.jpg)

## 3 安装必要的工具和插件
### 3.1 安装 Git 客户端
go 语言的插件主要在 Github 上，安装 git 客户端是首要工作。
```
$ sudo yum install git
```
### 3.2 安装 go 的一些工具
进入 vscode ，它提示要进行一些安装工作，按提示安装
![](http://p8imgfwl2.bkt.clouddn.com/18-9-28/61897084.jpg)

但安装过程中出现```failed to install```错误
![](http://p8imgfwl2.bkt.clouddn.com/18-9-28/54511984.jpg)

仔细检查，发现无法连接 golang.org ，导致安装失败。

1、下载源代码到本地
```
# 创建文件夹
$ mkdir $GOPATH/src/golang.org/x/
# 下载源码
$ go get -d github.com/golang/tools
# copy 
$ cp $GOPATH/src/github.com/golang/tools $GOPATH/src/golang.org/x/ -rf
```
2、安装工具包
```
$ go install golang.org/x/tools/go/buildutil
```
退出 vscode，再进入，按提示安装，即可安装成功。
![](http://p8imgfwl2.bkt.clouddn.com/18-9-28/83872252.jpg)

## 4 安装与运行 go tour
《Go 语言之旅》是官方 Go Tour 的中文翻译版。

要从源码安装教程，首先设置一个工作空间并执行，这会在你工作空间的 bin 目录中创建一个 gotour 可执行文件。
```
$ go get -u github.com/Go-zh/tour/gotour
```
执行即可，gotour 即可以通过浏览器打开。
```
$ gotour
```
![](http://p8imgfwl2.bkt.clouddn.com/18-9-28/47499569.jpg)

## 5 如何使用 Go 编程
### 5.1 第一个 Go 程序：hellogo
1、首先要选择包路径，并在你的工作空间内创建相应的包目录，其中 user 要改成你自己的用户名：
```
$ mkdir $GOPATH/src/github.com/user/hello
```
2、在该目录下新建一个名为```hellogo.go```的文件，写入以下代码：
```
package main

import "fmt"

func main() {
	fmt.Printf("Hello, Go!\n")
}
```
3、现在可以用 go 工具构建并安装此程序了：
```
$ go install github.com/user/hellogo
```
此命令会构建 hellogo 命令，产生一个可执行的二进制文件。 接着它会将该二进制文件作为 hellogo（在 Windows 下则为 hellogo.exe）安装到工作空间的 bin 目录中。 在我们的例子中为 ```$GOPATH/bin/hellogo```，具体一点就是 ```$HOME/go/bin/hellogo```。
4、运行程序
```
$ hellogo
Hello, Go.
```

### 5.2 编写一个库
让我们编写一个库，并让 hellogo 程序来使用它。
1、同样，第一步还是选择包路径（我们将使用 ```github.com/user/stringutil```） 并创建包目录：
```
$ mkdir GOPATH/src/github.com/user/stringutil
```
2、在该目录中创建名为 ```reverse.go``` 的文件，内容如下：
```
// stringutil 包含有用于处理字符串的工具函数。
package stringutil

// Reverse 将其实参字符串以符文为单位左右反转。
func Reverse(s string) string {
	r := []rune(s)
	for i, j := 0, len(r)-1; i < len(r)/2; i, j = i+1, j-1 {
		r[i], r[j] = r[j], r[i]
	}
	return string(r)
}
```
3、用 ```go build``` 命令来测试该包的编译：
```
$ go build github.com/user/stringutil
```
4、想要输出可执行文件，则使用 ```go install``` 命令，它会将包的对象放到工作空间的 pkg 目录中。
```
$ go install github.com/user/stringutil
```
5、确认 stringutil 包构建完毕后，修改原来的 ```hellogo.go``` 文件（它位于 ```$GOPATH/src/github.com/user/hellogo```）去使用它：
```
package main

import (
	"fmt"

	"github.com/user/stringutil"
)

func main() {
	fmt.Printf(stringutil.Reverse("\n!oG ,olleH"))
}
```
当我们通过 ```go install``` 来安装 ```hellogo``` 程序时，go 工具都会自动安装它所依赖的任何东西，因此 ```stringutil``` 也会被自动安装。 
```
$ go install github.com/user/hellogo
```
6、运行此程序的新版本，你应该能看到一条新的，反向的信息：
```
$ hellogo
Hello, Go!
```

### 5.3 测试
我们可通过创建文件 ```$GOPATH/src/github.com/user/stringutil/reverse_test.go ``` 来为 ```stringutil``` 添加测试，其内容如下：
```
package stringutil

import "testing"

func TestReverse(t *testing.T) {
	cases := []struct {
		in, want string
	}{
		{"Hello, world", "dlrow ,olleH"},
		{"Hello, 世界", "界世 ,olleH"},
		{"", ""},
	}
	for _, c := range cases {
		got := Reverse(c.in)
		if got != c.want {
			t.Errorf("Reverse(%q) == %q, want %q", c.in, got, c.want)
		}
	}
}
```
接着使用 ```go test``` 运行该测试：
```
$ go test github.com/user/stringutil
ok  	github.com/user/stringutil 0.004s
```
![](http://p8imgfwl2.bkt.clouddn.com/18-9-28/74745.jpg)

## 6 创建 git 本地仓库并绑定 Github 远程仓库
我们将 hellogo 发布到远程仓库。
1、创建 git 本地仓库
在 ```$GOPATH/src/github.com/user/hellogo``` 目录下建立本地仓库：
```
$ git init
```
2、 创建 Github 仓库并绑定
登陆 GitHub -> Create a new repo，创建一个新的仓库，Repository name 为 ```hellogo```。使用以下命令将本地库关联到远程库：

```
$ git remote add origin https://https://github.com/user/hellogo
```
3、推送本地库到 Github 远程库
使用 ```git add``` 与 ```git commit``` 提交修改：
```
$ git add hellogo.go
$ git commit -m "Hello,Go"
```
使用 ```git push``` 推送本地内容到远程库：
```
$ git push -u origin master
```
会提示输入用户名和密码，输入成功即可将本地库推送到 Github 上。从现在起，只要本地作了提交，就可以通过命令 ```git push``` 把本地 ```master``` 分支的最新修改推送至 Github，现在，你就拥有了真正的分布式版本库。


