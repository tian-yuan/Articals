#### 网络流量测试



使用 iperf 工具



```
tar zxvf iperf-2.0.5.tar.gz
cd iperf-2.0.5
sudo ./configure
sudo make
sudo make install clean

客户端： iperf -c 192.168.21.68 -f M -p 9099               
服务端： iperf -s -f M -p 9099
```

