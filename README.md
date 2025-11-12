| 参数              | STM32F407（PCLK1=42MHz） | STM32F103（PCLK1=36MHz） | 说明              |
| --------------- | ---------------------- | ---------------------- | --------------- |
| Prescaler (BRP) | 14                     | 12                     | 不同主频下调整以得到相同 Tq |
| Tq              | 333.3 ns               | 333.3 ns               | ✅ 一样            |
| BS1             | 4                      | 4                      | ✅ 一样            |
| BS2             | 1                      | 1                      | ✅ 一样            |
| SJW             | 1                      | 1                      | ✅ 一样            |
| Bit time        | 6 × 333.3 ns = 2 µs    | 6 × 333.3 ns = 2 µs    | ✅ 一样            |
| 波特率             | 500 kbps               | 500 kbps               | ✅ 一样            |

#在配置波特率的时候只有BRP可以改变其他的必须一样

#下面是接收函数

void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)
           
can.CAN_RxMsg = rxHeader;      //can数据帧的帧头
can.rxFrameFlag = true;        //接收完成标志位

#下面是发送函数

void can_SendCmd(__IO uint8_t *cmd, uint8_t len)


下面是can滤波器
void USER_CAN1_Filter_Init(void)

