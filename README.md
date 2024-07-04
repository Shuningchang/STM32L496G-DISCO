# UART
**Setup project on STM32CubeIDE**
1. `File` > `New` > `STM32 Project` > `Board Selector`
2. Search for `STM32L496G-DISCO` in `Commercial Part Number` and select it
3. Click `Next` and fill in `Project Name`
4. Select `C` in `Targeted Language` > `Finish`

**ioc設定**  
1. Inside `Pinout & Configuration` > `Connectivity` > `USART2`, change `Mode` to `Asynchronous`
2. Inside `Pinout & Configuration` > `Pinout` > `Claer Pinouts`
3. Change the pin configuration:PA2 -> USART2_TX / PD6 -> USART2_RX
   ![image](https://github.com/Shuningchang/UART/assets/174292413/2ba8b40b-21df-42f1-8c51-a0f67a7f2f51)
4. Press `Ctrl + S` and click `Yes` to generate code.

**Inside `main.c`, add**

 ```c
 /* USER CODE BEGIN PV */
uint8_t buffer[] = "Hello, UART!";
/* USER CODE END PV */
```
```c
while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
	HAL_Delay(1000);
	HAL_UART_Transmit(&huart2, buffer, sizeof(buffer), HAL_MAX_DELAY);
  }
```
**連上 UART**
1. Link STM32L496G-DISCO by USB 
2. Download [PuTTY](https://www.putty.org/)
3. `Search` > `device manager` > `Ports (COM & LPT)` > `STMicroelectronics STLink Virtual COM Port(COMx)`
4. Inside PuTTY >  `PuTTY Configuration` > `Connection type` : Serial / `Serial line` : COMx / `Speed` : 115200 >  `Open`
![image](https://github.com/Shuningchang/UART/assets/174292413/cce9611e-015e-4942-88d2-ce61baa793e7)
5. Click `Debug` (F11) > `Run` (F8), and then the output will be printed on COMx-PuTTY.

The output should look like this:

![image](https://github.com/Shuningchang/UART/assets/174292413/86367be0-1303-45ee-b8c0-c56735d6d246)
