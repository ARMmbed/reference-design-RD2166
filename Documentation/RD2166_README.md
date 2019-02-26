# Reference Application Introduction:

This Reference Application is targetted specifically at the RD2166 Reference Design and is built on mbedOS and uses the "mbed-cloud-client" library to handle the Pelion Platform connection, registration and LWM2M resources. The application simulates a button press every 5 seconds and relays the packet to Arm Pelion IoT platform.

The current version of the Reference Application uses: Mbed OS v5.11.5 and Mbed Cloud Client v2.0.1.1

Note: references to Mbed Cloud and Pelion Device Managament are interchangeable.  For example: “Mbed Cloud Client enables the device to communicate with Pelion Device Management” and “Mbed Cloud Client enables the device to communicate with Mbed Cloud” have the same meaning.  In later versions we will remove references to Mbed Cloud inline with branding guidelines.

# Pre-requisites:
1. Hardware:
    * RD2166 based board or platform (the application was developed and tested using the Example Implemenation Hardware - MCB/MTB
    * A micro USB cable
    * A SIM card from your network provider with SIM details available for SIM PIN, APN, SIM username and SIM password.
    * For development purposes of this reference design, an EE network SIM card was used in the UK with the following parameters:

    ```
            "nsapi.default-cellular-sim-pin"       : "\"1234\"",
            "nsapi.default-cellular-apn"           : "\"everywhere\"",
            "nsapi.default-cellular-username"      : "\"myusername\"",
            "nsapi.default-cellular-password"      : "\"mypassword\"",
            "cellular.use-apn-lookup"              : false,
    ```

2. Software:
    * mbed CLI - https://github.com/ARMmbed/mbed-cli
    * Pelion account for API keys & developer certificate.

# Getting started:
1. Import this example to your local machine with mbed CLI:
    ``` mbed import https://github.com/ARMmbed/reference-design-RD2166.git ```

2. Move into the reference-design-RD2166 directory ``` cd reference-design-RD2166 ```. Run the ``` mbed deploy ``` command to download the sources and to add them to your current directory.
    * Login to your Pelion Cloud account on a browser & follow the steps below:
        * Navigate to Device identity > Certificates
        * Select the certificate created by your account admin and click on "Download Developer C file"
        * Save the file?mbed_cloud_dev_credentials.c to the location of the RD2166 directory. You have to overwrite the contents of the existing file there.

3. Configure your network credentials:
    * In the mbed_app.json file in the root of the application, modify the SIM PIN, APN and password for your network.
    * Save the changes.

4. The supplied Bootloader in ../bootloader/ is currently built to use SPI Flash. If you are using the SPI Flash to store the certificates, then you are good to go. If you prefer to use the SD card for the storage, follow the steps below. If not, you may skip these & move to step 11.
   
5. To build the bootloader to use the SD card:  (TIP: You may want to use atleast a Class 10, 2GB SD card.) This application currently uses v3.5.0 of the mbed-bootloader.
    * Import the bootloader repo - https://github.com/ARMmbed/mbed-bootloader
    * Change the default storage component in targets/targets.json file of Mbed OS.
    * Modify mbed_app.json in bootloader: Change update storage-address and remove SPI pin defines.  
    
```
    "update-client.storage-address"  : "(1024*1024*64)",  
```

And

    ```
    "MTB_STM_L475": {  
    "flash-start-address"              : "0x08000000",  
    "flash-size"                       : "(1024*1024)",  
    "nvstore.area_1_address"           : "(MBED_CONF_APP_FLASH_START_ADDRESS + 1016*1024)",
    "nvstore.area_1_size"              : "(4*1024)",
    "nvstore.area_2_address"           : "(MBED_CONF_APP_FLASH_START_ADDRESS + 1020*1024)",
    "nvstore.area_2_size"              : "(4*1024)",
    "update-client.application-details": "(MBED_CONF_APP_FLASH_START_ADDRESS+64*1024)",  
    "application-start-address"        : "(MBED_CONF_APP_FLASH_START_ADDRESS+65*1024)",  
    "max-application-size"             : "DEFAULT_MAX_APPLICATION_SIZE",   
    }
     ```
6. Change mbed_app.json in source:
 ```   "update-client.storage-address"  : "(1024*1024*64)",  ```
   
7. Once changes are made in bootloader, build it:
``` mbed compile -t GCC_ARM -m MTB_STM_L475 ```

8. Copy the generated binary to ../bootloader/ directory. Rename to match the existing file name. Or if you prefer to use a different file name, make sure that the ```            "target.bootloader_img"                : "bootloader/mbed-bootloader-extended.bin",``` in mbed_app.json in the RD2166's root directory matches the file name you have used.

9. Change mbed_app.json in the root of the reference application i.e. 
"update-client.storage-address"  : "(1024*1024*64)"

10. Change main.cpp in source to match SD card - i.e. un-comment the respective header include files in main.cpp to include the FileSystem specific headers while commenting out / removing the ones related to the SPI Flash (LittleFileSystem.h). This RD uses LittleFileSystem and SPI Flash. If you want to use FAT FS with SD card, then please un-comment the line to include the respective header

### IMPORTANT: The required lines are already supplied in the code. Uncomment appropriate lines in main.cpp and build the binary.

11. Once you are happy with changes(if any, for the SD card) or you are going with the provided default options, you may now proceed to compile the application binary with the command:
    ``` mbed compile -t GCC_ARM -m MTB_STM_L475 --profile=tiny.json```. 

12. Flash the generated binary (RD2166.bin) to your device. Open a serial terminal at 115200, 8-N-1 & observe the output.

13. The device starts the example, connects to the Cellular network & after a few moments connects & registers to Pelion Cloud. It then prints out a device ID. You will need this for firmware update at a later stage.

* You can now check your simulated button counter's value in the Pelion portal as well.
* Navigate to "Device Directory" in the portal and click on your device's device ID. This will open up a new drawer on the page.
* Scroll down to LWM2M resource "Button_count" and click on it.
* A new window opens and you will be able to see the counter & a graph increment every time the resource value increments.

14. Congratulations, your device is now connected to Pelion IoT platform. Please proceed to the FW update section now.

## Recommendation:  
Increase stack size:         "MBED_CONF_APP_MAIN_STACK_SIZE=6000", to 7K if using newer versions of Mbed OS  


# Demonstrate a remote Firmware update:

1. In order to update FW on a connected device, you will need the mbed CLI.

2. A dependency is to install the python SDK along with the manifest tool. This is done using : ``` pip install mbed-cloud-sdk --upgrade ```

3. Now you need to setup Mbed OS on the device to receive FW updates:
    * Login to your Pelion Cloud portal.
    * Create an API device key (Access Management -> API Keys -> Create a new API key)
    * Ensure you copy the API key generated as this is visible only once.
    
4. Initialize the certificates for your application:
## IMPORTANT: You must initialize the certificates before you flash the device for the 1st time as these certificates need to be embedded within the device in order for it to receive remote updates.

``` mbed dm init -a <api key> -S <mbed cloud API URL> -d <domain name> -m <model ID> --force ``` 

Where mbed cloud API URL is https://api.us-east-1.mbedcloud.com/  . The domain name must have a ".com" as a mandatory requirement and the model ID can be an integer of your choice.

### Note the use of --force as this option overrides the defaults provided in the application. This is a mandatory parameter.

5. With the certificates initialized, there will now be a ".update-certificates" directory created within the application's root directory.

6. Now, compile the application:
``` mbed compile -t GCC_ARM -m MTB_STM_L475 ```

7. Unless you made any changes to bootloader before this, you do not need to re-build the bootloader. Otherwise, please re-build the bootloader using steps described earlier and copy the new binary to the respective folder.

8. Program your device with the ``` RD2166.bin ``` just generated. You can either drag-drop via the GUI of your OS or you can use the corresponding copy command from the terminal for your OS.

9. Open a serial terminal application such as PuTTY and observe the output. You will notice the device boots up, connects to the Cellular network, starts Pelion Cloud Client and registers to the Cloud.

10. Once the device registers, make a note of the device ID assigned to the device.

11. Modify the main.cpp in the source directory to add a small printf statement - this is already provided in the code. So, you can un-comment this line in order to demo a FW update.

12. Compile the new source with ``` mbed compile -t GCC_ARM -m MTB_STM_L475 ``` command.

13. You will now have a new binary in the BUILD\MTB_STM_L475\GCC_ARM\ directory called "RD2166_update.bin"

14. Use the mbed CLI to perform a remote update on the device:

``` mbed dm update device -p <payload name> -D <device id> ``` 

where payload name is the full path to your new binary built from step 13 and device ID is the one obtained from step 10.

15. You will notice from the serial terminal that the device gets a request to update the FW, then this is authorized, then the download of the new FW starts, finishes and the bootloader verifies the authencity of the new FW, installs it on the device and reboots.

16. You will now notice the new printf statement appear on the device logs which indicates that the newly built application binary is now installed and running on the device i.e. the device has been remotely updated.

The success logs should look like this on a serial terminal, although your device *will* be assigned a different device ID and will have a different IP address.

```
PS C:\Ashok\Modules\Reference_Apps\Cellular\pelion-ready-example> mbed sterm -b 115200
[mbed] Detecting connected targets/boards to your system...
[mbed] Opening serial terminal to "MTB_STM_L475"
--- Terminal on COM89 - 115200,8,N,1 ---
[BOOT] Mbed Bootloader
[BOOT] ARM: 00000000000000000000
[BOOT] OEM: 00000000000000000000
[BOOT] Layout: 0 8009280
[BOOT] Active firmware integrity check:
[BOOT] [++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++]
[BOOT] SHA256: 3E213FC00FCC310F95CCB22FF574F48E65BD923B186F80C10BAC5B54A4EFFA88
[BOOT] Version: 1551179671
[BOOT] Slot 0 firmware is of older date
[BOOT] Version: 1550750045
[BOOT] Active firmware up-to-date
[BOOT] Application's start address: 0x8010400
[BOOT] Application's jump address: 0x8011071
[BOOT] Application's stack address: 0x20018000
[BOOT] Forwarding to application...
     Starting Simple Pelion Device Management Client example
Connecting to the network...
Connected to the network successfully. IP address: 100.120.93.161
Initialized Pelion Client. Registering...
Connected to Pelion Device Management. Endpoint Name: 01690fe6e310000000000001001002b2
Simulated button clicked 1 times
Simulated button clicked 2 times
Simulated button clicked 3 times
Simulated button clicked 4 times
Simulated button clicked 5 times
Simulated button clicked 6 times
Simulated button clicked 7 times
Simulated button clicked 8 times
Simulated button clicked 9 times
Simulated button clicked 10 times
Simulated button clicked 11 times
Firmware download requested
Authorization granted
Simulated button clicked 12 times
Downloading: [++++\                                             ] 8 %Simulated button clicked 13 times
Downloading: [+++++++++++/                                      ] 22 %Simulated button clicked 14 times
Downloading: [++++++++++++++-                                   ] 29 %Simulated button clicked 15 times
Downloading: [+++++++++++++++++++\                              ] 39 %Simulated button clicked 16 times
Downloading: [+++++++++++++++++++++++++/                        ] 50 %Simulated button clicked 17 times
Downloading: [++++++++++++++++++++++++++++++++\                 ] 65 %Simulated button clicked 18 times
Downloading: [++++++++++++++++++++++++++++++++++++++/           ] 77 %Simulated button clicked 19 times
Downloading: [+++++++++++++++++++++++++++++++++++++++++++++/    ] 91 %Simulated button clicked 20 times
Downloading: [++++++++++++++++++++++++++++++++++++++++++++++++++] 100 %
Download completed
Simulated button clicked 21 times
Firmware install requested
Authorization granted
[BOOT] Mbed Bootloader
[BOOT] ARM: 00000000000000000000
[BOOT] OEM: 00000000000000000000
[BOOT] Layout: 0 8009280
[BOOT] Active firmware integrity check:
[BOOT] [++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++]
[BOOT] SHA256: 3E213FC00FCC310F95CCB22FF574F48E65BD923B186F80C10BAC5B54A4EFFA88
[BOOT] Version: 1551179671
[BOOT] Slot 0 firmware integrity check:
[BOOT] [++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++]
[BOOT] SHA256: B9B086982B0542249C0223BE25AEE43EEBDC2A1251EA35E1DC634156D1C25B62
[BOOT] Version: 1551179857
[BOOT] Update active firmware using slot 0:
[BOOT] [++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++]
[BOOT] Verify new active firmware:
[BOOT] [++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++]
[BOOT] New active firmware is valid
[BOOT] Application's start address: 0x8010400
[BOOT] Application's jump address: 0x8011071
[BOOT] Application's stack address: 0x20018000
[BOOT] Forwarding to application...
?     Starting Simple Pelion Device Management Client example
Connecting to the network...
Connected to the network successfully. IP address: 10.185.222.179
******** This line demonstrates a FW update ********
Initialized Pelion Client. Registering...
Connected to Pelion Device Management. Endpoint Name: 01690fe6e310000000000001001002b2
Simulated button clicked 1 times
Simulated button clicked 2 times
Simulated button clicked 3 times
Simulated button clicked 4 times
Simulated button clicked 5 times
Simulated button clicked 6 times
Simulated button clicked 7 times
Simulated button clicked 8 times
Simulated button clicked 9 times
Simulated button clicked 10 times
Simulated button clicked 11 times
Simulated button clicked 12 times
Simulated button clicked 13 times
Simulated button clicked 14 times
```

#### Also note that the device ID remains the same after the FW update. This indicates that the SOTP regions were not over-written while perfoming the update.
