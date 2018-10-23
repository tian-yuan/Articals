#### broadcast receiver 权限问题

##### 问题描述

```
AndroidManifest.xml

<receiver
  android:name=".MessageListener"
  android:exported="false"
  android:permission="com.android.permission.PRODUCT_SERVICE">
  <intent-filter>
  <action android:name="application_v4"/>
  </intent-filter>
</receiver>

<uses-permission android:name="com.android.permission.PRODUCT_SERVICE"/>


Intent broadcastIntent = new Intent();
broadcastIntent.setFlags(Intent.FLAG_INCLUDE_STOPPED_PACKAGES);
broadcastIntent.setAction("application_v4");
broadcastIntent.putExtra("message", "test message.");
sendBroadcast(broadcastIntent, "com.android.permission.PRODUCT_SERVICE");
```

MessageListener 里面的 onReceive 函数没有被调用

##### 问题解决

permission 需要申明

```
AndroidManifest.xml

<receiver
  android:name=".MessageListener"
  android:exported="false"
  android:permission="com.android.permission.PRODUCT_SERVICE">
  <intent-filter>
  <action android:name="application_v4"/>
  </intent-filter>
</receiver>

<permission android:name="com.android.permission.PRODUCT_SERVICE"></permission>

<uses-permission android:name="com.android.permission.PRODUCT_SERVICE"/>
```

