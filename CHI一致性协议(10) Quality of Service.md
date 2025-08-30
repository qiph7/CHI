# 10. Quality of Service

## 10.1 Overview

系统利用QoS的目的：

- 保证特定流中的transactin的最大latency
- 保证请求流的最小带宽
- 尽可能保证为特定流的请求提供的带宽和延迟。

## 10.2 QoS priority value

使用4-bit QoS Priority Value (PV)来表示优先级。一般会和source type and the class of traffic结合使用，QoS越大优先级越高。可能根据一些累积的延迟和所需的吞吐量度量动态地改变QoS值。

## 10.3 Repeating a transaction with higher QoS value

当带某个QoS的transaction发出后，允许发送相同的但是QoS不同的(一般是QoS更高的)transaction。Completer需要处理这种场景。如果发生Retry，允许cancel掉该transaction并返回credit。
