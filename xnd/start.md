
##### 运行
-DappName=otter-node
-Ddubbo.application.logger=slf4j
-Dlogback.configurationFile=/home/xiaoniudu/project_mx/otter/node/deployer/src/main/resources/logback.xml
-Dnid=1



##### 启动流程
1. OtterController.main();
        ->OtterController.start();
            initNid();
            (1)检测nid是否有效.
            NodeTaskService.addListener(this);
            (2)添加NodeTaskListener监听.
                notifyListener();
                (1)触发一次listener推送
                    OtterController.process(incNodeTask);



