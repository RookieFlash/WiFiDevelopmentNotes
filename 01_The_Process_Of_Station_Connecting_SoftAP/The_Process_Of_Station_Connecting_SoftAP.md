# Station连接SoftAP到底经历了哪些过程？

*******************************************************************************



打开手机 Wi-Fi，选择一个热点并成功连接。

从用户的角度来看，只是“点击热点 → 输入密码 → 连接成功”几个简单操作。

但从 Wi-Fi 协议的角度来看，Station 与 AP 之间实际上经历了一系列交互过程。

通过 Wi-Fi 空口 Sniffer 抓包，可以直观看到这些交互过程。

> 本文不深入分析每一个协议字段，只是通过一组完整的 Sniffer 抓包，从整体上认识 Station 连接 SoftAP 的全过程。




*******************************************************************************
*******************************************************************************



## Station 和 SoftAP 是什么？
| 术语         | 含义                | 简单理解                     |
| :---------: | :-----------------: | :-----------------------: |
| Station（STA）| Wi-Fi 中的客户端角色  | 连接 AP 的终端设备            |
| AP         | Access Point        | 为 Statioin 提供 Wi-Fi 接入服务的一端  |
| SoftAP     | Software AP         | 通过软件实现 AP 功能       |
| Station 与 STA | 二者指同一个角色    | `Station` 是完整叫法，`STA` 是缩写 |
| AP 与 SoftAP | SoftAP 是 AP 的一种实现方式 | 可理解为「软件实现的 AP」 |


```
                 Wi-Fi 网络

       Station                              SoftAP
     ┌─────────┐                          ┌─────────┐
     │ 手机/PC  │       802.11 Wi-Fi       │ 路由器 / │
     │         │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │ SoftAP  │
     └─────────┘                          └─────────┘
```

```
例如：

手机连接家里的路由器时：
   路由器 = AP
   手机 = Station

当手机开启个人热点，其他手机连接这个热点时：
   开启热点的手机 = SoftAP
   连接热点的手机 = Station
```



*******************************************************************************
*******************************************************************************



## Station 连接 SoftAP 的完整过程

```
Station                    SoftAP
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
| ⑥       | Data                 | STA 与 AP/SoftAP 之间的数据通信           |



*******************************************************************************
*******************************************************************************



## 抓 WiFi 空口的环境信息
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



## WiFi 空口 

### 完整连接过程抓包

![complete_process_of_station_connect_softap](./assets/complete_process_of_station_connect_softap.png)

> **说明：**
> Beacon 和 Probe Request / Probe Response 都属于发现 AP 的方式，二者并不是每次连接都会同时发生。
> Station 可以通过监听 Beacon 发现 AP，也可以通过主动发送 Probe Request 发现 AP。
> 为了便于展示完整流程，下面的示意图将两种发现方式都画出来。


#### Frame 类型
| 阶段 | 802.11 报文/帧 | 方向 | 主要作用 |
| :---: | :--- | :---: | :--- |
| ① | Beacon | AP → STA | AP 周期性广播自身信息 |
| ① | Probe Request | STA → AP | STA 主动寻找 AP |
| ① | Probe Response | AP → STA | AP 响应 STA 的 Probe Request |
| ② | Authentication | STA ↔ AP | 完成 802.11 Authentication |
| ③ | Association Request | STA → AP | STA 请求加入 AP |
| ③ | Association Response | AP → STA | AP 接受或拒绝 STA 的关联请求 |
| ④ | EAPOL | STA ↔ AP | 安全密钥协商 |
| ⑤ | DHCP | STA ↔ DHCP Server | 获取 IP、网关、DNS 等网络参数 |
| ⑥ | ARP | STA ↔ AP/其他设备 | IP 与 MAC 地址解析 |
| ⑥ | IP | STA ↔ AP/其他设备 | IP 层数据通信 |
| ⑥ | TCP / UDP | STA ↔ AP/其他设备 | 传输层数据通信 |
| ⑥ | HTTP / RTSP | STA ↔ AP/其他设备 | 应用层数据通信 |



### 第一阶段：扫描(Scan)发现 AP/SoftAP

#### 被动发现 AP/SoftAP - Beacon

Station 通过监听 Beacon 来发现附近的 AP/SoftAP

```
Station                    SoftAP
   |                          |
   | <------- Beacon -------- |
   | <------- Beacon -------- |
   | <------- Beacon -------- |
```

![passive_scan_beacon](./assets/passive_scan_beacon.png)



#### 主动发现 AP/SoftAP - Probe Request / Probe Response

Station通过主动发送Probe Request来发现附近的 AP/SoftAP

```
Station                     SoftAP
   |                           |
   | ---- Probe Request -----> |
   | <--- Probe Response ----- |
   |                           |
