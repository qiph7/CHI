# 9. Error Handling

## 9.1 Error types

The packet level error reporting types:

- Data Error, DERR: 

  Used when the correct address location has been accessed, but an error is detected within
  the data. Typically, this is used when data corruption has been detected by ECC or a parity
  check.
  Data Error reporting is supported by the RespErr, Poison, and DataCheck fields in the DAT
  packet.
  When processing of a request received by Home that is required to be propagated to the
  Slave results in a DERR, the Home must not stop propagating the request to the Slave.

- Non-data Error, NDERR

  Used when an error is detected that is not related to data corruption. This specification does
  not define all cases when this error type is reported. Typically, this error type is reported for:
  • An attempt to access a location that does not exist.
  • An illegal access, such as a write to a read only location.
  • An attempt to use a transaction type that is not supported.

## 9.2 Error response fields

| RespErr[1:0] | Name  | Description                                                  |
| ------------ | ----- | ------------------------------------------------------------ |
| 0b00         | OK    | Okay. Indicates that a Non-exclusive access has been successful. Also used to indicate an Exclusive access failure. |
| 0b01         | EXOK  | Exclusive Okay. Indicates that either the read or write portion of an Exclusive access has been successful. |
| 0b10         | DERR  | Data Error.                                                  |
| 0b11         | NDERR | Non-data Error.                                              |

## 9.3 Errors and transaction structure

即使是有error的response，所有transaction必须遵循协议且完成对应的行为。

因为同有机制来传播request或snoop的error，当interconnect检测到error时，request不能使用DMT或DCT。

如果transaction中包含data packet，则必须发送数量正确的data packet，但不要求data values有效。

如果transaction中没有一个合法的cache 状态，则RespErr域必须指示Non-data error。snoop response如果没有一个合法的cache 状态，则不能返回包含data的response。

每个packet中response中的Resp域必须相同，不管是否有错误指示。

## 9.4 Error response use by transaction type

对各种类型的transaction，允许使用的error field的值。包括Request Snoop DBIDResp Comp CompAck DVMOp Stash等。

## 9.5 Poison

Poison bit用于指示一组数据字节先前已被破坏。DAT packet中包含Poison bit，用于指示未来的使用者数据已被破坏。

Poison支持：

- DAT packet中，1个poison bit表示64 bits数据。
- Data marked as poisoned的数据，不能被任何master使用，允许poisoned的数据存储在cache和memory中。
- Poison设置后，数据传输过程中必须保留
- 当检测到中毒错误时，允许对数据进行过度中毒。

如果64 bit的chunk中有任何有效的字节，则Poison必须是准确的，这就是Poison粒度，否则Poison位是一个Don't Care，即当64位的chunk中所有8个字节都是无效的，那么它就是一个Don't Care。

Data_Poison属性用于表示该component是否支持Poison。

## 9.6 Data Check

DataCheck用于检测DAT packet中data errors。

当支持Data check时：

- DAT packet中每64bit数据包含8 bit Data Check。
- Data Check bit是产生奇字节奇偶校验的奇偶校验位。

Data_Check属性用于表示支持Data Check。

> Interface parity是可选扩展，用于DataCheck字段在DAT通道上提供的错误检测.

## 9.7 Use of interface parity

对于安全关键型应用，有必要检测并纠正SoC内单个导线上的瞬态和功能错误。
系统组件中的错误可能会在连接的组件中传播并导致多个错误。Error detection and correction错误检测和纠正（EDC）需要端到端操作，涵盖从源到目的的所有逻辑和线路。
实现端到端保护的一种方法是在组件中使用定制的EDC方案，并在组件之间实现简单的错误检测方案。这些组件之间没有逻辑，连接相对较短。本节介绍器件间接口单bit错误检测的奇偶校验方案。如果多比特错误发生在不同的奇偶校验信号组中，则可能会检测到它们。

### 9.7.1 Byte parity check signals

### 9.7.2 Error detection behavior

### 9.7.3 Interface parity check signals

## 9.8 Interoperability of Poison and DataCheck

如果数据包的接收者不支持毒性和数据检查功能，则互连必须枚举并根据需要将毒性和数据检查错误响应转换为DAT数据包中的数据错误。

如果对毒害和数据检查功能的支持跨接口不相似，则适用以下规则：

- 如果跨接口不支持毒药，则必须映射到DataCheck或DERR。在这样的接口中，如果支持DataCheck，则不需要映射到DataCheck而不是DERR。
- 如果跨接口不支持DataCheck，则DataCheck必须映射到毒药或DERR。在这样的接口中，DataCheck是预期的，但不要求映射到毒药，而不是DERR，如果支持毒药。

## 9.9 Hardware and software error categories

### 9.9.1 Software based error

software based error发生在多个access到相同的location，但是有不匹配的Snoopable或Memory属性。

software based error会导致loss of coherency和corruption of data values。协议要求系统不能因为software based error导致死锁，而且transaction能继续完成执行。

一个software based error，access在4KB的memory范围内，不能导致其他4KB范围的data corruption。

对于Normal memory loaction，适合的store和software cache maintenance可以使memory location回到定义的状态。

当访问外设时，外设的操作不能保证正确。唯一的要求是外设继续以符合协议的方式响应事务。将被错误访问的外围设备返回到已知工作状态可能需要的事件序列是IMPLEMENTATION DEFINED。

### 9.9.2 Hardware based error

hardware based error定义为非software based error的error。

如果发生基于硬件的错误，则无法保证从错误中恢复。系统可能崩溃、锁定或遭受其他不可恢复的故障。
