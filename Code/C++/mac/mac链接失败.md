#### MAC 应用链接失败问题定位

##### 问题现象

```
 ~/PartTimeJob/MainServerNew   master ●  ./mainserver
dyld: Library not loaded: libevpp.0.7.dylib
  Referenced from: /Users/tianyuan/PartTimeJob/MainServerNew/./mainserver
  Reason: image not found
[1]    36622 abort      ./mainserver
```

##### 问题分析

* otool

```
otool -L mainserver
mainserver:
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.0.0)
        libevpp.0.7.dylib (compatibility version 0.7.0, current version 0.7.0)
        /usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 400.9.0)
```

找不到 libevpp.0.7.dylib

```
ls -lht /Users/tianyuan/CPPWorkspace/vcpkg/installed/x64-osx/lib/libevpp.dylib
lrwxr-xr-x  1 tianyuan  staff    17B Apr 12 11:51 /Users/tianyuan/CPPWorkspace/vcpkg/installed/x64-osx/lib/libevpp.dylib -> libevpp.0.7.dylib
```

从输出可以看出 libevpp.dylib 存在一个软链接到 libevpp.0.7.dylib，由于没有绝对路径，所以找不到此文件

* install_name_tool

```
install_name_tool -change libevpp.0.7.dylib /Users/tianyuan/CPPWorkspace/vcpkg/installed/x64-osx/lib/libevpp.0.7.dylib mainserver
```

重新修改依赖：

```
otool -L mainserver
mainserver:
        /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1252.0.0)
        /Users/tianyuan/CPPWorkspace/vcpkg/installed/x64-osx/lib/libevpp.0.7.dylib (compatibility version 0.7.0, current version 0.7.0)
        /usr/lib/libc++.1.dylib (compatibility version 1.0.0, current version 400.9.0)
```

可以运行应用程序

* 设置 DYLD_LIBRARY_PATH 环境变量

export DYLD_LIBRARY_PATH=/Users/tianyuan/CPPWorkspace/vcpkg/installed/x64-osx/lib/

