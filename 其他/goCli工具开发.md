# Go 项目开发与跨平台打包指南

## 项目结构

![image-20200503095502150](https://i.loli.net/2020/05/19/kKIMHsxvmRlugGw.png)

## 技术栈

* GO
* Go mod
* Makefile



## 基础知识

* [runoob go 教程](https://www.runoob.com/go/go-map.html)

* [go 语言圣经](https://yar999.gitbooks.io/gopl-zh/content/ch4/ch4-05.html)

* [文档](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter06/06.2.html)



## 环境配置

go 仓库代理，开启go module

```bash
# http(s)代理
export proxy=http://xxx.xxx:8080/
export https_proxy=http://xxx.xxx:8080/

https://goproxy.cn/
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.cn,direct
```



## 入口文件

通过args 获取参数

```go
package main

import (
	"fmt"
	"os"
	scripts "gocli/scripts"
)

func main()  {
	// fmt.Println("starting go push...")
	args := os.Args[1:]
	fmt.Println("input args: ",args)
	if (len(args) == 0) {
		fmt.Println("you must type right commander(init/push/version)")
		return
	}
	switch args[0] {
		case "init":
			fmt.Println("----------init----------")
		case "push":
			fmt.Println("----------push----------")
			scripts.PushToGit()
		case "version":
			fmt.Println("----------version----------")
			scripts.GetVersion()
		default:
			fmt.Println("you must type right commander(init/push/version)")
	}
}
```



## Makefile 跨平台编译编写

整合编译打包到不通平台



```makefile
# Binary name
BINARY=myGoCli
# Binary dir
BUILDDIR=bin
# Builds the project
build:
		GO111MODULE=on go build -o ./${BUILDDIR}/${BINARY} -ldflags "-X main.Version=${VERSION}"
		GO111MODULE=on go test -v
# Installs our project: copies binaries
install:
		GO111MODULE=on go install
release:
		# Clean
		go clean
		rm -rf ./${BUILDDIR}/*.gz
		# Build for mac
		GO111MODULE=on go build -o ./${BUILDDIR}/${BINARY} -ldflags "-s -w -X main.Version=${VERSION}"
		tar czvf ./${BUILDDIR}/${BINARY}-mac64-${VERSION}.tar.gz ./${BUILDDIR}/${BINARY}
		# Build for arm
		go clean
		CGO_ENABLED=0 GOOS=linux GOARCH=arm64 GO111MODULE=on go build  -o ./${BUILDDIR}/${BINARY} -ldflags "-s -w -X main.Version=${VERSION}"
		tar czvf ./${BUILDDIR}/${BINARY}-arm64-${VERSION}.tar.gz ./${BUILDDIR}/${BINARY}
		# Build for linux
		go clean
		CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on go build -o ./${BUILDDIR}/${BINARY} -ldflags "-s -w -X main.Version=${VERSION}"
		tar czvf ./${BUILDDIR}/${BINARY}-linux64-${VERSION}.tar.gz ./${BUILDDIR}/${BINARY}
		# Build for win
		go clean
		CGO_ENABLED=0 GOOS=windows GOARCH=amd64 GO111MODULE=on go build -o ./${BUILDDIR}/${BINARY}.exe -ldflags "-s -w -X main.Version=${VERSION}"
		tar czvf ./${BUILDDIR}/${BINARY}-win64-${VERSION}.tar.gz ./${BUILDDIR}/${BINARY}.exe
		go clean
# Cleans our projects: deletes binaries
clean:
		go clean
		rm -rf ./${BUILDDIR}

.PHONY:  clean build
```



## go-simplejosn 的使用

读取json



```json
{
    "projectName":"myapp",
    "branch": "dev",
    "gitRepoUrl": "git@xx.xx.com:xxx/sample.git",
    "gitCommitMsg": "new commit msg",
    "tempGitDir": "./build",
    "extention": [".js", ".css", ".map"],
    "copyList": [
        {
            "from": "./dist/index.html",
            "to": "xxx/index.html"
        },
        {
            "from": "./dist/webstatic/remstart/",
            "to": "xxx"
        }
    ]
}
```



```go
func ReadJSON(path string) (*simplejson.Json, error) {
	filePtr, err := os.Open(path)
	if err != nil {
		return simplejson.New(), errors.New(`path can't be nil.`)
	}
    defer filePtr.Close()

	json, err := simplejson.NewFromReader(filePtr)
    if err != nil {
		return simplejson.New(), errors.New(`json err .`)
	}
	// fmt.Println(json)
	return json, nil
}
```





数组循环

```go
config, err := utils.ReadJSON(path)
....
// copy files
	copyList, _ := config.Get("copyList").Array()
	for i := range copyList {
		file := config.Get("copyList").GetIndex(i)
		from := file.Get("from").MustString()
		to := gitDir + "/" +file.Get("to").MustString()
		// fmt.Println(i)
		// fmt.Println(from)
		// fmt.Println(to)
		utils.CopyFiles(from, to)
	}
```



## PS

* golang的语法：模块中要导出的函数，必须首字母大写,最好加入注释
* 本地包需要用 module name 引入

```go
import (
	utils "myGoCli/utils" // 本地导入本地的模块
	"fmt"
	"os"
)
```



### 一些工具类函数

1. Cmd 类

```go
  // 查看详细的错误	
  c := "dir"
	fmt.Println("cmd:" + c)
	c1 := exec.Command("cmd", "/c", c) // windows 下
	// c1 := exec.Command("/bin/sh", "-c", c) linux mac 下
	var out bytes.Buffer
	var stderr bytes.Buffer
	c1.Stdout = &out
	c1.Stderr = &stderr
	err := c1.Run()
	if err != nil {
		fmt.Println(fmt.Sprint(err) + ": " + stderr.String())
		return
	}
	fmt.Println("Result: " + out.String())
```

```go
// 简单封装
package utils

import (
    "fmt"
    "errors"
    // "os"
    "os/exec"
    "runtime"
    "strings"
)

func runInLinux(cmd string) string{
    fmt.Println("Running Linux cmd:" , cmd)
    result, err := exec.Command("/bin/sh", "-c", cmd).Output()
    if err != nil {
        fmt.Println(err.Error())
    }
    return strings.TrimSpace(string(result))
}

func runInWindows(cmd string) string{
    fmt.Println("Running Win cmd:", cmd)
    result, err := exec.Command("cmd", "/c", cmd).Output()
    if err != nil {
        fmt.Println(err.Error())
    }
    return strings.TrimSpace(string(result))
}
// RunCommand win run 
func RunCommand(cmd string) string{
    if runtime.GOOS == "windows" {
        return runInWindows(cmd)
    }
	return runInLinux(cmd)
}
// RunLinuxCommand linux run
func RunLinuxCommand(cmd string) string{
    if runtime.GOOS == "windows" {
        return ""
    }
	return runInLinux(cmd)
}

func runInLinuxWithErr(cmd string) (string, error) {
    fmt.Println("Running Linux cmd:"+cmd)
    result, err := exec.Command("/bin/sh", "-c", cmd).Output()
    if err != nil {
        fmt.Println(err.Error())
    }
    return strings.TrimSpace(string(result)), err
}

func runInWindowsWithErr(cmd string) (string, error){
    fmt.Println("Running Windows cmd:"+cmd)
    result, err := exec.Command("cmd", "/c", cmd).Output()
    if err != nil {
        fmt.Println(err.Error())
    }
    return strings.TrimSpace(string(result)), err
}
// RunCommandWithErr run with error win
func RunCommandWithErr(cmd string) (string, error){
    if runtime.GOOS == "windows" {
        return runInWindowsWithErr(cmd)
    }
	return runInLinuxWithErr(cmd)
}
// RunLinuxCommandWithErr run with error 
func RunLinuxCommandWithErr(cmd string)(string, error){
    if runtime.GOOS == "windows" {
        return "", errors.New("could not run in Windows Os") 
    }
	return runInLinuxWithErr(cmd)
}
```



2. 文件类

```go
// ExistsPathCheck 判断目录或文件是否存在
func ExistsPathCheck(path string) (bool, error) {
    // 判断不存在
    if _, err := os.Stat(path); os.IsNotExist(err) {
        // 不存在
    }

    // 判断是否存在
    _, err := os.Stat(path)
    if err == nil {
        return true, nil
    }
    if os.IsNotExist(err) {
        return false, nil
    }
    return true, err
}

// CopyFile 拷贝文件
func CopyFile(source string, dest string) (err error) {
    sf, err := os.Open(source)
    if err != nil {
        return err
    }
    defer sf.Close()
    df, err := os.Create(dest)
    if err != nil {
        return err
    }
    defer df.Close()
    fmt.Println("copy file: " + source + "===>" +dest + "  ")
    _, err = io.Copy(df, sf)
    if err == nil {
        si, err := os.Stat(source)
        if err != nil {
            err = os.Chmod(dest, si.Mode())
        }

    }
    return
}

// CopyDir 拷贝目录
func CopyDir(source string, dest string) (err error) {
    fi, err := os.Stat(source)
    if err != nil {
        return err
    }
    if !fi.IsDir() {
        return errors.New(source + " is not a directory")
    }
    err = os.MkdirAll(dest, fi.Mode())
    if err != nil {
        return err
    }
    entries, err := ioutil.ReadDir(source)
    for _, entry := range entries {
        sfp := filepath.Join(source, entry.Name())
        dfp := filepath.Join(dest, entry.Name())
        if entry.IsDir() {
            err = CopyDir(sfp, dfp)
            if err != nil {
                fmt.Println(err)
            }
        } else {
            err = CopyFile(sfp, dfp)
            if err != nil {
                fmt.Println(err)
            }
        }

    }
    return nil
}
```

