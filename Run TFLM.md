# Detailed Guide: Run TFLM on STM32L496G-DISCO
reference:彥傑學長Run TFLM on STM32L496G-DISCO

## Prepare
1. Download [PuTTY](https://www.putty.org/)
2. Download FileZilla
3. Connect to Server
4. Clone the TensorFlow Lite Micro repository by running the following command:
```c
git clone --depth 1 https://github.com/tensorflow/tflite-micro.git
```
5. Navigate to the cloned repository:
```c
cd tflite-micro
```
## Run on MCU
### Compile as static library for Cortex-M4
1. Run the following command to compile the TensorFlow Lite Micro library for Cortex-M4:
```c
make -f tensorflow/lite/micro/tools/make/Makefile TARGET=cortex_m_generic TARGET_ARCH=cortex-m4+fp OPTIMIZED_KERNEL_DIR=cmsis_nn microlite
```
2. After the compilation is complete,
the `libtensorflow-micro.a` library will be generated in the following directory:`gen/cortex_m_generic_cortex-m4+fp_default/lib`
3. Transfer `tflite-micro` flie from the Server to your computer via FileZilla.

### Setup project on STM32CubeIDE
1. `File` > `New` > `STM32 Project` > `Board Selector`
2. Search for `STM32L496G-DISCO` in `Commercial Part Number` and select it
3. Click `Next` and fill `Project Name`
4. Select `C++` in `Targeted Language` > `Finish`
5. Click `No` when asked `Initialize all peripherals with their default mode?`
6. Inside `Pinout & Configuration` > `Connectivity` > `USART2`, change `Mode` to `Asynchrounous`
7. Press `Ctrl + S` and click `Yes` to generate code.
8. Right click on project > `Properties` > `C/C++ General` > `Paths and Symbols`, and add the following items (change to your own tflite-micro directory):
![](https://i.imgur.com/RdNaUGa.png)
![](https://i.imgur.com/7t1MLfB.png)
without `TF_LITE_STATIC_MEMORY`, compilation succeeds but TFLM will not work!!!
![](https://i.imgur.com/LkWXDCg.png)
![](https://i.imgur.com/RHyqrgq.png)
![](https://i.imgur.com/khnAnh1.png)
![](https://i.imgur.com/ejpoq2v.png)
9. Inside `Properties` > `C/C++ Build` > `Settings`, check `Use float with printf from newlib-nano (-u _printf_float)`
![](https://i.imgur.com/sHl413t.png)

### Model conversion
1. Take [hello_world_int8.tflite](https://github.com/tensorflow/tflite-micro/blob/main/tensorflow/lite/micro/examples/hello_world/models/hello_world_int8.tflite) for example, execute the following command:(on the Server)
   ```
   xxd -i tensorflow/lite/micro/examples/hello_world/models/hello_world_int8.tflite > /tmp/model.h
   ```
   then add `/tmp/model.h` to your STM32CubeIDE project
2. Transfer `model.h` from the Server to your computer via FileZilla.
3. Place `model.h` inside the `Core`>`Inc` directory

   Here is the accompanying image for clarification:

   ![image](https://github.com/Shuningchang/UART/assets/174292413/61a94e70-8d25-4bcd-bc0b-05edaacfe4e6)
4. Rename the variable name and add `const` qualifier in `model.h` like this:
   ```
   const unsigned char model_tflite[] = {
     0x28, 0x00, 0x00, 0x00, 0x54, 0x46, 0x4c, 0x33, 0x00, 0x00, 0x00, 0x00,
     ...
     0x09, 0x00, 0x00, 0x00
   };
   const unsigned int model_tflite_len = 2704;
   ```
### Inference

1. Inside `Core`> right click on `Inc` directory > `New` > `Header File` ,to create `inference.h` in your project

The content is as follows:
   ```cpp
   #ifndef INC_INFERENCE_H_
   #define INC_INFERENCE_H_

   #ifdef __cplusplus
   extern "C" {
   #endif

   #include "stm32l4xx_hal.h"
   extern UART_HandleTypeDef *huart;
   int inference(UART_HandleTypeDef *huart_);

   #ifdef __cplusplus
   }   // extern "C"
   #endif

   #endif /* INC_INFERENCE_H_ */
   ```
2. Inside `Core`> right click on `Src` directory > `New` > `Source File` ,to create `inference.cpp` in your project

The content is as follows:
```c
#include <cstdio>
#include "model.h"
#include "inference.h"
#include "tensorflow/lite/micro/cortex_m_generic/debug_log_callback.h"
#include "tensorflow/lite/micro/micro_log.h"
#include "tensorflow/lite/micro/system_setup.h"
#include "tensorflow/lite/micro/micro_interpreter.h"
#include "tensorflow/lite/micro/micro_mutable_op_resolver.h"

UART_HandleTypeDef *huart;

void debug_log_printf(const char* s) {
  HAL_UART_Transmit(huart, (uint8_t*)s, strlen(s), HAL_MAX_DELAY);
}

int inference(UART_HandleTypeDef *huart_) {
  huart = huart_;

  RegisterDebugLogCallback(debug_log_printf);
  MicroPrintf("started");

  tflite::InitializeTarget();

  const tflite::Model* model = tflite::GetModel(model_tflite);
  TFLITE_CHECK_EQ(model->version(), TFLITE_SCHEMA_VERSION);

  tflite::MicroMutableOpResolver<1> op_resolver;
  TF_LITE_ENSURE_STATUS(op_resolver.AddFullyConnected());

  constexpr int kTensorArenaSize = 3000;
  uint8_t tensor_arena[kTensorArenaSize];

  tflite::MicroInterpreter interpreter(model, op_resolver, tensor_arena, kTensorArenaSize);
  TF_LITE_ENSURE_STATUS(interpreter.AllocateTensors());

  TfLiteTensor* input = interpreter.input(0);
  TFLITE_CHECK_NE(input, nullptr);

  TfLiteTensor* output = interpreter.output(0);
  TFLITE_CHECK_NE(output, nullptr);

  float input_scale = input->params.scale;
  int input_zero_point = input->params.zero_point;
  MicroPrintf("input_scale = %f", input_scale);
  MicroPrintf("input_zero_point = %d", input_zero_point);

  float output_scale = output->params.scale;
  int output_zero_point = output->params.zero_point;
  MicroPrintf("output_scale = %f", output_scale);
  MicroPrintf("output_zero_point = %d", output_zero_point);

  // Check if the predicted output is within a small range of the
  // expected output
  float epsilon = 0.05f;
  constexpr int kNumTestValues = 4;
  float golden_inputs[kNumTestValues] = {0.77, 1.57, 2.3, 3.14};

  for (int i = 0; i < kNumTestValues; ++i) {
	input->data.int8[0] = (golden_inputs[i] / input_scale) + input_zero_point;
	MicroPrintf("input = %d", input->data.int8[0]);
	TF_LITE_ENSURE_STATUS(interpreter.Invoke());
	MicroPrintf("output = %d", output->data.int8[0]);
	float y_pred = (output->data.int8[0] - output_zero_point) * output_scale;
	MicroPrintf("y_pred = %f", y_pred);
	float y_gold = sin(golden_inputs[i]);
	MicroPrintf("y_gold = %f", y_gold);
	float difference = fabs(y_gold - y_pred);
	MicroPrintf("difference = %f", difference);
	TFLITE_CHECK_LE(difference, epsilon);
  }

  MicroPrintf("all correct");
  MicroPrintf("");

  return kTfLiteOk;
}
```
3. Inside `main.c`, add
```c
/* USER CODE BEGIN Includes */
#include "inference.h"
/* USER CODE END Includes */
```
```c
/* USER CODE BEGIN 2 */
inference(&huart2);
/* USER CODE END 2 */
```
### Build and run
Link STM32L496G-DISCO by USB and click `Run` 

The output will be printed on UART console, which can be viewed by
```
sudo screen /dev/ttyACM0 115200
```
The output should looks like this:
![](https://i.imgur.com/DaZFPwp.png)
