# 7. Cache Stashing

## 7.1 Overview

cache stashing是一个将数据放到特定cache的机制，使数据更靠近使用方，提升系统性能。

cache stashing只允许对Snoopable memory使用。

协议支持两种cache stash transaction:

- Write with stash hint : WriteUniqueStash. 用于当写数据时，数据应该allocated在哪个cache是已知的情况。
- Independent stash request：StashOnce。用于写数据和stash data request请求是分开的情况。

两种方式都支持stash数据到不同level的cache中。stash target可以是peer cache, 或者logical processor cache，也可以target到below peer cache。

所有的stash都是hint，允许receiver不执行stashing行为。

### 7.1.1 Snoop requests and Data Pull

以下snoop request用于peer cache的stash request:

• SnpUniqueStash.
• SnpMakeInvalidStash.
• SnpStashUnique.
• SnpStashShared.

Snoopee接收到stash类型的Snoop request时的行为：

- 像处理Read请求一样提供snoop response。包含read request和snoop response认为是Data Pull。Data Pull只在DoNotDataPull域deasserted时可用。
- 提供Snoop response不带Data Pull，即忽略stash hint。

## 7.2 Write with Stash hint

Requester / Home / Stash target需要分别处理Stash hint。

Requester responsibilities:

- 发送WriteUniqueFullStash or WriteUniquePtlStash
- request包含Stash target

Home responsibilities:

- 允许发RetryAck
- 发送SnpUniqueStash给Stash target
- 发送SnpUnique给其他share该cache line的requester
- 对于WriteUniqueFullStash.，允许发送SnpMakeInvalidStash and SnpMakeInvalid，而不是SnpUniqueStash and SnpUnique
- 允许忽略Write request中的stash hint，把request当作普通WriteUnique处理。
- 当Stash target没有指定时，也可以处理(查看Stash target not specified章节)
- 允许使用DMT从SN-F获取数据
- 允许使用分开的Non-data和Data-only response。

Stash target responsibilities:

- 包含Data Pull的response有：SnpResp_I_Read. SnpRespData_I_Read. SnpRespData_I_PD_Read. SnpRespDataPtl_I_PD_Read

- DoNotDataPull assert时，或者snoop地址hazard with an outstanding request时，不能使用Data Pull

- 当requesting Data Pull:

  - Stash target保证Read data接收不能导致死锁
  - Read request被当成ReadUnique对待
  - Stash target必须使用Home的Read transaction的TxnID来填充DBID。如果Data Pull的snoop response包含数据，则所有data packet的DBID必须相同。

- 允许忽略Stash hint，当作SnpUnique处理。

## 7.3 Independent Stash request

implementing cache stashing允许Stash request把请求和写数据分开。比如在以下例子中有用：

- 当数据不会立刻被target写，延迟stash避免cache polluting。
- 当数据已经在系统中，而且数据已经被预取到cache。
- 当写数据时，并没有明确知道哪个target会使用。

这种情况，Requester可以使用StashOnce请求发给Home或peer node获取一个cache line。

Requester / Home / Stash target需要分别处理Stash hint。

Requester Node responsibilities:

- 发送StashOnceUnique or StashOnceShared to Home
- 当stash到peer cache时，提供Stash target
- 当data allocated到next level cache时，不提供stash target

Home Node responsibilities:

- 允许RetryAck
- StashOnceUnique时，发送SnpStashUnique给target RN-F
- StashOnceShared时，发送SnpStashShared给target RN-F
- 允许不发送Snoop request
- 必须发送Comp response，即使放弃了Stash request
- 在建立了处理流程，保证同地址请求的顺序，才能发Comp
- 当StashOnce request没有指定Stash target时，从memory获取数据到shared system cache
- 允许接收到StashOnce request后，在发送SnpStash或接收Snoop response之前，发送Comp
- 如果request hit the cache line at Home, 发送Com_[X]，而不是Comp_I
- 当Home miss或Home没有loopup时，发送Comp_I
- 允许使用DMT
- 允许使用分离的Non-data and Data-only response

Stash target responsibilities:

- snoop不能改变cache line的状态
- snoop认为是一个hint，来获取cache line的一个copy
- 以下情况不能request Data Pull:
  - DoNotDataPull bit is set
  - Snoop has an address hazard with an outstanding request
  - response的发送早于local cache lookup
  - The snoop is SnpStashShared and the cache has a copy of the cache line.
- 以下情况requesting Data Pull：
  - Stash target保证Read data接收不能导致死锁
  - 对于SnpOnceShared, Home把DataPull request当作ReadNotSharedDirty
  - 对于SnpOnceUnique，Home把DataPull request当作ReadUnique
  - Stash target必须使用Home的Read transaction的TxnID来填充DBID。
- 当snoop是SnpStashUnique且有一个shared copy时，DataPull request可发可不发。
- 允许但不要求Stash目标在发送Snoop响应之前等待，直到它完成local cache lookup
- Snoop response的cache state不要求精确，不精确时必须为SnpResp_I，其他状态必须精确。

> 对于StashOnceShared or StashOnceUnique transactions，需要避免预期要用的cache line的deallocation。
>
> StashOnceUnique会引发cache line的invalidation，注意transaction不要和Exclusive access sequence关联。

## 7.4 Stash target identifiers

### 7.4.1 Stash target specified

如果Stash request中的Stash target是明确的，Home就给对应的target发送snoop。specified target可以是一个RN或一个RN中的一个logical processor。

### 7.4.2 Stash target not specified

对于没有Stash target的WriteUniquePtlStash or WriteUniqueFullStash，Home Node的处理如下：

- 如果在某个RN中该cache line的状态为Unique，Home可以认为该RN就是Stash target.
- 如果cache line状态不是Unique，则Home必须发送SnpUnique，且不能给任何RN发送SnpUniqueStash
- 对于WriteUniquePtlStash, 如果cache line不在任何cache中，协议建议Home预取并allocate该cache line到system cache中。也允许，但不推荐，执行partial write到main memory中。
- 对于WriteUniqueFullStash,，如果cache line不在任何cache中，允许Home allocate该cache line到shared system cache中。

对于没有Stash target的StashOnceUnique or StashOnceShared request，Home Node的处理如下：

- 如果cache line不在任何一个peer cache中，协议建议将cache line allocate到shared system cache中
- 如果cache line中某一个peer cache中，则是IMPLEMENTATION DEFINED的，snoop发送用于传输一份该cache line的copy然后allocate到shared system cache中。对于StashOnceUnique,，也是IMPLEMENTATION DEFINED的，先allocate该cache line到shared system cache中，然后所有peer cache的copy invalidated。

## 7.5 Stash messages

Stash messages are classified as:
• Write requests:
— WriteUniqueFullStash.
— WriteUniquePtlStash.

• Dataless requests:
— StashOnceUnique.
— StashOnceShared.

• Snoop requests:
— SnpUniqueStash.
— SnpMakeInvalidStash.
— SnpStashUnique.
— SnpStashShared.

### 7.5.1 Supporting REQ packet fields

The fields defined in the REQ packet to support Stash requests are:
• StashNID, StashLPID.
• StashNIDValid, StashLPIDValid.

### 7.5.2 Supporting SNP packet fields

The fields defined in the SNP packet to support Stash requests are:
• StashLPID.
• StashLPIDValid.
• DoNotDataPull.

### 7.5.3 Supporting RSP packet field

The field defined in the RSP packet to support Stash requests is:
• DataPull.

### 7.5.4 Supporting DAT packet fields

The field defined in the DAT packet to support Stash requests is:
• DataPull.
