# STM32F429-DCMI usage example code (OV9655)

This program is made for a STM32F429 discovery board AND OV9655 camera. The demo program samples a 320x240px image from a OV9655 camera into the SRAM and encodes it into JPEG in real time. It then transmits the image by serial to the computer where it can be displayed. The image is encoded into JPEG in real time because the STM32F429 doesn't provide enough memory to hold a raw image (320x240px) in it's SRAM. The STM32F429 discovery board has actually external SDRAM but the project's goal is it, to solve this problem without any external SDRAM.

**How it's done:** The image is encoded to JPEG while the microcontroller receives the picture from the OV9655 by DCMI at the same time. The JPEG encoding algorithm needs just 16x16px blocks at the same time, so ther is no need of much memory (SRAM).

The image itself is sent by the camera linewise. To get a full 16x16px block, full 16 lines have to be received. The DCMI interface on the microcontroller will generate an interrupt every 16 lines, so the algorithm can process all 16x16px blocks. For VGA (640x480px) The algoritm needs 15kB for buffering the image (480[px] x 16[lines] x 2[byte/px]). While we are receiving a stream, we have the need of 2 buffers, because we can't process the data in a buffer in which data gets written.

**Total amount of SRAM needed (for VGA)**

Purpose      | Memory
------------ | ------
DMA buffer 1 | 15kB
DMA buffer 2 | 15kB
JPEG Output  | 50kB
**Total**    | **80kB**


**Anvantages:** No external memory needed, small capture outputs

**Disadvantages:** Long shutter time (VGA 1/3sec, QVGA 1/12sec)


![Demo capture](https://raw.githubusercontent.com/DL7AD/STM32F429-DCMI/master/example.jpg)
Demo capture taken by STM32F429 with OV9655

![Demo capture](https://github.com/DL7AD/STM32F429-DCMI/blob/master/stm32f4_ov9655.jpg)
STM32F429 discovery board connected to Omnivision OV9655


Code sources
------------
- DCMI example http://mikrocontroller.bplaced.net/wordpress/?page_id=1115
- JPEGant http://sourceforge.net/projects/jpegant.berlios/
- To make this demo work ChibiOS is needed https://github.com/ChibiOS/ChibiOS

Hardware recommondations
------------------------
*ST STM32F429 discovery board*
- Datasheet disco board: http://www.st.com/st-web-ui/static/active/en/resource/technical/document/user_manual/DM00093903.pdf
- Datasheet Microcontroller: http://www.st.com/web/en/resource/technical/document/datasheet/DM00071990.pdf
- Reference Manual: http://www.st.com/web/en/resource/technical/document/reference_manual/DM00031020.pdf

*Omnivision OV9655 camera*
- Datasheet: http://www.waveshare.com/w/upload/0/0a/OV9655.pdf

Hardware connections
--------------------
The STM32F429 discovery board must be connected to the camera to spectific pins

Camera | Disco-Board   | _Notes_
------ | ------------- | -------
3.3V   | VDD           | Disco board supplies 3.0V but that's OK
GND    | GND           |
SIOC   | PB10          | I2C CLK
SIOD   | PB11          | I2C SDA
VSYNC  | PB7           |
HREF   | PA4           |
PCLK   | PA6           |
XCLK   | PA8           | Camera clock input (Generated by STM32)
D9     | PE6           | Data pin
D8     | PE5           | Data pin
D7     | PB4           | Data pin
D6     | PE4           | Data pin
D5     | PC9           | Data pin
D4     | PC8           | Data pin
D3     | PC7           | Data pin
D2     | PC6           | Data pin
RET    | _unconnected_ | Reset pin (clears all registers)
PWDN   | _unconnected_ | Power down

Installation/Building/Flashing
------------------------------
To be able to build this project, a modification in the ChibiOS submodule must be done:
- Make sure you have a GCC toolchain installed (http://www.wolinlabs.com/blog/linux.stm32.discovery.gcc.html)
- Open this file: ChibiOS/os/ext/CMSIS/ST/stm32f4xx.h
- and uncomment this following line:
```c
#define STM32F429xx           /*!< STM32F429VG, STM32F429VI, STM32F429ZG, STM32F429ZI, STM32F429BG, STM32F429BI, STM32F429NG, 
                                   STM32F439NI, STM32F429IG  and STM32F429II Devices */
```
- The compile it with *make*
- And then flash the STM32 with *make flash* (You probably have to adjust the variable *STLINK* in the Makefile)

Taking photos
-------------
- The programm initializes the camera once at startup by I2C.
- Thereafter the green LED on the discovery board will blink green to signal, that the board is idling.
- When the user button is pressed the picture will be taken and the JPEG encoded picture will be transmitted by UART on PIN PD5. The UART output must be dumped into a file on the computer. The green LED will blink again if UART transmission is done. (*Important* Programs like minicom are not sufficient to dump the JPEG because they to not dump unprintable chars like 0x00 into the file)
- The red LED on the discovery board displays when the algorithm is encoding JPEG.
- _Note: At the moment there's a bug in the software. The STM32 has to bet reset after each image transmission._
