---

layout: single  
title: "Cmd and workspace of go"  
date: 2022-05-08 16:28:00 +0800   
categories: Go Go-Base

---

## 重要的环境变量
* GOROOT: Go语言的安装路径
* GOPATH：我们定义的多个工作区目录路径
* GOBIN：Go程序生成的可执行文件  
* GOPROXY：设置获取包代理
* GOPRIVATE：配置私有仓库域名
* GO111MODULE：配置是否使用mod模块管理，on， off， auto（默认）

## GOPATH 与工作区
* 源码文件一般以包的形式放置在GOPATH包含的某个工作区目录下的src目录中；
* 安装后产生的归档文件会放置在GOPATH包含的某个工作区目录下的pkg目录中；
* 安装后产生的可执行文件会放置在GOPATH包含的某个工作区目录下的bin目录中；
* 归档文件的相对目录与 pkg 目录之间还有一级目录，叫做平台相关目录。平台相关目录的名称是由 build（也称“构建”）的目标操作系统、下划线和目标计算架构的代号组成的；因此GOPATH 与工作区的关系如下图：

	![GOPATH 与工作区](/assets/img/GOPATH与工作区.png)
	
	**注意**：GOMODULE模式下go build 下载的包存在于$GOPATH/pkg/mod中
	
## Go build 与 Go install 区别
* go build 构建库源码文件，构建后的结果文件存在于临时目录中，意义在于检查和验证；
* go build 构建命令源码文件，构建后的结果存在于源码文件所在的目录中；
* go install 安装库源码文件，安装后的结果存在于它所在工作区的pkg目录中；
* go install 安装命令源码文件，安装后的结果存在于它所在工作区的bin目录中，或者环境变量GOBIN指向的目录中；

## 常用命令
* go get（go 1.6以及之前）：安装代码包到GOPATH包含的第一个工作区目录中。如果存在环境变量GOBIN，那么仅包含命令源码文件的代码包会被安装到GOBIN指向的那个目录。
	* -u：下载并安装代码包，不论工作区中是否已存在它们。	* -d：只下载代码包，不安装代码包。
	* -fix：在下载代码包后先运行一个用于根据当前 Go 语言版本修正代码的工具，然后再安装代码包。
	* -t：同时下载测试所需的代码包。
	* -insecure：允许通过非安全的网络协议下载和安装代码包。HTTP 就是这样的协议。   

**注意**：go 1.6之后 go get -insecure 标志已弃用并已被删除。 要在获取依赖项时允许使用不安全的方案，请配置 GOINSECURE 环境变量。 
go get 在主模块之外安装命令时打印一个弃用警告（没有 -d 标志）。 应该使用 go install cmd@version 来安装特定版本的命令，使用像 @latest 或 @v1.2.3 这样的后缀。 在 Go 1.18 中，-d 标志将始终启用，并且 go get 仅用于更改 go.mod 中的依赖项。

## 命令源码文件
![文件类型](/assets/img/文件类型.png)   

* 含义：源码文件声明属于main包，并且包含一个无参数声明且无结果声明的main函数，那么它就是命令源码文件。
* go run 命令源码文件 -参数=“xxx”
* 查看用法：go run 命令源码文件 --help

## 库源码文件
* 含义：不能被直接运行的源码文件，它仅用于存放程序实体，这些程序实体可以被其他代码使用。
* 源码文件所在的目录相对于 src 目录的相对路径就是它的代码包导入路径，而实际使用其程序实体时给定的限定符要与它声明所属的代码包名称对应，因此应该让声明的包名与其父目录的名称一致。
* 模块级私有：internal代码包中声明的公开程序实体仅能被该代码包的直接父包及其子包中的代码引用。

	



