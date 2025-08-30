# 11. System Debug, Trace, and Monitoring

## 11.1 Data Source indication

Suggested DataSource values

| DataSource | Suggested mapping                                            |
| ---------- | ------------------------------------------------------------ |
| 0b0000     | Non-memory default. Source does not support sending a useful DataSource value |
| 0b0001     | Peer processor cache within local cluster                    |
| 0b0010     | Local cluster cache                                          |
| 0b0011     | Interconnect cache                                           |
| 0b0100     | Peer cluster caches                                          |
| 0b0101     | Remote chip caches                                           |

## 11.2 MPAM

Memory System Performance Resource Partitioning and Monitoring (MPAM) 机制用于多用户间高效使用memory资源和监控资源使用率。资源的分组通过Partition ID (PartID) and Performance Monitoring Group (PerfMonGroup).

MPAM域只在REQ和SNP channel使用：

- 在REQ channel，当发送者不想使用MPAM时，MPAM值必须为默认配置
- 在SNP channel，MPAM只用于Stash类型的snoop。在非stash类型snoop，MPAM不适用且必须为默认值。

MPAM域的宽度，要么是0 bit，表示不支持MPAM。要么是11 bit：

- PartID = 9 bit
- PerfMonGroup = 1 bit
- MPAMNS = 1 bit

MPAM域的值的使用是由receiver来implementation defined的

### 11.2.1 MPAMNS

MPAMNS用于指示Non-secure和Secure partition，与NS域分开。不允许MPAMNS=1, NS=0的情况。

### 11.2.2 MPAM value propagation

receiver允许但不强制支持所有partition和group。

允许MPAM域的值传给不支持MAPM的接口。

### 11.2.3 Stash transaction rules

MPAM的值，在Stash snoop时，必须与request相同。

### 11.2.4 Request to Slave rules

MPAM的值，在request to Slave时，必须与request to Home相同。

## 11.3 Completer Busy

Completer Busy用于指示当前的活动水平。

CBusy：3 bit域，适用于适合的DAT和RSP packets

## 11.3.1 Use case

协议没有规定CBusy在Completer如何设置和Requester如何理解。是IMPLEMENTATION DEFINED的。但是协议建议一些实现机制，以避免出现少数非常繁忙的Completer。

Example use case：

CBusy[2]：=1时表示有多个core的请求。

CBusy[1:0]：

• 00 = Less than 50% full.
• 01 = Greater than 50% full.
• 10 - Greater than 75% full.
• 11 = Greater than 90% full.

Requester的预取器可以通过CBusy来调整预取行为：

• If CBusy[2] = 1 and CBusy[1:0] = 11: Disable inaccurate prefetchers.
• If CBusy[1:0] = 10: Very conservative mode on inaccurate prefetchers.
• If CBusy[2] = 1 and CBusy[1:0] = 01: Moderately aggressive mode on inaccurate prefetchers.
• If CBusy[1:0] = 00: Fully aggressive mode on inaccurate prefetchers.

## 11.4 Trace Tag

协议在每个通道提供一个TraceTag bit来支持debugging，tracing，performance measurement。

### 11.4.1 TraceTag usage and rules

TraceTag bit设置和传播的规则：

- TraceTag可以由initiator和interconnect组件设置
- 收到TraceTag的组件，必须保存和并在response packet中返回该值
- 如果涉及多个response，比如Write请求会分开返回Comp和DBIDResp，这些response要返回TraceTag bit。
- 如果收到同一个transaction中的多个packets，只有相关的packet需要设置TraceTag：
  - Write transaction，CompAck不用TraceTag，Comp和DBIDResp要设置TraceTag
  - 如果Comp和DBIDResp包含TraceTag，NCBWrDataCompAck必须设置TraceTag
- 如果interconnect收到包含TraceTag bit的packet，必须保存，且不能reset该值。

