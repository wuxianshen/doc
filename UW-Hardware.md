# UW-Hardware

## 1. Modules

| Module      | Port | Connection                             | Comment                        |
| ----------- | ---- | -------------------------------------- | ------------------------------ |
| IMU         | PA   | [X2] Serial Port 13 (label), 12 (Code) | Data (Baud: 406800) <br> RS422 |
|             | PB   | NULL                                   |                                |
|             | PS   | [X2] Serial Port 3 (label), 2 (Code)   | Trigger<br>RS422               |
| GPS         | Data | [X1] Serial Port 2 (label), 1 (Code)   | Baud: 115200<br>RS232          |
| DVL         | Data | [X1] Serial Port 3 (label), 2 (Code)   | Baud: 19200<br>RS232           |
| DAM         | Data | [X1] Serial Port 8 (label), 7 (Code)   | Baud: 19200<br/>RS232          |
| Radio       | Data | [X1] Serial Port 7 (label), 6 (Code)   | Baud: 19200<br>RS232           |
| Iridium     | Data | [X1] Serial Port 11 (label), 10 (Code) | Baud: 19200<br/>RS232          |
| Acoustic    | Data | [X1] Serial Port 12 (label), 11 (Code) | Baud: 19200<br/>RS232          |
| Depthometer |      |                                        |                                |
| Humidometer |      |                                        |                                |



## 2. Installation

### 2.1 IMU

(1) Power supply



(2) Data Connection

RS232 - 2/3/5



## 3. 主控器继电器对应外设列表

红蓝黑及备注都为接线细节。

| X5接口 | 外设       | 红   | 蓝   | 黑   | 备注         |
| ------ | ---------- | ---- | ---- | ---- | ------------ |
| 0      | 深度计     | 5    | 4    | 10   | 蓝黑加2Pin针 |
| 1      | 前温盐深   | 60   | 59   | 54   | 蓝黑加2Pin针 |
| 2      | 声通讯     | 12   | 19   | 20   |              |
| 3      | 水听器     | 16   | 23   | 31   | 黑加一根线   |
| 4      | 无         |      |      |      |              |
| 5      | DVL        | 35   | 26   | 34   |              |
| 6      | 抛载       | 43   | 50   | 42   |              |
| 7      | 后温盐深   | 56   | 61   | 55   |              |
| 8      | 无         |      |      |      |              |
| 9      | IMU        | 18   | 11   | 17   | 蓝黑加2Pin针 |
| 10     | DAM板      | 3    | 2    | 8    |              |
| 11     | 无         |      |      |      |              |
| 12     | 无线电台   | 1    | 6    | 7    |              |
| 13     | 前关节电机 |      |      |      |              |
| 14     | 后关节电机 |      |      |      |              |



## 4.外设串口通讯接口列表

标号为线束上的标号

| X1(母头) | 外设     | 2(Tx) | 3(Rx) | 5(GND) |
| -------- | -------- | ----- | ----- | ------ |
| 0        | 声通讯   | 20    | 27    | 22     |
| 1        | DVL      | 6     | 13    | 21     |
| 2        | 无       | 8     | 1     | 5      |
| 3        | 前温盐深 | 16    | 3     | 15     |
| 4        | 后温盐深 | 23    | 10    | 25     |
| 5        | 铱星     | 39    | 44    | 41     |
| 6        | GPS      | 57    | 38    | 56     |
| 7        | 电池     | 59    | 53    | 51     |
| 8        | 无线电台 | 32    | 18    | 31     |
| 9        | DAM      | 43    | 35    | 36     |
| 10       | 湿度计   | 47    | 55    | 46     |
| 11       | 无       | 29    | 49    | 30     |

