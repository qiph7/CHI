# 4. Coherence Protocol

第4章比较重要，重点学习。表格和数据信息太多，没有全部记录。

## 4.1 Cache line states

I Invalid:
• The cache line is not present in the cache.

UC Unique Clean:
• The cache line is present only in this cache.
• The cache line has not been modified with respect to memory.
• The cache line can be modified without notifying other caches.
• In response to a snoop that requests data, the cache line is permitted, but not required to be:
— Returned to Home when requested.
— Forwarded directly to the Requester when instructed by the snoop.

UCE Unique Clean Empty:
• The cache line is present only in this cache.
• The cache line is in a unique state but none of the data bytes are valid.
• The cache line can be modified without notifying other caches.
• In response to a snoop that requests data, the cache line must not be:
— Returned to Home even when requested.
— Forwarded directly to the Requester even when instructed by the snoop.

UD Unique Dirty:
• The cache line is present only in this cache.
• The cache line has been modified with respect to memory.
• The cache line must be written back to next level cache or memory on eviction.
• The cache line can be modified without notifying other caches.
• In response to a snoop that requests data, the cache line must be:
— Returned to Home when requested.
— Forwarded directly to the Requester when instructed by the snoop.

UDP Unique Dirty Partial:
• The cache line is present only in this cache.
• The cache line is unique. Only a part of the cache line is Valid and Dirty.
• The cache line has been modified with respect to memory.
• When the cache line is evicted, it must be merged with data from next level cache or memory
to form the complete Valid cache line.
• The cache line can be modified without notifying other caches.
• In response to a snoop that requests data, the cache line must:
— Be returned to Home.
— Not forward the cache line directly to the Requester even when instructed by the
snoop.

SC Shared Clean:
• Other caches might have a shared copy of the cache line.
• The cache line might have been modified with respect to memory.
• It is not the responsibility of this cache to write the cache line back to memory on eviction.
• The cache line cannot be modified without invalidating any shared copies and obtaining
unique ownership of the cache line.
• In response to a snoop that requests data, the cache line:
— Is required to not return data if RetToSrc bit is not set.
— Can return data if RetToSrc bit is set.
— Is forwarded directly to the Requester when instructed by the snoop.

SD Shared Dirty:
• Other caches might have a shared copy of the cache line.
• The cache line has been modified with respect to memory.
• The cache line must be written back to next level cache or memory on eviction.
• The cache line cannot be modified without invalidating any shared copies and obtaining
unique ownership of the cache line.
• In response to a snoop that requests data, the cache line must be:
— Returned to Home when requested.
— Forwarded directly to the Requester when instructed by the snoop.

### 4.1.1 Empty cache line ownership

UCE状态，拥有该cache line的Unique状态，但data都是invalid的。

两个例子获取empty cache line:

- Requester故意获取empty cache line，以用于后面的写操作。
- Requester已有一个该cache line的copy，可以转换为空状态，以用于后面的写操作。

### 4.1.2 Ownership of cache line with partial Dirty data

UDP状态，允许store to the cache line，store之前仍为UDP状态。

## 4.2 Request types

• Read requests:
— A data response is provided to the Requester.
— Can result in data movement among other agents in the system.
— Can result in a cache state change at the Requester.
— Can result in a cache state change at other Requesters in the system.

• Dataless requests:
— No data response is provided to the Requester.
— Can result in data movement among other agents in the system.
— Can result in a cache state change at the Requester.
— Can result in a cache state change at other Requesters in the system.

• Write requests:
— Move data from the Requester.
— Can result in data movement among other agents in the system.
— Can result in a cache state change at the Requester.
— Can result in a cache state change at other Requesters in the system.

• Atomic requests:
— Move data from the Requester.
— A data response is provided to the Requester in some Request types.
— Can result in data movement among other agents in the system.
— Can result in a cache state change at the Requester.
— Can result in a cache state change at other Requesters in the system.

