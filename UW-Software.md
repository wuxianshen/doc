# UW-Software

”UW-Logic“文档描述潜器主要工作模式的工作逻辑，包括任务流、命令流和信息流的实现与交互。

潜器的主要工作模式包括甲板自检模式、水面模式、水下运动模式、下潜模式。



[toc]

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

