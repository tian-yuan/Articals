### conan FAQ

按照 https://docs.conan.io/en/latest/installation.html 安装 conan



#### 用例

* 克隆测试代码 git clone https://github.com/memsharded/example-poco-timer.git mytimer
* 编译

```
$ conan install .
```

之后执行 cmake . 出现如下错误：

```
cmake .
-- Current conanbuildinfo.cmake directory: /Users/tianyuan/GitHupProject/mytimer
CMake Error at conanbuildinfo.cmake:471 (message):
  Incorrect 'apple-clang' version 'compiler.version=8.1' is not the one
  detected by CMake: 'Clang=9.0'
Call Stack (most recent call first):
  conanbuildinfo.cmake:527 (conan_error_compiler_version)
  conanbuildinfo.cmake:598 (check_compiler_version)
  conanbuildinfo.cmake:235 (conan_check_compiler)
  CMakeLists.txt:7 (conan_basic_setup)


-- Configuring incomplete, errors occurred!
See also "/Users/tianyuan/GitHupProject/mytimer/CMakeFiles/CMakeOutput.log".
```

需要修改 