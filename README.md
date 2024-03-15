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
    
		
		
## Reset sequence of the Cortex M Processor      
     
- When you reset the processor (press reset button on your microcontroller), The PC (program counter register) is first loaded with the value 0x0000_0000
- Then processor read the _value_ @ memory location 0x0000_0000 in to the MSP (Main Stack Pointer register)
- That means, processor first initialises the Stack pointer
- After that processor read the value @ memory location 0x0000_0004 in to the PC, that value actually contains the address of reset handler and PC jumps to the reset handler
- A reset handler is just a C or assembly function written by you to carry out any initialisations required
- From reset handler you call your main() function of the application		 		         
    
<img src="images/reset_sequence.png" alt="Reset sequence of the Cortex M Processor" title="Reset sequence of the Cortex M Processor">   		       
    
		
		
## Memory aliasing      
       
<img src="images/memory_aliasing.png" alt="Memory aliasing" title="Memory aliasing">   		         
      
Memory aliasing is a technique used by the microcontroller manufacturer to map different addresses onto different other addresses.   
    
<img src="images/memory_aliasing2.png" alt="Memory aliasing" title="Memory aliasing">   
     
For example, in the image shown above, by default the base address of the user flash 0x0800_0000 is mapped onto the base address of the memory map 0x0000_0000. Hence, if you read 0th address 0x0000_0000 then that address somehow converted to the 0x0800_0000 address.   
     
Similarly, if you read 0x0000_0004 address then you will end up reading 0x0800_0000. That's because, memory region 0x0800_xxxx is mapped onto 0x0000_xxxx memory regions, Thats what we call Memory aliasing.           
    
		
		
## Boot configuration of STM32F446xx      
     
All the manufacturer need not to forward the 0th address 0x0000_0000 to the base address of user flash 0x0800_0000. Hence, it's manufacturer dependent.		 
      
       
As shown in the below image, There are 2 pins in the STs microcontrollers (Boot 1 pin and Boot 0 pin), and using those pins, you can control where exactly the processor has to fetch instructions during reset.    
       
<img src="images/boot_config_stm32f446xx.png" alt="Table 2. Boot modes" title="Table 2. Boot modes">     
      
Let's say, you place some code in the embedded SRAM and after reset if you want your processor to execute instructions from embedded SRAM, then during reset the Boot 1 and the Boot 0 pins has to be 1.		
     
In the case of TIs microcontroller, (Datasheet page 92) It says the _On-chip Flash_ starts at 0x0000_0000 whereas in ST it start from 0x0800_0000. Hence memory aliasing is not required for TIs microcontrollers. As the flash is already mapped at 0th address.  		 
       
<img src="images/ti_datasheet.png" alt="TIs Datasheet page 92 Table 2-4" title="TIs Datasheet page 92 Table 2-4">     
     
## Activating bootloader		    
     
Going back to _Table 2. Boot modes_, in order to access the **System memory** we have to keep the BOOT1 0 and BOOT0 at 1. When we look into the schematic and search for both the pins, then we will find that **BOOT0** is connected to **pin 7** of the NUCLEO (**This is same across all the NUCLEOs**), You can short it with pin 5 (VDD) **OR** 16 (3.3v) as shown below. Similar details can be found in the User Manual of the NUCLEO board _page 53_. 		 
       
<img src="images/boot0.png" alt="BOOT0 to VDD or 3.3v" title="BOOT0 to VDD or 3.3v">     
     
Similarly, **BOOT1** has to be 0 (GND)		 
     
<img src="images/boot1.png" alt="BOOT1 (PB2) to 0 (GND)" title="BOOT1 (PB2) to 0 (GND)">   		 
          
     
     
	 		 		   
     
	 		 		   
     
	 		 		   
    
		


				 	
