Install **pySerial** python library to communicate with the target over the serial port. pySerial python module provide all the opening, reading, and writing a python functions for the serial port.  

```bash
# for windows 
python -m pip install pyserial 

# for Ubuntu
sudo apt-get install python3-serial 

# for MacOSX 
brew install pipx
pipx install pyserial

# activate the venvs on every new shell login
python3 -m venv /Users/<MAC-USERNAME>/Library/Application\\\ Support/pipx/venvs
source /Users/<MAC-USERNAME>/Library/Application\\\ Support/pipx/venvs/bin/activate
```    

Now, cd into `host_app/python` directory and run the `STM32_Programmer_V1.py`      
      
```bash
cd host_app/python

# list all the serial ports
python3 STM32_Programmer_V1.py

# Run the above script again and type in the right serial port name 
python3 STM32_Programmer_V1.py
```    
    
> [!NOTE]		
> Upon RESET of MCU (or power up the MCU), User button has to be pressed to go to bootloader mode, and then use above python script to connect to MCU with Serial port and communicate with it.     
     
<img src="images/shell_1.png" alt="Output of Serial communication with target MCU's bootloader" title="Output of Serial communication with target MCU's bootloader">
<img src="images/shell_2.png" alt="Output of Serial communication with target MCU's bootloader" title="Output of Serial communication with target MCU's bootloader">			 
     
		 