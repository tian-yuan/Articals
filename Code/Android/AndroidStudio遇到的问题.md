# Android Studio 

#### Unable to load class ‘org.gradle.api.internal.component.Usage

解决办法：项目根目录的 build.gradle 中 修改如下代码：

```
buildscript {
    repositories {
        jcenter()
        google()
    }
    dependencies {
        ...
        classpath 'com.novoda:bintray-release:0.5.0'//修改此处版本号为 0.5.0---修改之前是0.3.4
        ...
    }
}
```

2.Error:Unable to find method 'com.android.build.gradle.internal.variant.BaseVariantData.getOutputs()

```
buildscript {
    ext.kotlin_version = '1.1.2-3'
    repositories {
        maven { url 'https://maven.google.com' }
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.0-alpha1'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
```

#### gradle sync too slow

国内访问 jcenter 太慢

解决办法，使用阿里镜像

```
buildscript {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public'}
        //jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.2.2'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        maven{ url 'http://maven.aliyun.com/nexus/content/groups/public'}
        //jcenter()
    }
}
```

#### Oppo 调试闪退

关闭 instant run 可以解决这个问题

```
默认情况下，Android Studio 仅需点击几下即可设置要部署至模拟器或物理设备的新项目。使用 Instant Run，您无需构建新的 APK，就可以将更改推送至方法，将现有应用资源推送至正在运行的应用，所以几乎立刻就能看到代码更改。
```



