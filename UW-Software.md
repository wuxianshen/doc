# UW-Software

”UW-Software“文档描述潜器软件开发相关内容。



[toc]

## Gitee

TJU_UW项目：https://gitee.com/TJU_UW/dashboard

uw-doc: https://gitee.com/TJU_UW/doc

Windows/Linux下安装Git；

生成ssh key：在git命令行(Windows)、终端(Linux)中，执行ssh-keygen，连续回车以默认配置生成ssh key；

进入~/.ssh文件夹，cat id_rsa.pub，获取公钥内容；

在Gitee中添加公钥。（右上角个人头像，settings，左侧栏选ssh keys，添加ssh key）

使用git clone命令同步工程。



## Git

版本控制 Version Control

二进制 差异管理

Git 操作：

git status: 查看当前仓库状态

git restore : 恢复文件修改；（ git reset HEAD <file> ）

git add: 添加修改/新文件到缓存区(staged)  <----> git rm

git commit -m "****" : 提交缓存区修改到git仓库

git reset --hard *** ： 回滚到某个版本

Github / Gitee：

git clone : 同步远程仓库代码

git pull : 从远程仓库获取最新提交（同步远程仓库）

git push : 将本地提交推送到远程仓库 (同步本地仓库)

"pull request"



## 交叉编译准备

在X86主机上编译ARM平台程序——使用arm gcc

arm平台gcc下载地址：

https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads

直接解压预编译好的gcc程序到某个路径，解压后文件夹下的bin子文件夹中即包含编译程序需要使用的程序：arm-linux-gnueabihf-gcc

后续交叉编译时，需要通过配置环境变量PATH或者修改cmake配置，指定使用arm gcc。



## 潜器主控程序

uw_all_in_one程序

### 1 构建与编译



### 2 运行



## Can-Festival

### 1 构建与编译

#### 本地编译（x86_64 Native）

① socket can
sudo ./configure --target=unix --can=socket --timers=unix --debug=ERR
make
sudo make install
② virtual can
sudo ./configure --prefix=$HOME/work/vcan --target=unix --can=virtual --timers=unix --debug=ERR
make
sudo make install

注意：

① --prefix用于指定make install时安装的路径，这个路径同样也是调用库时需要使用的路径（链接、头文件）；

② --can用于指定can设备，主要使用socket can；

#### 交叉编译（ARM）

以socket can为例：

① Configure arm-linux-gcc bin path
export PATH=/home/tao/tools/arm-linux-gcc-7.4/bin:$PATH

修改Linux系统环境变量，终端内运行后对此终端有效，目的是让终端可以找到arm-linux-gnueabihf-gcc程序；

② Compile Canfestival for ARM
sudo ./configure --cc=arm-linux-gnueabihf-gcc --arch=arm  --os=unix --prefix=$HOME/work/imx6/can-festival --target=unix --can=socket --timers=unix

## LINUX串口编程

### 串口初始化

波特率

起始位 + 数据位 + 奇偶校验位 + 停止位

特殊设置

https://www.cnblogs.com/li-hao/archive/2012/02/19/2358158.html

### 串口调用流程

打开串口

串口设置

https://blog.csdn.net/morixinguan/article/details/80898172

串口读写

关闭串口

### cmake

https://blog.csdn.net/whahu1989/article/details/82078563