### mosquitto 在 mac 上的编译



This post describes the steps I needed to go through to build Mosquitto with TLS on a MAC. :

First I needed to get openssl into MAC. For this brew can be used.

```
brew install openssl
```

This should install openssl under: 

```
/usr/local/Cellar/openssl/1.0.2e 
```

Next I needed to change library paths in CMakeCache.txt for the make command to succeed.  (Xcode and the developer tools are needed for the make to work.)

```
OPENSSL_CRYPTO_LIBRARY:FILEPATH=/usr/local/Cellar/openssl/1.0.2e/lib/libcrypto.dylib

OPENSSL_INCLUDE_DIR:PATH=/usr/local/Cellar/openssl/1.0.2e/include

OPENSSL_SSL_LIBRARY:FILEPATH=/usr/local/Cellar/openssl/1.0.2e/lib/libssl.dylib
```

The make should succeed with these changes, but when running ./mosquitto, it is possible that there is a linker error:

```
dyld: Library not loaded: /usr/local/opt/openssl/lib/libssl.1.0.0.dylib

  Referenced from: /Users/cigdem/Development/mqtt-trials/mosquitto/src/./mosquitto

  Reason: image not found

Trace/BPT trap: 5

```

To see what the problem is, otool comes handy. 

otool -L ./mosquitto

```
./mosquitto:

 /usr/local/opt/openssl/lib/libssl.1.0.0.dylib (compatibility version 1.0.0, current version 1.0.0)

 /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib (compatibility version 1.0.0, current version 1.0.0)

 /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)

```



So, the problem is obvious - libraries are searched in the wrong directory.

To solve this, install_name_tool is used:

 ```
install_name_tool -change /usr/local/opt/openssl/lib/libssl.1.0.0.dylib /usr/local/Cellar/openssl/1.0.2e/lib/libssl.1.0.0.dylib mosquitto

install_name_tool -change /usr/local/opt/openssl/lib/libcrypto.1.0.0.dylib /usr/local/Cellar/openssl/1.0.2e/lib/libcrypto.1.0.0.dylib mosquito
 ```



otool -L ./mosquitto

```
./mosquitto:

 /usr/local/Cellar/openssl/1.0.2e/lib/libssl.1.0.0.dylib (compatibility version 1.0.0, current version 1.0.0)

 /usr/local/Cellar/openssl/1.0.2e/lib/libcrypto.1.0.0.dylib (compatibility version 1.0.0, current version 1.0.0)

 /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)

```

This fixes the problem, and mosquito runs successfully. 
These problems will need to be fixed similarly for the clients: mosquitto_sub and mosquitto_pub. Also, you have to fix it for the library for libmosquitto.1.dylib that again refers to the wrong libraries. 

./mosquitto -c ../mosquitto.conf -v

./mosquitto_sub -t \$SYS/broker/bytes/\# -v --cafile ../../ssl/ca/ca.crt -p 8883 -h localhost



> if there is a linker error:
>
> ld: symbol(s) not found for architecture x86_64 for openssl
>
> you should donwload openssl and compile it use ./Configure darwin64-x86_64-cc