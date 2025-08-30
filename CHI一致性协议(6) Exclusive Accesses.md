# 6. Exclusive Accesses

## 6.1 Overview

一个logical processor (LP)执行exclusive操作的流程：

- 执行Exclusive Load
- 计算一个值
- 执行Exclusive Store将值保存

支持2种Exclusive access:

- Exclusive accesses to a Snoopable memory location.
- Exclusive accesses to a Non-snoopable memory location

## 6.2 Exclusive monitors

Exclusive 流程通过exclusive monitor来跟踪。

### 6.2.1 Snoopable memory location

对于Snoopable memory location，需要两种monitor：

- LP monitor：每个在一个RN-F中的LP都需要实现一个exclusive monitor，用于观察某个地址的exclusive sequence。
- PoC monitor：一个HN-F必须实现一个PoC monitor，来决定一个Exclusive Store transaction是pass还是fail。

### 6.2.2 Additional address comparison

地址比较时，只比较部分bit，而不是full address.

### 6.2.3 Non-snoopable memory location

System monitor用于跟踪Non-snoopable memory location的Exclusive accesses.

## 6.3 Exclusive transactions

transaction通过设置Excl bit域来支持Exclusive accesses。支持以下transaction：

• Exclusive Load transaction to a Snoopable location:
— ReadClean.
— ReadNotSharedDirty.
— ReadShared.
• Exclusive Store transaction to a Snoopable location:
— CleanUnique.
• Exclusive Load transaction to a Non-snoopable location:
— ReadNoSnp.
• Exclusive Store transaction to a Non-snoopable location:
— WriteNoSnp.

### 6.3.1 Responses to exclusive requests

对exclusive transaction的response与normal response相似，差异是RespErr域可用于表示exclusive request成功还是失败。

RespErr field value of 0b01, Exclusive Okay, indicates a pass 

RespErr field value of 0b00, Normal Okay, indicates an Exclusive access failure.

### 6.3.2 System responsibilities

CHI协议系统必须有以下责任：

- 每个LP应该包含一个monitor，用于管理Exclusive accesses.
- 必须有一个starvation prevention机制。
- 协议建议安全和非安全的Exclusive request相互独立。

### 6.3.3 Exclusive accesses to Snoopable locations

Snoopable Exclusive Load

Snoopable Exclusive Load to Snoopable Exclusive Store

Snoopable Exclusive Store

Exclusive accesses to Non-snoopable locations
