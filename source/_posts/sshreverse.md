title: ssh 反向代理    
date: 2014-10-16 14:01:01
categories: 网络技术 
tags: [ssh反向代理,ssh,穿越内网] 

---

为了解决无法突破对远端设备所在内网访问的限制，ssh提供了一套反向代理机制来解决问题。

<!--more-->
## 1.原理图

![image description](http://7xkz95.com1.z0.glb.clouddn.com/15-8-9/58003707.jpg)

## 2.使用方法
##### 2.1 在A下

```
ssh -NfR 2222:localhost:22 httpd@100.100.100.100 -p 22
```
（连接对方22的端口，并把B主机的2222端口 与 本地A主机的22端口映射，即建立通信线路。用httpd这个用户名登陆B主机，B主机ip为100.100.100.100）
##### 2.2 在B下

```
ssh -p 2222 exe@localhost
```

（B主机通过ssh客户端连接自己本地的2222端口，并用exe登陆远程A主机）

（也可以不用输入exe@，直接输入ssh -p 2222 localhost即可）

**这里需要说明的是，为了使B区分不同的远程主机，即可用端口号加以区分。**

##### 2.3 为了解决内网A主机 第一次主动连接B主机时，需要输入密码的问题，还有ssh生命周期时间太短的问题。我们可以采用ssh-keygen和autossh解决。

a.在A主机上键入：
```
ssh-keygen
```
 ，依次点下去，会在~目录下的.ssh/中生成一个ida_rsa.pub的文件，把这个文件通过scp拷贝到远程主机B上

b.在B主机上键入：
```
cat ida_rsa.pub >> ~/.ssh/authorized_keys
```


此时，A主机已经免输密码。

c.接着我们在A主机下安装一个autossh （ubuntu可采用apt-get install autossh安装，嵌入式设备可能需要移植）

~~d.在A主机上键入 
```
autossh -M 5678 -NR 2222:localhost:22 root@100.100.100.100 -p 22  & 
```

此时，A主机与B主机的SSH通道一直会保持在线了。~~


##### 2.4 经测试发现，这条命令并不适用于3G网卡传输的方式，因为3G的无线传输方式本身就有传输可能会断掉的缺点。如果A主机端为一个3G网络的环境，那么需要在A主机上使用以下方法解决。
d1.    
```
autossh -M 0 -q -N -o ServerAliveInterval=10 -o ServerAliveCountMax=3 -R 44444:localhost:22 root@100.100.100.100
```
这条命令的作用比上一条多了一个心跳信息，就是说A主机的ssh每隔10秒会主动给B主机的sshd发送心跳信息，这时候B主机sshd会给予A主机ssh一个响应信息，这是一个过程，如果B主机没有给予ssh一个响应信息，那么ServerAliveCountMax值+1，当ServerAliveCountMax等于3时，ssh客户端认为已经断掉，这时候ssh会自己关闭相关端口和进程，然后由autossh自动启动ssh客户端，再一次连接，也就是说完成一次检测网络状态，需要30秒，如果30秒，由于网络阻塞或者其他问题，那么ssh会主动关闭。

d2.   另外，由于是3G网络环境（测试发现，如果3G网络异常断开，那么autossh进程会死掉），所以需要加入一小段监控的shell脚本，定期检测autossh是否存在，如果不存在则重启，如果存在阻塞几秒继续监控。

d3.   上边配置了A主机，还需要配置B主机（服务器），首先要说的是，当用d1的那条命令后，会使得服务器端ssh连接异常，也就是说putty可能登陆不上去，重启也无效（这个可能是特例，总之咱不知道具体原因），查了一些资料，说是要更改ssh客户端的配置即可（这里不明白），具体做法是/etc/ssh/ssh_config这个文件中，加入ServerAliveInterval 60即可，据说是防被踢，但是不理解。。。。。

d4. 另外我们使用netstat -antp命令可以看到linux当前网络的端口占用情况。测试的时候我发现A主机的3G网络异常断开之后，A主机的autossh自然死掉了。但是在B主机netstat -antp时，还是可以看到系统在监听44444端口，后边的进程是sshd 服务端程序启动的，如果我们在B主机上不做任何设置，那么会导致B主机44444端口一直处于无效状态，即便A主机设置重新启动autossh，也无法再B主机通过44444与A主机建立连接。解决方法是打开/etc/ssh/sshd_config配置文件，在最后边添加或者更改ClientAliveIntercal 10 还有 ClientAliveCountMax 3即可，意思就是说sshd服务端程序主动去发送小心给客户端（A主机），A主机如果与B保持连接，那么肯定会给予B回复，否则不回，这样就是使得10秒之后，count值+1，直到3，sshd程序就会主动把占用端口的sshd子进程杀掉，让出端口来。这时候就等待A主机建立连接就可以了！
> 
> 副：测试发现，也有小概率出现，在A主机端发现autossh进程在，ssh进程也在，但是在B主机端用netstat -antp发现没有在监听44444端口。这时，需要在B主机端主动杀掉netstat -antp结果中那个本地22的端口后边对应的sshd:root所在的pid号。这时，你再netstat -antp，会发现44444已经出来了，在A主机上，ps可以看到autossh进程号没有改变，但是ssh进程改变了，是因为B主机端主动杀掉了连接，所以ssh又一次重连。 
> 
如果想要A主机重启之后，也可以保持自动ssh反向连接，那么需要追加到相应脚本下实现开机自启动。

## 参考文献

http://www.jb51.net/hack/58514.html

http://blog.chinaunix.net/uid-20109107-id-2954579.html

http://www.cnblogs.com/eshizhan/archive/2012/07/16/2592902.html