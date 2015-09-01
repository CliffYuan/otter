##### SEDA模型
1. 包括3个步骤:
    (1)await
    (2)notify
    (3)single
2. 以Select为例子说明
    //###otter调度模型步骤1 await
    final EtlEventData etlEventData = arbitrateEventService.selectEvent().await(pipelineId);
    //###otter调度模型步骤2 notify 提交任务到thread pools
    executorService.execute(extractFuture);
    //###otter调度模型步骤3 single
    arbitrateEventService.selectEvent().single(etlEventData);

##### setl流程

0. setl过程与pipeline挂钩,必须是某个固定的pipeline

1. Select
    (1)核心线程 SelectTask
    (2)node/etl下面
    (3)流程:


##### 数据传输
有了一层SEDA调度模型的抽象，S/E/T/L模块之间互不感知，那几个模块之间的数据传递，需要有一个机制来处理，这里抽象了一个pipe(管道)的概念.
1. 原理：stage | pipe | stage
2. 基于pipe实现：
    in memory (两个stage经过仲裁器调度算法选择为同一个node时，直接使用内存传输)
    rpc call (<1MB)
    file(gzip) + http多线程下载
3. 在pipe中，通过对数据进行TTL控制，解决TCP协议中的丢包问题控制.

##### stage
每个stage都会有个block queue，接收上一个stage的single信号通知，
当前stage会阻塞在该block queue上，直到有信号通知.
共3种类型stage,如下:
1. MemoryStageController
2. RpcStageController
3. StageMonitor(基于zookeeper)
    (1)SelectStageListener
    (2)ExtractStageListener
    (3)TransformStageListener
    (4)LoadStageListener

##### pipe


