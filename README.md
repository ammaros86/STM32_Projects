# STM32 FreeRTOS UART Communication
This project demonstrates UART communication on an STM32 microcontroller with FreeRTOS integration. It is designed to receive data via UART in DMA mode and process the received packets in a FreeRTOS task. The main functionality includes packet validation and acknowledgment via UART.

# Project Overview
The project is based on the STM32 HAL and FreeRTOS. It configures UART for communication and uses DMA for efficient data reception. A task running in FreeRTOS processes the incoming data and sends an acknowledgment back to the sender.

# Key Features:
- UART communication using DMA for efficient data transfer.
- FreeRTOS task that handles data reception and validation.
- Packet checksum validation to ensure data integrity.
- Acknowledgment of received packets.

# Code Explanation
UART Initialization
````
static void MX_USART3_UART_Init(void)
{
  huart3.Instance = USART3;
  huart3.Init.BaudRate = 115200;
  huart3.Init.WordLength = UART_WORDLENGTH_8B;
  huart3.Init.StopBits = UART_STOPBITS_1;
  huart3.Init.Parity = UART_PARITY_NONE;
  huart3.Init.Mode = UART_MODE_TX_RX;
  huart3.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart3.Init.OverSampling = UART_OVERSAMPLING_16;
  huart3.Init.OneBitSampling = UART_ONE_BIT_SAMPLE_DISABLE;
  huart3.Init.ClockPrescaler = UART_PRESCALER_DIV1;
  huart3.AdvancedInit.AdvFeatureInit = UART_ADVFEATURE_NO_INIT;
  if (HAL_UART_Init(&huart3) != HAL_OK)
  {
    Error_Handler();
  }
  HAL_UART_Receive_DMA(&huart3, rxData, 10);
}
`````

This function initializes the UART interface (USART3) with the following parameters:

- Baud rate: 115200
- 8-bit data frame
- 1 stop bit
- No parity
- No flow control
  
It also enables UART in receive mode using DMA, allowing efficient data reception.

# DMA UART Receive Callback

````
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
  countFullRcv++;
  HAL_UART_Receive_DMA(huart, rxData, 10);
}
````


This callback function is triggered when DMA completes receiving data. It increments the countFullRcv variable, indicating a new packet has been received. It then reinitiates the DMA receive operation for the next 10 bytes.

# FreeRTOS Task for Packet Validation and Acknowledgment

````
void StartDefaultTask(void const * argument)
{
  for(;;)
  {
    int packet_validation = isPacketValide();
    if (packet_validation >= 0)
    {
      send_ack(packet_validation);
    }
    osDelay(1);
  }
}
````

This FreeRTOS task continuously checks if a packet is valid by calling the isPacketValide() function. If a valid packet is received, it sends an acknowledgment back to the sender via UART.

# Packet Validation
````
int isPacketValide()
{
  uint8_t checksum = 0;
  int data_length = 5;
  if (countFullRcv != 0 && countFullRcv != countLastReceive)
  {
    countLastReceive = countFullRcv;
    memcpy(packetCopy, rxData, PACKET_SIZE);
    for(int i = 0; i < data_length; i++)
    {
      checksum ^= packetCopy[i];
    }
    if (checksum == packetCopy[data_length])
    {
      return 1;  // Valid packet
    }
    return 0;  // Invalid packet
  }
  else
  {
    return -1;  // No new packet
  }
}
````
This function checks the validity of the received packet by calculating a checksum (XOR). If the checksum matches the expected value, the packet is considered valid.

# Sending Acknowledgment
````
void send_ack(int ack)
{
  char ackPacket[50];
  if (ack)
  {
    snprintf(ackPacket, sizeof(ackPacket), "received: %d \r\n", countLastReceive);
  }
  else
  {
    snprintf(ackPacket, sizeof(ackPacket), "invalid data: %d \r\n", countLastReceive);
  }
  HAL_UART_Transmit(&huart3, (uint8_t*)ackPacket, strlen(ackPacket), HAL_MAX_DELAY);
}
````
This function sends an acknowledgment message over UART, indicating whether the received packet was valid or invalid.




