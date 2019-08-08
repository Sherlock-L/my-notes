七月份实在是太忙了，拖更了一把。于是乎无从下手，现在就来水一篇吧。
事情是这样的，本地启动多个不同laravel项目时，数据库连接混乱了。然后针对这种情况，网上也有不少解决方法，下面我就简单说说一种简单的解决办法。

## 原因
本地是windows系统，集成环境中php是线程安全的版本,laravel 项目启动时，加载.env文件的时候用了 getenv() 和 setenv() 。由于getenv()/setenv()不是的[线程安全的](https://www.cnblogs.com/ChandlerVer5/p/php_ts_nts.html)。所以导致apache启动多个项目时，会改变环境变量，出现错乱。linux没有此现象，因为使用的是单一进程，同时安装的默认也是使用非线程安全版本，可以使用 php -v 查看


## 网上的解决办法
1、更改PHP版本为非线程安全( nts )版本。（个人觉得麻烦得一腿）

2、在config/database.php中写死数据库连接配置，不用 env()。（不太灵活）

3、如果使用 apache，可以将工作模式设置为 [prefork](https://www.cnblogs.com/qiujun/p/6861773.html) 模式。


## 不用改任何配置的方法
**不用改配置的方法**:利用php 内置的web 服务，虽然消耗资源，但是本地开启也无妨，在对应的根目录 php -S  ip:port   如在laravel 的public 下 php -S 127.0.0.1:123456，这样每个项目都是独立的web进程，环境变量也就不会冲突了。

## 最后说两句

其实[php -S](https://www.php.net/manual/en/features.commandline.webserver.php) 真的挺好用的。在某些时候，临时拉入新项目，配虚拟域名什么的多麻烦，还得重启。现在，又水完一篇，给七月补上一份。