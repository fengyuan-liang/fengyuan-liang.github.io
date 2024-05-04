# 分享一个很好用的热更新插件ArthasHotSwap

>- github地址：https://github.com/xxxtai/ArthasHotSwap

我们在**测试环境**进行开发调试的时候，会有想要热更新几个文件的需求

例如想要把下面的`张三`改成`李四`

![image-20240427165838595](https://cdn.fengxianhub.top/resources-master/image-20240427165838595.png)

如果只是做了一小部分的修改，就去重新发布的，有点得不偿失（因为发布常常得几分钟甚至十几分钟），这时候我们就可以使用`ArthasHotSwap`这个插件帮我们进行热更新，`并且操作特别简单`

>只需要下载该插件，然后修改代码，进行编译（因为需要class文件）

![image-20240427171359813](https://cdn.fengxianhub.top/resources-master/image-20240427171359813.png)

>然后使用插件

![image-20240427171524170](https://cdn.fengxianhub.top/resources-master/image-20240427171524170.png)

>再到服务器上粘贴就好了！😁
>
>这里需要注意的是，该插件是将修改后的字节码文件上传到了oss中，然后目标服务器再下载下来进行热更新的，如果需要自定义上传的对象存储的话可以去作者github issue中找到方法

![image-20240427171651771](https://cdn.fengxianhub.top/resources-master/image-20240427171651771.png)

我们可以看到修改已经成功了！

![image-20240427183912366](https://cdn.fengxianhub.top/resources-master/image-20240427183912366.png)

![image-20240427183948357](https://cdn.fengxianhub.top/resources-master/image-20240427183948357.png)