• Other requests:
— Do not involve any data movement in the system.
— Can be used to assist with Distributed Virtual Memory (DVM) maintenance.
— Can be used to warm the memory controller for a following read request.

### 4.2.1 Read transactions

ReadNoSnp

ReadNoSnpSep

ReadOnce

ReadOnceCleanInvalid

ReadOnceMakeInvalid

ReadClean

ReadNotSharedDirty

ReadShared

ReadUnique

### 4.2.2 Dataless transactions

CleanUnique

MakeUnique

Evict

StashOnceUnique

StashOnceShared

Cache maintenance transactions：

CleanShared

CleanSharedPersist

CleanSharedPersistSep

CleanInvalid

MakeInvalid

### 4.2.3 Write transactions

Write transactions从Requester搬移数据到Completer，可能是next level cache, meory, 外设。根据不同的transaction type，可以是coherent或non-coherent。

WriteNoSnpFull

WriteNoSnpPtl

WriteUniqueFull

WriteUniquePtl

WriteUniqueFullStash

WriteUniquePtlStash

CopyBack transactions：

WriteBackFull

WriteBackPtl

WriteCleanFull

WriteEvictFull

### 4.2.4 Atomic transactions

Atomic transaction操作允许Requester发送transaction给interconnect，包含memory address和operation。该transaction把operation移动到更靠近数据保存的位置，这对原子执行操作来更新memory很有用，可提升性能。

如果没有Atomic transaction，一个原子操作需要由一系列的memory accesses完成，这些access依赖如Exclusive reads和wriates.

使用Atomic操作后：

- 可以有更确定性的时延
- block access memory的时延减少了，也就减少了其他agent memory access的影响。
- 不同Requester的memory access的公开性问题变得简单，因为atomic operation是由PoS或PoC来仲裁。

协议定义两个相关的术语：

- Atomic Operation：是一个execution of a function
- Atomic Transaction: 是一个transaction

四种Atomic transaction types:

- • AtomicStore.
- • AtomicLoad.
- • AtomicSwap.
- • AtomicCompare.

Atomic Operation中的data elements：

- TxnData： The write data in the AtomicLoad, and AtomicStore transactions.
- CompareData： The compare value in the AtomicCompare transaction.
- SwapData： The swap value in the AtomicCompare, and AtomicSwap transactions.
- InitialData： The content of the addressed location before the atomic operation.

AtomicStore：STADD STCLR STEOR STSET STSMAX STSMIN STUMAX STUMIN

AtomicLoad：LDADD LDCLR LDEOR LDSET LDSMAX LDSMIN  LDUSMAX LDUMIN

AtomicSwap：只有一个支持的Atomic Operation

AtomicCompare：只有一个支持的Atomic Operation

### 4.2.5 Other transactions

DVM transactions: DVMOp

Prefetch transaction: PrefetchTgt

## 4.3 Snoop request types

ICN生成一个snoop request，用于回应RN的一个请求，或因为内部cache或snoop filter维护操作。除了SnpDVMOp，snoop transaction在RN-F的cache data上operates。SnpDVMOp的操作在target node上执行DVM maintenance operation。

SnpOnceFwd, SnpOnce

SnpStashUnique

SnpStashShared

SnpCleanFwd, SnpClean

SnpNotSharedDirtyFwd, SnpNotSharedDirty

SnpSharedFwd, SnpShared

SnpUniqueFwd, SnpUnique

SnpUniqueStash

SnpCleanShared

SnpCleanInvalid

SnpMakeInvalid

SnpMakeInvalidStash

SnpDVMOp

## 4.4 Request types and corresponding snoop requests

对某一种Requster，ICN会有Expected Snoop，也可能没有Snoop。

## 4.5 Response types

### 4.5.1 Completion response

除了PCrdReturn and PrefetchTgt，所有transaction都要求completion response。completion保证request已经达到PoS或PoC，会保证同地址保序。

Resp域用于表示response的cache state和pass dirty。

