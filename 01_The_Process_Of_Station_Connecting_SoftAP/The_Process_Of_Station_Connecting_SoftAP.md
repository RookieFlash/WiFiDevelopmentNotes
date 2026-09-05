打开手机 Wi-Fi，点击一个热点，输入密码并成功连接时，从 Wi-Fi 协议的角度来看，Station 和 AP 之间经历了一系列交互。抓取WiFi 空口 Sniffer 可以看到完整的交互过程。

> 本文不深入分析每一个协议字段，只是通过一组完整的 Sniffer 抓包，从整体上认识 Station 连接 SoftAP 的全过程。


*******************************************************************************
*******************************************************************************

# Station 和 SoftAP 是什么？
| 术语         | 含义                | 简单理解                     |
| :---------: | :-----------------: | :-----------------------: |
| Station（STA）| Wi-Fi 中的客户端角色  | 连接 Wi-Fi AP 的设备         |
| AP         | Access Point        | 为 Statioin 提供 Wi-Fi 接入服务的一端  |
| SoftAP     | Software AP         | 通过软件实现 AP 功能       |
| Station 与 STA | 二者指同一个角色    | `Station` 是完整叫法，`STA` 是缩写 |
| AP 与 SoftAP | SoftAP 是 AP 的一种实现方式 | 可理解为「软件实现的 AP」 |

> Station/STA 描述的是 Wi-Fi 角色，而 SoftAP 描述的是 AP 的一种实现方式。

```
Station                   AP
   |       Wi-Fi 连接      |
   |<--------------------->|
```

*******************************************************************************
*******************************************************************************


# Station 连接 SoftAP 的完整过程

```
Station                      AP
   |                          |
   | <------- Beacon -------- |
   |                          |
   | ---- Probe Request ----> |
   | <--- Probe Response ---- |
   |                          |
   | ---- Authentication ---> |
   | <--- Authentication ---- |
   |                          |
   | ----- Association -----> |
   | <---- Association ------ |
   |                          |
   | <------ EAPOL 1 -------- |
   | ------- EAPOL 2 -------> |
   | <------- EAPOL 3 ------- |
   | ------- EAPOL 4 -------> |
   |                          |
   | ---- DHCP Discover ----> |
   | <------ DHCP Offer ----- |
   | ----- DHCP Request ----> |
   | <------- DHCP ACK ------ |
   |                          |
   | <===== 正常数据通信 =====> |
```

| 阶段编号 | 阶段名称             | 主要目的                                   |
| :------: | :----------------- | :--------------------------------------- |
| ①       | 发现 AP              | STA (Station) 找到 AP                    |
| ②       | Authentication       | 建立 802.11 认证关系                      |
| ③       | Association          | STA(Station) 与 AP 建立关联            |
| ④       | 4-Way Handshake      | 协商并建立数据加密所需的密钥               |
| ⑤       | DHCP                 | STA 获取 IP 地址等网络参数                 |

*******************************************************************************
*******************************************************************************

# 抓 WiFi 空口的环境信息
```
Sniffer 平台：Ubuntu 22.04
Sniffer 工具：aircrack-ng / airodump-ng / Wireshark
STA：Android 手机
SoftAP：Android 手机热点 / 路由器WiFi热点

SoftAP信息：
    SSID：TestAP
    Band：5 GHz
    Channel：149
    Bandwidth：80 MHz
    Security：WPA2-PSK
```

*******************************************************************************
*******************************************************************************

# WiFi 空口 

## 完整连接过程抓包

![complete_process_of_station_connect_softap](./assets/complete_process_of_station_connect_softap.png)


### Frame 类型
| 阶段编号 | 过程 | 802.11 帧/协议        | 作用                |
| ------ | -- | -------------------- | ----------------- |
| ①     | 发现 | Beacon               | AP 周期性广播自身信息    |
| ①     | 发现 | Probe Request        | STA 主动寻找 AP       |
| ①     | 发现 | Probe Response       | AP 响应 STA 的 Probe |
| ②     | 认证 | Authentication       | 802.11 认证         |
| ③     | 关联 | Association Request  | STA 请求加入 AP       |
| ③     | 关联 | Association Response | AP 接受/拒绝 STA      |
| ④     | 安全 | EAPOL                | 安全密钥协商            |
| ⑤     | 网络 | DHCP                 | 获取 IP 等网络参数       |
| ⑥     | 通信 | ARP / IP / TCP / UDP | 上层网络通信            |



