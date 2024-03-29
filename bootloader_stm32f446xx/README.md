## USART Modules       

    
* _UART2_ of STM32's peripheral of the microcontroller to receive the commands which is sent by the Host, as well as replies that bootloader sends to the PC `#define C_UART     &huart2` (We make use of the virtual COM port, **PA2** and **PA3** pins of NUCLEO) (to use with Desktop Application to get the Debug messages). [Nucleo F466re User Guide: page 25, 6.8 USART communication](https://www.st.com/resource/en/user_manual/um1724-stm32-nucleo64-boards-mb1136-stmicroelectronics.pdf)     
    
* _UART3_ of STM32's peripheral as a Debug port to get the debug prints `#define D_UART     &huart3` from the bootloader. As, in the early development stage we can put some print statements `printmsg()` in our code to see what is happening under the hood when different functions is being called (Unidirectional flow from Bootloader to PC only).	Off course we need the USB to UART converter (USB-TTL) hardware to push the data to PC from the Nucleo board. (**During custom bootloader development we will only use USART2**) 		   
     
<img src="images/usb_ttl_usart3_connection.png" alt="USB to TTL, USART3 connection with NUCLEO-F446RE" title="USB to TTL, USART3 connection with NUCLEO-F446RE">   	
     
```c
// prints formatted string to console over UART
void printmsg(char* format, ...)
{
#ifdef BL_DEBUG_MSG_EN

  char str[80];

  // extract the argument list using VA apis
  va_list args;
  va_start(args, format);
  vsprintf(str, format, args);
  HAL_UART_Transmit(D_UART, (uint8_t*)str, strlen(str), HAL_MAX_DELAY);
  va_end(args);

#endif
}
```		 
     
     
## Bootloader code flow chart     
     
<img src="images/flow_chart_usart.png" alt="Bootloader code flow chart" title="Bootloader code flow chart"> 
<img src="images/flow_chart_usart2.png" alt="Bootloader code flow chart" title="Bootloader code flow chart">
<img src="images/flow_chart_usart3.png" alt="Bootloader code flow chart" title="Bootloader code flow chart"> 
     
When the User button **isn't** pressed during RESET of MCU, the `bootloader_jump_to_user_app()` will be called and it will make a jump to User application (project name `user_app_stm32f446re`).  		
     
		 
### bootloader_jump_to_user_app(void)    
    
```c
/**
 * Jump to User application, Here we are assuming
 * FLASH_SECTOR2_BASE_ADDRESS is where the User
 * application stored.
 */
void bootloader_jump_to_user_app(void)
{
  // function pointer to hold the address of Reset Handler of User app.
  void (*app_reset_handler)(void);

  printmsg("BL_DEBUG_MSG: bootloader_jump_to_user_app\n");

  //1. configure the MSP by reading the value from the base address of Sector 2
  uint32_t msp_value = *(volatile uint32_t*)FLASH_SECTOR2_BASE_ADDRESS;
  printmsg("BL_DEBUG_MSG: MSP value: %#x\n", msp_value);

  // CMSIS function
  __set_MSP(msp_value);

//  SCB->VTOR = FLASH_SECTOR1_BASE_ADDRESS;

  //2. Now fetch the reset handler address of the User application
  //   from the location FLASH_SECTOR2_BASE_ADDRESS+4
  uint32_t resethandler_address = *(volatile uint32_t*)(FLASH_SECTOR2_BASE_ADDRESS + 4);
  app_reset_handler = (void*)resethandler_address;

  printmsg("BL_DEBUG_MSG: app reset handler address: %#x\n", app_reset_handler);

  //3. jump to reset handler of the user application
  app_reset_handler();
}
```				 
     
     
## Vector table relocation feature (ARM Cortex Mx processor)    
    
Let's discuss about the vector table relocation feature of the ARM Cortex Mx Processor.    
    
We have Flash memory with starting address 0x0800_0000 and 2 of its Sectors contains Bootloader along with its Vector table. We have another Vector table for User app starting from 0x0800_8000 as well as User app.		
    
Now, once the bootloader runs (on MCU Reset), and if the User button isn't pressed during power up, then bootloader will give control to the User application (By calling Reset Handler of User application). That's how User application gets triggered.				  
     
<img src="images/vector_tables.png" alt="Flash memory and different components" title="Flash memory and different components">     
    
Once User application starts running and User application wants to handle any interrupt, as ARM Cortex Mx processor by default look for Vector table at 0x0000_0000 (Memory alias to 0x0800_0000). However, There's no  Vector table for User application, rather a Vector table for Bootloader.	
     
		 
**VTOR**      
     
