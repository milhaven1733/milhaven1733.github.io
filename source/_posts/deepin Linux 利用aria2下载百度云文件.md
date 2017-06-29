### deepin Linux 利用aria2下载百度云文件

#### 目的

解决deepin用chrome网页下载百度云资源限速、crossover难以安装百度云Win版客户端等问题。

#### 需要工具

下载器：

aria2

##### chrome插件：

YAAW for Chrome （管理aria2下载任务）

百度网盘助手（导出百度云链接到aria2）

#### 搭建过程

##### 安装配置aria2：

参考了[Debian 如何搭建使用 aria2c 作为下载工具](http://www.w2bc.com/article/215354)

安装aria2：

```
sudo apt-get update
sudo apt-get install aria2
```

测试版本：

```
aria2c --version
```

![](/home/milhaven1733/Desktop/1.png)

启动服务：创建后端启动需要的配置文件

```
mkdir /root/.aria2

wget http://webdir.cc/aria2.conf /root/.aria2/aria2.conf

wget http://webdir.cc/dht.dat /root/.aria2/dht.dat
```

测试一下，服务是否启动成功：

```
netstat -anp|grep 6800
```

加载插件：

YAAW

可以直接从Google商店安装：

![](/home/milhaven1733/Desktop/2.png)

百度网盘助手

google应用商店已下架，可以从[作者github](https://github.com/acgotaku/BaiduExporter/tree/a9cabe7624993eb98613dca18183e0bf113fe34d)上下载插件包手动加载。

步骤：

- 插件github主页download ZIP包，解压
- 进入Chrome扩展程序
- 勾选“开发者模式”
- 点击“加载已解压的扩展程序”
- 选中之前解压好的包中的“chrome”文件夹并载入

#### 下载文件

重启chrome浏览器,随便打开一个百度网盘的链接，会发现网页上多出一个「导出下载」按钮，点击它弹出的「ARIA2 RPC」就自动添加到你的下载队列里了,此时点击浏览器右上角的YAAW插件即可管理下载任务. 