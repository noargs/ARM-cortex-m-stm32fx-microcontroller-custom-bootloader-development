## ARM Cortex Mx custom bootloader development for STM32Fx MCUs     
     
		 
Bootloader is small piece of code stored in the MCU flash or ROM to act as an application loader as well as a mechanism to update the applications whenever required. It is the first thing runs after microcontroller reset. It also helps to Download/Upload binaries (code)	from Host to microcontroller (IAP) when we flash a program to microcontroller.	 
    
- You sometime activate (upon MCU reset) the bootloader on some microcontrollers by changing the status of boot pins   
     
- Microcontrollers like NUCLEO/STM32 has In-Circuit Debugger/Programmer (ICDP), a circuitary, attached to them. Hence we don't take the help of bootloader rather use ICDP also known as ST link (for ST MCUs) to program (In System Programming ISP) the microcontroller. Whereas in Arduinos this circuitary is simply not there and it takes the help of bootloader to progam (IAP) the Arduino.		  	
     
		 
## STM32F446xx Memory organisation     
      
- Internal Flash memory also called Embedded Flash memory of 512kb.    
- Internal SRAM1 of 112kb.    
- Internal SRAM2 of 16kb.    
- System memory (ROM) of 30kb.    
- OTP memory of 528 bytes.     
- Option bytes memory of 16bytes.    
- Backup RAM of 4kb.				
     
		 
		 
## Internal Flash memory     
      
- Size if 512kb.    
- Begins @ 0x0800 0000    
- Ends @ 0x0807 FFFF   
- Used to store your application code and read only data of the program (i.e. Vector table)    
- Non volatile       
      
			
			
## Internal SRAM1   
      
- Size is 112kb   
- Begins @ 0x2000 0000   
- Ends @ 0x2001 BFFF   
- Used to store your application global data, static variables    
- Also used for Stack and Heap purpose    
- Volatile    
- You can also execute code from this memory     
      
			
			
## Internal SRAM2   
      
- Size is 16kb   
- Begins @ 0x2001 C000   
- Ends @ 0x2001 FFFF   
- Used to store your application global data, static variables    
- Also used for Stack and Heap purpose    
- Volatile    
- You can also execute code from this memory    
      
			
			
## System Memory (ROM)   
      
- Size is 30kb   
- Begins @ 0x1FFF C000   
- Ends @ 0x1FFF 77FF   
- All the ST MCUs store the Bootloader in this memory   
- This memory is Read only.   
- By default MCU will not execute any code from this memory but you can configure MCU to boot or execute bootloader from this memory.           
    
<img src="images/memory_organisation.png" alt="MCUs Embedded Memory Organisation" title="MCUs Embedded Memory Organisation">     


				 	
