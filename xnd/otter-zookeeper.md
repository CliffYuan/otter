
#### otter在zookeeper中的节点

##### 1)node节点是否启动
/otter/node/1 代表manger中id为1的节点处于运行状态
/otter/node/3
/otter/node/4

##### 2)canal对应消费记录
记录对应的channel下pipeline的canal消费到了哪里。

    /otter/canal/destinations/M-A-175-DEBUG/8/cursor
        节点内容：{
            "@type": "com.alibaba.otter.canal.protocol.position.LogPosition",
            "identity": {
                "slaveId": -1,
                "sourceAddress": {
                    "address": "172.16.82.175",
                    "port": 3306
                }
            },
            "postion": {
                "included": false,
                "journalName": "mysql-bin.000004",
                "position": 114281,
                "timestamp": 1442904553000
            }
        }
    /otter/canal/destinations/M-A-175-DEBUG/8/filter
    /otter/canal/destinations/M-A-174-DEBUG/7/cursor
        节点内容：{
            "@type": "com.alibaba.otter.canal.protocol.position.LogPosition",
            "identity": {
                "slaveId": -1,
                "sourceAddress": {
                    "address": "172.16.82.174",
                    "port": 3306
                }
            },
            "postion": {
                "included": false,
                "journalName": "mysql-bin.000004",
                "position": 50882,
                "timestamp": 1442394796000
            }
        }
    /otter/canal/destinations/M-A-174-DEBUG/7/filter



##### 3)channel,pipeline对应的处理过程
/otter/channel/4/7
[lock, process, termin, remedy, mainstem]
/otter/channel/4/7/mainstem
    节点内容：{"active":true,"nid":1,"pipelineId":7,"status":"OVERTAKE"}

/otter/channel/4/8
[lock, process, termin, remedy, mainstem]

4对应channel。
7,8对应pipeline



##### 4)在pipeline中选择node和canal
如:
    Pipeline序号：	8
    Pipeline名字：	175-174
    Select机器：	180-node-debug;                 ###选择node
    Load机器：	180-node-debug;                 ###选择node
    并行度：	5
    数据反查线程数：	10
    数据载入线程数：	15
    文件载入线程数：	15
    主站点：	false
    同步数据来源：	Canal
    Canal名字：	M-A-175-DEBUG                   ###选择canal
    主道消费批次大小：	6000
    获取批次数据超时时间：	-1
    描述信息：
    是否显示高级设置：