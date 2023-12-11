# 为阿里云域名配置泛解析SSL证书并自动刷新

最近阿里云免费的SSL过期了，需要更换，打开控制台一看，好家伙，居然已经用完了😱

![image-20231210235422273](https://cdn.fengxianhub.top/resources-master/image-20231210235422273.png)

而且阿里云证书有几个痛点：

- 一年刷新一次，不能自动更新
- 免费的没有泛解析证书，每个域名一个证书有点麻烦
- 免费证书有限

基于以上痛点，我打算使用开源免费的泛解析证书

- 基于docker容器进行一键部署（可以自动刷新），并且异常情况会邮件进行通知
- 使用`Let's Encrypt`或者`ZeroSSL`的泛解析证书
- 泛解析证书理论上没有限制

>下面是介绍，可以跳过😁

市面上主流的证书生成工具有 `certbot`、`acme.sh`、`acme-tiny`，其中[acme.sh](https://github.com/acmesh-official/acme.sh)申请泛解析证书最为方便

市面上使用最多的免费泛解析证书是`Let's Encrypt`家的，[acme.sh](https://link.zhihu.com/?target=http%3A//acme.sh) v3.0之前默认申请的也是这家的证书，但是在[acme.sh](https://link.zhihu.com/?target=http%3A//acme.sh) v3.0之后，默认申请的证书是`ZeroSSL`，[ZeroSSL](https://app.zerossl.com/)同样可以申请`90`天的免费泛解析证书，相较于`Let's Encrypt`，ZeroSSL提供了一个管理界面可以管理我们的证书，并且提供api可以进行二次开发，可玩性方面比Let's Encrypt更多一些；缺点是稳定性方面可能没有老牌Let's Encrypt证书好，但是笔者并没有遇到过；当然也可以使用Let's Encrypt🎈



## docker容器一键部署

>为了方便你我他，笔者构建了docker容器进行一键部署，并且能通过邮件对证书异常进行自动通知
>
>这里分为两步
>
>- 自动使用`acme.sh`刷新证书
>- 自动调用阿里云API更新OSS证书

```shell
version: '3'
services:
  acme-sh:
    image: neilpang/acme.sh
    container_name: acme.sh
    restart: always
    command: daemon
    environment:
      - Ali_Key="xxxxx"
      - Ali_Secret="xxxxx"
    volumes:
      - ./acme/out:/acme.sh
    network_mode: host
```

**启动容器**

```shell
$ docker-compose -f docker-compose-acme.sh.yaml -p acme.sh up -d
```

**申请SSL证书**

```shell
1、首先将默认证书服务器切换为 letsencrypt
docker exec acme.sh acme.sh --set-default-ca --server letsencrypt

# --debug 可以去掉，我们加上能直接打印出每一步详细日志，如果出错的话可以知道在哪一步出错
# -d 后面换成你自己的域名，可以使用*号 泛指所有子域名,eg *.3658k.com
docker exec acme.sh  --issue  --dns dns_ali -d example.com -d *.example.com --debug
# 其他云厂商 例如腾讯云只需要修改--dns dns_dp 
```

**设置 crontab 任务自动续签**

在宿主主机的cronta，添加一条定时任务，每天让acme容器去自动检查下那些域名过期，如果过期的话，acme会自动续签,这样我们就不用担心证书过期了

`crontab -e` 添加一下内容

```shell
0 0 * * * docker exec acme.sh --cron
```

**升级 acme**

有的时候续签可能会出问题，这时候就需要升级一下acme看能不能自动解决了

```shell
docker exec acme.sh acme.sh --upgrade
```















