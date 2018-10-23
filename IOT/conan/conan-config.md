### conan 配置信息参考

#### ~/.conan/profiles/default

```
[settings]
arch = x86_64
build_type = Release
compiler = apple-clang
compiler.version = 9.0
os = Macos
compiler.libcxx = libc++

```

#### ~/.conan/settings.yml

```
os:
    Windows:
    Linux:
    Macos:
    Android:
        api_level: ANY
    iOS:
        version: ["7.0", "7.1", "8.0", "8.1", "8.2", "8.3", "9.0", "9.1", "9.2", "9.3", "10.0", "10.1", "10.2", "10.3"]
    FreeBSD:
    SunOS:
    Arduino:
        board: ANY
arch: [x86, x86_64, ppc64le, ppc64, armv6, armv7, armv7hf, armv8, sparc, sparcv9, mips, mips64, avr]
compiler:
    sun-cc:
        version: ["5.10", "5.11", "5.12", "5.13", "5.14"]
        threads: [None, posix]
        libcxx: [libCstd, libstdcxx, libstlport, libstdc++]
    gcc:
        version: ["4.1", "4.4", "4.5", "4.6", "4.7", "4.8", "4.9", "5.1", "5.2", "5.3", "5.4", "6.1", "6.2", "6.3", "6.4", "7.1", "7.2"]
        libcxx: [libstdc++, libstdc++11]
        threads: [None, posix, win32] #  Windows MinGW
        exception: [None, dwarf2, sjlj, seh] # Windows MinGW
    Visual Studio:
        runtime: [MD, MT, MTd, MDd]
        version: ["8", "9", "10", "11", "12", "14", "15"]
    clang:
        version: ["3.3", "3.4", "3.5", "3.6", "3.7", "3.8", "3.9", "4.0"]
        libcxx: [libstdc++, libstdc++11, libc++]
    apple-clang:
        version: ["5.0", "5.1", "6.0", "6.1", "7.0", "7.3", "8.0", "8.1", "9.0"]
        libcxx: [libstdc++, libc++]

build_type: [None, Debug, Release]

```

#### ~/.conan/conan.conf

```
[storage]
path = ~/.conan/data

[remotes]
conan.io = https://server.conan.io
local = http://conan.local

[proxies]


[log]
run_to_output = True        # environment CONAN_LOG_RUN_TO_OUTPUT
run_to_file = False         # environment CONAN_LOG_RUN_TO_FILE
level = 50                  # environment CONAN_LOGGING_LEVEL
# trace_file =              # environment CONAN_TRACE_FILE
print_run_commands = False  # environment CONAN_PRINT_RUN_COMMANDS

[general]
compression_level = 9       # environment CONAN_COMPRESSION_LEVEL
sysrequires_sudo = True     # environment CONAN_SYSREQUIRES_SUDO
# cmake_generator           # environment CONAN_CMAKE_GENERATOR
```