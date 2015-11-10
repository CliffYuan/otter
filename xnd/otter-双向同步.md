##### 双向同步过程

目前双向同步（单一回环同步算法）有个主站和从站的概念。主的数据一定会同步一次到从。

1.例子：库174（主站）和175，
    情况一：从174写入数据
        1)写入数据到174库。
        2)otter解析174库数据(这时未带同步标注)+同步标注Mark写入到175库

            1)Select阶段：
            [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@94] - ---Otter select解析数据,是否主站单项回环:true,[同步回环数据],PipelineParameter[pipelineId=<null>,parallelism=5,lbAlgorithm=Stick,home=true,selectorMode=Canal,destinationName=M-A-174-DEBUG,mainstemClientId=7,mainstemBatchsize=6000,extractPoolSize=10,loadPoolSize=15,fileLoadPoolSize=15,dumpEvent=true,dumpSelector=true,dumpSelectorDetail=true,pipeChooseType=AUTOMATIC,useBatch=true,skipSelectException=false,skipLoadException=false,arbitrateMode=AUTOMATIC,batchTimeout=-1,fileDetect=false,skipFreedom=false,useLocalFileMutliThread=false,useFileEncrypt=false,useExternalIp=false,useTableTransform=true,enableCompatibleMissColumn=true,skipNoRow=false,channelInfo=<null>,dryRun=false,ddlSync=false,skipDdlException=false,enableRemedy=true,remedyAlgorithm=LOOPBACK,remedyDelayThresoldForMedia=60,syncMode=ROW,syncConsistency=BASE,systemSchema=retl,systemMarkTable=retl_mark,systemMarkTableColumn=channel_id,systemMarkTableInfo=channel_info,systemBufferTable=retl_buffer,systemDualTable=xdual,retriever=ARIA2C],
            [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@144] - @@@@-select-不是回环数据，添加到待处理列表，tableName:T_SMS_USER
            [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@232] - @@@@-select-解析数据后EventDatas:1

            2)Load阶段：
            [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@579] - @@@@-load--阶段---执行批量JdbcTemplate操作开始
            [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [erceptor.operation.CanalMysqlInterceptor@172] - ########回环标记#######--添加mark标记,begin,sql:INSERT INTO retl.retl_mark (id, channel_id) VALUES (?, ?) ON DUPLICATE KEY UPDATE channel_id = VALUES(channel_id),值：2,4
            [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@600] - @@@@-load--阶段---执行JdbcTemplate sql 入库:insert into meizu_sms.T_SMS_USER(`FUSER_NAME` , `FPASSWORD` , `FID`) values (? , ? , ?) on duplicate key update `FUSER_NAME`=values(`FUSER_NAME`) , `FPASSWORD`=values(`FPASSWORD`) , `FID`=values(`FID`)
            [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [erceptor.operation.CanalMysqlInterceptor@172] - ########回环标记#######--添加mark标记,end,sql:UPDATE retl.retl_mark SET channel_id = 0 WHERE id = ? and channel_id = ?,值：2,4
            [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@609] - @@@@-load--阶段---执行批量JdbcTemplate操作结束

        3)otter解析175库数据(这时带有同步标注)，因为不是主站，所以丢弃数据。

            1)Select阶段：
            [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@94] - ---Otter select解析数据,是否主站单项回环:false,[丢弃回环数据],PipelineParameter[pipelineId=<null>,parallelism=5,lbAlgorithm=Stick,home=false,selectorMode=Canal,destinationName=M-A-175-DEBUG,mainstemClientId=8,mainstemBatchsize=6000,extractPoolSize=10,loadPoolSize=15,fileLoadPoolSize=15,dumpEvent=true,dumpSelector=true,dumpSelectorDetail=true,pipeChooseType=AUTOMATIC,useBatch=true,skipSelectException=false,skipLoadException=false,arbitrateMode=AUTOMATIC,batchTimeout=-1,fileDetect=false,skipFreedom=false,useLocalFileMutliThread=false,useFileEncrypt=false,useExternalIp=false,useTableTransform=true,enableCompatibleMissColumn=true,skipNoRow=false,channelInfo=<null>,dryRun=false,ddlSync=false,skipDdlException=false,enableRemedy=true,remedyAlgorithm=LOOPBACK,remedyDelayThresoldForMedia=60,syncMode=ROW,syncConsistency=BASE,systemSchema=retl,systemMarkTable=retl_mark,systemMarkTableColumn=channel_id,systemMarkTableInfo=channel_info,systemBufferTable=retl_buffer,systemDualTable=xdual,retriever=ARIA2C],
            [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@113] - @@@@-select-mark tableName:retl_mark，loopback:2
            [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@151] - @@@@-select-是回环标记表数据，丢弃，tableName:retl_mark
            [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@148] - @@@@-select-是回环数据,但不是主站单项回环，丢弃，tableName:T_SMS_USER,isLoopback:true,主站单项回环：false，needLoopback：true
            [communication-async-3                   ] [protocol.dubbo.LazyConnectExchangeClient@42] -  [DUBBO] Lazy connect to dubbo://127.0.0.1:1099/endpoint?acceptEvent.timeout=50000&client=netty&codec=dubbo&connections=30&heartbeat=60000&iothreads=4&lazy=true&send.reconnect=true&serialization=java&threads=50, dubbo version: 2.5.3, current host: 172.16.136.180
            [communication-async-2                   ] [.dubbo.remoting.transport.AbstractClient@42] -  [DUBBO] Successed connect to server /172.16.136.180:1099 from NettyClient 172.16.136.180 using dubbo version 2.5.3, channel is NettyChannel [channel=[id: 0x065a214b, /172.16.136.180:35464 => /172.16.136.180:1099]], dubbo version: 2.5.3, current host: 172.16.136.180
            [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@113] - @@@@-select-mark tableName:retl_mark，loopback:2
            [communication-async-2                   ] [.dubbo.remoting.transport.AbstractClient@42] -  [DUBBO] Start NettyClient /172.16.136.180 connect to the server /172.16.136.180:1099, dubbo version: 2.5.3, current host: 172.16.136.180
            [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@151] - @@@@-select-是回环标记表数据，丢弃，tableName:retl_mark
            [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@232] - @@@@-select-解析数据后EventDatas:0

    情况二：从175写入数据
        1)写入数据到175库。
        2)otter解析175库数据(这时未带同步标注)+同步标注Mark写入到174库
            2015-09-30 15:47:59.811 INFO  [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@94] - ---Otter select解析数据,是否主站单项回环:false,[丢弃回环数据],PipelineParameter[pipelineId=<null>,parallelism=5,lbAlgorithm=Stick,home=false,selectorMode=Canal,destinationName=M-A-175-DEBUG,mainstemClientId=8,mainstemBatchsize=6000,extractPoolSize=10,loadPoolSize=15,fileLoadPoolSize=15,dumpEvent=true,dumpSelector=true,dumpSelectorDetail=true,pipeChooseType=AUTOMATIC,useBatch=true,skipSelectException=false,skipLoadException=false,arbitrateMode=AUTOMATIC,batchTimeout=-1,fileDetect=false,skipFreedom=false,useLocalFileMutliThread=false,useFileEncrypt=false,useExternalIp=false,useTableTransform=true,enableCompatibleMissColumn=true,skipNoRow=false,channelInfo=<null>,dryRun=false,ddlSync=false,skipDdlException=false,enableRemedy=true,remedyAlgorithm=LOOPBACK,remedyDelayThresoldForMedia=60,syncMode=ROW,syncConsistency=BASE,systemSchema=retl,systemMarkTable=retl_mark,systemMarkTableColumn=channel_id,systemMarkTableInfo=channel_info,systemBufferTable=retl_buffer,systemDualTable=xdual,retriever=ARIA2C],
            2015-09-30 15:47:59.812 INFO  [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@144] - @@@@-select-不是回环数据，添加到待处理列表，tableName:T_SMS_USER
            2015-09-30 15:47:59.836 INFO  [pipelineId = 8,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@232] - @@@@-select-解析数据后EventDatas:1

            2015-09-30 15:47:59.868 INFO  [pipelineId = 8 , pipelineName = 175-174 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@579] - @@@@-load--阶段---执行批量JdbcTemplate操作开始
            2015-09-30 15:47:59.931 INFO  [pipelineId = 8 , pipelineName = 175-174 , DbLoadAction] [erceptor.operation.CanalMysqlInterceptor@172] - ########回环标记#######--添加mark标记,begin,sql:INSERT INTO retl.retl_mark (id, channel_id) VALUES (?, ?) ON DUPLICATE KEY UPDATE channel_id = VALUES(channel_id),值：3,4
            2015-09-30 15:47:59.934 INFO  [pipelineId = 8 , pipelineName = 175-174 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@600] - @@@@-load--阶段---执行JdbcTemplate sql 入库:insert into meizu_sms.T_SMS_USER(`FUSER_NAME` , `FPASSWORD` , `FID`) values (? , ? , ?) on duplicate key update `FUSER_NAME`=values(`FUSER_NAME`) , `FPASSWORD`=values(`FPASSWORD`) , `FID`=values(`FID`)
            2015-09-30 15:47:59.934 INFO  [pipelineId = 8 , pipelineName = 175-174 , DbLoadAction] [erceptor.operation.CanalMysqlInterceptor@172] - ########回环标记#######--添加mark标记,end,sql:UPDATE retl.retl_mark SET channel_id = 0 WHERE id = ? and channel_id = ?,值：3,4
            2015-09-30 15:47:59.949 INFO  [pipelineId = 8 , pipelineName = 175-174 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@609] - @@@@-load--阶段---执行批量JdbcTemplate操作结束


        3)otter解析174库数据(这时带有同步标注)，因为是主站，所以添加同步标注Mark写入到175库。
            2015-09-30 15:47:59.959 INFO  [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@94] - ---Otter select解析数据,是否主站单项回环:true,[同步回环数据],PipelineParameter[pipelineId=<null>,parallelism=5,lbAlgorithm=Stick,home=true,selectorMode=Canal,destinationName=M-A-174-DEBUG,mainstemClientId=7,mainstemBatchsize=6000,extractPoolSize=10,loadPoolSize=15,fileLoadPoolSize=15,dumpEvent=true,dumpSelector=true,dumpSelectorDetail=true,pipeChooseType=AUTOMATIC,useBatch=true,skipSelectException=false,skipLoadException=false,arbitrateMode=AUTOMATIC,batchTimeout=-1,fileDetect=false,skipFreedom=false,useLocalFileMutliThread=false,useFileEncrypt=false,useExternalIp=false,useTableTransform=true,enableCompatibleMissColumn=true,skipNoRow=false,channelInfo=<null>,dryRun=false,ddlSync=false,skipDdlException=false,enableRemedy=true,remedyAlgorithm=LOOPBACK,remedyDelayThresoldForMedia=60,syncMode=ROW,syncConsistency=BASE,systemSchema=retl,systemMarkTable=retl_mark,systemMarkTableColumn=channel_id,systemMarkTableInfo=channel_info,systemBufferTable=retl_buffer,systemDualTable=xdual,retriever=ARIA2C],
            2015-09-30 15:47:59.968 INFO  [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@146] - @@@@-select-是回环数据，添加到待处理列表，tableName:T_SMS_USER
            2015-09-30 15:47:59.968 INFO  [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@113] - @@@@-select-mark tableName:retl_mark，loopback:2
            2015-09-30 15:47:59.968 INFO  [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@151] - @@@@-select-是回环标记表数据，丢弃，tableName:retl_mark
            2015-09-30 15:47:59.968 INFO  [pipelineId = 7,taskName = ProcessSelect ] [r.node.etl.select.selector.MessageParser@232] - @@@@-select-解析数据后EventDatas:1


            2015-09-30 15:47:59.969 INFO  [pipelineId = 7,taskName = ProcessTermin ] [alibaba.otter.node.etl.select.SelectTask@388] - start process termin SETL处理完成: BatchTermin [batchId=5, needWait=true, processId=3]
            2015-09-30 15:47:59.973 INFO  [Otter-Seda-Executor-19                  ] [ter.node.etl.load.loader.db.DbLoadAction@125] - @@@@@--load-阶段开始,EventData Size:1,interceptor:com.alibaba.otter.node.etl.load.loader.interceptor.ChainLoadInterceptor@1aa12810
            2015-09-30 15:47:59.974 INFO  [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@579] - @@@@-load--阶段---执行批量JdbcTemplate操作开始
            2015-09-30 15:47:59.981 INFO  [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [erceptor.operation.CanalMysqlInterceptor@172] - ########回环标记#######--添加mark标记,begin,sql:INSERT INTO retl.retl_mark (id, channel_id) VALUES (?, ?) ON DUPLICATE KEY UPDATE channel_id = VALUES(channel_id),值：4,4
            2015-09-30 15:47:59.983 INFO  [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@600] - @@@@-load--阶段---执行JdbcTemplate sql 入库:insert into meizu_sms.T_SMS_USER(`FUSER_NAME` , `FPASSWORD` , `FID`) values (? , ? , ?) on duplicate key update `FUSER_NAME`=values(`FUSER_NAME`) , `FPASSWORD`=values(`FPASSWORD`) , `FID`=values(`FID`)
            2015-09-30 15:47:59.983 INFO  [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [erceptor.operation.CanalMysqlInterceptor@172] - ########回环标记#######--添加mark标记,end,sql:UPDATE retl.retl_mark SET channel_id = 0 WHERE id = ? and channel_id = ?,值：4,4
            2015-09-30 15:47:59.993 INFO  [pipelineId = 7 , pipelineName = 174-175 , DbLoadAction] [ter.node.etl.load.loader.db.DbLoadAction@609] - @@@@-load--阶段---执行批量JdbcTemplate操作结束
            2015-09-30 15:47:59.993


        4)otter解析175库数据(这时带有同步标注)，因为不是主站，所以丢弃数据。



##### 单一回环同步实现

使用在事务前后添加标记的方式。

1.什么情况下写标记？
    1)当channel下配置了两个pipeline，
    2)或者在pipeline配置中设置了channelInfo。


