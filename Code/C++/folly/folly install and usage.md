### folly install and usage

#### environment

* CMake

```
cmake version 3.9.0
```

* c++

```
Apple LLVM version 9.0.0 (clang-900.0.39.2)
Target: x86_64-apple-darwin17.2.0
Thread model: posix
InstalledDir: /Library/Developer/CommandLineTools/usr/bin
```

* vcpkg

```
8dc8d0e0c97550a95b764287adbad90b7df7d11d, use the master branch
```



#### install folly

```
./vcpkg install folly:x64-osx
```

```
folly:x64-osx                                      2018.10.29.00    An open-source C++ library developed and used at....
```

> May be occur may error, search solution from google

> folly use c++11 and c++14



#### use folly

* CMakeList.txt

```
# CMakeLists.txt
cmake_minimum_required(VERSION 3.0)

set(CMAKE_TOOLCHAIN_FILE /Users/tianyuan/CPPWorkspace/vcpkg/scripts/buildsystems/vcpkg.cmake)
project(test)
message("CMAKE_MODULE_PATH=${CMAKE_MODULE_PATH}")
message("VCPKG_TARGET_TRIPLET=${VCPKG_TARGET_TRIPLET}")
message("CMAKE_TOOLCHAIN_FILE=${CMAKE_TOOLCHAIN_FILE}")

### folly use c++14
set(CMAKE_CXX_FLAGS -std=c++14)

find_package(Sqlite3 REQUIRED)
find_package(folly REQUIRED)
find_path(FOLLY_INCLUDE_DIR folly/GLog.h)
find_library(FOLLY_LIBRARRY folly)

include_directories(${FOLLY_INCLUDE_DIR})
link_libraries(${FOLLY_LIBRARRY})

message("FOLLY_INCLUDE_DIR=${FOLLY_INCLUDE_DIR}")
message("FOLLY_LIBRARRY=${FOLLY_LIBRARRY}")

message("CMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}")
message("CMAKE_C_FLAGS=${CMAKE_CXX_FLAGS}")

add_executable(futextest futex-test.cpp)
### target_link_libraries link folly librarry
target_link_libraries(futextest ${FOLLY_LIBRARRY})
```

* futex_test.cpp

```
#include <folly/detail/Futex.h>
#include <iostream>

int main() {
	std::cout << "futex test." << std::endl;
	return 0;
}
```

