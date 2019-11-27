

# 甲板软件通讯协议V2.0

| 版本 | 时间       | 说明                                 |
| ---- | ---------- | ------------------------------------ |
| V2.0 | 2019-11-25 | 定义通讯帧格式、字段ID、主要数据类型 |



[toc]

## 1.概述

甲板软件通讯协议用于甲板软件和潜器主控（IMX6、1052）之间的命令和数据通讯，包括协议格式和功能字段定义。

甲板软件与潜器主控之间的通讯通过串口通讯实现，数据链路包括无线电台、声通讯和铱星三种，对应三个不同的串口，串口默认配置为：

| 波特率 | 数据位 | 停止位 | 奇偶校验 |
| ------ | ------ | ------ | -------- |
| 19200  | 8      | 1      | None     |

## 2.帧格式

帧格式包括帧头段(FrameHeader)、帧数据段(FrameData)、帧尾段(FrameEnd)三部分，帧格式见表2.1所示。帧数据段是一个完整的数据包(Package)，数据包长度为**N** (N<128), 格式见表2.2所示。

### 2.1 帧格式

表2.1 甲板软件通讯协议帧格式

| 字节序号 / ByteSeq | 内容 / Content  | 说明 / Description                              |
| ------------------ | --------------- | ----------------------------------------------- |
| FrameHeader[0]     | 帧头起始标志[0] | 0x55                                            |
| FrameHeader[1]     | 帧头起始标志[1] | 0xAA                                            |
| FrameHeader[2]     | 帧长度[0]       | uint16_t低字节，帧长度为Data段的长度N           |
| FrameHeader[3]     | 帧长度[1]       | uint16_t高字节，帧长度为Data段的长度N           |
| FrameHeader[4]     | 帧序号          | 0x00                                            |
| FrameHeader[5]     | 帧头校验字节    | CRC8 (FrameHeader[0]~FrameHeader[4])            |
| FrameData[0]       | 帧数据段        | 数据包格式见表2.2                               |
| ...                |                 |                                                 |
| FrameData[N-1]     | 帧数据段        | 数据包格式见表2.2                               |
| FrameEnd[0]        | 帧尾[0]         | CRC16校验的低字节 (FrameData[0]~FrameData[N-1]) |
| FrameEnd[1]        | 帧尾[1]         | CRC16校验的高字节 (FrameData[0]~FrameData[N-1]) |

表2.2 甲板软件通讯协议包格式

| 字节序号 / ByteSeq | 内容 / Content | 说明 / Description                            |
| ------------------ | -------------- | --------------------------------------------- |
| FrameData[0]       | 源链路ID       | 用于区分甲板软件和潜器的三个串口链路          |
| FrameData[1]       | 目的链路ID     | 用于区分甲板软件和潜器的三个串口链路          |
| FrameData[2]       | 模块ID         | 针对的模块ID，e.g. DAM：2，详见模块ID定义Enum |
| FrameData[3]       | 功能ID         | 命令/数据对应的功能ID，e.g. Power off：0      |
| FrameData[4]       | 数据段         |                                               |
| ...                |                |                                               |
| FrameData[N-1]     | 数据段         |                                               |

设备ID定义

```
enum uw_ccom_device_e
{
	e_deck_radio    = 0,
	e_deck_iridium  = 1,
	e_deck_acoustic = 2,
	e_imx6_radio    = 3,
	e_imx6_iridium  = 4,
	e_imx6_acoustic = 5,
}
```

### 2.2 串口匹配机制

#### 1.默认串口匹配机制

甲板软件、潜器主控软件维护一个实时更新的**“功能包——>串口链路”**的映射数组(uint8_t类型数组)，存储当前进行通讯时默认的串口链路(初始默认链路为无线电台)。

甲板软件发送的数据包中填写源设备ID，指明发送的串口链路信息，潜器主控收到数据后，解析链路ID，根据数据包模块ID、功能ID，将链路ID对应写入数组对应位置。

