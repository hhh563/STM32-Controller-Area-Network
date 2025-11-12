| 参数              | STM32F407（PCLK1=42MHz） | STM32F103（PCLK1=36MHz） | 说明              |
| --------------- | ---------------------- | ---------------------- | --------------- |
| Prescaler (BRP) | 14                     | 12                     | 不同主频下调整以得到相同 Tq |
| Tq              | 333.3 ns               | 333.3 ns               | ✅ 一样            |
| BS1             | 4                      | 4                      | ✅ 一样            |
| BS2             | 1                      | 1                      | ✅ 一样            |
| SJW             | 1                      | 1                      | ✅ 一样            |
| Bit time        | 6 × 333.3 ns = 2 µs    | 6 × 333.3 ns = 2 µs    | ✅ 一样            |
| 波特率             | 500 kbps               | 500 kbps               | ✅ 一样            |

在配置波特率的时候只有BRP可以改变其他的必须一样

下面是接收
void HAL_CAN_RxFifo0MsgPendingCallback(CAN_HandleTypeDef *hcan)
{
    if (hcan->Instance == CAN1)
    {
        CAN_RxHeaderTypeDef rxHeader;
        uint8_t rxData[8] = {0};

        if (HAL_CAN_GetRxMessage(hcan, CAN_RX_FIFO0, &rxHeader, rxData) == HAL_OK)
        {
            // 保证数据长度有效
            for (uint8_t i = rxHeader.DLC; i < 8; i++)
                rxData[i] = 0;

            // 将数据复制到全局缓冲区
            memcpy(can.rxData, rxData, 8);
            can.CAN_RxMsg = rxHeader;      //can数据帧的帧头
            can.rxFrameFlag = true;        //接收完成标志位
        }
        else
        {
            printf("CAN RX Error!\r\n");
        }
    }
}


下面是发送
void can_SendCmd(__IO uint8_t *cmd, uint8_t len)
{
	static uint32_t TxMailbox; __IO uint8_t i = 0, j = 0, k = 0, l = 0, packNum = 0;

	// 除去ID地址和功能码后的数据长度
	j = len - 2;

	// 发送数据
	while(i < j)
	{
		// 数据个数
		k = j - i;

		// 填充缓存
		can.CAN_TxMsg.StdId = 0x00;
		can.CAN_TxMsg.ExtId = ((uint32_t)cmd[0] << 8) | (uint32_t)packNum;
		can.txData[0] = cmd[1];
		can.CAN_TxMsg.IDE = CAN_ID_EXT;
		can.CAN_TxMsg.RTR = CAN_RTR_DATA;

		// 小于8字节命令
		if(k < 8)
		{
			for(l=0; l < k; l++,i++) { can.txData[l + 1] = cmd[i + 2]; } can.CAN_TxMsg.DLC = k + 1;
		}
		// 大于8字节命令，分包发送，每包数据最多发送8个字节
		else
		{
			for(l=0; l < 7; l++,i++) { can.txData[l + 1] = cmd[i + 2]; } can.CAN_TxMsg.DLC = 8;
		}

		// 发送数据
		while(HAL_CAN_AddTxMessage((&hcan1), (CAN_TxHeaderTypeDef *)(&can.CAN_TxMsg), (uint8_t *)(&can.txData), (&TxMailbox)) != HAL_OK);

		// 记录发送的第几包的数据
		++packNum;
	}
}

下面是can滤波器
void USER_CAN1_Filter_Init(void)
{
    CAN_FilterTypeDef sFilterConfig;
    //全接收，不做任何特殊处理
    sFilterConfig.FilterBank = 0;
    sFilterConfig.FilterMode = CAN_FILTERMODE_IDMASK;
    sFilterConfig.FilterScale = CAN_FILTERSCALE_32BIT;
    sFilterConfig.FilterIdHigh = 0x0000;
    sFilterConfig.FilterIdLow = 0x0000;
    sFilterConfig.FilterMaskIdHigh = 0x0000;
    sFilterConfig.FilterMaskIdLow = 0x0000;
    sFilterConfig.FilterFIFOAssignment = CAN_RX_FIFO0;
    sFilterConfig.FilterActivation = ENABLE;

    if (HAL_CAN_ConfigFilter(&hcan1, &sFilterConfig) != HAL_OK)
    {
        printf("CAN Filter config failed!\r\n");
        Error_Handler();
    }
}