## 第一阶段：扫描(Scan)发现 AP/SoftAP

### 被动发现 AP/SoftAP - Beacon

Station 通过监听 Beacon 来发现附近的 AP/SoftAP

```
    AP
    |
    | ---- Beacon ----> Station
    | ---- Beacon ----> Station
    | ---- Beacon ----> Station
```

![passive_scan_beacon](./assets/passive_scan_beacon.png)



### 主动发现 AP/SoftAP - Probe Request / Probe Response

Station通过主动发送Probe Request来发现附近的 AP/SoftAP

```
Station                       AP
   |                           |
   | ---- Probe Request -----> |
   | <--- Probe Response ----- |
   |                           |
```

![active_scan_probe](./assets/active_scan_probe.png)



## 第二阶段：认证(Authentication)

```
Station                           AP
   |                               |
   | -- Authentication Request - > |
   | <-- Authentication Response --|
```

![authentication](./assets/authentication.png)

```
Authentication 并不是我们通常理解的“输入 Wi-Fi 密码进行身份验证”。

在现代 Wi-Fi 网络中，这一步主要是完成 802.11 层面的 Authentication，
为后续的 Association 做准备。

对于 WPA2-PSK 等安全网络，Wi-Fi 密码相关的安全密钥协商并不是在这里完成，
而是在后续的 4-Way Handshake 中进行。
```



## 第三阶段：关联(Association)

```
Station                      AP
   |                          |
   | --- Association Req ---> |
   | <-- Association Resp --- |
```

![association](./assets/association.png)


```
Authentication 完成后，Station 会通过 Association Request 向 AP 请求加入网络。

AP 接收到请求后，会根据 Station 携带的信息进行判断，如果允许该 Station 加入，则返回 Association Response。

Association 成功后，AP 与 Station 就建立了关联关系。
```



## 第四阶段：四次握手(4-Way Handshake)

```
Station                     AP
   |                         |
   | <------- EAPOL 1 -------|
   | ------- EAPOL 2 ------->|
   | <------- EAPOL 3 -------|
   | ------- EAPOL 4 ------->|
```

![4-way-handshake](./assets/4-way-handshake.png)

```
对于 WPA2/WPA3 等需要安全机制的网络，Association 成功后，
Station 和 AP 还需要进行 4-Way Handshake。

如果是开放网络（Open Network），则不会进行 WPA2-PSK 这种意义上的 4-Way Handshake。
```


## 第五阶段：DHCP 获取 IP

```
Station                      AP
   |                          |
   | ---- DHCP Discover ----> |
   | <----- DHCP Offer -------|
   | ---- DHCP Request ------>|
   | <------ DHCP ACK --------|
```

![dhcp](./assets/dhcp.png)

```
本文以 SoftAP 同时提供 DHCP 服务的典型场景为例，因此图中将 DHCP Server 简化为 AP。实际网络中 DHCP Server 也可能是其他设备。
```



## 第六阶段：数据通信

```
Station                      AP
   |                          |
   | <======= Wi-Fi =======>  |
   |                          |
   | <======= IP ===========> |
   |                          |
   | <======= TCP ==========> |
   |                          |
   | <======= HTTP =========> |
```


*******************************************************************************
*******************************************************************************

# 从 Sniffer 抓包重新看整个过程

```
    Station 连接 AP
             │
             ▼
    ┌───────────────────┐
    │ ① 发现 AP/SoftAP │
    │ Beacon / Probe   │
    └────────┬──────────┘
             │
             ▼
    ┌───────────────────┐
    │ ② Authentication │
    │    802.11认证     │
    └────────┬──────────┘
             │
             ▼
    ┌───────────────────┐
    │ ③ Association   │
    │    建立关联关系    │
    └────────┬──────────┘
             │
             ▼
    ┌───────────────────┐
    │ ④ 4-way Handshake │
    │   WPA2/WPA3 安全建立 │
    └────────┬──────────┘
             │
             ▼
    ┌───────────────────┐
    │  ⑤  DHCP       │
    │ 获取 IP 网络参数  │
    └────────┬──────────┘
             │
             ▼
    ┌───────────────────┐
    │  ⑥ 数据通信      │
    │ ARP / DNS         │
    │ TCP / UDP         │
    │ HTTP / RTSP / ... │
    └───────────────────┘
```