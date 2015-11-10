
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
channel在zookeeper中的数据

4对应channel
7,8对应pipeline

(1)/otter/channel/4/7
        [lock, process, termin, remedy, mainstem]表示为子节点
   /otter/channel/4/8
        [lock, process, termin, remedy, mainstem]表示为子节点


   [lock,
   process,  //当前在处理的processId处于那个状态
   termin,
   remedy,
   mainstem  //pipeline运行情况
   ]

(2)/otter/channel/4/7/mainstem：表示当前的pipeline的运行情况。
    节点内容：{"active":true,     --处于运行状态
             "nid":1,           --运行的node节点
             "pipelineId":7,    --pipeline id
             "status":"OVERTAKE"--OVERTAKE表示已追上，TAKEING表示追赶中   （主道控制信号的状态）
            }







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