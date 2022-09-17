
# Exercise 3: Make a Blinky
## William Goethals
## September 11, 2022

## Code:
IDE: STM32CubeMXIDE
Board: STM32F411E-DISCO
Board2: STM32F407G-DISCO

### Relevant Code:
Description: 
- Setup: PA0 to be external interrupt on PA0; Enable the NVIC interrupt; GPIOD port for the LEDs to be outputs
- Function: When the interrupt hits, the Callback is initiated - checks that the pin that triggered the interrupt is PA0 then checks that it’s been 10 ms since the last trigger to deal with debounce.  
- Feature: Added a case statement to click through each of the 4 LEDs on the board

```C
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin){
    //check for GPIO
    static int hits = 0;
    // Only hit the interrupt lights for this pin interrupt and debounce
    currentMs = HAL_GetTick();
    if(GPIO_Pin == GPIO_PIN_0 && (currentMs - previousMs > 10) ){
        //LD 3, 4, 5, 6 are outside
        switch(hits){
        case 0:
            HAL_GPIO_TogglePin(GPIOD, LD6_Pin);
            hits++;
            break;
        case 1:
            HAL_GPIO_TogglePin(GPIOD, LD3_Pin);
            hits++;
            break;
        case 2:
            HAL_GPIO_TogglePin(GPIOD, LD4_Pin);
            hits++;
            break;
        case 3:
            HAL_GPIO_TogglePin(GPIOD, LD5_Pin);
            hits = 0;
        }
        previousMs = currentMs;
    }
}
```

### Bonus: Debounce Both High and Low
Function: 
Reads if the pin is stable for 10 ms, once it is, then checks if this is a rising or falling edge then either sets or zeros out the LED. 

```C
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin){
    currentMs = HAL_GetTick();
    if((GPIO_Pin == GPIO_PIN_0) && (currentMs - previousMs > 10) ){
        if(HAL_GPIO_ReadPin(GPIOA, GPIO_Pin)){
            // Rising
            HAL_GPIO_WritePin(GPIOD, LD6_Pin, GPIO_PIN_SET);
        }
        else{
            // Falling
            HAL_GPIO_WritePin(GPIOD, LD6_Pin, GPIO_PIN_RESET);
        }
    }
    previousMs = currentMs;
}

Also Changed:
  /*Configure GPIO pin : PA0 */
  GPIO_InitStruct.Pin = GPIO_PIN_0;
  GPIO_InitStruct.Mode = GPIO_MODE_IT_RISING_FALLING;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
```


## Investigate Further:
What are the hardware registers that cause the LED to turn on and off? (From the processor manual, don’t worry about initialization.) 
- LD3,4,5 and 6 are connected to PORTD, base address 0x4002 0C00, output is offset at 0x14, the bits for these guys are 12,13,14,15
Enabling the RCC for PORTD needs to occur first, then setting the port to output
What are the registers that you read in order to find out the state of the button?
- For the user button it’s PA0 - 0x4002 0000 input offset at 0x10, bit 0
Can you read the register directly and see the button change in a debugger or by printing out thes value of the memory at the register’s address? 
- Yes you can read the value at the register. For the debugger you can open the SFRs in the debug interface, select GPIOA and look at the IDR register to watch it change 

## Development Board Info:
### 6.3 LEDs
- LD1 COM: The LD1 default status is red. LD1 turns to green to indicate that communications are in progress between the PC and the ST-LINK/V2. 
- LD2 PWR: The red LED indicates that the board is powered. 
- User LD3: The orange LED is a user LED connected to the I/O PD13 of the STM32F411VET6. 
- User LD4: The green LED is a user LED connected to the I/O PD12 of the STM32F411VET6. 
- User LD5: The red LED is a user LED connected to the I/O PD14 of the STM32F411VET6.
- User LD6: The blue LED is a user LED connected to the I/O PD15 of the STM32F411VET6. 
- USB LD7: The green LED indicates when VBUS is present on CN5 and is connected to PA9 of the STM32F411VET6. 
- USB LD8: The red LED indicates an overcurrent from VBUS of CN5 and is connected to the I/O PD5 of the STM32F411VET6. 

### 6.4 Pushbuttons 
- B1 USER: User and Wake-Up button connected to the I/O PA0 of the STM32F411VE. 
- B2 RESET: The pushbutton connected to NRST is used to RESET the STM32F411VE.
- STM32F411VE Info:
```C
0x4002 0000 - 0x4002 03FF GPIOA
0x4002 0C00 - 0x4002 0FFF GPIOD
0x4002 3800 - 0x4002 3BFF RCC 
```
- 8.3.9 RCC AHB1 peripheral clock enable register (RCC_AHB1ENR) Address offset: 0x30
GPIODEN is bit 3, enabled with 1
- 8.4.1 GPIO port mode register (GPIOx_MODER) (x = A..E and H) Address offset: 0x00
Input is 00; 01 is output
- 8.4.6 GPIO port output data register (GPIOx_ODR) (x = A..E and H) Address offset: 0x14
Write to each bit with a 1


## References:
[Board Documentation](https://www.st.com/en/evaluation-tools/32f411ediscovery.html#documentation)
[STM32F411VE Documentation](https://www.st.com/en/microcontrollers-microprocessors/stm32f411ve.html#documentation)