需要对数据包进行回复，或潜器主动发送数据包到甲板软件时，从链路映射数组中取出链路ID，选择对应串口进行发送。

#### 2.设置串口匹配

甲板软件和潜器通讯的三个串口可以根据发送数据的模块ID和功能ID进行配置，由甲板软件进行设置。

#### 3.串口链路优先级

根据具体模式、场景和需求定义。



## 3.字段定义

### 说明

字段定义包括模块ID和功能ID的定义，一帧的数据包作用由模块ID和功能ID的组合确定。

水下设备(IMX6,1052-1)回复甲板软件时，对应修改源设备ID和目的ID，帧序号、模块ID和命令ID相同。说明中没有提到回复内容时表示默认不回复此帧。

甲板软件发出查询类命令后，潜器回复对应数据；某种工作模式下，潜器主动按照约定频率发回指定数据，模块ID、功能ID与查询命令的回复一致。

### 3.1 模块ID定义

见packet_def.h中枚举packet_type_e定义。

| 枚举定义                | 模块名称   | 模块ID | 说明             |
| ----------------------- | ---------- | ------ | ---------------- |
| e_general_packet        | 通用模块   | 0x00   | 保留             |
| e_deck_packet           | 甲板通信   | 0x01   |                  |
| e_propeller_packet      | 主推进器   | 0x02   |                  |
| e_can_motor_packet      | 关节电机   | 0x03   |                  |
| e_ejection_packet       | 抛载装置   | 0x04   |                  |
| e_dam_packet            | DAM        | 0x05   |                  |
| e_imu_packet            | IMU        | 0x06   |                  |
| e_gps_packet            | GPS        | 0x07   |                  |
| e_dvl_packet            | DVL        | 0x08   |                  |
| e_depthomoter_packet    | 深度计     | 0x09   |                  |
| e_humidometer_packet    | 湿度计     | 0x0A   |                  |
| e_radio_station_packet  | 无线电台   | 0x0B   |                  |
| e_iridium_comm_packet   | 铱星       | 0x0C   |                  |
| e_acoustic_comm_packet  | 声通讯     | 0x0D   |                  |
| e_battery_packet        | 电池模块   | 0x0E   |                  |
| e_auv_model_packet      | 潜器模式   | 0x0F   | 潜器工作模式控制 |
| e_auv_model_init_packet | 初始化模式 | 0x10   | 潜器初始化控制   |
| e_auv_model_deck_packet | 甲板模式   | 0x11   | 潜器甲板自检测试 |
| e_auv_model_surf_packet | 水面模式   | 0x12   | 潜器水面测试     |
| e_auv_model_sail_packet | 远航模式   | 0x13   | 潜器远航测试     |
| e_auv_model_dive_packet | 下潜模式   | 0x14   | 潜器下潜测试     |
| e_motion_param_packet   | 运动参数   | 0x15   | 潜器运动参数反馈 |
| e_telemetry_packet      | 遥测参数   | 0x16   | 潜器遥测参数     |

packet_type_e定义如下。

```
enum packet_type_e
{
    e_general_packet   = 0,
    e_deck_packet      = 1,
    //actuators
    e_propeller_packet = 2,
    e_can_motor_packet = 3,
    e_ejection_packet  = 4,
    //sensors
    e_dam_packet       = 5,
    e_imu_packet       = 6,
    e_gps_packet       = 7,
    e_dvl_packet       = 8,
    e_depthomoter_packet   = 9,
    e_humidometer_packet   = 10,
    //communication device packet
    e_radio_station_packet = 11,
    e_iridium_comm_packet  = 12,
    e_acoustic_comm_packet = 13,
    //battery
    e_battery_packet       = 14,
    //auv model
    e_auv_model_packet      = 15,
    e_auv_model_init_packet = 16,
    e_auv_model_deck_packet = 17,
    e_auv_model_surf_packet = 18,
    e_auv_model_sail_packet = 19,
    e_auv_model_dive_packet = 20,
    //motion param
    e_motion_param_packet   = 21,
    //telemetry param
    e_telemetry_packet      = 22,
    e_packet_type_num,
};
```



