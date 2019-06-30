最近写了一个获取光纤交换机端口连接wwn 号的python脚本，用来对比与哪些服务器的的HBA卡口相连。虽然没啥难度，但是还是要水一下。下面就简单说一说实现的步骤。

## 核心思路
本身光纤交换机支持ssh协议 访问，所以可以使用python 的 paramiko模块 ，ssh登录到交换机。确保python的pycrypto和paramiko模块都完成安装。最后发送对应的交换机指令获取返回结果，正则提取就完事了。
## paramiko 
paramiko是用python语言写的一个模块，遵循SSH2协议，支持以加密和认证的方式，进行远程服务器的连接。

## 查看端口信息命令

```
  switchshow
```
命令会返回光纤交换机所有的端口连接信息，如果需要获取其他的内容，可以通过help  查看命令提示
## 脚本获取解析后的结果展示

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190625200215830.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xzd19kYWJhb2ppYW4=,size_16,color_FFFFFF,t_70)

## 参考链接
- [获取交换机端口信息的代码地址](https://github.com/Sherlock-L/automation-py/blob/master/brocade_fs_query.py)
- [paramiko 的安装步骤](https://www.cnblogs.com/qianyuliang/p/6433250.html)   （此文介绍的paramiko的版本不是最新的，可以到此下载=》[github](https://github.com/paramiko/paramiko)，自行安装）
- [pycrypto 资源下载地址参考](https://www.jb51.net/article/118044.htm)
- [windows下python SSH的使用——paramiko模块](https://blog.csdn.net/wangyuling1234567890/article/details/21656471)