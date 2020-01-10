### 使用vcpkg 管理 c++ 包

Vcpkg helps you manage C and C++ libraries on Windows, Linux and MacOS. This tool and ecosystem are constantly evolving; your involvement is vital to its success!

https://github.com/Microsoft/vcpkg



### FAQ

#### CMAKE_TOOLCHAIN_FILE seem useless

```
➜  build git:(master) ✗ cmake .. "-DCMAKE_TOOLCHAIN_FILE=/Users/tianyuan/CPPWorkspace/vcpkg/scripts/buildsystems/vcpkg.cmake"
CMAKE_MODULE_PATH=
VCPKG_TARGET_TRIPLET=
CMAKE_TOOLCHAIN_FILE=/Users/tianyuan/CPPWorkspace/vcpkg/scripts/buildsystems/vcpkg.cmake
CMake Error at CMakeLists.txt:8 (find_package):
  By not providing "FindSqlite3.cmake" in CMAKE_MODULE_PATH this project has
  asked CMake to find a package configuration file provided by "Sqlite3", but
  CMake did not find one.

  Could not find a package configuration file provided by "Sqlite3" with any
  of the following names:

    Sqlite3Config.cmake
    sqlite3-config.cmake

  Add the installation prefix of "Sqlite3" to CMAKE_PREFIX_PATH or set
  "Sqlite3_DIR" to a directory containing one of the above files.  If
  "Sqlite3" provides a separate development package or SDK, be sure it has
  been installed.


-- Configuring incomplete, errors occurred!
See also "/Users/tianyuan/CPPWorkspace/vcpkg/test/build/CMakeFiles/CMakeOutput.log".
```

solved

```
Whenever you run cmake always use a clean directory. if you ls -a you'll see a lot of cache files. If you try to run cmake again it will probably use a lot of those cached files. I ran into this issue recently too. just cd ..; rm -rf build/**;cd build to clean your directory and run cmake again
```