### 3.2 功能ID定义

#### 1.通用模块(e_general_packet)

模块ID：0x00

保留模块，暂用于包数据内容打印。

| 枚举定义        | 功能名称 | 功能ID | 数据长度 | 说明                                            |
| --------------- | -------- | ------ | -------- | ----------------------------------------------- |
| e_general_print | 通用打印 | 0x00   | 任意     | 按字节打印包数据内容中可显示字符到终端和log文件 |



#### 2.甲板通信模块(e_deck_packet)

模块ID：0x01

甲板通信功能模块，包括用于甲板通信检测的基本功能。

| 枚举定义     | 功能名称 | 功能ID | 数据长度 | 说明                                                         |
| ------------ | -------- | ------ | -------- | ------------------------------------------------------------ |
| e_deck_print | 打印     | 0x00   | 任意     | 按字节打印包数据内容中可显示字符到终端和log文件              |
| e_deck_echo  | 回显     | 0x01   | 任意     | 按字节打印包数据内容中可显示字符到终端和log文件<br />回复包数据内容（修改源设备ID和目的设备ID） |



#### 3.主推进器模块(e_propeller_packet)

模块ID：0x02

主推进器模块，包括主推进器控制、状态查询等功能。

| 枚举定义                | 功能名称     | 功能ID | 数据长度 | 说明                                                         |
| ----------------------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| e_propeller_power_off   | 下电         | 0x00   | 0        | 主推进器下电<br />回复与0x02查询相同                         |
| e_propeller_power_on    | 上电         | 0x01   | 0        | 主推进器上电<br />回复与0x02查询相同                         |
| e_propeller_power_query | 供电状态查询 | 0x02   | 0        | 主推进器电源状态查询<br />回复主推进器输入电压：int16_t，单位0.01V |
| e_propeller_rpm_set     | 转速设置     | 0x03   | 4        | 主推进器电源转速设置<br />数据类型uint32_t，单位0.1rpm<br />回复与0x04相同 |
| e_propeller_rpm_query   | 转速查询     | 0x04   | 0        | 主推进器电源转速查询<br />回复主推进器转速rpm：uint32_t，单位0.1rpm |



#### 4.关节电机模块(e_can_motor_packet)

模块ID：0x03

关节电机模块，包括关节电机控制、状态查询等功能。

| 枚举定义                | 功能名称     | 功能ID | 数据长度 | 说明                                                         |
| ----------------------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| e_can_motor_power_off   | 下电         | 0x00   | 1        | 关节电机下电<br />数据：关节组合ID<br />回复与0x02查询相同   |
| e_can_motor_power_on    | 上电         | 0x01   | 1        | 关节电机上电<br />数据：关节组合ID<br />回复与0x02查询相同   |
| e_can_motor_power_query | 供电状态查询 | 0x02   | 1        | 关节电机电源状态查询<br />数据：关节组合ID<br />回复：关节电机组合输入电压：int16_t，单位0.01V |
| e_can_motor_angle_set   | 关节角度设置 | 0x03   | 3        | 关节角度设置<br />数据：<br />关节组合ID，uint8_t<br />目标角度值，int16_t，单位0.1度<br /> |
| e_can_motor_angle_query | 关节角度查询 | 0x01   | 1        | 关节角度查询<br />数据：关节组合ID<br />回复：关节组合I的角度，int16_t，单位0.1度 |



关节电机组合ID

| 枚举定义          | 组合名称 | 组合ID | 说明                |
| ----------------- | -------- | ------ | ------------------- |
| e_all_motor_group | 全部电机 | 0      | 针对全部的四个电机  |
| e_joint1_group    | 关节1    | 1      | 针对关节1的两个电机 |
| e_joint2_group    | 关节2    | 2      | 针对关节2的两个电机 |

