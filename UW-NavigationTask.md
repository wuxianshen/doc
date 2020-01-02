# Navigation Task
## 1. Introduction

Navigation task is conducting in 1052-0, with abbreviation of "INS Task".

This markdown records how to get navigation data in IMX6 from 1052-0.


## 2. Hardware Setup

**Note: These serial ports are occupied by INS Task, so they cannot be used for other use.**

IMU is required for INS tasks results update.

### 2.1 IMU

(1) IMU Electrical Connection

PA-Serial Port 13(labeled), real Port 12 in codes

PB-Null

PS-Serial Port 3(labeled), real Port 2 in codes

(2) IMU Power Supply

Power lines (+/-) connect to imx6 relay 0

Power on command: send deck command with module id e_imu_packet and function id e_imu_power_on;

Power off command: send deck command with module id e_imu_packet and function id e_imu_power_off;

Power state query command: send deck command with module id e_imu_packet and function id e_imu_power_query, reply like:

### 2.2 DVL

DVL- Serial Port 3(labeled), real Port 2 in codes

### 2.3 GPS

GPS-Serial Port 2(labeled), real Port 1 in codes



## 3. Software Setup

### 3.1 Environment Checkup

#### 1. INS firmware version

How to figure out the firmware version of INS task in 1052-0?

Which version is the right version?

**<font color=red>To be done</font>**



#### 2. 



### 3.2 Task Setup

#### 1. Data Definition for INS task

The following codes define the data objects required by the INS task.

```c
//User data define
DT_UINT8 ui8_cnt_frame = 0x0;
DT_UINT8 ui8_cnt_frame_10ms = 0x0;

DT_SCHAR8 buf_des[0x400];
DT_SCHAR8 ins_data[400] = {0};
DT_SCHAR8 ins_data_process[400] = {0};

DT_SCHAR8 path_original_gps[40];
DT_SCHAR8 path_original_dvl[40];
DT_SCHAR8 path_original_dep[40];

DT_SCHAR8 path_recv_data[40];
DT_SCHAR8 path_recv_data_uc[40];
DT_SCHAR8 path_recv_data_pc[40];

FILE* file_original_gps = NULL;
FILE* file_original_dvl = NULL;
FILE* file_original_dep = NULL;

FILE* file_recv_data = NULL;
FILE* file_recv_data_uc = NULL;
FILE* file_recv_data_pc= NULL;
FILE* file_recv_data_pc_all= NULL;


int8_t uw_imx6_data_config()
{
	strcpy(path_original_gps, filepath_save);
	strcpy(path_original_dvl, filepath_save);
	strcpy(path_original_dep, filepath_save);
	strcpy(path_recv_data, filepath_save);
	strcpy(path_recv_data_uc, filepath_save);
	strcpy(path_recv_data_pc, filepath_save);

	strcat(path_original_gps, "original_gps.txt");
	strcat(path_original_dvl, "original_dvl.txt");
	strcat(path_original_dep, "original_dep.txt");
	strcat(path_recv_data, "recv_data.txt");
	strcat(path_recv_data_uc, "recv_data_uc.txt");
	strcat(path_recv_data_pc, "recv_data_pc.txt");

	if((file_original_gps = fopen(path_original_gps, "wb")) == NULL)
	{
		log_e("Cannot open file %s", path_original_gps);
	}

	if((file_original_dvl = fopen(path_original_dvl, "wb")) == NULL)
	{
		log_e("Cannot open file %s", path_original_dvl);
	}

	if((file_original_dep = fopen(path_original_dep, "wb")) == NULL)
	{
		log_e("Cannot open file %s", path_original_dep);
	}

	if((file_recv_data = fopen(path_recv_data, "wb")) == NULL)
	{
		log_e("Cannot open file %s", path_recv_data);
	}
	if((file_recv_data_uc = fopen(path_recv_data_uc, "wb")) == NULL)
	{
		log_e("Cannot open file %s", path_recv_data_uc);
	};
	if((file_recv_data_pc = fopen(path_recv_data_pc, "wb")) == NULL)
	{
		log_e("Cannot open file %s", path_recv_data_pc);
	}

    return 0;
}
```



#### 2. Interruption function

Define a 400Hz interrupt callback for data update checking.

```c
DT_SINT32 INTER_CALLBACK_400Hz(void)  //400Hz Interrupter
{
	ui8_cnt_frame++;
	if(0 == ui8_cnt_frame % 2)
	{
		ui8_cnt_frame_10ms++;
	}

#ifdef USE_ARM
    extern DT_VOID RT2IMXDPRAM(DT_VOID);

	RT2IMXDPRAM();
#endif

    memcpy(ins_data_process, ins_data, sizeof(ins_data));

    //Process INS data
    //Todo
    static uint64_t count = 0;
    //if ( count % 100 == 0 )
    {
        log_i("%x %x %x %x", ins_data_process[0], ins_data_process[1], ins_data_process[2], ins_data_process[3]);
    }
    count++;

	return OK;
}
```

Register this interrupt function.

```c
int32_t imx6_mandatory_config()
{
#ifdef USE_ARM
    isrCallbackFunReg (2, INTER_CALLBACK_400Hz);
#endif
    return 0;
}
```



#### 3. Init INS Task

Call the initial function of INS task.

```c
//Navigation task with IMU
ret = INS_Task_Init();
if ( ret != 0 )
{
    log_e("UW hardware process INS_Task_Init failed %d", ret);
    return ret;
}
log_i("[Hardware] UW INS Task init ok!");
```



## 4. Data Parsing

### 4.1 Data format example

**<font color=red>To be done</font>**



### 4.2 Data format parsing

Meaning of different segments.

**<font color=red>To be done</font>**



### 4.3 Log file

How to parse the log file?

**<font color=red>To be done</font>**



##  Note