```

![active_scan_probe](./assets/active_scan_probe.png)



### 第二阶段：认证(Authentication)

```
Station                         SoftAP
   |                               |
   | -- Authentication Request - > |
   | <-- Authentication Response --|
```

![authentication](./assets/authentication.png)

```
Authentication 并不是我们通常理解的“输入 Wi-Fi 密码进行身份验证”。

在本文的 WPA2-PSK 场景中，可以先简单理解为：
Station 和 AP 先完成 802.11 层面的 Authentication，为后续的 Association 做准备。

需要注意的是，Wi-Fi 密码相关的安全密钥建立并不是在这个 Authentication 过程中完成的，而是在后续的 4-Way Handshake 中进行。

不同 Wi-Fi 安全机制的认证过程有所不同，例如 WPA3-SAE 会涉及不同的认证机制，
本文暂不展开。
```



### 第三阶段：关联(Association)

```
Station                    SoftAP
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



### 第四阶段：四次握手(4-Way Handshake)

```
Station                   SoftAP
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



### 第五阶段：DHCP 获取 IP

```
Station                   DHCP Server
   |                          |
   | ---- DHCP Discover ----> |
   | <----- DHCP Offer -------|
   | ---- DHCP Request ------>|
   | <------ DHCP ACK --------|
```

> **注意：**
> “Wi-Fi 连接成功(**Wi-Fi Connection**)”和“已经获得 IP 地址(**IP Network Connection**)”是两个不同的概念。
>
> 前面的 Authentication、Association 和安全密钥建立，主要解决的是 Station 如何加入 Wi-Fi 网络。
>
> DHCP 则主要负责让 Station 获取 IP 地址、网关、DNS 等网络参数，从而能够进一步进行 IP 层面的网络通信。


![dhcp](./assets/dhcp.png)

```
在 Android SoftAP 等典型场景中，SoftAP 设备通常会同时提供 DHCP Server 服务，
因此 DHCP 报文实际上仍然是在 Station 和 SoftAP 设备之间传输。

需要注意的是，AP 和 DHCP Server 是两个不同的概念：
AP 是提供 Wi-Fi 接入能力的角色，DHCP Server 是负责分配 IP 网络参数的网络服务。
```



### 第六阶段：数据通信

```
┌───────────────────────┐
│       HTTP            │
├───────────────────────┤
│       TCP             │
├───────────────────────┤
│       IP              │
├───────────────────────┤
│   Wi-Fi / 802.11      │
└───────────────────────┘
```

```
完成 Wi-Fi 连接并获取 IP 地址后，Station 和 AP 就可以进行正常的网络通信。

例如访问 HTTP 服务时，可以简单理解为：

HTTP
 ↓
TCP
 ↓
IP
 ↓
Wi-Fi（802.11）
 ↓
无线空口
```



*******************************************************************************
*******************************************************************************



## 从 Sniffer 抓包重新看整个过程

```
    Station 连接 SoftAP
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

简化后的流程

```
Station                    SoftAP
   |                          |
   | <------- 发现 AP -------> |
   |                          |
   | <--- Authentication ---> |
   |                          |
   | <---- Association -----> |
   |                          |
   | <--- 4-Way Handshake --> |
   |                          |
   | <-------- DHCP --------> |
   |                          |
   | <===== 正常数据通信 =====> |
```

```
发现 AP → Authentication → Association → 4-Way Handshake → DHCP → IP 通信
```


> 到这里可以先记住一个最核心的概念：
>
> **Wi-Fi 连接解决的是“怎么加入这个 Wi-Fi 网络”，
> DHCP 解决的是“加入网络之后，怎么获得 IP 等网络参数”，
> 后续的 ARP / IP / TCP / UDP / HTTP 等协议则负责真正的数据通信。**



*******************************************************************************
*******************************************************************************



## 为什么 Sniffer 抓包可能看不到完整流程？

实际抓包时，不一定能够看到本文示意图中的所有报文。

常见原因包括：

1. **Beacon 和 Probe 是两种不同的 AP 发现方式**
   
   Station 可能只通过 Beacon 发现 AP，也可能主动发送 Probe Request。
   因此不一定同时看到两种报文。

2. **抓包开始得太晚**
   
   如果 Station 在开始抓包之前已经完成了 Authentication 或 Association，
   那么这些报文就不会出现在当前抓包文件中。

3. **Sniffer 没有抓到正确的信道**
   
   如果 AP 工作在 5 GHz Channel 149，而 Sniffer 没有监听对应信道，
   就无法正常捕获该信道上的 Wi-Fi 报文。

4. **连接过程失败**
   
   如果 Authentication、Association 或 4-Way Handshake 失败，
   后面的报文自然不会出现。

5. **网络安全配置不同**
   
   例如 Open Network 不会出现 WPA2-PSK 场景下的 4-Way Handshake。

因此，Sniffer 中看到的报文不一定严格等于本文的完整流程图。