```
enum can_motor_group_e
{
    e_all_motor_group = 0,
    e_joint1_group    = 1,
    e_joint2_group    = 2,
};
```





#### 5.抛载电机模块(e_ejection_packet)

模块ID：0x04

抛载模块，包括抛载上电、状态查询等功能。

| 枚举定义               | 功能名称     | 功能ID | 数据长度 | 说明         |
| ---------------------- | ------------ | ------ | -------- | ------------ |
| e_ejection_power_off   | 下电         | 0x00   | 0        | 抛载模块下电 |
| e_ejection_power_on    | 上电         | 0x01   | 0        | 抛载模块上电 |
| e_ejection_power_query | 供电状态查询 | 0x02   | 0        |              |



#### 6.DAM模块(e_dam_packet)

模块ID：0x05

DAM模块，包括DAM控制、状态查询等功能。**【待补充】**

| 枚举定义          | 功能名称     | 功能ID | 数据长度 | 说明                                                       |
| ----------------- | ------------ | ------ | -------- | ---------------------------------------------------------- |
| e_dam_power_off   | 下电         | 0x00   | 0        | DAM模块下电                                                |
| e_dam_power_on    | 上电         | 0x01   | 0        | DAM模块上电                                                |
| e_dam_power_query | 供电状态查询 | 0x02   | 0        | DAM电源状态查询<br />回复：DAM输入电压：int16_t，单位0.01V |



#### 7.IMU模块(e_imu_packet)

模块ID：0x06

IMU模块，包括IMU状态查询等功能。**【待补充】**

| 枚举定义          | 功能名称     | 功能ID | 数据长度 | 说明                                                       |
| ----------------- | ------------ | ------ | -------- | ---------------------------------------------------------- |
| e_imu_power_off   | 下电         | 0x00   | 0        | IMU模块下电                                                |
| e_imu_power_on    | 上电         | 0x01   | 0        | IMU模块上电                                                |
| e_imu_power_query | 供电状态查询 | 0x02   | 0        | IMU电源状态查询<br />回复：IMU输入电压：int16_t，单位0.01V |



#### 8.GPS模块(e_gps_packet)

模块ID：0x07

GPS模块，包括GPS状态查询等功能。**【待补充】**

| 枚举定义           | 功能名称     | 功能ID | 数据长度 | 说明                                                       |
| ------------------ | ------------ | ------ | -------- | ---------------------------------------------------------- |
| e_gps_power_off    | 下电         | 0x00   | 0        | GPS模块下电                                                |
| e_gps_power_on     | 上电         | 0x01   | 0        | GPS模块上电                                                |
| e_gps_power_query  | 供电状态查询 | 0x02   | 0        | GPS电源状态查询<br />回复：GPS输入电压：int16_t，单位0.01V |
| e_gps_query        | 原始GPS查询  | 0x03   | 0        | 当前原始GPS数据查询<br />回复：见4.2.1中GPS数据结构定义    |
| e_gps_fusion_query | 融合GPS查询  | 0x04   | 0        | 当前融合GPS数据查询<br />回复：见4.2.1中GPS数据结构定义    |



#### 9.DVL模块(e_dvl_packet)

模块ID：0x08

DVL模块，包括DVL状态查询等功能。**【待补充】**

| 枚举定义          | 功能名称     | 功能ID | 数据长度 | 说明                                                       |
| ----------------- | ------------ | ------ | -------- | ---------------------------------------------------------- |
| e_dvl_power_off   | 下电         | 0x00   | 0        | DVL模块下电                                                |
| e_dvl_power_on    | 上电         | 0x01   | 0        | DVL模块上电                                                |
| e_dvl_power_query | 供电状态查询 | 0x02   | 0        | DVL电源状态查询<br />回复：DVL输入电压：int16_t，单位0.01V |



#### 10.深度计模块(e_depthomoter_packet)

模块ID：0x09

深度计模块，包括深度计状态查询等功能。**【待补充】**

