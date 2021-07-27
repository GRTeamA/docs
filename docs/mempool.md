# Mempool
## 结构
```
commit a13ca8112c17b1e19a648f6ad4ef8a068aea5fb1 (HEAD -> main)
Author: Sam Blackshear <shb@fb.com>
Date:   Thu Jul 15 08:30:08 2021 -0700
```

```
      26 text files.
      26 unique files.
       0 files ignored.

github.com/AlDanial/cloc v 1.82  T=0.02 s (1169.5 files/s, 285938.5 lines/s)
-------------------------------------------------------------------------------
Language                     files          blank        comment           code
-------------------------------------------------------------------------------
Rust                            24            654            664           4925
TOML                             1              6              0             52
Markdown                         1             19              0             37
-------------------------------------------------------------------------------
SUM:                            26            679            664           5014
-------------------------------------------------------------------------------
```

```
mempool
├── Cargo.toml
├── README.md
└── src
    ├── core_mempool
    │   ├── index.rs
    │   ├── mempool.rs
    │   ├── mod.rs
    │   ├── transaction.rs
    │   ├── transaction_store.rs
    │   └── ttl_cache.rs
    ├── counters.rs
    ├── lib.rs
    ├── logging.rs
    ├── shared_mempool
    │   ├── coordinator.rs
    │   ├── mod.rs
    │   ├── network.rs
    │   ├── peer_manager.rs
    │   ├── runtime.rs
    │   ├── tasks.rs
    │   └── types.rs
    └── tests
        ├── common.rs
        ├── core_mempool_test.rs
        ├── fuzzing.rs
        ├── mocks.rs
        ├── mod.rs
        ├── multi_node_test.rs
        ├── node.rs
        └── shared_mempool_test.rs

4 directories, 26 files
```

## 特点
+ 创建独立的Tokio Runtime
+ 将启动三个不同的异步任务
    + coordinator：处理入站网络事件与出站交易广播，使用**BoundedExecutor**限制并发任务数
    + gc_coordinator：对所有到达SystemTTL的过期交易进行垃圾回收
    + snapshot_job：周期性地对core mempool内存在的交易记录快照，用于观察内部状态
+ core_mempool 内存中的核心数据结构
+ shared_mempool 运行时 

## core_mempool

### index.rs
+ PriorityIndex
+ TTLIndex
+ TimelineIndex
+ ParkingLotIndex

## shared_mempool

### coordinator.rs
处理以下网络事件：

+ client_events
+ consensus_requests
+ state_sync_requests
+ mempool_reconfig_events
+ scheduled_broadcasts
+ events
    + Event::NewPeer
    + Event::LostPeer
    + Event::Message

## 关于broadcast
Event::NewPeer生成，scheduled_broadcasts调度，peer_manager发送

## 关于gc
+ SystemTTL gc由gc_coordinator周期性地触发
+ expiration time gc在process_consensus_request和process_state_sync_request(commit_txns)时触发

## 疑问
+ coordinator中的异步处理逻辑是否合理？（串行处理事件）
+ SharedMempool中subscriber模式的用处？
