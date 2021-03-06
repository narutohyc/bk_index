# 电子书目录

| gitbook版本                                           | github版本                                                   | 阿里云版本                                       |
| ----------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------ |
| [python学习笔记](https://narutohyc.gitbook.io/python) | [python学习笔记](https://narutohyc.github.io/bk_python)      | [python学习笔记](http://112.126.102.142:4000/)   |
| [java学习笔记](https://narutohyc.gitbook.io/jdk)      | [java学习笔记](https://narutohyc.github.io/bk_jdk)           | [java学习笔记](http://112.126.102.142:4001/)     |
| [机器学习学习笔记](https://narutohyc.gitbook.io/ml)   | [机器学习学习笔记](https://narutohyc.github.io/bk_machineLearning) | [机器学习学习笔记](http://112.126.102.142:4002/) |



# 阿里云部署

---

>阿里云部署笔记(centos版本)

## 购买[阿里云主机](https://www.aliyun.com/)，并开启安全组

云主机实例

![image-20200328203641733](res/README/image-20200328203641733.png)

配置安全组

![image-20200328203832056](res/README/image-20200328203832056.png)

## 安装git、jdk和nodejs

> 安装jdk

```cmd
# 安装jdk
rpm -ivh jdk-8u241-linux-x64.rpm
```

> 配置jdk环境变量
>
> vi /etc/profile

```
export JAVA_HOME=/usr/java/jdk1.8.0_241-amd64
export JRE_HOME=/usr/java/jdk1.8.0_241-amd64/jre
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
```

> ps: 若原先存在openjdk，需要先卸载
>
> ```cmd
> # 查看jdk列表
> rpm -qa|grep java
> # 卸载
> rpm -e --nodeps java-1.8.0-openjdk-1.8.0.181-7.b13.el7.x86_64
> ```

> 安装和配置nodejs

```cmd
# 安装和配置nodejs
cp node-v12.16.1-linux-x64.tar.xz /usr/local/
cd /usr/local/
tar -xvf node-v12.16.1-linux-x64.tar.xz
mv node-v12.16.1-linux-x64/ nodejs
cd nodejs
mkdir node_global
mkdir node_cache
npm config set prefix "/usr/local/nodejs/node_global"
npm config set cache "/usr/local/nodejs/node_cache"
# 配置nodejs
vim /etc/profile
# 在末尾添加
export PATH=$PATH:/usr/local/nodejs/bin # config nodejs
# 执行生效命令
source /etc/profile
```

> 安装和配置git


```cmd
# 安装和配置git
yum install git
git config --global user.name "yourname"
git config --global user.mail "xxxxxxxxxx@mail.com"
# 配置远程仓库 将pub_key黏贴到git
ssh-keygen -t rsa -C "xxxxxxxxxx@mail.com"
```

> 安装和配置git


```cmd
# 安装和配置git
yum install git
git config --global user.name "narutohyc"
git config --global user.mail "1832044043@qq.com"
# 配置远程仓库 将pub_key黏贴到git
ssh-keygen -t rsa -C "1832044043@qq.com"
```

> 安装gitbook

```cmd
# 安装gitbook
npm install gitbook-cli -g
# 添加gitbook软链接
ln -s /usr/local/nodejs/node_global/bin/gitbook /usr/bin/gitbook
# gitbook指令测试
gitbook -V
```

## git访问慢配置

> **通过修改 hosts文件，重定向网站的访问到代理服务器上**
>
> [**Github下载速度太慢怎么办？完美解决**](https://yq.aliyun.com/articles/713169)
>
> [服务器查询](https://www.ipaddress.com/?spm=a2c4e.10696291.0.0.1d6f19a4MrZtLO)
>
> ![image-20200328210756641](res/README/image-20200328210756641.png)

| **系统**        | **hosts文件路径**                             |
| :-------------- | :-------------------------------------------- |
| **Window**      | **`%systemroot%\system32\drivers\etc\hosts`** |
| **Mac / Linux** | **`/etc/hosts`**                              |

```properties
# github
219.76.4.4 github-cloud.s3.amazonaws.com
140.82.113.4 github.com
151.101.185.194 github.global.ssl.fastly.net
```

> 刷新DNS缓存
>
> [Linux/Mac/Windows-刷新DNS的方法](https://baijiahao.baidu.com/s?id=1616353495084098905&wfr=spider&for=pc)
>
> `yum install -y nscd`
>
> `systemctl restart nscd`

## 常用命令

> `netstat -atunlp`查看端口占用

![image-20200328210123194](res/README/image-20200328210123194.png)

> `杀死指定进程`

![image-20200328210327497](res/README/image-20200328210327497.png)

> `gitbook指定端口并后台执行`

```cmd
# 指定端口为4001，并放在后台执行
setsid gitbook serve --lrport=35729 --port=4001
```

> 若启动gitbook报错

```cmd
Starting server ...
Serving book on http://localhost:4000

Error: ENOSPC: System limit for number of file watchers reached, watch '/home/hyc/github/bk_python/node_modules/entities/mathjax/jax/output/HTML-CSS/fonts/Neo-Euler/Symbols'
```

> 则执行以下命令即可

```shell
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p
sudo sysctl --system
```

## 批量杀死进程

> 查看进程号

```shell
[root@iZ2zeh6nc313zs4azjijjnZ ~]# ps -ef | grep gitbook | grep -v grep|awk '{print $2}'
24064
29387
30425
```

> 方式一：batch_shell

```shell
#!/bin/sh
dsp_admin_id=`ps -ef | grep gitbook | grep -v grep|awk '{print $2}'`
echo $dsp_admin_id
for id in $dsp_admin_id
do
    kill -9 $id  
    echo "killed $id" 
done
```

> 方式二：单行执行

```shell
#!/bin/sh
ps -ef | grep gitbook | grep -v grep|grep 4000|  awk '{print $2}' | xargs kill -9
```



# 重装系统

## u盘启动盘制作

### 方式一

1. 百度搜索[微软官网工具](https://www.microsoft.com/zh-cn/software-download/windows10/)

   ![image-20210121171520084](res/README/image-20210121171520084.png)

2. 下载后，使用管理员身份运行
3. 按步骤操作，其中下载镜像会比较耗时(好像只能下载win10)
4. 制作启动盘完成后，就可以开始安装系统了



### 方式二

1. 百度搜索[itellyou](https://msdn.itellyou.cn/)，下载系统镜像

   ![image-20210121172227945](res/README/image-20210121172227945.png)

2. 下载u盘启动盘制作工具[UltraISO](https://cn.ultraiso.net/xiazai.html)，安装后执行

   ![image-20210121172408486](res/README/image-20210121172408486.png)

   ![image-20210121172424293](res/README/image-20210121172424293.png)

3. 制作启动盘完成后，就可以开始安装系统了

## 主板启动热键

![img](res/README/21a4462309f790525fa44160a3f343cf7bcbd53f.jpeg)