| 枚举定义                  | 功能名称     | 功能ID | 数据长度 | 说明                                                         |
| ------------------------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| e_depthomoter_power_off   | 下电         | 0x00   | 0        | 深度计模块下电                                               |
| e_depthomoter_power_on    | 上电         | 0x01   | 0        | 深度计模块上电                                               |
| e_depthomoter_power_query | 供电状态查询 | 0x02   | 0        | 深度计电源状态查询<br />回复：深度计输入电压：int16_t，单位0.01V |



#### 11.湿度计模块(e_humidometer_packet)

模块ID：0x0A

湿度计模块，包括深度计状态查询等功能。【待补充】

| 枚举定义                  | 功能名称     | 功能ID | 数据长度 | 说明                                                         |
| ------------------------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| e_humidometer_power_off   | 下电         | 0x00   | 0        | 湿度计模块下电                                               |
| e_humidometer_power_on    | 上电         | 0x01   | 0        | 湿度计模块上电                                               |
| e_humidometer_power_query | 供电状态查询 | 0x02   | 0        | 湿度计电源状态查询<br />回复：湿度计输入电压：int16_t，单位0.01V |



#### 12.无线电台模块(e_radio_station_packet)

模块ID：0x0B

无线电台模块，包括无线电台状态查询等功能。【待补充】

| 枚举定义                  | 功能名称     | 功能ID | 数据长度 | 说明                                                         |
| ------------------------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| e_radio_station_power_off | 下电         | 0x00   | 0        | 无线电台模块下电                                             |
| e_radio_station_power_on  | 上电         | 0x01   | 0        | 无线电台模块上电                                             |
| e_radio_comm_power_query  | 供电状态查询 | 0x02   | 0        | 无线电台电源状态查询<br />回复：无线电台输入电压：int16_t，单位0.01V |



#### 13.铱星模块(e_iridium_comm_packet)

模块ID：0x0C

铱星模块，包括铱星状态查询等功能。【待补充】

| 枚举定义                   | 功能名称     | 功能ID | 数据长度 | 说明                                                         |
| -------------------------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| e_iridium_comm_power_off   | 下电         | 0x00   | 0        | 铱星模块下电                                                 |
| e_iridium_comm_power_on    | 上电         | 0x01   | 0        | 铱星模块上电                                                 |
| e_iridium_comm_power_query | 供电状态查询 | 0x02   | 0        | 铱星电源状态查询<br />回复：铱星输入电压：int16_t，单位0.01V |



#### 14.声通讯模块(e_acoustic_comm_packet)

模块ID：0x0D

声通讯模块，包括声通讯状态查询等功能。【待补充】

| 枚举定义                    | 功能名称     | 功能ID | 数据长度 | 说明                                                         |
| --------------------------- | ------------ | ------ | -------- | ------------------------------------------------------------ |
| e_acoustic_comm_power_off   | 下电         | 0x00   | 0        | 声通讯模块下电                                               |
| e_acoustic_comm_power_on    | 上电         | 0x01   | 0        | 声通讯模块上电                                               |
| e_acoustic_comm_power_query | 供电状态查询 | 0x02   | 0        | 声通讯电源状态查询<br />回复：声通讯输入电压：int16_t，单位0.01V |



#### 15.电池模块(e_battery_packet)

模块ID：0x0E

潜器模式模块，包括电池电量查询等功能。**【待补充】**

| 枚举定义               | 功能名称     | 功能ID | 数据长度 | 说明                                   |
| ---------------------- | ------------ | ------ | -------- | -------------------------------------- |
| e_battery_remain_query | 剩余电量查询 | 0x00   | 0        | 电池组剩余电量查询<br />回复：【待定】 |



#### 16.潜器模式(e_auv_model_packet)

模块ID：0x20

潜器模式模块，包括潜器工作模式切换和查询等功能。

