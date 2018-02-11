#### flag参数解析

### simple example:

```
package main  

import (  
   "flag"  
   "fmt"  
   "os"      
)  

var (  
    levelFlag = flag.Int("level", 0, "级别")    
    bnFlag int    
)  

func init() {  
    flag.IntVar(&bnFlag, "bn", 3, "份数")    
}  

func main() {  

   flag.Parse()    
   count := len(os.Args)  
   fmt.Println("参数总个数:",count)  

   fmt.Println("参数详情:")  
   for i := 0 ; i < count ;i++{  
       fmt.Println(i,":",os.Args[i])  
   }  

   fmt.Println("\n参数值:")  
   fmt.Println("级别:", *levelFlag)  
   fmt.Println("份数:", bnFlag)  
}  
```

```
运行结果:

C:\TEMP\testflag>go run tf2.go -level 3 -bn=2
参数总个数: 4
参数详情:
0 : C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\go-build158983983\command-line-arguments\_obj\exe\tf2.exe
1 : -level
2 : 3
3 : -bn=2

参数值:
级别: 3
份数: 2
```

### more complicatied example:
```
package main  

import (  
   "flag"  
   "fmt"  
   "os"  
   "time"  
)  

var (  
/*
   参数解析出错时错误处理方式
   switch f.errorHandling {
       case ContinueOnError:
           return err
       case ExitOnError:
           os.Exit(2)
       case PanicOnError:
           panic(err)
       }  
       */  

   //flagSet = flag.NewFlagSet(os.Args[0],flag.PanicOnError)   
   flagSet = flag.NewFlagSet(os.Args[0],flag.ExitOnError)   
   //flagSet = flag.NewFlagSet("xcl",flag.ExitOnError)   
   verFlag = flagSet.String("ver", "", "version")  
   xtimeFlag  = flagSet.Duration("time", 10*time.Minute, "time Duration")  

   addrFlag = StringArray{}  
)  

func init() {  
   flagSet.Var(&addrFlag, "a", "b")  
}  

func main() {  
   fmt.Println("os.Args[0]:", os.Args[0])  
   flagSet.Parse(os.Args[1:]) //flagSet.Parse(os.Args[0:])  

   fmt.Println("当前命令行参数类型个数:", flagSet.NFlag())    
    for i := 0; i != flagSet.NArg(); i++ {    
       fmt.Printf("arg[%d]=%s\n", i, flag.Arg(i))    
   }    

   fmt.Println("\n参数值:")  
   fmt.Println("ver:", *verFlag)  
   fmt.Println("xtimeFlag:", *xtimeFlag)  
   fmt.Println("addrFlag:",addrFlag.String())  

   for i,param := range flag.Args(){  
       fmt.Printf("---#%d :%s\n",i,param)  
   }  
}  


type StringArray []string  

func (s *StringArray) String() string {  
   return fmt.Sprint([]string(*s))  
}  

func (s *StringArray) Set(value string) error {  
   *s = append(*s, value)  
   return nil  
}  
```

```
运行结果:
C:\TEMP\testflag>go run tfs.go -ver 9.0 -a ba -a ca -a d2 -ver 10.0 -time 2m0s
os.Args[0]: C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\go-build341936307\command-line-arguments\_obj\exe\tfs.exe
当前命令行参数类型个数: 3

参数值:
ver: 10.0
xtimeFlag: 2m0s
addrFlag: [ba ca d2]



C:\TEMP\testflag>go run tfs.go -ver 9.0 -a ba -a ca -a d2 -ver 10.0
os.Args[0]: C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\go-build712958211\command-line-arguments\_obj\exe\tfs.exe
当前命令行参数类型个数: 2

参数值:
ver: 10.0
xtimeFlag: 10m0s
addrFlag: [ba ca d2]



-- flagSet = flag.NewFlagSet(os.Args[0],flag.PanicOnError) 结果:
C:\TEMP\testflag>go run tfs.go -ver 9.0 -a ba -a ca -a d2 -ver 10.0 -time 2m0s33
os.Args[0]: C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\go-build841833143\command-line-arguments\_obj\exe\tfs.exe
invalid value "2m0s33" for flag -time: time: missing unit in duration 2m0s33
Usage of C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\go-build841833143\command-line-arguments\_obj\exe\tfs.exe:
 -a=[]: b
 -time=10m0s: time Duration
 -ver="": version
panic: invalid value "2m0s33" for flag -time: time: missing unit in duration 2m0s33

goroutine 1 [running]:
flag.(*FlagSet).Parse(0x10b18180, 0x10b42008, 0xc, 0xc, 0x0, 0x0)
       c:/go/src/flag/flag.go:814 +0xee
main.main()
       C:/TEMP/testflag/tfs.go:41 +0x163
exit status 2


-- flagSet = flag.NewFlagSet(os.Args[0],flag.ExitOnError) 结果:
C:\TEMP\testflag>go run tfs.go -ver 9.0 -a ba -a ca -a d2 -ver 10.0 -time 2m0s33
os.Args[0]: C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\go-build501686683\command-line-arguments\_obj\exe\tfs.exe
invalid value "2m0s33" for flag -time: time: missing unit in duration 2m0s33
Usage of C:\DOCUME~1\ADMINI~1\LOCALS~1\Temp\go-build501686683\command-line-arguments\_obj\exe\tfs.exe:
 -a=[]: b
 -time=10m0s: time Duration
 -ver="": version
exit status 2
```

flagSet类可以让参数处理更加灵活.
 其中NewFlagSet:
 flagSet = flag.NewFlagSet("xcl",flag.ExitOnError)
 NewFlagSet的第一个参数是可以任意定的.但第二个参数,则决定了参数解析出错时错误处理方式.

如果要扩展参数定义,只要实现下面的接口:
[cpp] view plain copy
type Value interface {  
String() string  
Set(string) error  
依例子中的 StringArray 一样.就可以实现如:
   -a ba -a ca -a d2
    addrFlag: [ba ca d2]
  这种效果.
   要知道,同样的:
        -ver 9.0 -ver 10.0
   最后的结果是 ver: 10.0 即参数只识别最末一次的.
   没想到还能通过interface进行扩展,并且其使用比起Linux的getopt()系列函数.flag包这种方式还是更清晰直观.

参考: https://blog.komand.com/build-a-simple-cli-tool-with-golang
