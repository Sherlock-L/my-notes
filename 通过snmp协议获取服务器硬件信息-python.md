**前言**
最近有个需求，就是在新机器上架的时候，获取硬件信息，用来核对新采购的服务器是否和购买的配置一致。其中包含cpu、内存，硬盘，Raid卡、网卡等硬件信息。由于新机器可能没有装操作系统，考虑到公司使用的服务器主要为DELL品牌的服务器，集成了戴尔远程控制卡-iDRAC，一个远程控制卡。然后iDRAC开启snmp 协议，供远程机器访问其硬件信息。

**[代码地址](https://github.com/Sherlock-L/automation-py/blob/master/dell_host_model.py)**

**iDRAC**
iDRAC又称为Integrated Dell Remote Access Controller，也就是集成戴尔远程控制卡，这是戴尔服务器的独有功能，iDRAC卡相当于是附加在服务器上的一计算机，可以实现一对一的服务器远程管理与监控，通过与服务器主板上的管理芯片BMC进行通信，监控与管理服务器的硬件状态信息。它拥有自己的系统和IP地址，与服务器上的OS无关。是管理员进行远程访问和管理的利器，戴尔服务器集成了iDRAC控制卡，我们就可以扔掉价格昂贵的KVM设备了。在戴尔第12代服务器中，iDRAC的版本升级到了iDRAC 7。

**SNMP**
简单网络管理协议（SNMP，Simple Network Management Protocol），由一组网络管理的标准组成，包含一个应用层协议（application layer protocol）、数据库模型（database schema）和一组资源对象。该协议能够支持网络管理系统，用以监测连接到网络上的设备是否有任何引起管理上关注的情况。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190426211405841.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xzd19kYWJhb2ppYW4=,size_16,color_FFFFFF,t_70)
 **依赖工具**
 snmpwalk，snmpwalk是SNMP的一个工具，它使用SNMP的GETNEXT请求查询指定[OID](https://baike.baidu.com/item/OID/2723970?fr=aladdin)（SNMP协议中的对象标识）入口的所有OID树信息，并显示给用户。[安装链接](http://www.cnblogs.com/lsdb/p/8481954.html)
 例如，获取硬件设备序列号，在linux环境下执行以下指令，即可在控制台看到对应信息


**snmpwalk -v 2c -c public   192.168.12.123（IP地址） 1.3.6.1.4.1.674.10892.2.1.1.11.0** 

 

**python 依赖模块 **
我的是python2.7版本用的commands 模块，3.X可以用 subprocess模块。[代码地址](https://github.com/Sherlock-L/automation-py/blob/master/dell_host_model.py)
代码片段如下：

```

import commands 

def execComand(self,command, bJson=True):
		#执行命令，获取返回结果
        data = commands.getoutput(command)
        if bJson == False:
            return data

        data = data.replace('"', '')
        arr = data.split("\n")
        return json.dumps(arr)
#获取cpu颗数和型号
 def getCpu(self):
        modelCommand = 'snmpwalk  -c public -v 2c %s 1.3.6.1.4.1.674.10892.5.4.1100.30.1.23.1 |cut -d : -f4 |cut -b 3-' % ( self.__ip)
        ret = self.execComand(modelCommand, False)
        lines = ret.split("\n")
        if not lines:
            return
        for line in lines:
            self.__cpu['num'] += 1
            self.__cpu['model'].append(line.replace("\"", ''))
```
python 执行结果返回效果

```
{
    "raid":{
        "model":"PERC H730P Mini (Embedded)"
    },
    "power":{

    },
    "networkCard":{
        "model":[
            "Broadcom Gigabit Ethernet BCM5720 ",
            "Broadcom Gigabit Ethernet BCM5720 ",
            "Broadcom Gigabit Ethernet BCM5720 ",
            "Broadcom Gigabit Ethernet BCM5720 "
        ],
        "num":4
    },
    "brand":"Dell",
    "hba":{

    },
    "sn":"CF3JD92",
    "memory":{
        "num":24,
        "MemoryStickItems":Array[24]
    },
    "model":"PowerEdge R730",
    "disk":{
        "diskList":[
            "285568",
            "285568"
        ],
        "num":2
    },
    "cpu":{
        "model":[
            "Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz",
            "Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz"
        ],
        "num":2
    },
    "remoteManageCard":{
        "name":"iDRAC8"
    }
}
```

**注意的地方**
不同型号，有些配置在OID树信息可能为空。SNMP  OID树列表：
[参考链接1](http://www.ttlsa.com/monitor/snmp-oid/)
[参考链接2](https://blog.csdn.net/qq_29663071/article/details/50771575)


