# cluster-test
## tx_emitter
### 关键参数

+ workers_per_ac：每个节点对应的worker数量
+ accounts_per_client：每个worker将发出的交易数

### 测试逻辑

令节点数量为**n**

参与测试账户数=**n** \* **workers_per_ac** \* **accounts_per_client**

测试程序对每个节点分配**workers_per_ac**个异步任务，每个异步任务在每个周期中将构造**accounts_per_client**个交易请求，并向该节点的JSON-RPC端口提交交易请求，该过程不断重复直至测试程序结束。

测试程序在产生**n** \* **workers_per_ac**个异步任务后，由**Tokio runtime**负责调度执行（通常为线程池）。