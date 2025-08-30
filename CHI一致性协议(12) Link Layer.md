# 12. Link Layer

## 12.1 Introduction

链路层为节点和互连之间的基于数据包的通信提供了简化的机制。

链路层定义：

- Packet和filt的格式
- link之间的控制流

## 12.2 Link

flit通信发生在一对发送者和接收者之间。

连接发送者和接收者的链接叫作link。

link是单向的，一个Node和interconnect的双向连接，需要两个link，一个link1从Node Transmitter到Interconnect的Receiver，一个link2从Interconnect Transmitter到Node Receiver。

### 12.2.1 Outbound and inbound links

outbound link是发送packet的link

inbound link是接收packet的link

## 12.3 Flit

flit是Link层传输的基本单元。

Packet可以分成多个flit传输。flit的类型：

- Protocol flit：运载一个协议packet，有效负载。本协议中，每一个protocol packet与一个protocol flt一一对应。
- Link flit: 运载link维护相关信息。比如transmitter使用Link flit发送Link layer Credit给receiver。Link flit起于一个link transmitter，结束于对面的receiver。

## 12.4 Channel

协议提供一组channel通道，用于flit通信。

每个channel包含定义好的flit格式，格式中定义多个域段以及相关的值。在一些case中，定义好的flit格式同时适用于inbound和outbound。

### 12.4.1 Channel dependencies

## 12.5 Port

描述links, channels, and port的关系。

link分成outbound和inbound link两种，多个link组成一个channel，多个link和多个channel组成一个port。

## 12.6 Node interface definitions

### 12.6.1 Request Nodes

#### RN-F

```
RN-F            ICN
TXREQ ---REQ--> REREQ
RXRSP <--RSP--- TXRSP
RXDAT <--DAT--- TXDAT
RXSNP <--SNP--- TXSNP
TXRSP ---RSP--> RXRSP
TXDAT ---DAT--> RXDAT
```

#### RN-D

SNP channel只能处理DVM transaction。

```
RN-F            ICN
TXREQ ---REQ--> REREQ
RXRSP <--RSP--- TXRSP
RXDAT <--DAT--- TXDAT
RXSNP <--SNP(DVM)--- TXSNP
TXRSP ---RSP--> RXRSP
TXDAT ---DAT--> RXDAT
```

#### RN-I

不需要SNP channel

```
RN-F            ICN
TXREQ ---REQ--> REREQ
RXRSP <--RSP--- TXRSP
RXDAT <--DAT--- TXDAT
TXRSP ---RSP--> RXRSP
TXDAT ---DAT--> RXDAT
```

### 12.6.2 Slave Nodes

```
ICN               SN-F&SN-I
TXREQ ---REQ--> REREQ
RXRSP <--RSP--- TXRSP
RXDAT <--DAT--- TXDAT
TXDAT ---DAT--> RXDAT
```

## 12.7 Channel interface signals

REQ/RSP/SNP/DAT四种channel的接口类似：

```
Transmitter                         Receiver
-------------------------------------------------
TXxxxFLITPEND  ----xxxFLITPEND--->  RXxxxFLITPEND
TXxxxFLITV     ----xxxFLITV------>  RXxxxFLITV
TXxxxFLIT      ====xxxFLIT[X:0]==>  RXxxxFLIT
TXxxxLCRDV     <---xxxLCRDV-------  RXxxxLCRDV
```

通道信号作用：

- FLITPEND：用于指示将有Flit传输，在FLITV的前一拍必须有效。

- FLITV：Flit Valid，指示Flit有效。

- LCRDV：Link-Credit Valid，指示发送LCredit。只有当receiver有空余的Lcredit时，才能把该信号置高。当requester(TX)收到LCredit时，才能发包(FLITV)有效。

- FLIT：传输flit信息。

  除了通道信号，Port连接时还有TXLINKACTIVEREQ和TXLINKACTIVEACK，RXLINKACTIVEREQ和RXLINKACTIVEACK，用于指示Link的状态。

## 12.8 Flit packet definitions

四种flit:

- Request flit: Total R = (121 to 141) + M + X
- Response flit: Total T = 58 to 66
- Snoop flit: Total S = 88 to 104 + M
- Data flit:

Data flit field bits: 

- Total D = (210 to 222) + Y + DC + P for 128 bit Data
- Total D = (354 to 366) + Y + DC + P for 256 bit Data
- Total D = (642 to 654) + Y + DC + P for 512 bit Data

## 12.9 Protocol flit fields

共50个flit fields

## 12.10 Link flit

一个Link flit用于在link deactivation sequence期间给receiver返回L-Credits。Link flit起于一个link transmitter，结束于对面的receiver。

link flit的Opcode filed设置为0。TxnID filed为0。其他filed不需要而且可以设置为任意值。
