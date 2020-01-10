### Go vendor 问题

存在问题的树结构如下：

```
➜  test find . -print | sed -e 's;[^/]*/;|____;g;s;____|; |;g'
.
|____Makefile
|____src
| |____main.go
| |____vendor
| | |____wer
| | | |____s.go
```

> mac 下并没有 tree 命令 

这里的 GOPATH 为 /Users/tianyuan/Temp/test

```
➜  test pwd
/Users/tianyuan/Temp/test
```



* Makefile

```
export GOPATH=$(PWD)
$(info $(GOPATH))
$(info $(GOROOT))
VERSION?=0.0.1

all:
	cd src; \
	go build -o ${GOPATH}/test
```

* main.go

```
package main

import(
    "fmt"
    "wer"
)

func main(){
    fmt.Println(wer.Echo())
}
```

* s.go

```
package wer

func Echo() int{
    return 123
}
```

make 之后出现如下问题：

```
➜  test make
/Users/tianyuan/Temp/test

cd src; \
	go build -o /Users/tianyuan/Temp/test/test
main.go:5:5: cannot find package "wer" in any of:
	/usr/local/go/src/wer (from $GOROOT)
	/Users/tianyuan/Temp/test/src/wer (from $GOPATH)
make: *** [all] Error 1
```

#### 把目录结构改成如下方式时就没有问题：

```
➜  test tree
.
├── Makefile
├── src
│   └── main
│       ├── main.go
│       └── vendor
│           └── wer
│               └── s.go
└── test

4 directories, 4 files
```

* Makefile 修改为

```
➜  test cat Makefile
export GOPATH=$(PWD)
$(info $(GOPATH))
$(info $(GOROOT))
VERSION?=0.0.1

all:
	cd src/main; \
	go build -o ${GOPATH}/test
```

> cd 到了 src 下的 main 目录

主要做的修改就是把 main.go 和 vendor 目录挪到了 main 目录下

编译成功：

```
➜  test make
/Users/tianyuan/Temp/test

cd src/main; \
	go build -o /Users/tianyuan/Temp/test/test
```

#### 其中原因如下

```
// vendoredImportPath returns the expansion of path when it appears in parent.
// If parent is x/y/z, then path might expand to x/y/z/vendor/path, x/y/vendor/path,
// x/vendor/path, vendor/path, or else stay path if none of those exist.
// vendoredImportPath returns the expanded path or, if no expansion is found, the original.
func vendoredImportPath(parent *Package, path string) (found string) {
    if parent == nil || parent.Root == "" {
        return path
    }
    
这个代码可以从 go 源码中找到。    
```



这个里面有一个概念parent，我们可以看到Go官方关于vendor的文档设计的时候都有这个，也就是说源码要有一个包存起来，我们看一下这个实验里面的代码。

首先`a.go`放在了GOPATH的目录，也就是他的parent是nil，所以这个返回了`wer`,而这个包又不在GOPATH下面，所以第一个报错了。

第二个你创建了main目录，那么他的parent就是main，那么就会去找main/vendor/path和vendor/path下面找，所以vendor不管你放在main下面还是GOPATH下面都是可以找到的。



github上有对vendor的明确解释，vendor的目录必须是所在项目的包内，不能允许为non-package的目录，也就是说vendor的目录路径应该位于GOPATH下的包内，这是对vendor顶层目录路径的要求。所以 GOPATH 下的 go 文件引用 vendor 下的包，肯定是编译不过的，因为 GOPATH 下的 go 文件不在包内。