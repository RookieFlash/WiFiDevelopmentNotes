# Station 是如何发现 SoftAP的？


## 1. 手机打开Wi-Fi后，是怎么发现附近的热点的？

> 打开手机 Wi-Fi，很快就能看到附近的 Wi-Fi 列表，那么手机是怎么发现附近 Wi-Fi的？

简单来说，手机有 主动扫描 + 被动扫描/监听 两种方式来发现附近的热点。
主动扫描：手机在支持的信道上轮询发送 Probe Request，等待 SoftAP 回复 Probe Response 主动发现附近的 Wi-Fi热点。
被动扫描：手机在支持的信道上轮询监听 SoftAP 周期性广播的 Beacon 来发现 Wi-Fi热点。

*******************************************************************************
*******************************************************************************

## 2. Wi-Fi扫描大致发生了什么？

```
Station                    SoftAP
   |                          |
   | <------- Beacon -------- |
   |                          |
   | ---- Probe Request ----> |
   | <--- Probe Response ---- |
```

SoftAP 可以通过 Beacon 主动“广播自己的存在”，STA也可以通过 Probe Request 主动询问附近是否存在目标网络。

*******************************************************************************
*******************************************************************************

## 3. Beacon：SoftAP 主动告诉啊别人“我在这里”

```
Station                    SoftAP
   |                          |
   | <------- Beacon -------- |
   | <------- Beacon -------- |
   | <------- Beacon -------- |
```

Beacon 可以理解成 SoftAP 周期性发送一张“自我介绍”。例如：

```
我是 MySoftAP
我工作在 5GHz 5745信道上
我的能力是这些
我的安全配置是这些
...
```

在 802.11 网络中，SoftAP 会周期性(约102ms)发送 Beacon Frame，STA 可以通过接受 Beacon 了解附近 BSS 的基本信息。

### Beacon 长什么样子
![beacon](./assets/beacon.png)

| 字段             | 简单理解为       |
| --------------- | ------------ |
| SSID            | Wi-Fi 名称      |
| BSSID           | SoftAP 的 MAC 地址  |
| Beacon Interval | Beacon 发送间隔   |
| RSN             | 安全相关信息       |
| HT/VHT/HE       | AP 支持的 Wi-Fi 能力 |

> Beacon 中还有大量 Information Elements（IE），本文暂时不逐个展开，后续会单独分析。

*******************************************************************************
*******************************************************************************

## 4. Probe Request：STA 主动寻找 SoftAP

```
Station                    SoftAP
   |                          |
   | ---- Probe Request ----> |
   | <--- Probe Response ---- |
```

Beacon 是 SoftAP 主动广播自己的信息。
STA 也可以主动发送 Probe Request，询问附近的 SoftAP

### Probe Request 长什么样子

#### Probe Request - Wildcard - 广播 Probe Request

不指定任何特定的 SSID，相当于客户端在“广撒网”式地询问周围的 AP：“谁在？有哪些网络？”

- 目的地址：通常是广播地址 ff:ff:ff:ff:ff:ff。
- SSID 字段：通配符或留空。
- 用途：用于发现周围所有可用的无线网络
- Wireshark 过滤表达式：wlan.fc.type_subtype == 0x04 && wlan.da == ff:ff:ff:ff:ff:ff

![probe_request_wildcast](./assets/probe_request_wildcast.png)


#### Probe Request - Directed - 定向 Probe Request

有明确目标地询问：“‘TesttAP’这个网络在不在附近？”

- 目的地址：虽然标准流程中通常也是广播地址，但它的核心特征是携带了特定的 SSID。
- SSID 字段：包含客户端想要寻找的那个具体的网络名称。
- 用途：通常用于快速连接之前连接过的、已保存的网络
- Wireshark 过滤表达式：wlan.fc.type_subtype == 0x04 && wlan.ssid == "TestAP"

![probe_request_directed](./assets/probe_request_directed.png)


*******************************************************************************
*******************************************************************************

## 5. Probe Response：SoftAP 回应 STA

```
Station                    SoftAP
   |                          |
   | ---- Probe Request ----> |
   | <--- Probe Response ---- |
```

SoftAP 收到 Probe Request 后，可以回复 Probe Response，把自己的网络信息告诉 STA。

![probe_response](./assets/probe_response.png)


*******************************************************************************
*******************************************************************************

## 6. Beacon 与 Probe 有什么区别？

|      | Beacon   | Probe Request | Probe Response |
| ---- | -------- | ------------- | -------------- |
| 发送方  | AP       | STA           | AP             |
| 方向   | AP → STA | STA → AP      | AP → STA       |
| 主要目的 | 广播 AP 信息 | STA 主动寻找 AP   | AP 回复 STA      |
| 是否主动 | AP主动     | STA主动         | AP响应           |
| 常见用途 | 被动扫描     | 主动扫描          | 主动扫描           |

Beacon 是 AP 主动“喊话”；Probe Request 是 STA 主动“询问”；Probe Response 是 AP 对 STA 的“回答”。

*******************************************************************************
*******************************************************************************

## 7. Active Scan 与 Passive Scan

### Passive Scan
不主动发送 Probe Request，主要通过监听 AP 的 Beacon 来发现网络。

```
Station                    SoftAP
   |                          |
   | <------- Beacon -------- |
   | <------- Beacon -------- |
   | <------- Beacon -------- |
```

### Active Scan

```
Station                    SoftAP
   |                          |
   | ---- Probe Request ----> |
   | <--- Probe Response ---- |
```

*******************************************************************************
*******************************************************************************

## 8. 从 Sniffer 抓包完整看一次扫描

![beacon_probe](./assets/beacon_probe.png)

① Beacon
   AP周期性广播自己的存在

② Probe Request - Directed - **定向 Probe Request**
   STA主动发起扫描 - 定向扫描

③ Probe Response
   AP回应STA

④ Beacon
   AP继续周期性发送Beacon

⑤ Probe Request - Wildcard - **广播 Probe Request**
   STA主动发起扫描 - 广播扫描


*******************************************************************************
*******************************************************************************

## 9. 总结

```
         Station发现SoftAP
                │
      ┌─────────┴─────────┐
      ↓                   ↓
 Passive Scan        Active Scan
      │                   │
      ↓                   ↓
   Beacon          Probe Request
      │                   │
      │                   ↓
      │            Probe Response
      └─────────┬─────────┘
                ↓
            发现SoftAP
                │
                ↓
          Authentication
                │
                ↓
          Association
                │
                ↓
         4-Way Handshake
                │
                ↓
               DHCP
```

*******************************************************************************
*******************************************************************************
