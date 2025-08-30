# 15. Properties, Parameters, and Broadcast Signals

## 15.1 Interface properties and parameters

properties 用于声明一个功能。如果没有声明一个properties，则认为它是False。

properties和parameters：

- Atomic_Transactions：是否支持atomic
- Cache_Stash_Transactions：是否支持cache stashing
- Direct_Memory_Transfer：是否支持DMT
- Direct_Cache_Transfer：是否支持DCT
- Data_Poison：是否支持Poision
- Data_Check：是否支持data check
- Check_Type：当设置为Odd_Parity_Byte_Data时，支持Odd parity checking
- CleanSharedPersistSep_Request：是否支持CleanSharedPersistSep
- MPAM_Support：是否支持MPAM
- CCF_Wrap_Order: Critical chunk first wrap order
- Req_Addr_Width：支持的最大物理地址，合法范围是44到52
- NodeID_Width: 支持的NodeID域宽度，7到11
- Data_Width：data width in DAT channel，合法值128, 256, 512
- Enhanced_Features：
  - Data return from SC state.
  - I/O Deallocation transactions.
  - ReadNotSharedDirty transaction.
  - CleanSharedPersist transaction.
  - Receiving of Forwarding snoops.

## 15.2 Optional interface broadcast signals

协议包含一些可选管脚optional pins，用于确定互连中某些事务组的广播：

• BROADCASTINNER and BROADCASTOUTER.
• BROADCASTCACHEMAINTENANCE and BROADCASTPERSIST.
• BROADCASTATOMIC.

缩写：

BI      BROADCASTINNER.
BO    BROADCASTOUTER.
BCM BROADCASTCACHEMAINTENANCE.
BP     BROADCASTPERSIST.

## 15.3 Atomic transaction support

### 15.3.1 Request Node support

### 15.3.2 Interconnect support

### 15.3.3 Slave Node support
