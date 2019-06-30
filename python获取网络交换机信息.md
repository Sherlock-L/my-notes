@[TOC](目录)

# 前言
**要想真正的弄清楚，就得自己登录到交换机查看，分析文本，总结多个例子。这是必经之路。**
文中涉及到的交换机，包括思科（Cisco）、博科（Brocade）、华为（Huawei）、华三（H3C）四个品牌。获取的信息包括：主机、内存、flash、固件版本（os_version、设备序列号）、物理端口、逻辑端口（聚合口）、vlan、以及互连端mac和ip信息。
如果你对交换机不是很熟悉，我建议你首先得先去了解几个关键的知识：

 - [VLAN](https://blog.csdn.net/cwm_meng_home/article/details/49762807)
 - [管理端口](https://zhidao.baidu.com/question/557987470619164652.html?qbl=relate_question_0&word=%BD%BB%BB%BB%BB%FA%20%B9%DC%C0%ED%BF%DA)
 - [逻辑端口](https://jingyan.baidu.com/article/c14654138b129e0bfcfc4c01.html)
 - 物理端口[命名规则](https://www.cnblogs.com/sddai/p/8947310.html)、
 - [速率](https://zhidao.baidu.com/question/298753446.html)
 - 连接方式：[trunk、access、hybrid](https://zhidao.baidu.com/question/271064330.html?qbl=relate_question_1&word=%BD%BB%BB%BB%BB%FA%20%B9%DC%C0%ED%BF%DA) 
 - 
# github地址（敲黑板）
思科、华为、H3C脚本程序设计为常规思路，由于项目需要一次性获取多项数据，所以导致执行多条命令花费的时间很长，可能获取失败。执行消耗快的3、4分钟，慢的有可能10分钟。
**重点：** 博科交换机获取脚本是我经过不断的尝试，用了新的设计思路，其他老的脚本来不及优化了。执行时间从减少到1分钟之内。当然，如果你的网络不好那就另当别论。
[***git地址***](https://github.com/Sherlock-L/switch-info-script)

# Python依赖的工具库

- 正则匹配函数库，用于提取交换机返回的关键信息
- telnet模块函数库，用于与交换机进行通信
- 睡眠，用于等待交换机的响应
```
import re  
import telnetlib
import time
```


# 核心思路（必看）

**中心思想**：减少睡眠等待的时间，若要最短响应时间需要多线程
|常规思路（执行时间长）| 优化思路（执行时间短） |备注
|:--|:--|:--|
| 1. 登录交换机，建立一个连接 |  1. 登录交换机，建立一个连接| |
| 2.一次发送一条”下一页”指令=》睡眠=》读取；<br>**结果：执行N条需要睡眠N次** |2. 一次发送N条“下一页指令”（N可以自定义）<br> =》睡眠（我一般设置1.5秒）<br>=》读取 <br>**结果：执行N条需要睡眠1次** | 查询内容过多，交换机一般只输出一部分内容，然后尾部为“----more-------”字符串，这时候需要不断发送敲空格（下一页）指令，获取更多，直到内容获取完毕。<br>**另外**，如果需要获取所有端口的信息，尽量不要单个循环获取，如show interfaces GigabatEthernet1/0/1，show interfaces GigabatEthernet1/0/2.... 。这样就需要多条命令执行;直接 show  interfaces，一次读取所有的端口,然后在代码中自己分隔文本去提取。
| 4 等待交换机返回结果（睡眠等方法）|4 等待交换机返回结果（睡眠等方法）  | |
| 5. 正则提取结果| 5.正则提取结果 | 优化思路的做法因为一次获取了所有文本数据，所以需要根据关键字切割文本，然后遍历提取
| 6. 重复2、3、4操作 |6.重复2、3、4操作  | |
|7. 存入变量或者数据库存入变量或者数据库 | 7. 存入变量或者数据库存入变量或者数据库 | 

# 兼容性 （必看）

 **同一个品牌，不同型号两个地方不同，需要代码做一些兼容**
 - 睡眠时间：考虑到保证所有交换机都能获取成功，睡眠时间不宜太短，也不宜太长
 - 同一内容，展示文本格式不同，同时命令可能有一丝丝的差别，可能就多个单词
 - 同一命令，部分型号include 后面有些需要 双引号包含关键字  （目前只发现**H3C**有这种情况）。<br>
 如 display  interfaces | include "GigabatEthernet" 或者 display  interfaces | include GigabatEthernet （不带双引号）
 - 博科交换机 回车命令 是 \r\n，而其他是\n
 


# 登录，建立一个连接
## 命令
 

 1. telnet + ip
 2. 输入账号
 3. 输入一级密码
 4. enable
 5. 输入二级密码（思科，博科交换机有，华为，h3c没有）

 telnet  + 交换机ip
 - 
## 关键代码

```
#以思科为例
  self.tnIp = "192.168.1.1" #你的ip
  self.username = "admin"
  self.password1 = "pwd1" #你的ip
  self.password2 = "pwd2" #你的ip
  
  tnconn = telnetlib.Telnet()
        try:
        	//telnet  ip
            tnconn.open(self.tnIp)
        except:
            print "Cannot open host"
            return
        #读取交换机返回的文本提示，读到‘username：’， 说明需要输入账户名，不同品牌设备不一样的提示
        tnconn.read_until('Username:') #表示程序一直等待，直到 收到交换机返回'Username:'字符串提示。
        tnconn.write(self.username + '\n') #模拟输入账户名命令。并加一个回车按钮。表示输入完毕，模拟人工敲键盘
       
        #读取交换机返回的文本提示，直到读到Password，说明需要输入密码
        tnconn.read_until('Password:')
        tnconn.write(self.password1 + '\n')
       #程序睡眠一秒，视情况而调整，考虑到所有交换机都能成功，所以需要照顾反应慢的机器。
        time.sleep(1) 
        tnconn.write('enable\n')

        tnconn.read_until('Password:')
        tnconn.write(self.password2 + '\n')

        time.sleep(1)

        self.__tnConn = tnconn #返回建立成功的连接，

        return tnconn
```

## 注意事项
- 不同交换机登录的时候，输入命令响应时间有快有慢，有可能read_until()方法会超时，所以可以适当在登录的步骤加上程序睡眠逻辑，给交换机一定的处理命令返回的等待时间。避免死在开头。
# 获取主机名等基础信息
## 命令

| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
show running-config  \| include hostname | show running-config  \| include hostname |display current-configuration \| include sysname |display current-configuration | include sysname
## 注意事项
考虑不同H3C设备的命令 include 后面的字符串是否需要加上双引号。具体可以参考H3C获取脚本
# 获取内存信息

## 命令
| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
| show memory summary |show version  | display memory-usage|dis version

## 注意事项
- 单位，不同类型有些是kb  有些是 b等。同时数字 可能有逗号 如 9,991KB。
- 如果是多台交换机使用堆叠的连接方式，共用一个管理ip。那么会按照slot（插槽顺序）分块显示，比如有四台，那么会有四份数据。可以通过设备sn（序列号）区分哪个插槽属于哪一台交换机。一般它们的设备型号、内存配置是一模一样的。所以输入命令，会显示一样的总内存，只不过使用率会有不同。

# 获取flash信息
## 命令
| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
|  dir| dir |dir |dir
## 注意事项
- **单位**:有些品牌类型有些是kb  有些是 b等。同时数字 可能有逗号 如 9,991KB。
- **出现多个重复数据**：如果是多台交换机堆叠在一起，共用一个管理ip。比如有四台，那么会有四份数据，具体依据返回文本进行分析。可以通过设备sn（序列号）区分哪个插槽属于哪一台交换机。一般它们的设备型号、内存配置是一模一样的。所以输入命令，会显示一样的总内存，只不过使用率会有不同。
# 获取物理端口信息
## 命令
| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
| show interfaces<br> 或者 show interfaces \| include  xxxx | show interfaces<br> 或者 show interfaces \| include  xxxx |display current-configuration <br> 或者 display current-configuration \| include  xxxx  |display current-configuration <br> 或者 display current-configuration \| include  xxxx
## 注意事项
- **当前的速率** ：如果端口关掉可能不一定有值，
- **最大速率**: 最大值可以根据端口名判断。如GigabatEthernet代表千兆口ten-GigabatEthernet，代表万兆口。fastEthernet代表百兆口。
- **duplex**：不一定能读到，需要另一个指令，具体看获取端口信息的代码。
- **交换机端口命名**：如 GigabatEthernet1/0/1 解释： https://zhidao.baidu.com/question/539909844.html
- **连接方式**：trunk、还是access的名号不一定能读到，需要另一个指令，具体看获取端口信息的代码
- **光口电口**：文本内容可能显示双绞线，或者光纤等单词，总之就是不同英文展示，甚至是缩写，我整理了一波：
- **大小写兼容**：如up或者UP。
```
//PHP代码
 //电口
            if (stripos($value['port_type'], 'twisted') !== false
                ||$value['port_type'] == 'twistedpair'
                || $value['port_type'] == 'TX'
                || $value['port_type'] == 'COMMON COPPER'
                || $value['port_type'] == 'CX'
                || $value['port_type'] == 'T') {
                $value['port_type'] = AtDevNetPort::PORT_TYPE_T;
                $allContent['ihmDevNet']['port_num']++;

            }
            //光口
             else if (stripos($value['port_type'], 'fiber') !== false
                || $value['port_type'] == 'opticalfiber'
                || $value['port_type'] == 'FX'
                || $value['port_type'] == 'COMMON FIBER'
                || $value['port_type'] == 'SX'
                || $value['port_type'] == 'LX') {
```

# 获取逻辑端口信息
## 命令
**这一栏目内容就具体看代码了**
| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
|show interfaces \| include Port-channel|show trunk|display interfaces  Eth-Trunk \| include Eth-Trunk|display interface Bridge-Aggregation \| include Bridge-Aggregation
## 注意事项
- 大小写兼容，如up或者UP
- Brocade 在比较老的型号中，没有逻辑口的说法，只是在查看物理端口的时候，能看到此物理口和哪几个口是一个组的，可以把这种组当成是逻辑口，姑且这么认为。
- 获取逻辑口包含哪些物理口，可能需要额外指令，具体看获取逻辑口信息的代码。
- 逻辑口命名：可以通过指令发现，逻辑口的叫法命名不同品牌是不一样的。具体看代码或者百度。
# 获取vlan信息
## 命令
| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
|show vlan |show vlan|display vlan | include VLAN|display vlan -all
## 注意事项
- **关键还是要理解端口和vlan的关系**
- **默认vlan**：名称可能就不是vlan+id，可能交换机直接叫做default，一般代表vlan 1（有些叫vlan0001）。
- **部分端口被省略思**：如思科类型交换机，如在vlan侧查看已经被配置了哪些口的时候，如果物理口1,2,3属于逻辑口1，name可能物理口1,2,3就不会出现在某个vlan中。只会出现逻辑口1的名称；其他品牌可能都会出现，既出现逻辑口1，也出现物理口1,2,3，在同一个vlan中。
- **物理端口名为缩写**：需要自己重新转化名称，如可能会有这样的文本：
 ***
<br>vlan   2:<br>Gi0/5,Fa0/12,XGE2<br>vlan   25:<br>Gi0/1,Fa0/1,XGE1（这个是万兆口的缩写）.脚本中获取vlan的方法中有提到。
***

# 互连端mac和ip信息

## 命令
| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
| show cdp neighbors detail <br>https://jingyan.baidu.com/article/37bce2be4bcc671002f3a226.html|show lldp  | display lldp nei|dis lldp neighbor-information<br>（另一个型号：dis lldp neighbor-information verbose）https://jingyan.baidu.com/article/86fae3461bb6cc3c49121ad1.html
## 注意事项
-  可以依赖  [lldp 协议](https://baike.baidu.com/item/LLDP/6849522?fr=aladdin)，可以知道端口连的对端设备信息
- 思科有自己独立的协议，可以知道端口连的对端设备信息。但是对端应该也是思科设备才行
- 要获取连接端的设备信息，首先端口要开启，同时协议也要开启。否则是无法知道的

# 设备序列号
## 命令
| 思科 |  博科（Brocade）|华为|H3C
|:--|:--|:--|:--|
|show version|可能在version里，找你的网络工程师|可能在version里，找你的网络工程师|display device manuinfo

## 注意事项
- 如果是堆叠（多台交换机组合到一起），可能读到多个序列号。但是可以根据插槽区分。




