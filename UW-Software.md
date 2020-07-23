# UW-Software

”UW-Software“文档描述潜器软件开发相关内容。



[toc]

## 1 Gitee

TJU_UW项目：https://gitee.com/TJU_UW/dashboard

uw-doc: https://gitee.com/TJU_UW/doc

Windows/Linux下安装Git；

生成ssh key：在git命令行(Windows)、终端(Linux)中，执行ssh-keygen，连续回车以默认配置生成ssh key；

进入~/.ssh文件夹，cat id_rsa.pub，获取公钥内容；

在Gitee中添加公钥。（右上角个人头像，settings，左侧栏选ssh keys，添加ssh key）

使用git clone命令同步工程。

注意，从Gitee上同步仓库后，远程源只连接到Gitee上，我们需要同步连接到github上，执行如下操作：

对于doc仓库：

git remote set-url --add origin git@github.com:wuxianshen/doc.git

对于uw_all_in_one：

git remote set-url --add origin git@github.com:wuxianshen/uw_all_in_one.git

移除某个新添加的url：git remote set-url --delete origin





## 2 Git

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



## 3 交叉编译准备

在X86主机上编译ARM平台程序——使用arm gcc

arm平台gcc下载地址：

https://developer.arm.com/tools-and-software/open-source-software/developer-tools/gnu-toolchain/gnu-rm/downloads

直接解压预编译好的gcc程序到某个路径，解压后文件夹下的bin子文件夹中即包含编译程序需要使用的程序：arm-linux-gnueabihf-gcc

后续交叉编译时，需要通过配置环境变量PATH或者修改cmake配置，指定使用arm gcc。



## 4 潜器主控程序

uw_all_in_one程序

### 1 同步代码

git clone git@gitee.com:TJU_UW/uw_all_in_one.git --recursive

如果忘记添加--recursive选项，可以在仓库目录下运行：

git submodule update --init --recursive

同步代码后应注意添加github的push源

### 2 构建与编译

① Arm gcc编译器：arm-linux-gcc-7.4.tar.gz解压，例如解压到/home/tao/tool/gcc-arm-7；

尝试运行解压后的arm gcc，如果提示找不到文件，需要安装32位运行库：

sudo apt-get install lib32ncurses5 lib32z1

② 使用本地gcc和arm gcc编译can-festival，详见5，假设Arm版本安装到/home/tao/tool/can-festival，x86版本默认安装到/usr/local下；

③ 修改代码目录下cmake-modules文件夹中的compiler_config.cmake文件，配置if (CTYPE MATCHES Arm)条件下的gcc路径：

set(TOOLS /home/tao/tool/gcc-arm-7) 

④ 修改代码目录下cmake-modules文件夹中的can_motor.cmake文件，配置if (CTYPE MATCHES Arm)条件下的can festival路径：

set(CAN_FESTIVAL_DIR /home/tao/tool/can-festival)

⑤ 编译x86版本程序：

进入代码目录下：

mkdir build_x86 & cd build_x86

cmake -DARCH=x86 ..

make

⑥ 编译arm版本程序：

回到代码目录下，

mkdir build_arm & cd build_arm

cmake -DARCH=Arm ..

make

⑦ make后，build目录的bin文件夹下包含编译生成的可执行程序uw_all_in_one.

### 3 运行



## 5 Can-Festival

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



## 6 LINUX串口编程

### 开启串口权限

sudo usermod -a -G dialout $USER，注销后重新登录；

Linux下串口助手：sudo apt-get install cutecom

### 串口初始化（串口设置）

1. 波特率

2. 起始位 + 数据位 + 奇偶校验位 + 停止位

3. 特殊设置

https://www.cnblogs.com/li-hao/archive/2012/02/19/2358158.html

### 串口调用流程

打开串口-----ttyS1(COM1)---open函数

串口设置

https://blog.csdn.net/morixinguan/article/details/80898172

串口读写-----read和write函数

关闭串口---close函数

### cmake

https://blog.csdn.net/whahu1989/article/details/82078563



## 7 GPS编程

### 进程与线程

https://blog.csdn.net/beidaol/article/details/89135277

https://blog.csdn.net/tjcwt2011/article/details/89561927

https://blog.csdn.net/wojiuguowei/article/details/82993697

线程创建

互斥锁

条件变量

信号量

### GPS串口数据---状态机

### 程序流程