Read and Atomic transaction completion

Dataless transaction completion

Write and Atomic transaction completion

### 4.5.2 WriteData response

CopyBackWrData

NonCopyBackWrData

NCBWrDataCompAck

### 4.5.3 Snoop response

Snoop response without data

Snoop response without Data to Home and Direct Cache Transfer (DCT)

Snoop response with data

Snoop response with partial data

Snoop response with Data to Home and DCT

### 4.5.4 Miscellaneous response

CompAck

RetryAck

PCrdGrant

ReadReceipt

DBIDResp

## 4.6 Silent cache state transitions

cache可以因为内部事件改变状态，而不用通知其他的系统。

合法的silent cache state转换：

| RN_F action      | RN-F cache state present | RN-F cache state Next | Notes                            |
| ---------------- | ------------------------ | --------------------- | -------------------------------- |
| cache eviction   | UC                       | I                     | 可以使用Evict或WriteEvictFull    |
|                  | UCE                      | I                     | 可以使用Evict                    |
|                  | SC                       | I                     | 可以使用Evict                    |
| Local sharing    | UC                       | SC                    |                                  |
|                  | UD                       | SD                    |                                  |
| Store            | UC                       | UD                    | Full or partial cache line store |
|                  | UCE                      | UDP                   | Partial cache line store         |
|                  | UCE                      | UD                    | Full cache line store            |
|                  | UDP                      | UD                    | Store that fills the cache line  |
| Cache Invalidate | UD                       | I                     | 可以使用Evict                    |
|                  | UDP                      | I                     | 可以使用Evict                    |

不允许UC到UCE转换。

## 4.7 Cache state transitions at a Requester

一个cache line数据，要么缓存在RN的cache中(信息记录在RN对应的directory中)，要缓存在HN的cache中(信息记录在HN对应的Tag中)，要么二者都有。一个transaction的前后，都会导致该cache line在RN或HN状态变化。

## 4.8 Cache state transitions at a Snoopee

RN-F收到Snoop后，有两个action。一个是改变cache line状态，第二个是发送response。

## 4.9 Returning Data with Snoop response

The rules for returning a copy of the cache line with the Snoop response are detailed below:
For Non-forwarding snoops, except SnpMakeInvalid, the rules for returning a copy of the cache line to the Home
are:
• Irrespective of the value of RetToSrc, must return a copy if the cache line is Dirty.
• Irrespective of the value of RetToSrc, optionally can return a copy if the cache line is Unique Clean.
• If the RetToSrc value is 1, must return a copy if the cache line is Shared Clean and the Snoopee retains a copy
of the cache line.
• If the RetToSrc value is 0, must not return a copy if the cache line is Shared Clean.
For forwarding snoops, the rules for returning a copy of the cache line to the Home are:
• Irrespective of the value of RetToSrc, must return a copy if a Dirty cache line cannot be forwarded or kept.
• If the RetToSrc value is 1, must return a copy if the cache line is Dirty or Clean.
• If the RetToSrc value is 0, must not return a copy if the cache line is Clean.
RetToSrc is applicable and must be set to zero in:
• Stash snoops.
• SnpCleanShared, SnpCleanInvalid, and SnpMakeInvalid,
• SnpOnceFwd and SnpUniqueFwd.
RetToSrc is applicable and can take any value in all other snoops except SnpDVMOp.
RetToSrc is inapplicable and must be set to zero in SnpDVMOp.
Home must only set RetToSrc on the Snoop request to a single Request Node.

## 4.10 Do not transition to SD

DoNotGotoSD用于指示the Snoopee must not transition to SD state as a result of the Snoop request.

The field is applicable and can take any value in:
• SnpOnce, SnpOnceFwd.
• SnpClean, SnpCleanFwd.
• SnpNotSharedDirty, SnpNotSharedDirtyFwd.
• SnpShared, SnpSharedFwd.
The field is applicable and must be set to 1 in:
• SnpUnique, SnpUniqueFwd.
• SnpCleanShared.
• SnpCleanInvalid.
• SnpMakeInvalid.
The field is inapplicable and must be set to 0 in SnpDVMOp.
The field is not present in Stash Snoops.

