1. 事先准备：

xdebug dll文件 [https://xdebug.org/download](https://xdebug.org/download)<br />phpstorm jetbrain全家桶的php工具<br />debug helper Chrome浏览器插件 

2. 配置xdebug环境

这里从一开始我就踩了一个大坑，我用的7.4.3nts的php去配置（很多网上的教程都是这么写的），但其实，在这个[https://xdebug.org/wizard](https://xdebug.org/wizard)网站中使用phpinfo（命令行输入php -i）的信息去测试后发现，它并不能检测出7.4.3对应的xdebug版本，然后我们自己随便下载的其实都有问题，没办法完全适配，所以在加载模块的时候就会报错：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1641538035991-313ac994-2084-4883-830d-485646461997.png#clientId=u16773827-4496-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=304&id=u39ec3720&margin=%5Bobject%20Object%5D&name=image.png&originHeight=304&originWidth=603&originalType=binary&ratio=1&rotation=0&showTitle=false&size=7234&status=done&style=none&taskId=u6b3e7b8c-5cce-4091-86c2-a16233b1af5&title=&width=603)<br />然后也可以发现，确实xdebug模块没有被加载出来，无奈之下，我只好使用7.3.4版本，没想到就成功了：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1641538104761-efb23a1a-0025-4a87-9768-1b72be4527bd.png#clientId=u16773827-4496-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1238&id=uef42539b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1238&originWidth=1133&originalType=binary&ratio=1&rotation=0&showTitle=false&size=105222&status=done&style=none&taskId=ud58deab6-9369-41a1-abd3-067d2a9ff4b&title=&width=1133)<br />这里给出了应该下载的对应版本。在对应的php.ini的最后添加下面的字段：
```
[xdebug]
zend_extension="C:\phpstudy_pro\Extensions\php\php7.3.4nts\ext\php_xdebug-3.1.2-7.3-vc15-nts-x86_64.dll"
xdebug.mode="debug"
xdebug.output_dir="C:\phpstudy_pro\tmp\xdebug"
xdebug.remote_handler="dbgp"
xdebug.idekey="PHPSTORM"
xdebug.client_host=127.0.0.1
xdebug.client_port=9100
```
不要加中文注释，也会报错的。<br />保存后重启apache服务器，这个模块就被成功加载了：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1641538232833-245dbfda-704c-4fd5-b9f6-0efa2397f4a0.png#clientId=u16773827-4496-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=356&id=u9f6a0595&margin=%5Bobject%20Object%5D&name=image.png&originHeight=356&originWidth=404&originalType=binary&ratio=1&rotation=0&showTitle=false&size=3684&status=done&style=none&taskId=ucd1b1a19-f9b5-496e-9dd9-1413ad40e0a&title=&width=404)

3. 配置phpstorm

[https://blog.51cto.com/u_13640989/2709976](https://blog.51cto.com/u_13640989/2709976)<br />按照上面的教程配置，需要注意：phpstorm最新版本的debug配置是属于PHP目录下的，不在language下了：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1641538491144-0205ee89-ff6d-4dcc-b079-ad3a1ff64b84.png#clientId=u16773827-4496-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=228&id=ufdaa4eda&margin=%5Bobject%20Object%5D&name=image.png&originHeight=228&originWidth=225&originalType=binary&ratio=1&rotation=0&showTitle=false&size=5731&status=done&style=none&taskId=ude76859c-0171-45ac-a878-72fb3b95c26&title=&width=225)<br />这里改一下就可以了。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1641538415699-bd07e43b-1b79-4904-baf5-1e261db5ffcb.png#clientId=u16773827-4496-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=523&id=ue8f6a5f1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=523&originWidth=580&originalType=binary&ratio=1&rotation=0&showTitle=false&size=51484&status=done&style=none&taskId=u47bb50e0-39f3-4413-ad92-b87430677bb&title=&width=580)<br />出现这样的界面就是成功了。

4. 浏览器插件配置

因为php的调试需要带有自己的cookie，两种方法可以实现：<br />[https://segmentfault.com/a/1190000018961750](https://segmentfault.com/a/1190000018961750) 提供的，使用debug的方法，无需安装浏览器插件；<br />[https://johnfrod.top/%E5%B7%A5%E5%85%B7/wsl2xdebug3phpstorm%E8%B0%83%E8%AF%95%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B/](https://johnfrod.top/%E5%B7%A5%E5%85%B7/wsl2xdebug3phpstorm%E8%B0%83%E8%AF%95%E9%85%8D%E7%BD%AE%E6%95%99%E7%A8%8B/)  提供的浏览器插件的方法（个人认为这个更好用）

这样一番操作之后，就可以成功的进行PHP的动态调试了！<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1641538703551-a3d3bb3f-288b-4b59-a10f-06534d6b697c.png#clientId=u16773827-4496-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=858&id=udd53eead&margin=%5Bobject%20Object%5D&name=image.png&originHeight=858&originWidth=1443&originalType=binary&ratio=1&rotation=0&showTitle=false&size=333307&status=done&style=none&taskId=ua19d1e81-329b-4bb1-9e50-1f796e61853&title=&width=1443)


另外，MYSQLMONITOR也可以用来检测一些运行过程中的sql语句，方便检查出sql注入，配置方法如下：<br />[https://github.com/fupinglee/MySQLMonitor/releases/tag/1.1](https://github.com/fupinglee/MySQLMonitor/releases/tag/1.1)<br />下载jar包后，直接运行打开就好，需要注意，这里需要使用root用户的账号进行登录：<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1641541020588-7d3711e5-48c3-4ecd-8f53-167ac11bce0f.png#clientId=uf81ad7c9-177a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=639&id=u4052f8f3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=639&originWidth=1020&originalType=binary&ratio=1&rotation=0&showTitle=false&size=71397&status=done&style=none&taskId=uba628c3e-6434-4182-b9b6-360dbedcac0&title=&width=1020)<br />很方便可以监控到所有的mysql语句了。<br />需要注意，每一次下断点之前都需要连接一下数据库，否则会报错。

