### FreeRTOS-Notes

1.数据类型在portmacro.h中进行宏定义。
```
#define portCHAR		char
#define portFLOAT		float
#define portDOUBLE		double
#define portLONG		long
#define portSHORT		short
#define portSTACK_TYPE	uint32_t
#define portBASE_TYPE	long

typedef portSTACK_TYPE StackType_t;
typedef long BaseType_t;
typedef unsigned long UBaseType_t;

```


### 内存申请
```
//支持动态内存申请
#define configSUPPORT_DYNAMIC_ALLOCATION        1    
//支持静态内存
#define configSUPPORT_STATIC_ALLOCATION			1	

```
使用静态内存的方法需要用户预先定义，在实际开发中使用较少，主要使用动态内存分配的方法。动态内存申请的宏定义在FreeRTOS.h里面是默认开启的，可以不用再在FreeRTOSConfig.h中去开启。一些所需功能的宏定义可以在FreeRTOS.h中找到，1是开启，0是关闭。




### 任务管理
FreeRTOS中的任务调度器是基于优先级的全抢占式调度：除了中断处理函数、调度器上锁部分的代码和禁止中断的代码是不可抢占的之外，其他都是可以抢占的。**优先级数值越大，任务优先级越高，0为最低优先级（分配给空闲任务使用）。** 而对于相同优先级的任务，则采用**时间片轮转方式（分时调度器）** 

#### 任务状态
系统中每一个任务都有多种运行状态，系统初始化完成后，创建的任务就可以在系统中竞争一定的资源，由内核进行调度。
1. 就绪（ready）态
    新创建的任务初始化为就绪态。
2. 运行（running）态
    任务开始运行的那一刻，任务状态就转变为运行态。
3. 阻塞（blocked）态
   如果当前任务正在等待某个时序或外部中断，我们就说这个任务处于阻塞态，任务不在就绪列表中。包含任务挂起、任务延时、任务正在等待信号量，读写队列或者等待读写事件等。
4. 挂起（suspended）态
   处于挂起态的任务对调度器不可见，让任务进入挂起态的唯一方法就是调用vTaskSuspend()函数；而取消挂起态，恢复任务的唯一途径是调用vTaskResume()或vTaskResumeFromISR()函数。

挂起与阻塞的区别：**挂起时，调度器不会理会被挂起任务的任何信息。而当阻塞时，系统还需要判断阻塞态任务是否超时，是否可以解除阻塞。**


#### 常用任务函数

##### 任务挂起与恢复函数
启用： 
INCLUDE_vTaskSuspend                                     定义为1
INCLUDE_vTaskResumeFromISR（调用vTaskResumeFromISR时）    定义为1     
**1.vTaskSuspend()**
用于挂起指定任务，当要挂起自身时，只要将任务句柄设置为NULL，作为参数传递进去即可。

**恢复函数**
恢复一个挂起态的任务唯一途径是调用vTaskResume()或者vTaskResumeFromISR()函数。vTaskResumeFromISR()专门用在中断服务程序中。
想要调用vTaskSuspend()，则必须将宏定义**INCLUDE_vTaskSuspend**配置为**1**。想使用**vTaskResumeFromISR()**函数还需要将**INCLUDE_vTaskResumeFromISR**也定义为**1**。

**2.vTaskSuspendAll()**
可将所有任务挂起（本质上就是挂起任务调度器，中断还是可以被启用）。

**全部恢复**
恢复调度器则可以调用xTaskResumeAll()。调用了多少次vTaskSuspendAll()就要调用多少次xTaskResumeAll()进行恢复。


##### 任务删除函数
**vTaskDelete()**
启用：INCLUDE_vTaskDelete 定义为1
形参为要删除的任务创建时返回的**任务句柄**，如果是删除自身，则形参为**NULL**。被删除的任务将从所有状态和事件列表中删除。

##### 任务延时函数
**相对延时函数vTaskDelay()**

启用：
INCLUDE_vTaskDelay 定义为1
该函数的延时单位为系统节拍周期，可以理解为systick。该函数的延时时间是相对的、其他任务和中断活动会影响到vTaskDelay()的调用，影响到任务下一次执行的时间。

**绝对延时函数vTaskDelayUntil()**
启用：
INCLUDE_vTaskDelayUntil 定义为1
常用于较精确的周期运行任务。



***





#### 常见问题
1. Error: L6218E: Undefined symbol vApplicationGetIdleTaskMemory (referred from tasks.o).
   在FreeRTOSConfig.h中打开了支持静态内存的定义：configSUPPORT_STATIC_ALLOCATION
