## User application 
    
When the User button **isn't** pressed during the RESET of MCU (while running `bootloader_stm32f446xx`). It will make a jump to this User application.    
    
We also enable the UART peripheral (USART2) in this User application `user_app_stm32f446re`, which prints to the console that the _User application is running..._. This ensures that `bootloader_stm32f446xx` passed the control to the User application. 	 
    
**PA5** `LD2 [Green Led]` and **PC13** `B1 [Blue PushButton]`	is already enabled. However we will enable the `EXTI line[15:10] interrupts` at _System Code > NVIC > NVIC_ and code generation at _System Code > NVIC > Code generation_ in CubeMX settings and finally generate the code.     
      
When **PC13** `B1 [Blue PushButton]` is pressed, let's toggle an 	**PA5** `LD2 [Green Led]` in Interrupt handler   
    
```
uint8_t flag=0;

void EXTI15_10_IRQHandler(void)
{
  HAL_GPIO_WritePin(LD2_GPIO_Port, LD2_Pin, !flag);
  flag = !flag;

  HAL_GPIO_EXTI_IRQHandler(B1_Pin);
}
```		
     
And finally print `char userdata[] = "Hello from User application \r\n"` in the `main()` function as follows:     
    
```
while (1)
{
  HAL_UART_Transmit(&huart2, (uint8_t*)userdata, sizeof(userdata), HAL_MAX_DELAY);
  uint32_t current_tick = HAL_GetTick();
  while (HAL_GetTick() <= current_tick + 1000);
}
```				 		 		 
		 
     