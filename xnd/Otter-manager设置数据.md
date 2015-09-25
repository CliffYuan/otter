##### channel设置的数据:

{"channelId":4,
"enableRemedy":true,             //是否开启数据一致性
"remedyAlgorithm":"LOOPBACK",    //单向回环补救
"remedyDelayThresoldForMedia":60,//一致性反查数据库延迟阀值
"syncConsistency":"BASE",        //BASE:基于当前日志变更   MEDIA:基于数据库反查
"syncMode":"FIELD"}              //FIELD:列记录模式 ROW:行记录模式


##### pipeline设置的数据：
mysql:
{
    "arbitrateMode": "AUTOMATIC",
    "batchTimeout": -1,
    "ddlSync": false,
    "destinationName": "M-A-174-DEBUG",
    "dryRun": false,
    "dumpEvent": false,
    "dumpSelector": true,
    "dumpSelectorDetail": false,
    "enableCompatibleMissColumn": true,
    "extractPoolSize": 10,
    "fileDetect": false,
    "fileLoadPoolSize": 15,
    "home": true,
    "lbAlgorithm": "Stick",
    "loadPoolSize": 15,
    "mainstemBatchsize": 6000,
    "parallelism": 5,
    "pipeChooseType": "AUTOMATIC",
    "selectorMode": "Canal",
    "skipDdlException": false,
    "skipFreedom": false,
    "skipLoadException": false,
    "skipNoRow": false,
    "skipSelectException": false,
    "useBatch": true,
    "useExternalIp": false,
    "useFileEncrypt": false,
    "useLocalFileMutliThread": false,
    "useTableTransform": true
}
otter应用中:
PipelineParameter[
    pipelineId=<null>,
    parallelism=5,
    lbAlgorithm=Stick,
    home=true,
    selectorMode=Canal,
    destinationName=M-A-174-DEBUG,
    mainstemClientId=7,
    mainstemBatchsize=6000,
    extractPoolSize=10,
    loadPoolSize=15,
    fileLoadPoolSize=15,
    dumpEvent=false,
    dumpSelector=true,
    dumpSelectorDetail=false,
    pipeChooseType=AUTOMATIC,
    useBatch=true,
    skipSelectException=false,
    skipLoadException=false,
    arbitrateMode=AUTOMATIC,
    batchTimeout=-1,
    fileDetect=false,
    skipFreedom=false,
    useLocalFileMutliThread=false,
    useFileEncrypt=false,
    useExternalIp=false,
    useTableTransform=true,
    enableCompatibleMissColumn=true,
    skipNoRow=false,
    channelInfo=<null>,
    dryRun=false,
    ddlSync=false,
    skipDdlException=false,
    enableRemedy=true,
    remedyAlgorithm=LOOPBACK,
    remedyDelayThresoldForMedia=60,
    syncMode=ROW,
    syncConsistency=BASE,
    //单项回环数据
    systemSchema=retl,
    systemMarkTable=retl_mark,
    systemMarkTableColumn=channel_id,
    systemMarkTableInfo=channel_info,
    systemBufferTable=retl_buffer,
    systemDualTable=xdual,

    retriever=ARIA2C
]

start:channel_id=0,channel_info="";