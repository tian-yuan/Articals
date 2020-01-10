#### JAVA MQTT 对接示例

目前可以直接使用 org.eclipse.paho 的 mqtt sdk，认证方式有变化

#### maven 配置

```
<dependency>
      <groupId>org.eclipse.paho</groupId>
      <artifactId>org.eclipse.paho.client.mqttv3</artifactId>
      <version>1.2.0</version>
    </dependency>
    <dependency>
      <groupId>commons-codec</groupId>
      <artifactId>commons-codec</artifactId>
      <version>1.9</version>
    </dependency>
```

#### 测试代码

```
package com.netease;

import org.apache.commons.codec.binary.Hex;
import org.eclipse.paho.client.mqttv3.MqttClient;
import org.eclipse.paho.client.mqttv3.MqttConnectOptions;
import org.eclipse.paho.client.mqttv3.MqttException;
import org.eclipse.paho.client.mqttv3.MqttMessage;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;

import javax.crypto.Mac;
import javax.crypto.spec.SecretKeySpec;

/**
 * Hello world!
 *
 */
public class App 
{
    public static String calcPassword(String secret) {
        long counter = System.currentTimeMillis() / (5 * 60 * 1000);
        try {
            Mac sha256_HMAC;
            sha256_HMAC = Mac.getInstance("HmacSHA256");
            SecretKeySpec secret_key;
            secret_key = new SecretKeySpec(secret.getBytes("UTF-8"), "HmacSHA256");
            sha256_HMAC.init(secret_key);
            return "v1:" + Hex.encodeHexString(sha256_HMAC.doFinal(Long.toString(counter).getBytes("UTF-8"))).substring(0, 20);
        } catch (Exception e) {
            return null;
        }

    }

    public static String genUsername(String deviceName, String productKey) {
        if (deviceName == null || deviceName.isEmpty()) {
            return productKey;
        } else {
            return productKey + ":" + deviceName;
        }
    }

    public static void main( String[] args )
    {
        System.out.println( "Hello World!" );
        String productKey   = "ZxYroV2JmruLkY";
        String content      = "Message from MqttPublishSample";
        int qos             = 1;
        String broker       = "ssl://iot-test1.push.126.net:8883";
        String clientId     = "6ad80850-57a7-4b7a-9fd2-ae0654a3d63e";
        String secret       = "2c46934c-d506-4063-ad31-6af8e03576d2";
        String topic        = productKey + "/" + clientId + "/user/report/info/update";
        MemoryPersistence persistence = new MemoryPersistence();

        try {
            MqttClient sampleClient = new MqttClient(broker, clientId, persistence);
            MqttConnectOptions connOpts = new MqttConnectOptions();
            connOpts.setCleanSession(true);
            String username = genUsername("", productKey);
            System.out.println("gen username : " + username);
            connOpts.setUserName(username);
            String password = calcPassword(secret);
            System.out.println("calc password : " + password);
            connOpts.setPassword(password.toCharArray());
            System.out.println("Connecting to broker: "+broker);
            sampleClient.connect(connOpts);
            System.out.println("Connected");
            System.out.println("Publishing message: "+content);
            MqttMessage message = new MqttMessage(content.getBytes());
            message.setQos(qos);
            sampleClient.publish(topic, message);
            System.out.println("Message published");
            sampleClient.disconnect();
            System.out.println("Disconnected");
            System.exit(0);
        } catch(MqttException me) {
            System.out.println("reason "+me.getReasonCode());
            System.out.println("msg "+me.getMessage());
            System.out.println("loc "+me.getLocalizedMessage());
            System.out.println("cause "+me.getCause());
            System.out.println("excep "+me);
            me.printStackTrace();
        }
    }
}
```