| 枚举定义           | 功能名称 | 功能ID | 数据长度 | 说明                                                         |
| ------------------ | -------- | ------ | -------- | ------------------------------------------------------------ |
| e_auv_model_switch | 模式切换 | 0x00   | 1        | 潜器工作模式切换<br />数据：uint8_t，潜器模式ID，定义如下所示<br />回复与功能0x01相同 |
| e_auv_model_query  | 模式查询 | 0x01   | 0        | 潜器工作模式查询                                             |

潜器工作模式定义：

| 枚举定义         | 组合名称   | 组合ID | 说明             |
| ---------------- | ---------- | ------ | ---------------- |
| e_auv_model_init | 初始化模式 | 0      | 潜器初始化阶段   |
| e_auv_model_deck | 甲板模式   | 1      | 潜器甲板自检阶段 |
| e_auv_model_surf | 水面模式   | 2      | 潜器水面测试阶段 |
| e_auv_model_sail | 远航模式   | 3      | 潜器远航测试     |
| e_auv_model_dive | 下潜模式   | 4      | 潜器下潜阶段     |

```
enum auv_model_e
{
    e_auv_model_init = 0,
    e_auv_model_deck = 1,
    e_auv_model_surf = 2,
    e_auv_model_sail = 3,
    e_auv_model_dive = 4,
};

```



#### 17.潜器初始化模式(e_auv_model_init_packet)

模块ID：0x21

潜器初始化模式模块，包括潜器初始化状态查询等功能。

| 枚举定义               | 功能名称       | 功能ID | 数据长度 | 说明                                                         |
| ---------------------- | -------------- | ------ | -------- | ------------------------------------------------------------ |
| e_auv_model_reinit     | 重新初始化     | 0x00   | 0        | 潜器端程序重新启动<br />                                     |
| e_auv_model_init_query | 初始化状态查询 | 0x01   | 0        | 初始化状态查询<br />回复：uint8_t，初始化状态<br />(0-成功，非0不成功) |



#### 18.潜器甲板模式(e_auv_model_deck_packet)

模块ID：0x22

潜器水面模式模块，包括潜器甲板自检的控制、查询等功能。**【待补充】**

| 枚举定义              | 功能名称 | 功能ID | 数据长度 | 说明 |
| --------------------- | -------- | ------ | -------- | ---- |
| e_auv_model_deck_null | ？       | 0x00   | ？       | ？   |



#### 19.潜器水面模式(e_auv_model_surf_packet)

模块ID：0x23

潜器水面模式模块，包括潜器水面运动控制、状态查询等功能。**【待补充】**

| 枚举定义                    | 功能名称 | 功能ID | 数据长度 | 说明         |
| --------------------------- | -------- | ------ | -------- | ------------ |
| e_auv_model_surf_left_turn  | 水面左转 | 0x00   | ？       | 水面左转运动 |
| e_auv_model_surf_right_turn | 水面右转 | 0x01   | ？       | 水面右转运动 |



#### 20.潜器远航模式(auv_model_sail_event_e)

模块ID：0x23

潜器远航模式模块，包括潜器远航运动控制、状态查询等功能。**【待补充】**

| 枚举定义              | 功能名称 | 功能ID | 数据长度 | 说明 |
| --------------------- | -------- | ------ | -------- | ---- |
| e_auv_model_sail_null | ？       | 0x00   | ？       | ？   |



#### 21.潜器下潜模式(e_auv_model_dive_packet)

模块ID：0x25

潜器下潜模式模块，包括潜器下潜运动控制、状态查询等功能。**【待补充】**

| 枚举定义              | 功能名称 | 功能ID | 数据长度 | 说明 |
| --------------------- | -------- | ------ | -------- | ---- |
| e_auv_model_dive_null | ？       | 0x00   | ？       | ？   |



#### 22.潜器运动参数(e_motion_param_packet)

模块ID：0x26

潜器运动参数模块，包括潜器运动参数反馈等功能。**【待补充】**

