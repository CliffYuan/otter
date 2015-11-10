##### 核心概念
1. Arbitrate(仲裁)
    （1）仲裁分为3种：仲裁管理服务/仲裁事件服务/仲裁状态试图服务
        (a)ArbitrateManageService:仲裁器管理服务，提供给console进行干预仲裁器的行为：比如开始/停止channel同步
            1.NodeArbitrateEvent：机器Node节点管理
            2.PipelineArbitrateEvent：
            3.ChannelArbitrateEvent：
                (1)核心方法：通过管理后台操作
                    start():启动channel。
                    stop():停止channel。
            4.SystemArbitrateEvent:

        (b)ArbitrateEventService
            a.MainStemArbitrateEvent:主线仲裁,竞争&判断是否为活动节点
                (1)核心方法：都是SelectTask调用此方法
                    await(Long pipelineId):检查当前的Permit，阻塞等待其授权(解决Channel的pause状态处理)。
                                           尝试创建对应的mainStem节点(非持久化节点)。
                    release(Long pipelineId):释放mainStem的节点，重新选择工作节点。
            b.SelectArbitrateEvent
            c.ExtractArbitrateEvent
            d.TransformArbitrateEvent
            e.LoadArbitrateEvent
            f:TerminArbitrateEvent
            g:ToolArbitrateEvent
        (c)ArbitrateViewService

2. PermitMonitor(许可)


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

2.整个过程串联流程
(1).在SETL过程中的每个阶段的Task中，都会通过如下方式获取到要处理的processId
    ExtractStageListener extractStageListener = ArbitrateFactory.getInstance(pipelineId, ExtractStageListener.class);
    // 符合条件的processId
    Long processId = extractStageListener.waitForProcess();

(2).然后通过processId从zookeeper中获取到EtlEventData。
(3).然后通过etlEventData.getDesc()从pipe中获取到真实的关联数据，然后进行处理。
    List<PipeKey> keys = (List<PipeKey>) etlEventData.getDesc();
    DbBatch dbBatch = rowDataPipeDelegate.get(keys);
    ...
    该阶段任务处理
    ...
    // 传递给下一个流程
    List<PipeKey> nextKeys = rowDataPipeDelegate.put(dbBatch, etlEventData.getNextNid());
    etlEventData.setDesc(nextKeys);

(4).整体格式



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



##### SETL中两个很重要的ID
(1).batchId 批处理Id，对应一批处理的数据,  // 即canal生成的messageId,/otter/canal/destinations/{0}/{1}/mark add xnd
    在从canal获取数据的时候生成：
    CanalServerWithEmbedded{
        ...
        Long batchId = canalInstance.getMetaManager().addBatch(clientIdentity, events.getPositionRange());
        ...
    }
(2).processId 同步进程id,用于控制SETL过程的流转。
    在Select阶段生成:
    SelectTask{
        ...
        final EtlEventData etlEventData = arbitrateEventService.selectEvent().await(pipelineId);//令牌生成
        ...
    }





