

# IO

## IO 定义

``` C
#include "TM_BOARD.h"

#define LED0     TM_BOARD_OUTPUT_CHANNEL_NAME(0)
#define LED1     TM_BOARD_OUTPUT_CHANNEL_NAME(1)
#define LED2     TM_BOARD_OUTPUT_CHANNEL_NAME(2)
#define KEY0     TM_BOARD_INPUT_CHANNEL_NAME(0)

#define SET      1
#define RESET    0

void Init(void)
{
	TM_HAL_IO_Open(LED0);
	TM_HAL_IO_Open(KEY0);
}

void Run(void)
{
    uint8_t readValue = 0;
    
	readValue = TM_HAL_IO_Read(KEY0);
	TM_HAL_IO_Write(LED0, !!readValue);

	
	TM_HAL_IO_Write(LED1, SET);
	TM_HAL_IO_Toggle(LED2);
}
```



# ADC 


``` C
#include "TM_BOARD.h"

#define ADC1     TM_BOARD_ADC_CHANNEL_NAME(0)

void Init(void)
{
	TM_HAL_ADC_Open(ADC1);
}

void Run(void)
{
    uint16_t readValue = 0;
    
	readValue = TM_HAL_ADC_Read(ADC1);
}
```


