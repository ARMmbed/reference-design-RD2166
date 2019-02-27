# RD2166 Introduction:

The RD2166 Reference Design is a Pelion Ready design based on the Quectel BG96 cellular module and the STM32L475 Microcontroller.

The reference design is a tested and proven starting point targetted towards the creation of both Development Boards and End Products.


# Reference Design Contents:
**Reference Schematic** 
This is the core of reference design, and illustrates where and how the essential components are connected. 

**Reference Application**
Built on mbedOS, and includes Pelion Device Manager. This application has been configured to run on the Reference Design hardware, and will connect to the Pelion Cloud, and is capable of running the cloud based update service.
Please refer to the [\Documentation\RD2166_Readme.md](https://github.com/ARMmbed/reference-design-RD2166/blob/master/Documentation/RD2166_README.md) for instructions on how to build & use this example application.

**Example Implementation** 
Based on the Reference Design, this example implementation is included to demonstrate the Reference Design being used to create real-world hardware. The example implemenation is used within Arm to include the Reference Design into the internal test infrastructure, it is comprised of three parts:

*MCB (Module Carrier Board)* - this has been designed using the Reference Design schematic. In this reference design, the MCB is a standalone STM32L475 MCU.

*MEB (Modulude Expansion Board)* - this has been designed using the Reference Design schematic. In this reference design, the MEB is a Quectel BG96 module.

*MTB (Module Test Bed)* - this is part of the test infrastructure, and includes DAPLink circuit for programming the device, a range of I/O and loopback paths for automated testing.
 


# Version Information:
		
| Release | mbed OS | mbed Cloud Client | Release Date | Comment |
| --- | --- | --- | --- | --- | 
|v1.0 | 5.11.5	| 2.0.1.1	| 27/02/2019 | Initial Release
| |  |  |   | 
