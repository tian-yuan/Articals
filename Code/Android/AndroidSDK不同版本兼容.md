# Android应用在不同版本间兼容性处理

2017年06月19日 20:41:12

阅读数：853

最近遇见了安卓低版本兼容高版本的问题，在网上发现了一篇文章讲的超好，特此转载。

转自：http://www.cnblogs.com/yaowen/p/5013366.html

在Android系统中向下兼容性比较差，但是一个应用APP经过处理还是可以在各个版本间运行的。向下兼容性不好，不同版本的系统其API版本也不同，自然有些接口也不同，新的平台不能使用旧的API，旧的平台也使用不了新的API。

​        为了应用APP有更好的兼容性，咱们可以利用高版本的SDK开发应用，并在程序运行时（Runtime）对应用所运行的平台判断，旧平台使用旧的API，而新平台可使用新的API，这样可以较好的提高软件兼容性。

 

​        那么，如何在软件运行时做出这样的判断呢？答案下边揭晓：

 

　　在Android ＳＤＫ开发文档中有段话这样的话：

## Check System Version at Runtime（在软件运行时检查判断系统版本）

------

Android provides a unique code for each platform version in the `Build` constants class. Use these codes within your app to build conditions that ensure the code thatdepends on higher API levels is executed only when those APIs are available on the system.

```
private void setUpActionBar() {
    // Make sure we're running on Honeycomb or higher to use ActionBar APIs
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {

         ActionBar actionBar = getActionBar();
         actionBar.setDisplayHomeAsUpEnabled(true);
    }
}
```

Note: When parsing XML resources, Android ignores XML attributes that aren’t supported by the current device. So you can safely use XML attributes thatare only supported by newer versions without worrying about older versions breaking when theyencounter that code. For example, if you set the `targetSdkVersion="11"`, your app includes the `ActionBar` by defaulton Android 3.0 and higher. To then add menu items to the action bar, you need to set `android:showAsAction="ifRoom"` in your menu resource XML. It's safe to do this in a cross-version XML file, because the older versions of Android simply ignore the `showAsAction` attribute (that is, you do not need a separate version in `res/menu-v11/`).

 

​           从上面可以知道Android为我们提供了一个常量类Build，其中最主要是Build中的两个内部类VERSION和VERSION_CODES，

VERSION表示当前系统版本的信息，其中就包括SDK的版本信息，用于成员SDK_INT表示；

对于VERSION_CODES在ＳＤＫ开发文档中时这样描述的，Enumeration of the currently known SDK version codes. These are the values that can be found in `SDK`. Version numbers increment monotonically with each official platform release.

其成员就是一些从最早版本开始到当前运行的系统的一些版本号常量。

　　在我们自己开发应用过程中，常常使用如下的代码形式判断运行新API还是旧的API:

 

```
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) 
    {
            // 包含新API的代码块
    }
    else
    {
            // 包含旧的API的代码块
    }



```

​     OK，大家都知道原理了吧！ 需要实例的百度蛮多的，这里就不提供了。

 

 

 

android 10（2.2.3/2.2.4）及以下的版本是没有fragment的，从 11（3.0.x） 就有了，这就是新特性，诸如此类的还有很多呢。

不明白题主的“向下兼容”具体指哪方面，就我理解的来说吧：

为了使老版本的sdk能用上新版本的特性和功能，官方都会给出额外的jar包，还是以 fragment 为例，如果我开发的app必须要能在 2.3的系统上运行，但同时要使用 fragment 怎么办呢？此时就可以用引入android.support.v4.jar包，这就是官方给的兼容性解决方案了。

可以发现，随着 SDK 版本的不断升级，官方给出的jar包也越来越多，android.support.v7.jar,v13......