Hence, at the start of User application you have to inform the ARM Controller that Vector table of User application is at 0x0800_8000 using one of the register of the ARM controller **VTOR (Vector Table Relocation Register)** and VTOR has value of 0 by default. You suppose to change the content of this register (put base address of sector 2 i.e. 0x0800_8000) when the control jump to the User application in the Reset Handler.	
		 
		 
		 
### bootloader_uart_read_data(void)    
     
We have Rx buffer `uint8_t bl_rx_buffer[200]` of 200 bytes to keep the receive data. First element of `bl_rx_buffer[0]` will always be the **length** information to follow and Second element bl_rx_buffer[1] will be the **command code** to try in `swtich(bl_rx_buffer[1])` to execute right function handle.

```c
void bootloader_uart_read_data(void)
{
  uint8_t rcv_len = 0;

  while (1)
  {
	memset(bl_rx_buffer, 0, 200);

	// here we will read and decode the commands coming from Host
	//
	// first read only one byte from the host, which is the "length"
	// field of the command packet
	HAL_UART_Receive(C_UART, bl_rx_buffer, 1, HAL_MAX_DELAY);
	
	rcv_len = bl_rx_buffer[0];
	
	HAL_UART_Receive(C_UART, &bl_rx_buffer[1], rcv_len, HAL_MAX_DELAY);

	switch (bl_rx_buffer[1])
	{
	case BL_GET_VER:
	  bootloader_handle_getver_cmd(bl_rx_buffer);
	  break;
	case BL_GET_HELP:
	  bootloader_handle_gethelp_cmd(bl_rx_buffer);
	  break;
	case BL_GET_CID:
	  bootloader_handle_getcid_cmd(bl_rx_buffer);
	  break;
	
	...
	
	case BL_OTP_READ:
		bootloader_handle_read_otp(bl_rx_buffer);
		break;
	default:
	  printmsg("BL_DEBUG_MSG: Invalid command code received from host\n");
		break;
	}
 }
}	
```    
     
		 
     
## Command: BL_GET_VER		 
      
<img src="images/bl_get_ver.png" alt="Command BL_GET_VER" title="Command BL_GET_VER"> 	
    
```c
void bootloader_handle_getver_cmd(uint8_t* bl_rx_buffer)
{
  uint8_t bl_version;

  //1. verify the checksum
  printmsg("BL_DEBUG_MSG: bootloader_handle_getver_cmd\n");

  // total length of the command packet
  uint32_t command_packet_len = bl_rx_buffer[0]+1;

  // extract the CRC32 send by the Host
  uint32_t host_crc = *((uint32_t*)(bl_rx_buffer + command_packet_len-4));

  if (!bootloader_verify_crc(&bl_rx_buffer[0], command_packet_len-4, host_crc))
  {
	printmsg("BL_DEBUG_MSG: checksum success !!\n");

	// checksum is correct (send ACK)
	bootloader_send_ack(bl_rx_buffer[0], 1);
	bl_version = get_bootloader_version();
	printmsg("BL_DEBUG_MSG: BL_VER: %d %x#x\n", bl_version, bl_version);
	bootloader_uart_write_data(&bl_version, 1);
  }
  else
  {
	printmsg("BL_DEBUG_MSG: checksum fail !!\n");
	// checksum is wrong (send NACK)
	bootloader_send_nack();
}  
```					
    
## Command: BL_GET_HELP     
     
<img src="images/bl_get_help.png" alt="Command BL_GET_HELP" title="Command BL_GET_HELP"> 			
     
```c
uint8_t supported_commands[] = {BL_GET_VER, BL_GET_HELP, BL_GET_CID, .. };
								
void bootloader_handle_gethelp_cmd(uint8_t* bl_rx_buffer)
{
  printmsg("BL_DEBUG_MSG: bootloader_handle_gethelp_cmd\n");
  ...
  if (!bootloader_verify_crc(&bl_rx_buffer[0], command_packet_len-4, host_crc))
  {
    ...
	  bootloader_send_ack(bl_rx_buffer[0], sizeof(supported_commands));
	  bootloader_uart_write_data(supported_commands, sizeof(supported_commands));
  }
  ...
}
```		 
    
		
## Command: BL_GET_CID     			
      
<img src="images/bl_get_cid.png" alt="Command BL_GET_CID" title="Command BL_GET_CID"> 		
    
