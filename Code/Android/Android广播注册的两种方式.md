# android注册广播的两种方式与区别

2016年02月24日 12:21:47

阅读数：22217

Android广播机制概述 
Android广播分为两个方面：广播发送者和广播接收者，通常情况下，BroadcastReceiver指的就是广播接收者（广播接收器）。广播作为Android组件间的通信方式，可以使用的场景如下： 
1.同一app内部的同一组件内的消息通信（单个或多个线程之间）； 
2.同一app内部的不同组件之间的消息通信（单个进程）； 
3.同一app具有多个进程的不同组件之间的消息通信； 
4.不同app之间的组件之间消息通信； 
5.Android系统在特定情况下与App之间的消息通信。

静态注册:

定义一个广播接收器继承BroadcastReceiver

```
package com.example.administrator.broadcastdemo;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class MyReceiver extends BroadcastReceiver {
    public MyReceiver() {
    }
    @Override
    public void onReceive(Context context, Intent intent) {       
        Toast.makeText(context,intent.getStringExtra("info"),Toast.LENGTH_SHORT).show();
    }
}123456789101112131415
```

AndroidManifest.xml

```
  <receiver
        android:name=".MyReceiver"
        android:enabled="true"
        android:exported="true">
        <!-- 静态注册广播 -->
        <!-- intent过滤器,指定可以匹配哪些intent, 一般需要定义action 可以是自定义的也可是系统的 -->  
        <intent-filter>
        <!--action-->
            <action android:name="com.broadcast.test" />
       </intent-filter>
 </receiver>1234567891011
```

MainActivity.class

```
package com.example.administrator.broadcastdemo;

import android.content.Intent;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //静态注册广播
        Intent intent=new Intent();
        //与清单文件的receiver的anction对应
        intent.setAction("com.broadcast.test");
        intent.putExtra("info","测试静态注册广播");
        //发送广播
        sendBroadcast(intent);
    }
}
123456789101112131415161718192021
```

动态注册: 
定义一个广播接收器继承BroadcastReceiver

```
package com.example.administrator.broadcastdemo;

import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class DynamicReceiver extends BroadcastReceiver {
    public DynamicReceiver () {
    }

    @Override
    public void onReceive(Context context, Intent intent) {

        Toast.makeText(context,intent.getStringExtra("name"),Toast.LENGTH_SHORT).show();
    }
}
123456789101112131415161718
```

Main2Activity.class

```
package com.example.administrator.broadcastdemo;

import android.content.Intent;
import android.content.IntentFilter;
import android.content.pm.ActivityInfo;
import android.content.res.Configuration;
import android.net.ConnectivityManager;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.widget.Toast;

public class Main2Activity extends AppCompatActivity {
    DynamicReceiver dynamicReceiver;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main2);
        //动态注册广播
        dynamicReceiver = new DynamicReceiver();
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("com.broadcast.test2");
        registerReceiver(dynamicReceiver, intentFilter);
        //发送信息
        Intent intent=new Intent();
        intent.setAction("com.broadcast.test2");
        intent.putExtra("name", "动态注册广播");
        sendBroadcast(intent);
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        //解除广播
        unregisterReceiver(dynamicReceiver);
    }
 }123456789101112131415161718192021222324252627282930313233343536
```

AndroidManifest.xml

```
  <receiver
     android:name=".DynamicReceiver"
     android:enabled="true"
     android:exported="true">
  </receiver>12345
```

总结:

1)静态注册：在AndroidManifest.xml注册，android不能自动销毁广播接收器，也就是说当应用程序关闭后，还是会接收广播。 
2)动态注册：在代码中通过registerReceiver()手工注册.当程序关闭时,该接收器也会随之销毁。当然，也可手工调用unregisterReceiver()进行销毁。

android:enabled: 
这个属性用于定义系统是否能够实例化这个广播接收器，如果设置为true，则能够实例化，如果设置为false，则不能被实例化。默认值是true。 
`<application>`元素有它自己的enabled属性，这个属性会应用给应用程序的所有组件， 
包括广播接收器。`<application>`和`<receiver>`元素的这个属性都必须是true，这个广播接收器才能够被启用。如果有一个被设置为false，该广播接收器会被禁止实例化。 
android:exported: 
这个属性用于指示该广播接收器是否能够接收来自应用程序外部的消息，如果设置true，则能够接收，如果设置为false，则不能够接收。如果设置为false，这该接收只能接收那些由相同应用程序组件或带有相同用户ID的应用程序所发出的消息。