## 4.11 Hazard conditions

介绍RN-F和HN-F处理Snoopable transaction的地址冲突和竞争条件。

多个Requester会同时发送transactions，每个Requester也会outstanding多个requests，接收多个outstanding的snoop requests。ICN（HN-F,HN-I,MN)负责定义这些transactions的顺序，保证这些顺序对所有component是相同的。

### 4.11.1 At the RN-F node

除了SnpDVMOp(sync)，RN-F必须respond收到的Snoop request，及时的，不新产生任何依赖协议层的请求。

如果有一个同cache line的一个pending request，而且这个pending request没有收到任何数据包：

- snoop request必须正常执行。
- cache状态转换必须对每一种request类型可用。
- cache data或CopyBack request data必须通过snoop response返回，或forwarded给Reqeuster，如果Snoop request type、Snoop request属性、cache状态需要。

如果有一个同cache line的一个pending request，而且这个pending request收到至少一个数据包：

- RN-F必须等待收到所有Data response packets后，再responding to the Snoop request.
- 一旦RN-F收到所有Data response packets：
  - Snoop request必须正常执行
  - cache状态转换必须对每一种request类型可用。
  - cache data必须通过snoop response返回，或forwarded给Reqeuster，如果Snoop request type、Snoop request属性、cache状态需要。

如果pending request是CopyBack reqeust:

- 收到CompDBIDResp后request transaction flow必须完成。
- 在WriteData response中，cache状态必须是snoop request后的状态，而不是发送CopyBack request时的状态。
- 如果Snoop response后cache的状态是I或SC，允许RN不发送有效的CopyBack Data。在WriteData response中，cache状态在CopyBack data后被snoop取走了，必须设置为I，所有bytes必须设置为deaaserted，对应的data必须为zero。
- 如果WriteData中包含data，则数据必须与Snoop response中的data一样或更新。

对于non-ordered transaction，RN可以不用等待DataSepResp再发CompAck。对于ordered transactions，RN需要等待收到DataSepData后再发CompAck。两种情况中，RN必须等收到所有data packets后再respond to Snoop request。

对于同一个cache line，在收到一个pending CopyBack request的response前，RN-F可能收到多个snoop request。这时data response包含的cache line状态是最后一个snoop request对应的response完成的cache line状态。这种场景是可能的，因为CopyBack request可能在HN-F排队在多个Read和Dataless request的后面。

### 4.11.2 At the ICN(HN-F) node

HN-F通过对transaction response和snoop transaction进行排序到同一个cache line。因为interconnect不要求保序，所以消息的到达顺序，并不是和消息在HN-F发送的顺序一致。

如果Response message包含的数据需要多个packet或beats来在interconnect传输，发送或接收这些message意味着发送或接收所有对应的packets。也就是说，当Home发送message时，它必须发送所有的packet，不依赖其他任何请求或response。

相似地，当Home接收一部分data message时，必须接收message剩下的packets，不依赖其他任何请求或response。

后续依赖于一个Data message的行为，必须等所有packet收到后，才继续向前执行。

当Snoop transaction response is pending，同地址的transaction response只允许：

• RetryAck for a CopyBack.
• RetryAck and DBIDResp for a WriteUnique and Atomics.
• RetryAck and, if applicable, a ReadReceipt for a Read request type.
• RetryAck for a Dataless request type.

一旦completion is sent for a transaction，对于同一条cache line，HN-F不能发送snoop request，直到收到：

• A CompAck for any Read and Dataless requests except for ReadOnce* and ReadNoSnp.
• A WriteData response for CopyBack and Atomic requests.
• For WriteUnique, a WriteData response and, if applicable, CompAck.
