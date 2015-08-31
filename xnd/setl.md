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