如果你想详细了解下某些版本的升级带来了哪些新特性，欢迎访问[Android 5.0 Behavior Changes](http://developer.android.com/about/versions/android-5.0-changes.html)，当然，感兴趣的话也可以找到历史版本的升级记录，在这里就不多说了。。。

 

 

 

Android 版本更替，新的版本带来新的特性，新的方法。

新的方法带来许多便利，但无法在低版本系统上运行，如果兼容性处理不恰当，APP在低版本系统上，运行时将会crash。

本文以一个具体的例子说明如何在使用高API level的方法时处理好兼容性问题。

例子：根据给出路径，获取此路径所在分区的总空间大小。

在[安卓中的文件存储使用参考](http://www.liaohuqiu.net/storage-in-android/)中提到:

> 获取文件系统用量情况，在API level 9及其以上的系统，可直接调用`File`对象的相关方法，以下需自行计算

##### 一般实现

就此需求而言，API level 9及其以上，调用 `File.getTotalSpace()` 即可, 但是在API level 8 以下系统`File`对象并不存在此方法。

如以下方法：

```
/**
 * Returns the total size in bytes of the partition containing this path.
 * Returns 0 if this path does not exist.
 * 
 * @param path
 * @return -1 means path is null, 0 means path is not exist.
 */
public static long getTotalSpace(File path) {
    if (path == null) {
        return -1;
    }
    return path.getTotalSpace();
}

```

##### 处理无法编译通过

如果`minSdkVersion`设置为8，那么build时候会报以下错误：

```
Call requires API level 9 (current min is 8)

```

为了编译可以通过，可以添加 `@SuppressLint("NewApi")` 或者 `@TargeApi(9)`。

> 用`@TargeApi($API_LEVEL)`显式表明方法的API level要求，而不是`@SuppressLint("NewApi")`;

但是这样只是能编译通过，到了API level8的系统运行，将会引发 `java.lang.NoSuchMethodError`。

##### 正确的做法

为了运行时不报错, 需要:

1. 判断运行时版本，在低版本系统不调用此方法

2. 同时为了保证功能的完整性，需要提供低版本功能实现

   如下：

   ```
   /**
    * Returns the total size in bytes of the partition containing this path.
    * Returns 0 if this path does not exist.
    * 
    * @param path
    * @return -1 means path is null, 0 means path is not exist.
    */
   @TargetApi(Build.VERSION_CODES.GINGERBREAD) 
       // using @TargeApi instead of @SuppressLint("NewApi")
   @SuppressWarnings("deprecation")
   public static long getTotalSpace(File path) {
       if (path == null) {
           return -1;
       }
       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.GINGERBREAD) {
           return path.getTotalSpace();
       }
       // implements getTotalSpace() in API lower than GINGERBREAD
       else {
           if (!path.exists()) {
               return 0;
           } else {
               final StatFs stats = new StatFs(path.getPath());
               // Using deprecated method in low API level system, 
               // add @SuppressWarnings("description") to suppress the warning
               return (long) stats.getBlockSize() * (long) stats.getBlockCount();
           }
       }
   }

   ```

#### 总结

在使用高于`minSdkVersion` API level的方法需要:

1. 用`@TargeApi($API_LEVEL)` 使可以编译通过, 不建议使用`@SuppressLint("NewApi")`;
2. 运行时判断API level; 仅在足够高，有此方法的API level系统中，调用此方法;
3. 保证功能完整性，保证低API版本通过其他方法提供功能实现。

 

 

 

# Android 开发之API兼容问题

## 问题背景

鉴于ANDROID SDK 更新较快，很多新的特性和API在低版本中的可能没有。所以开发过程中尽量要保持对新功能接口的兼容。

一般开发过程中APP都会有一个最低版本的配置，例如如果要兼容到android 2.2系统，则可以设置minSdkVersion=8，这就表明能向下兼容到android 2.2版本，即APP能在android2.2版本上的手机也能正常运行，即使可能某些新特性的功能支持失效，但至少保证不会出现崩溃的问题，而避免此问题的方式就要求开发者在代码中做好兼容和适配。

 

## 兼容原则

一般选择APP的最低支持版本原则是尽量向下保持兼容，但也不是说越向下越好，主要的考虑因素有以下几点：

1.      各个低版本手机的市场占有率，比如2013年android 2.2的手机还占用一定的市场份额，但到现在为止基本上该份额可以忽略不计了（目前android 最高的版本已达到android 5.1了）

2.      APP的针对用户群体，比如是高端的用户群体，屌丝用户群体，还是中低端用户群体，根据不同的用户群体可以综合出来决定对最低版本的支持。

## 基于SDK高低开发优缺点

基于低版本的SDK开发

优点就是你可以支持的手机用户会更多，基本上各个版本的用户都可以用你的应用。

但缺点也是非常明显，特别是对开发者来说，需要做好每一个新特性功能的适配和开发，随着版本越来越高，这对开发者后期的维护会越来越困难，越来越多。

基于高版本的SDK开发

如果你用最新的版本的SDK, 优点就是你可以使用最新的功能的api,而且编译也不会出现任何问题。

但是缺点就是你需要时刻对你调用的api保持向下兼容性，因为很有可能你现有调用的某个api在低版本中根本就不存在。这时候你需要考虑低版本系统的用户的运行问题了。

 

 

## 实战分析

如某个工程配置中的最低版本是android2.2，也就是正常来说开发过程中需要基于android SDK为8来做工程开发。但如果你没有基于adroid  2.2 SDK版本开发，而是支持了一个更高的版本，比如android 4.0 SDK开发，那么很多高版本的功能特性（2.3—4.0）在4.0以下的手机中运行就可以存在问题，一般的结果就是直接crash。

下面是基于android2.2 SDK 开发环境编译的最新的工程，其中就有一些直接编译运行不过的错误。下面可以看几个实例：

SampleActivity.java有一处这样写的：

​      if (savedInstanceState !=null) {

​         mOrderId =savedInstanceState.getString(EXTRA_ORDER_ID);

​         mPaySuccess =savedInstanceState.getString(EXTRA_PAY_SUCCESS,"");

​      }

代码中使用Bundle对象在新版本中才提供的方法而没有加兼容处理，如下官方文档中解释，该方法在android 3.1后才有。

#### public [String](http://developer.android.com/reference/java/lang/String.html) getString ([String](http://developer.android.com/reference/java/lang/String.html) key, [String](http://developer.android.com/reference/java/lang/String.html) defaultValue) Added in [API level 12](http://developer.android.com/guide/topics/manifest/uses-sdk-element.html#ApiLevels)

如果在低于android 3.0下机器运行和编译该代码，如果不做任何处理，会直接编译通不过。

 

解决方法：

1.       用android提供的注解 @TargetApi(11)+ 版本号控制做兼容

如果是基于高版本的SDK开发，则新的api肯定会有该方法，如果想让编译的版本在低版本中也能运行，则需要考虑到版本兼容的问题，可以用如下的方式：

/***

​     * 该api版本兼容获取指定参数

​     *

​     * @param savedInstanceState

​     * @return

​     */

   @TargetApi(12)

   privateString getPaySucess(Bundle savedInstanceState) {

​        if (Build.VERSION.SDK_INT >= 12) {

​            mPaySuccess = savedInstanceState.getString(EXTRA_PAY_SUCCESS,"");

​        } else {

​            mPaySuccess = savedInstanceState.getString(EXTRA_PAY_SUCCESS);

​            if (mPaySuccess ==null){

​                mPaySuccess = "";

​            }

​        }

​        returnmPaySuccess;

}

 

2.       用反射的方式调用高版本中的新功能接口进行调用。

如果是基于低版本SDK开发，那么新版本中的新接口肯定会编译不过，这时候可以考虑反射的方式先去查找是否存在这个方法，如果有就代表用户的手机支持该调用方法，如果没有则采用低版本的处理方式。

 

   /***

​     * 通过放射的方式来获取Bundle中的

​     * getString(String key,String value)方法

​     *

​     * @return

​     */

   privateStringgetPaySucessInvoke(Bundle savedInstanceState) {

 

​        try {

​            Class<?> c = Class.forName("android.os.bundle");

​            Method mGetString2Params =c.getDeclaredMethod("getString", String.class,String.class);

 

​            if (mGetString2Params !=null) {

​                mPaySuccess = (String)mGetString2Params.invoke(null,EXTRA_PAY_SUCCESS,"");

​            } else {

​                mPaySuccess = savedInstanceState.getString(EXTRA_PAY_SUCCESS);

​                if (mPaySuccess ==null){

​                    mPaySuccess ="";

​                }

​            }

​        } catch (Exception e) {

​            // TODO: handle exception

​        }

 

​        returnmPaySuccess;

​    }

 

3.       分离代码，分别在不同的SDK上编译运行，最后ClassLoader动态加载高版本中的相关类接口

此方法应用场景如2，可以将高版本的api接口封装后在高版本的SDK中编译运行jar包，供旧版本的工程中动态加载。

## SDK相关对应表

| Platform Version                         | API Level                                | VERSION_CODE                             | Notes                                    |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| [Android 5.1](http://developer.android.com/about/versions/android-5.1.html) | [22](http://developer.android.com/sdk/api_diff/22/changes.html) | [LOLLIPOP_MR1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#LOLLIPOP_MR1) | [Platform Highlights](http://developer.android.com/about/versions/lollipop.html) |
| [Android 5.0](http://developer.android.com/about/versions/android-5.0.html) | [21](http://developer.android.com/sdk/api_diff/21/changes.html) | [LOLLIPOP](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#LOLLIPOP) |                                          |
| Android 4.4W                             | [20](http://developer.android.com/sdk/api_diff/20/changes.html) | [KITKAT_WATCH](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#KITKAT_WATCH) | KitKat for Wearables Only                |
| [Android 4.4](http://developer.android.com/about/versions/android-4.4.html) | [19](http://developer.android.com/sdk/api_diff/19/changes.html) | [KITKAT](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#KITKAT) | [Platform Highlights](http://developer.android.com/about/versions/kitkat.html) |
| [Android 4.3](http://developer.android.com/about/versions/android-4.3.html) | [18](http://developer.android.com/sdk/api_diff/18/changes.html) | [JELLY_BEAN_MR2](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#JELLY_BEAN_MR2) | [Platform Highlights](http://developer.android.com/about/versions/jelly-bean.html) |
| [Android 4.2, 4.2.2](http://developer.android.com/about/versions/android-4.2.html) | [17](http://developer.android.com/sdk/api_diff/17/changes.html) | [JELLY_BEAN_MR1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#JELLY_BEAN_MR1) | [Platform Highlights](http://developer.android.com/about/versions/jelly-bean.html#android-42) |
| [Android 4.1, 4.1.1](http://developer.android.com/about/versions/android-4.1.html) | [16](http://developer.android.com/sdk/api_diff/16/changes.html) | [JELLY_BEAN](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#JELLY_BEAN) | [Platform Highlights](http://developer.android.com/about/versions/jelly-bean.html#android-41) |
| [Android 4.0.3, 4.0.4](http://developer.android.com/about/versions/android-4.0.3.html) | [15](http://developer.android.com/sdk/api_diff/15/changes.html) | [ICE_CREAM_SANDWICH_MR1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#ICE_CREAM_SANDWICH_MR1) | [Platform Highlights](http://developer.android.com/about/versions/android-4.0-highlights.html) |
| [Android 4.0, 4.0.1, 4.0.2](http://developer.android.com/about/versions/android-4.0.html) | [14](http://developer.android.com/sdk/api_diff/14/changes.html) | [ICE_CREAM_SANDWICH](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#ICE_CREAM_SANDWICH) |                                          |
| [Android 3.2](http://developer.android.com/about/versions/android-3.2.html) | [13](http://developer.android.com/sdk/api_diff/13/changes.html) | [HONEYCOMB_MR2](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#HONEYCOMB_MR2) |                                          |
| [Android 3.1.x](http://developer.android.com/about/versions/android-3.1.html) | [12](http://developer.android.com/sdk/api_diff/12/changes.html) | [HONEYCOMB_MR1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#HONEYCOMB_MR1) | [Platform Highlights](http://developer.android.com/about/versions/android-3.1-highlights.html) |
| [Android 3.0.x](http://developer.android.com/about/versions/android-3.0.html) | [11](http://developer.android.com/sdk/api_diff/11/changes.html) | [HONEYCOMB](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#HONEYCOMB) | [Platform Highlights](http://developer.android.com/about/versions/android-3.0-highlights.html) |
| [Android 2.3.4Android 2.3.3](http://developer.android.com/about/versions/android-2.3.3.html) | [10](http://developer.android.com/sdk/api_diff/10/changes.html) | [GINGERBREAD_MR1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#GINGERBREAD_MR1) | [Platform Highlights](http://developer.android.com/about/versions/android-2.3-highlights.html) |
| [Android 2.3.2Android 2.3.1Android 2.3](http://developer.android.com/about/versions/android-2.3.html) | [9](http://developer.android.com/sdk/api_diff/9/changes.html) | [GINGERBREAD](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#GINGERBREAD) |                                          |
| [Android 2.2.x](http://developer.android.com/about/versions/android-2.2.html) | [8](http://developer.android.com/sdk/api_diff/8/changes.html) | [FROYO](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#FROYO) | [Platform Highlights](http://developer.android.com/about/versions/android-2.2-highlights.html) |
| [Android 2.1.x](http://developer.android.com/about/versions/android-2.1.html) | [7](http://developer.android.com/sdk/api_diff/7/changes.html) | [ECLAIR_MR1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#ECLAIR_MR1) | [Platform Highlights](http://developer.android.com/about/versions/android-2.0-highlights.html) |
| [Android 2.0.1](http://developer.android.com/about/versions/android-2.0.1.html) | [6](http://developer.android.com/sdk/api_diff/6/changes.html) | [ECLAIR_0_1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#ECLAIR_0_1) |                                          |
| [Android 2.0](http://developer.android.com/about/versions/android-2.0.html) | [5](http://developer.android.com/sdk/api_diff/5/changes.html) | [ECLAIR](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#ECLAIR) |                                          |
| [Android 1.6](http://developer.android.com/about/versions/android-1.6.html) | [4](http://developer.android.com/sdk/api_diff/4/changes.html) | [DONUT](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#DONUT) | [Platform Highlights](http://developer.android.com/about/versions/android-1.6-highlights.html) |
| [Android 1.5](http://developer.android.com/about/versions/android-1.5.html) | [3](http://developer.android.com/sdk/api_diff/3/changes.html) | [CUPCAKE](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#CUPCAKE) | [Platform Highlights](http://developer.android.com/about/versions/android-1.5-highlights.html) |
| [Android 1.1](http://developer.android.com/about/versions/android-1.1.html) | 2                                        | [BASE_1_1](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#BASE_1_1) |                                          |
| Android 1.0                              | 1                                        | [BASE](http://developer.android.com/reference/android/os/Build.VERSION_CODES.html#BASE) |                                          |

 

 

参考：

<http://developer.android.com/reference/packages.html>