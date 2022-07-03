# 记一次linux挖矿处理过程（kthreaddk）

一大早就收到了阿里云的短信，说服务器很危险

![image-20220331084553479](https://cdn.fengxianhub.top/resources-master/202203310845563.png)

打开控制台一看，有个进程占用了很大一部分CPU资源

![image-20220331084815320](https://cdn.fengxianhub.top/resources-master/202203310848497.png)

上百度上查了查说这个东西叫`门罗币挖矿木马`,伪装的实现是太好了和系统中的正常进程`kthreadd`太像了

先不急着kill掉，先按照网上的做法使用命令`crontab -l`查看一下定时任务，再用`ll /proc/pid(参数)/exe`看一下其进程目录

![image-20220331085901873](https://cdn.fengxianhub.top/resources-master/202203310859948.png)

我在想`xxx-job`是我用来做分布式调度任务的，难道这玩意出了什么问题吗？

去看了一下跑的程序的配置

![image-20220331090122921](https://cdn.fengxianhub.top/resources-master/202203310901981.png)

怎么会出现在我日志的保存路径呢，很奇怪，去进程目录下看一下

![image-20220331091846459](https://cdn.fengxianhub.top/resources-master/202203310918596.png)

发现压根就没那玩意

不管了，先鲨掉这个进程再说

![image-20220331092011504](https://cdn.fengxianhub.top/resources-master/202203310920651.png)

鲨掉后服务器就正常了，但是过了几秒钟这个进程又跑起来了，发现有一个定时任务

![image-20220331092241561](https://cdn.fengxianhub.top/resources-master/202203310922652.png)

这次先清理掉定时任务再鲨掉这个进程

![image-20220331092628427](https://cdn.fengxianhub.top/resources-master/202203310926510.png)

但是过了几秒钟它又回来了，😂绝了，这没完了

索性把该进程存在目录都删掉（我找不到具体的，那就把这个目录都删掉）

但是它过了几秒钟又回来了！绝了

无奈之下去stackOverflow查看，发现原来被挖矿的不止我一个，按照stackOverflow的说法，crontab定时定时任务是每个用户都可以进行提交执行的，我们在/etc/cron.deny 中关闭某一用户的 crontab

![image-20220331100406058](https://cdn.fengxianhub.top/resources-master/202203311004219.png)

我来执行一下：

先看看那个用户在创建crontab

![image-20220331100827684](https://cdn.fengxianhub.top/resources-master/202203311008784.png)

禁止他创建crontab任务

```java
 vim /etc/cron.deny
```

按`i`进入编辑模式，写入用户root再按`Esc`回到末行模式，按`shift`+冒号`:`并输入wq，保存并退出

载入一下crontab的配置

```shell
/sbin/service crond reload 
```

清理定时任务

![image-20220331101130257](https://cdn.fengxianhub.top/resources-master/202203311011319.png)

鲨掉挖矿进程

![image-20220331101200384](https://cdn.fengxianhub.top/resources-master/202203311012429.png)

>然后还是一点用都没有，我感到很奇怪，为什么禁止使用crontab了还是又会产生，难道这就是`门罗币挖矿木马`的厉害之处嘛

继续查阅，发现有一段这样的回答：

![image-20220331135337246](https://cdn.fengxianhub.top/resources-master/202203311411377.png)

最终解决：https://blog.csdn.net/weixin_41599103/article/details/115403332?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.vipsorttest&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.vipsorttest



解决方案：

执行netstat -ltnp，可以看见一个奇怪的进程名在监听52335端口（忘记截图了，端口可能会不一样），先把这个进程kill掉，再执行下面的步骤。

1.  du -b zwmdcg（其中一个文件的名字）获取到文件的大小；

2.  find ./ -size  4099324c -type f -exec rm {} \;；（查找并删除文件，可以先用<find ./ -size  4099324c -type f>查找文件   {} \;是{}+空格+；都是必要的，4099324换成文件实际大小）

3. crontab -l 查看定时任务，如果还有定时任务 crontab -r删除定时任务

4. kill kthreaddi进程。

5. 如果kthreaddi继续重启，反复执行2,3,4即可。

最后把防火墙什么的都检查一下，看后续会不会再被黑吧。



















