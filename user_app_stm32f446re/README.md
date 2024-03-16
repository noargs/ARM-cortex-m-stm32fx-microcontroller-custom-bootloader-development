## User application 
    
When the User button **isn't** pressed during the RESET of MCU (while running `bootloader_stm32f446xx`). It will make a jump to this User application.    
    
We also enable the UART peripheral (USART2) in this User application `user_app_stm32f446re`, which prints to the console that the _User application is running..._. This ensures that `bootloader_stm32f446xx` passed the control to the User application. 	 
    
**PA5** `LD2 [Green Led]` and **PC13** `B1 [Blue PushButton]`	is already enabled. However we will enable the `EXTI line[15:10] interrupts` at _System Code > NVIC > NVIC_ and code generation at _System Code > NVIC > Code generation_ in CubeMX settings and finally generate the code.
     
		 		 		 
		 
     