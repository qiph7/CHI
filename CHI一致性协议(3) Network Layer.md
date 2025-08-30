# 3. Network Layer

确定目的节点ID

## 3.1 System address map

每一个Requester，即系统中的每一个RN和HN，必须包含一个System Address Map (SAM)，用于决定target ID。SAM的范围可能为每一个requester提供一个简单的fixed的node ID。

SAM的格式和结构是implementation defined的，本协议中不规定。

SAM必须提供一个完整的解码地址空间。协议建议任何非法的不存在的地址，都发到一个agent以作适合的错误响应。

## 3.2 Node ID

每个连接到Port的组件，都有一个node ID，用于指示source和destination。一个Port可以有多个node ID，一个node ID只能对应一个Port。

协议中NodeID支持7到11 bits。

node ID的定义和分配是implementation defined的，本协议中不规定。

## 3.3 Target ID determination

### 3.3.1 Target ID determination for Request messages

除了PCrdReturn，规定如下：

- 如果没有pre-allocated credit，那target ID由DVMop Opcode决定，或由其他node ID决定。
- 如果使用pre-allocated credit，那target ID必须是RetryAck中对应的source ID。另外，PrefetchTgtr nodeID mapper与其他请求不同。PrefetchTgt target一直是SN，而其他请求使用地址和HN的 node ID。

对于PCrdReturn:

- 由上一次收到PCrdGrant的source ID决定。

interconnect可能会remap target ID。

### 3.3.2 Target ID determination for Response messages

### 3.3.3 Target ID determination for Snoop Request messages

snoop Request不包含target ID。协议没有定义机制用于确定一个snoop request的target。这个机制是是implementation defined的，本协议中不规定。
