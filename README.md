
![.jpg](/doc/FPGA_UART.jpg)

# FPGA_UART

This is a project implemented on the ALTERA EP4CE10F17C8 chip. It will be used in ELEC5552 design2 unit. In this unit, the host computer will import the filter coefficients to the fir filter inside the FPGA through UART. Therefore, the coefficients in the fir filter can be modified. Also, the project defines a buzzer output module as well as an LED digital display module to indicate that the coefficient has been successfully imported.

## How to use

1. Creat a new project in Quartus ii 13.1
2. Import all '.v' files in 'rtl' folder
3. Set file 'uart_syn.v' as top level entity
4. Assign the pins according to the FPGA used and compile the project
5. Upload the .sof file to the FPGA board
6. Connect the FPGA with host laptop through input pin 'uart_r' and the output pin 'uart_s'
7. Install RS232 driver in host laptop
8. Send serial signal through COM port manager 

## Introduction to uart

UART (universal asynchronous receiver transmitter) needs two wires: an input pin to receive signal and an output pin to send signal. A frame of UART signal usually includes: start bit, data bit, parity bit and a stop bit. (there is no parity bit in this project as the signal being transmitted will be simple). 

The UART system of this project is mainly composed of two modules: receiver and transmitter. The receiver will store the data in an 8-bit register after receiving the signal. When all the signals are received and reach the midpoint of the stop bit time, the receiver will output all the data in the register and send out a 1-bit signal to remind the data reception is complete. When the stop bit ends, this reminder signal 'uart_en' will be reset to zero. After detecting the rising edge of the reminder signal, the sender will receive the data from the 8 bit input and output them on the sending pin in order. 

This structure is only to detect whether the signal is transmitted correctly. The data sent has not been processed in any way. The data sent in the serial port manager should be consistent with the received data. The struct of the overall entity is shown below:
![.png] (/doc/overall_rtl.png)

### Receive signal

 The input signal should always be maintained at a high level when the host computer does not send any signal. Usually, the start bit of a frame is low level. A counter will be started when the edge detection circuit detects a low level. base on the transmitted bit rate and the system clock, the counter will calculate how many system clock cycles it will take to transmit a bit. When the duration of one bit is reached, this counter will be reset to zero and time again for the next bit. Therefore, the receiver will know which position the input signal is in a frame at the current time.

The data bits come after the start bit ends. The receiver will store these data in different bits of a register. In this project, one frame of data will have 8 data bits. After the 8 data bits are transmitted, the input signal will return to the high level (the stop bit is high level as well),  the receiver will start waiting for the arrival of the next signal falling edge. 

At the tenth bit of a frame, that is, when the stop bit, although no signal is registered, the eight-bit data signal stored in the register will be output for further processing, and the receiver will output a signal called'uat_en' The signal indicates that a frame of signal has been successfully received.

### Send signal

In addition to the two signal inputs of the system clock and rst, the sender also has 8-bit data input and 'uart_en' signal input. When the uart_en signal becomes 1, the sender starts to receive the signal and start timing. (The sender also has a same counter to indicate which bit is currently in a frame of signal). According to the time indicated, the sender will send different bits of the received data to the output pin.

## USB to TTL as well as CH340

RS232 requires a big port and the according cables are already not that popular nowadays. Fortunately, USB to TTL is a good solution to this. A CH340 chip can convert USB signal to TTL signal and thus a host laptop can communicate with the FPGA board directly through a usb cable. To achieve this, just connect the FPGA's signal output port (in this project called 'uart_s') with CH340 TXD port and connect the FPGA's signal receive port (in this project called 'uart_r') with CH340 RXD port. Then connect CH340 D+ with D+ port in usb port, so as CH340 D- port and D- port in usb port.



## Dynamic digit LED display

This project also creates a LED display to indicate that the signal has been successfully received by the receiver. the digit LEDs will firstly not turn on when the '.sof' file is upload to the FPGA. Those LEDs will be turned on only after successfully received a signal from the UART port.  For every time when a signal is received, the 6 segment digit LED will display  'LOAdEd' for 2 seconds, then turn to next state: showing 'ELEC' and '5552' each for 1 second and loop.

the entity created by this project will have a 6-bit segment select signal (in this project called 'sel') to select which segment to turn on. This output signal has a finate state machine and will be iterated for every 0.01s (if the system clk signal is 50MHz) to select different segment. Then, according to different segment chosen, a 8-bit led control signal 'set_led' will control the character shown in the chosen segment. Therefore, the process goes like this: firstly, the first segment is turned on, and shows 'L', it then turns off after 0.01s and the second segment turns on and show 'O', then it turns off as well after 0.01s and the third one turns on... as long as this procedure is going fast, human beings can not distinguish the on and off of those segments and see a complete word 'LOAdEd'.



## Reference