| 枚举定义              | 功能名称           | 功能ID | 数据长度 | 说明              |
| --------------------- | ------------------ | ------ | -------- | ----------------- |
| e_motion_gps_position | 融合后GPS位置      | 0x00   | 0        | 回复：见4.2.1     |
| e_motion_vel          | 潜器各向运动速度   | 0x01   | 0        | 回复：见4.2.2 (1) |
| e_motion_acc          | 潜器各向运动加速度 | 0x02   | 0        | 回复：见4.2.2 (2) |
| e_motion_rpy_vel      | 潜器rpy角速度      | 0x03   | 0        | 回复：见4.2.2 (3) |
| e_motion_rpy          | 潜器rpy角度        | 0x04   | 0        | 回复：见4.2.2 (4) |



#### 23.潜器遥测参数(e_telemetry_packet)

模块ID：0x27

潜器遥测参数模块，包括潜器主要模块上电情况、工作情况、关键传感器参数反馈等。**【待补充】**

| 枚举定义          | 功能名称     | 功能ID | 数据长度 | 说明          |
| ----------------- | ------------ | ------ | -------- | ------------- |
| e_telemetry_start | 开启遥测反馈 | 0x00   | 0        | 回复：见4.2.3 |
| e_telemetry_stop  | 关闭遥测反馈 | 0x01   | 0        |               |



## 4.数据格式定义

### 4.1 数据类型说明

int8_t：1Bytes有符号整型，8bit

uint8_t：1Bytes无符号整型，8bit

int16_t：2Bytes有符号整型，16bit，发送时数据低字节在前

uint16_t：2Bytes无符号整型，16bit，发送时数据低字节在前

int32_t：4Bytes有符号整型，32bit，发送时数据低字节在前

uint32_t：4Bytes无符号整型，32bit，发送时数据低字节在前

浮点数类型数据根据实际精度转换为对应整型发送

### 4.2 数据结构说明

#### 1.GPS数据

经纬度单位1E-6度；

GPS状态字state：0-正常，非0-不正常

```
typedef struct _auv_position_gps_s
{
	uint8_t state;
	int32_t longitude;
	int32_t latitude;
}auv_position_gps_s
```

(2) 各轴加速度

#### 2.运动参数

运动参数包括潜器当前运动各向速度、各轴加速度、rpy角速度等数据类型。

(1) 各向速度

单位0.1m/s，分别为侧向速度(向右为正)、前向速度(向前为正)、垂向速度(向上为正)；

```
typedef struct _auv_motion_vel_s
{
	int8_t lateral_vel;
	int8_t forward_vel;
	int8_t vertical_vel;
}auv_motion_vel_s
```

(1) 各向速度

单位0.01m/s2，分别为侧向速度(向右为正)、前向速度(向前为正)、垂向速度(向上为正)；

```
typedef struct _auv_motion_acc_s
{
	int8_t lateral_acc;
	int8_t forward_acc;
	int8_t vertical_acc;
}auv_motion_acc_s
```

(3) rpy角速度

单位0.01度/s，分别为俯仰角速率、滚转角速率和偏航角速率。

```
typedef struct _auv_angular_vel_s
{
	int8_t roll_vel;
	int8_t pitch_vel;
	int8_t yaw_vel;
}auv_motion_acc_s
```

(4) rpy角度

单位0.01度，分别为俯仰角度、滚转角度和偏航角度。

```
typedef struct _auv_angular_vel_s
{
	int8_t roll;
	int8_t pitch;
	int8_t yaw;
}auv_motion_acc_s
```

#### 3.遥测参数

**【待完善】**

遥测参数将当前潜器主要的状态参数、传感器数据进行组合，得到一个数据紧凑的压缩帧进行传送。

遥测帧数据结构：

(1) 按位表示的各模块上电状态

(2) 按位表示的各模块工作状态

(3) 综合报警、错误信息

(4) 潜器工作模式信息

(5) 潜器位置、运动参数

**备注：需参考北航潜器协议“遥测帧”格式，根据需求进行设计。**