```c
uint16_t get_mcu_chip_id(void)
{
  // The STM32F446xx MCUs intergrate an MCU ID code. This ID identifies the ST MCU part number
  // and the die revision. It is part of the DBG_MCU component and is mapped on the
  // external PPB bus (see Section 33.16 on page 1304). This code is accessed using the
  // JTAG debug pCat.2ort (4 to 5 pins) or the SW debug port (two pins) or by the user software
  // It is even accessible while the MCU is under system reset.
  uint16_t cid;
  cid = (uint16_t)(DBGMCU->IDCODE) & 0x0FFF;
  return cid;
}

void bootloader_handle_getcid_cmd(uint8_t* bl_rx_buffer)
{
  uint16_t bl_cid_num = 0;
  ...
  if (!bootloader_verify_crc(&bl_rx_buffer[0], command_packet_len - 4, host_crc))
  {
	  ...
	  bl_cid_num = get_mcu_chip_id();
	...
}
```			
    
<img src="images/DBGMCU_IDCODE.png" alt="DBGMCU_IDCODE" title="DBGMCU_IDCODE"> 		
     
		 
## Command: BL_GO_TO_ADDR		 
    
<img src="images/bl_go_to_addr.png" alt="Command BL_GO_TO_ADDR" title="Command BL_GO_TO_ADDR"> 			
      
## Flash memory commands:    
    
All the remaining commands are used to deal with flash memory i.e. erasing flash memory, jump to different sector of flash memory etc.					
    
<img src="../images/bootplacement.png" alt="Code placement (i.e. Bootloader and User app) in Flash memory" title="Code placement (i.e. Bootloader and User app) in Flash memory"> 				
     
As previously described, we store bootloader on first two sectors, Sector 0 and Sector 1, Whereas User app on Sector 2 onward. **By default all these sectors (0 to 7) are writable, erasable and readable**. However the microcontroller also gives us the provision to secure the contents of the Flash memory. We use Option bytes (16 bytes size from 0x1FFF_C000 - 0x1FFF_C00F) to manage security level on the Flash memories / sectors (Reference Manual 3.6 Option bytes, page: 72 ).	 
   
> [!NOTE]	 	 
> Memory read protection Level 2 is an irreversible operation. When Level 2 is activated, the level of protection cannot be decreased to Level 0 or Level 1.    
    
     
## Command: BL_GET_RDP_STATUS    
		 		 		
```c
uint8_t get_flash_rdp_level(void)
{
  uint8_t rdp_status =0;
#if 0
  FLASH_OBProgramInitTypeDef ob_handle;
  HAL_FLASHEx_OBGetConfig(&ob_handle);
  rdp_status = (uint8_t)ob_handle.RDPLevel;
#else
  volatile uint32_t *pOB_addr = (uint32_t*) 0x1FFFC000; // extract bit 15-8
  rdp_status = (uint8_t)(*pOB_addr >> 8);
#endif

  return rdp_status;
}

void bootloader_handle_getrdp_cmd(uint8_t* bl_rx_buffer)
{
  uint8_t rdp_level = 0x00;
  ...

  if (!bootloader_verify_crc(&bl_rx_buffer[0], command_packet_len - 4, host_crc))
  {
	  ...
	  rdp_level = get_flash_rdp_level();
	  ...
}
```						
    
Same can also be done using `HAL_FLASHEx_OBGetConfig(&ob_handle)` as guarded with if macro inside `get_flash_rdp_level(void)` 		
   
<img src="images/hal_rdp.png" alt="HAL RDP" title="HAL RDP"> 			
     
		 
## Command: BL_GO_TO_ADDR		 
    
<img src="images/bl_go_to_addr.png" alt="Command BL_GO_TO_ADDR" title="Command BL_GO_TO_ADDR"> 				 
						
<img src="images/bl_flash_erase.png" alt="Command BL_FLASH_ERASE" title="Command BL_FLASH_ERASE"> 					
      
<img src="images/bl_mem_write.png" alt="Command BL_MEM_WRITE" title="Command BL_MEM_WRITE"> 					
      
<img src="images/bl_mem_read.png" alt="Command BL_MEM_READ" title="Command BL_MEM_READ"> 					
      
<img src="images/bl_en_r_w_protect.png" alt="Command BL_EN_R_W_PROTECT" title="Command BL_EN_R_W_PROTECT"> 					
      
<img src="images/bl_dis_r_w_protect.png" alt="Command BL_DIS_R_W_PROTECT" title="Command BL_DIS_R_W_PROTECT"> 					
      
<img src="images/bl_read_sector_status.png" alt="Command BL_READ_SECTOR_STATUS" title="Command BL_READ_SECTOR_STATUS"> 					
		
		
		
		
		
		
		
		
		
		
     
		    
     
		    
     
		 		 		 
		 
     