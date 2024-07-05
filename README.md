# STM32L496G-DISCO
### STM32 Functions
**HAL_Delay()**
1. Unit: ms
2. Specifies the length of the delay time

**GPIO(General-purpose input/output)**
1. A shorthand for general-purpose input/output, allowing users to perform input and output operations on the pins.
2. `filename.ioc`> `Pinout&Configuration` > PB13->GPIO_Output

**HAL_GPIO_WritePin**
```c
HAL_GPIO_WritePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_PIN, GPIO_PinState PinState);
```
1. The first parameter is GPIOx, which stands for "GPIO" + the pin name in English.
2. The second parameter is GPIO_PIN_x, which stands for "GPIO_PIN_" + the pin number.
3. The last parameter:
   
    If you want to output HIGH, use GPIO_PIN_SET.
   
    If you want to output LOW, use GPIO_PIN_RESET.
   
4. The LED blinks at a frequency of 1Hz by running the following command:
```c
/* USER CODE BEGIN WHILE */
while (1) {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_13, GPIO_PIN_SET);
    HAL_Delay(500);
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    HAL_Delay(500);
}
/* USER CODE END 3 */
```

**HAL_GPIO_TogglePin**
```c
HAL_GPIO_TogglePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState);
```
1. It will output based on your current state, each time outputting the opposite state of the current one.Therefore, we can further simplify our code.
   For example, if the current state is HIGH, the next output will be LOW.
2.  The LED blinks at a frequency of 1Hz by running the following command:
```c
/* USER CODE BEGIN WHILE */
while (1) {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
    HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_13);
    HAL_Delay(500);
}
/* USER CODE END 3 */
```

**HAL_GPIO_ReadPin**
```c
HAL_GPIO_ReadPin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
```
1. It can read whether the current input is high or low voltage.

### Tools
**Live Expression**
1. `Window` > `Short View` > `Live Expression`
2. In debug mode
3. A small window for variable monitoring: allows you to "Add New Expression" to get the memory location (&var) and value (var) of a variable
4. Can only monitor global variables (variables not enclosed in any braces)

**Memory**
1. `Window` > `Short View`> `Memory`
2. In debug mode
3. Adjust as needed:`New Renderings` > `Signed Integer`
4. Can change or view the value at the memory location

