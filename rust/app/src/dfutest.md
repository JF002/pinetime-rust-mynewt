# Wireless Firmware Update In Action on PineTime Smart Watch (nRF52)

This is a continuation of the articles...

_["MCUBoot Bootloader for PineTime Smart Watch (nRF52)"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/mcuboot)_

_["Firmware Update over Bluetooth Low Energy on PineTime Smart Watch"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/dfu)_

_["Configure Mynewt for SPI Flash on PineTime Smart Watch (nRF52)"](https://lupyuen.github.io/pinetime-rust-mynewt/articles/spiflash)_

We'll show step by step Wireless Firmware Update running on PineTime Smart Watch with nRF Connect App, MCUBoot Bootloader, NimBLE Bluetooth LE Stack and Apache Mynewt.

Here's the video of the Firmware Update...

[Watch on YouTube](https://youtu.be/thLhGUl9-CU)

[Download the video](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.3)

Now let's learn exactly what happened in that video.

# Test Firmware for PineTime Firmware Update

The PineTime firmware files used for testing Firmware Update are located here...

[`pinetime-rust-mynewt/releases/tag/v4.1.1`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.1)

[`pinetime-rust-mynewt/releases/tag/v4.1.2`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2)

[`pinetime-rust-mynewt/releases/tag/v4.1.4`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.4)

We provide 2 versions of each firmware file to be flashed...

1. __Arm Semihosting Enabled:__ These files should be selected if you're debugging with ST-Link or Raspberry Pi. They will display debugging messages on the Arm Semihosting Console via OpenOCD. These files will hang when used with JLink.

1. __Arm Semihosting Disabled:__ Files named `nosemi` should be selected if you're debugging with JLink. They will not display debugging messages via Arm Semihosting.

The Semihosting Build Options are defined here: [Bootloader Options](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/targets/nrf52_boot/pkg.yml), [Application Firmware Options](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/targets/nrf52_my_sensor/pkg.yml)

Before the Firmware Update test, the following files were flashed manually to PineTime (via ST-Link or Raspberry Pi)...

| Firmware Component | Version | Binary File | From | Flash To | At Address | Remarks |
| :--- | :--- |  :--- |  :--- |  :--- |  :--- | :--- |
| MCUBoot Bootloader for Semihosting | 1.0.0 | [`mynewt.elf.bin`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.1/mynewt.elf.bin) | [`v4.1.1`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.1) | Internal Flash ROM | `0x0000 0000` | Use this bootloader if you're debugging with ST-Link or Raspberry Pi (Semihosting is enabled)
| MCUBoot Bootloader for JLink | 1.0.0 | [`mynewt-nosemi.elf.bin`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.4/mynewt_nosemi.elf.bin) | [`v4.1.4`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.4) | Internal Flash ROM | `0x0000 0000` | Use this bootloader if you're debugging with JLink (Semihosting is disabled)
| Application Firmware for Semihosting | 1.0.0 | [`my_sensor_app.img`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.1/my_sensor_app.img) | [`v4.1.1`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.1) | Internal Flash ROM | `0x0000 8000` | Contains MCUBoot Image Header and Mynewt OS with Rust. Shows "`I AM PINETIME`" (Semihosting is enabled)
| Application Firmware for JLink | 1.0.0 | [`my_sensor_app_nosemi.img`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.4/my_sensor_app_nosemi.img) | [`v4.1.4`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.4) | Internal Flash ROM | `0x0000 8000` | Contains MCUBoot Image Header and Mynewt OS with Rust. Shows "`I AM PINETIME`" (Semihosting is disabled)
| Boot Graphic (Optional) | 1.0.0 | [`boot-graphic.bin`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.1/boot-graphic.bin) | [`v4.1.1`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.1) | External SPI Flash | `0x0000 0000` | Hand-drawn PineTime Logo in RGB565 format, 240 x 240 pixels, 2 bytes per pixel

If MCUBoot Bootloader for Semihosting doesn't start on PineTime, try using MCUBoot Bootloader for JLink.

For the Boot Graphic: Use this tool to convert a 240 x 240 PNG file to RGB565...

[`github.com/lupyuen/pinetime-graphic`](https://github.com/lupyuen/pinetime-graphic)

During the Firmware Update test, one of the following files will be selected in nRF Connect or Newt Manager for Firmware Update...

| Firmware Component | Version | Binary File | From | Flash To | At Address | Remarks |
| :--- | :--- |  :--- |  :--- |  :--- |  :--- | :--- |
| Application Firmware for Semihosting | 1.1.0 | [`my_sensor_app_1.1.img`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.2/my_sensor_app_1.1.img) | [`v4.1.2`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2) | External SPI Flash | `0x0004 0000` | Contains MCUBoot Image Header and Mynewt OS with Rust. Shows "`PINETIME 1.1`" (Semihosting is enabled)
| Application Firmware for JLink | 1.1.0 | [`my_sensor_app_nosemi_1.1.img`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.2/my_sensor_app_nosemi_1.1.img) | [`v4.1.2`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2) | External SPI Flash | `0x0004 0000` | Contains MCUBoot Image Header and Mynewt OS with Rust. Shows "`PINETIME 1.1`" (Semihosting is disabled)
| Application Firmware for Semihosting | 1.2.0 | [`my_sensor_app_1.2.img`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.2/my_sensor_app_1.2.img) | [`v4.1.2`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2) | External SPI Flash | `0x0004 0000` | Contains MCUBoot Image Header and Mynewt OS with Rust. Shows "`PINETIME 1.2`" (Semihosting is enabled)
| Application Firmware for JLink | 1.2.0 | [`my_sensor_app_nosemi_1.2.img`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/download/v4.1.2/my_sensor_app_nosemi_1.2.img) | [`v4.1.2`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2) | External SPI Flash | `0x0004 0000` | Contains MCUBoot Image Header and Mynewt OS with Rust. Shows "`PINETIME 1.2`" (Semihosting is disabled)

We provide 2 versions of the new firmware for easier testing. The build version numbers were specified in [`scripts/nrf52/image-app.sh`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/scripts/nrf52/image-app.sh)

To use the Firmware Image Files with the nRF Connect App, we need to rename the file extension from `.img` to `.bin` after downloading (e.g. `my_sensor_app_1.1.bin`)

Note that `.bin` files may be found in the [`v4.1.1`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.1) and [`v4.1.2`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2) downloads, but these `.bin` files should not be used for Firmware Updates because they don't have the MCUBoot Image Header.

So always download the `.img` files and then rename them to `.bin`.

The `.img` files were created with the `imgtool.py` command-line tool [described here](https://lupyuen.github.io/pinetime-rust-mynewt/articles/dfu).

# Video of PineTime Firmware Update Test

Here's the video of the Wireless Firmware Upgrade from version 1.0.0 to 1.1.0...

- [Watch on YouTube](https://youtu.be/thLhGUl9-CU)

- [Download the video](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.3)

Step by step, by timecode...

- `00:00` - PineTime is preloaded with...

    1. MCUBoot Bootloader for Semihosting

    1. Application Firmware Image v1.0.0

    1. Boot Graphic

    The Application Firmware is running, showing "`I AM PINETIME`"

    ST-Link debugger is connected, showing the debug log. Here are the messages generated by MCUBoot during booting, `Swap type` is set to `perm` because we have previously Confirmed the firmware via nRF Connect...

    ```
    [INF] Primary image: magic=good, swap_type=0x2, copy_done=0x1, image_ok=0x1
    [INF] Scratch: magic=bad, swap_type=0x1, copy_done=0x2, image_ok=0x2
    [INF] Boot source: none
    [INF] Swap type: perm
    ```
    _From [`mcuboot/main.c`](https://github.com/JuulLabs-OSS/mcuboot/blob/master/boot/mynewt/src/main.c#L239-L245)_

- `00:03` - We start the [Nordic nRF Connect](https://www.nordicsemi.com/Software-and-tools/Development-Tools/nRF-Connect-for-mobile) mobile app on an Android phone.

    We tap on `pinetime` and the `DFU` icon to start the Firmware Update

- `00:12` - We __select the firmware file__ to be flashed...

    Application Firmware Image v1.1.0 <br>
    `my_sensor_app_1.1.img` <br>
    Downloaded from [`v4.1.2`](https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2) 

    The file was renamed to `my_sensor_app_1.1.bin` after downloading, because nRF Connect only accepts `.bin` files.

    Wireless upload to PineTime begins. The new firmware will be staged in PineTime's External SPI Flash at address `0x0004 0000`.

- `02:11` - __Upload completed__. PineTime attempts to reboot, but because we are running ST-Link, the reboot attempt was caught by the debugger. We restart the OpenOCD debugger to reboot PineTime manually.

    ```
    target halted due to breakpoint, current mode: Thread 
    xPSR: 0x61000000 pc: 0x000081d8 psp: 0x20006a78, semihosting
    ```

- `02:13` - PineTime __reboots__. MCUBoot starts.

    ```
    Starting Bootloader...
    ```
    _From [`libs/pinetime_boot/pinetime_boot.c`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/libs/pinetime_boot/src/pinetime_boot.c#L35)_

- `02:14` - MCUBoot renders the __Boot Graphic__ (hand-drawn PineTime logo) in under 1 second

    ```
    Displaying image...
    Image displayed
    ```
    _From [`libs/pinetime_boot/display.c`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/libs/pinetime_boot/src/display.c#L112-L183)_

    MCUBoot checks whether the watch button is pressed, for Manual Firmware Rollback

    ```
    Button: 0
    ```
    _From [`libs/pinetime_boot/pinetime_boot.c`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/libs/pinetime_boot/src/pinetime_boot.c#L45)_

- `02:16` - MCUBoot starts swapping the new firmware (External SPI Flash) with the old firmware (Internal Flash ROM)

- `02:18` - MCUBoot swapping completed. New firmware is now in Internal Flash ROM, old firmware is now in External SPI Flash.

    The message `Swap type: test` means that the swapping to new firmware has been completed successfully.

    ```
    [INF] Primary image: magic=good, swap_type=0x3, copy_done=0x1, image_ok=0x1
    [INF] Scratch: magic=bad, swap_type=0x1, copy_done=0x2, image_ok=0x2
    [INF] Boot source: none
    [INF] Swap type: test
    ```
    _From [`mcuboot/main.c`](https://github.com/JuulLabs-OSS/mcuboot/blob/master/boot/mynewt/src/main.c#L239-L245)_

- `02:30` - MCUBoot wait 5 seconds and checks whether the watch button is pressed, for __Manual Firmware Rollback__

    ```
    Button: 0
    Button: 0
    ```
    _From [`libs/pinetime_boot/pinetime_boot.c`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/libs/pinetime_boot/src/pinetime_boot.c#L54-L58)_

- `02:36` - MCUBoot starts the __New Application Firmware__

    ```
    Bootloader done
    ```
    _From [`pinetime_boot/pinetime_boot.c`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/libs/pinetime_boot/src/pinetime_boot.c#L61)_

    Mynewt Application Firmware starts the Temperature Stub Driver and dsplays the Hardware ID    

    ```
    TMP create temp_stub_0
    NET hwid 4a f8 cf 95 6a be c1 f6 89 ba 12 1a 
    NET standalone node 
    ```

    Mynewt Application Firmware reads the Internal Flash ROM and External SPI Flash for testing 

    ```
    Testing flash...
    Read Internal Flash ROM...
    Read 0x0 + 20
    0x0000: 0x00 0x00 0x01 0x20 0xd9 0x00 0x00 0x00 
    0x0008: 0x35 0x01 0x00 0x00 0x37 0x01 0x00 0x00 
    0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
    0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
    Read External SPI Flash...
    Read 0x0 + 20
    0x0000: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
    0x0008: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
    0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
    0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
    Flash OK
    ```
    _From [`apps/my_sensor_app/flash_test.c`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/apps/my_sensor_app/src/flash_test.c#L218-L298)_

- `02:37` - Mynewt Application Firmware __resets the Backlight and Display Controller,__ causing the screen to blank (needs to be fixed)

- `02:43` - Mynewt Application Firmware __switches on the Backlight__ via the Rust driver for ST7789 Display Controller. The hand-drawn PineTime logo previously rendered is now visible.

    Mynewt Application Firmware __erases the screen very slowly__ via the Rust driver for ST7789 Display Controller

    ```
    Rust test display
    ```
    _From [`rust/app/display.rs`](https://github.com/lupyuen/pinetime-rust-mynewt/blob/ota2/rust/app/src/display.rs#L23)_

- `02:55` - Mynewt Application Firmware __renders some shapes__ and the message "`PINETIME 1.1`"

# Test Log

TODO

Here's the complete log from the ST-Link OpenOCD debugger...

```
# Luppy at Luppys-MacBook-Pro.local in ~/PineTime/pinetime-rust-mynewt on git:ota2 ✖︎ [9:41:49]
→ ./scripts/nrf52/flash-app.sh
+ source scripts/config.sh
++ swd_device=scripts/nrf52/swd-stlink.ocd
+ openocd/bin/openocd -f scripts/nrf52/swd-stlink.ocd -f scripts/nrf52/flash-app.ocd
GNU MCU Eclipse 64-bit Open On-Chip Debugger 0.10.0+dev-00462-gdd1d90111 (2019-01-15-13:49)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
debug_level: 0
adapter speed: 1000 kHz
force hard breakpoints
Stopping...
target halted due to debug-request, current mode: Thread 
xPSR: 0x61000000 pc: 0x000001c6 msp: 0x2000ffd8

Flashing Application...
***** DISABLED Flashing Application...

Restarting...
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x000000d8 msp: 0x20010000
Enabled ARM Semihosting to show debug output
semihosting is enabled

**** Done! Press Ctrl-C to exit...
Starting Bootloader...
Displaying image...
Image displayed
Button: 0
[INF] Primary image: magic=good, swap_type=0x2, copy_done=0x1, image_ok=0x1
[INF] Scratch: magic=bad, swap_type=0x1, copy_done=0x2, image_ok=0x2
[INF] Boot source: none
[INF] Swap type: perm
Button: 0
Button: 0
Bootloader done
TMP create temp_stub_0
NET hwid 4a f8 cf 95 6a be c1 f6 89 ba 12 1a 
NET standalone node 
Testing flash...
Read Internal Flash ROM...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x01 0x20 0xd9 0x00 0x00 0x00 
  0x0008: 0x35 0x01 0x00 0x00 0x37 0x01 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Read External SPI Flash...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0008: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Flash OK
Rust test display
target halted due to breakpoint, current mode: Thread 
xPSR: 0x61000000 pc: 0x000081d8 psp: 0x20006a78, semihosting
^Cshutdown command invoked


# Luppy at Luppys-MacBook-Pro.local in ~/PineTime/pinetime-rust-mynewt on git:ota2 ✖︎ [9:45:03]
→ ./scripts/nrf52/flash-app.sh
+ source scripts/config.sh
++ swd_device=scripts/nrf52/swd-stlink.ocd
+ openocd/bin/openocd -f scripts/nrf52/swd-stlink.ocd -f scripts/nrf52/flash-app.ocd
GNU MCU Eclipse 64-bit Open On-Chip Debugger 0.10.0+dev-00462-gdd1d90111 (2019-01-15-13:49)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
debug_level: 0
adapter speed: 1000 kHz
force hard breakpoints
Stopping...
target halted due to debug-request, current mode: Thread 
xPSR: 0x61000000 pc: 0x000001ca msp: 0x2000ffd8

Flashing Application...
***** DISABLED Flashing Application...

Restarting...
target halted due to debug-request, current mode: Thread 
xPSR: 0x01000000 pc: 0x000000d8 msp: 0x20010000
Enabled ARM Semihosting to show debug output
semihosting is enabled

**** Done! Press Ctrl-C to exit...
Starting Bootloader...
Displaying image...
Image displayed
Button: 0
[INF] Primary image: magic=good, swap_type=0x3, copy_done=0x1, image_ok=0x1
[INF] Scratch: magic=bad, swap_type=0x1, copy_done=0x2, image_ok=0x2
[INF] Boot source: none
[INF] Swap type: test
Button: 0
Button: 0
Bootloader done
TMP create temp_stub_0
NET hwid 4a f8 cf 95 6a be c1 f6 89 ba 12 1a 
NET standalone node 
Testing flash...
Read Internal Flash ROM...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x01 0x20 0xd9 0x00 0x00 0x00 
  0x0008: 0x35 0x01 0x00 0x00 0x37 0x01 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Read External SPI Flash...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0008: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Flash OK
Rust test display
```

# Test with Newt Mananger

TODO

To understand the Firmware Update in more detail, we'll use Newt Manager to retrace the steps...

https://github.com/lupyuen/pinetime-rust-mynewt/blob/master/scripts/nrf52/test-dfu.sh

```bash
# Build Newt Manager on Raspberry Pi
# cd ~/go
# mkdir -p src/mynewt.apache.org
# cd src/mynewt.apache.org/
# git clone https://github.com/apache/mynewt-newtmgr
# mv mynewt-newtmgr newtmgr
# cd newtmgr/newtmgr
# export GO111MODULE=on
# go build

# Run Newt Manager on Raspberry Pi
cd ~/go/src/mynewt.apache.org/newtmgr/newtmgr

# Step 1: Creating a Newtmgr Connection Profile
# PineTime sets and advertises "pinetime" as its bluetooth device name. 
# Run the newtmgr conn add command to create a newtmgr connection profile 
# that uses this peer name to communicate with the device over BLE:

sudo ./newtmgr conn add pinetime type=ble connstring="peer_name=pinetime"
# Connection profile pinetime successfully added

# Connect to PineTime and list firmware images
sudo ./newtmgr image list -c pinetime --loglevel debug

# Step 2: Uploading an Image to the Device
# We upload my_sensor_app_1.1.img (downloaded from https://github.com/lupyuen/pinetime-rust-mynewt/releases/tag/v4.1.2)

# sudo ./newtmgr image upload -c pinetime \
#    ~/my_sensor_app_1.1.img 

# Connect to PineTime and list firmware images
# sudo ./newtmgr image list -c pinetime

# Step 3: Testing the Image
# The image is uploaded to the secondary slot but is not yet active. 
# We must run the newtmgr image test command to set the image status to pending 
# and reboot the device. When the device reboots, the bootloader copies this image 
# to the primary slot and runs the image.

# Set my_sensor_app_1.1.img to pending
# sudo ./newtmgr image test -c pinetime \
#    66a23f4f8f5766b5150711eb8c7c4be326cebabef37429fd21879f6e0eacffe5

# Connect to PineTime and list firmware images
# sudo ./newtmgr image list -c pinetime

# echo Reboot PineTime now

# Step 4: Confirming the Image
# Run the newtmgr image confirm command to confirm and make the uploaded image permanent. 
# Since the uploaded image is currently the active image, we can confirm the image setup 
# without specifying the image hash value in the command:

# sudo ./newtmgr image confirm -c pinetime

# Connect to PineTime and list firmware images
# sudo ./newtmgr image list -c pinetime

exit

# Log

# Upload image and set pending:

+ cd /home/ubuntu/go/src/mynewt.apache.org/newtmgr/newtmgr
+ sudo ./newtmgr conn add pinetime type=ble connstring=peer_name=pinetime
Connection profile pinetime successfully added
+ sudo ./newtmgr image list -c pinetime --loglevel debug
DEBU[2020-05-19 04:46:13.693] Using connection profile: name=pinetime type=ble connstring=peer_name=pinetime 
DEBU[2020-05-19 04:46:14.023] Connecting to peer                           
DEBU[2020-05-19 04:46:14.244] Exchanging MTU                               
DEBU[2020-05-19 04:46:14.256] Exchanged MTU; ATT MTU = 256                 
DEBU[2020-05-19 04:46:14.256] Discovering profile                          
DEBU[2020-05-19 04:46:14.503] Subscribing to NMP response characteristic   
DEBU[2020-05-19 04:46:14.518] {add-nmp-listener} [bll_sesn.go:392] seq=66  
DEBU[2020-05-19 04:46:14.519] Encoded &{NmpBase:{hdr:{Op:0 Flags:0 Len:0 Group:1 Seq:66 Id:0}}} to:
00000000  a0                                                |.| 
DEBU[2020-05-19 04:46:14.519] Encoded:
00000000  00 00 00 01 00 01 42 00  a0                       |......B..| 
DEBU[2020-05-19 04:46:14.519] Tx NMP request: 00000000  00 00 00 01 00 01 42 00  a0                       |......B..| 
DEBU[2020-05-19 04:46:14.542] rx nmp response: 00000000  01 00 00 86 00 01 42 00  bf 66 69 6d 61 67 65 73  |......B..fimages|
00000010  9f bf 64 73 6c 6f 74 00  67 76 65 72 73 69 6f 6e  |..dslot.gversion|
00000020  65 31 2e 30 2e 30 64 68  61 73 68 58 20 70 3e bb  |e1.0.0dhashX p>.|
00000030  f8 11 45 8b 1f ad 18 9e  64 e3 a5 e0 f8 09 cb e6  |..E.....d.......|
00000040  ba d8 83 c7 6b 3d d7 12  79 1c 82 2f b5 68 62 6f  |....k=..y../.hbo|
00000050  6f 74 61 62 6c 65 f5 67  70 65 6e 64 69 6e 67 f4  |otable.gpending.|
00000060  69 63 6f 6e 66 69 72 6d  65 64 f5 66 61 63 74 69  |iconfirmed.facti|
00000070  76 65 f5 69 70 65 72 6d  61 6e 65 6e 74 f4 ff ff  |ve.ipermanent...|
00000080  6b 73 70 6c 69 74 53 74  61 74 75 73 00 ff        |ksplitStatus..| 
DEBU[2020-05-19 04:46:14.542] Received nmp rsp: &{NmpBase:{hdr:{Op:1 Flags:0 Len:134 Group:1 Seq:66 Id:0}} Rc:0 Images:[{NmpBase:{hdr:{Op:0 Flags:0 Len:0 Group:0 Seq:0 Id:0}} Image:0 Slot:0 Version:1.0.0 Hash:[112 62 187 248 17 69 139 31 173 24 158 100 227 165 224 248 9 203 230 186 216 131 199 107 61 215 18 121 28 130 47 181] Bootable:true Pending:false Confirmed:true Active:true Permanent:false}] SplitStatus:N/A} 
DEBU[2020-05-19 04:46:14.543] {remove-nmp-listener} [bll_sesn.go:392] seq=66 
Images:
 image=0 slot=0
    version: 1.0.0
    bootable: true
    flags: active confirmed
    hash: 703ebbf811458b1fad189e64e3a5e0f809cbe6bad883c76b3dd712791c822fb5
Split status: N/A (0)

+ sudo ./newtmgr image upload -c pinetime /home/ubuntu/my_sensor_app_1.1.img
 205.27 KiB / 205.27 KiB [=====================================================================] 100.00% 6.47 KiB/s 31s
Done

+ sudo ./newtmgr image list -c pinetime
Images:
 image=0 slot=0
    version: 1.0.0
    bootable: true
    flags: active confirmed
    hash: 703ebbf811458b1fad189e64e3a5e0f809cbe6bad883c76b3dd712791c822fb5
 image=0 slot=1
    version: 1.1.0
    bootable: true
    flags: 
    hash: 66a23f4f8f5766b5150711eb8c7c4be326cebabef37429fd21879f6e0eacffe5
Split status: N/A (0)

+ sudo ./newtmgr image test -c pinetime 66a23f4f8f5766b5150711eb8c7c4be326cebabef37429fd21879f6e0eacffe5
Images:
 image=0 slot=0
    version: 1.0.0
    bootable: true
    flags: active confirmed
    hash: 703ebbf811458b1fad189e64e3a5e0f809cbe6bad883c76b3dd712791c822fb5
 image=0 slot=1
    version: 1.1.0
    bootable: true
    flags: pending
    hash: 66a23f4f8f5766b5150711eb8c7c4be326cebabef37429fd21879f6e0eacffe5
Split status: N/A (0)

+ sudo ./newtmgr image list -c pinetime
Images:
 image=0 slot=0
    version: 1.0.0
    bootable: true
    flags: active confirmed
    hash: 703ebbf811458b1fad189e64e3a5e0f809cbe6bad883c76b3dd712791c822fb5
 image=0 slot=1
    version: 1.1.0
    bootable: true
    flags: pending
    hash: 66a23f4f8f5766b5150711eb8c7c4be326cebabef37429fd21879f6e0eacffe5
Split status: N/A (0)

# Reboot PineTime...

# First update failed:

Starting Bootloader...
Displaying image...
Image displayed
Button: 0
[INF] Primary image: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[INF] Scratch: magic=unset, swap_type=0x1, copy_done=0x3, image_ok=0x3
[INF] Boot source: primary slot
[INF] Swap type: test
Button: 0
Starting Bootloader...
Displaying image...
Image displayed
Button: 0
[INF] Primary image: magic=good, swap_type=0x2, copy_done=0x1, image_ok=0x3
[INF] Scratch: magic=bad, swap_type=0x1, copy_done=0x2, image_ok=0x2
[INF] Boot source: none
[INF] Swap type: revert
Button: 0
Button: 0
Bootloader done
TMP create temp_stub_0
NET hwid 4a f8 cf 95 6a be c1 f6 89 ba 12 1a 
NET standalone node 
Testing flash...
Read Internal Flash ROM...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x01 0x20 0xd9 0x00 0x00 0x00 
  0x0008: 0x35 0x01 0x00 0x00 0x37 0x01 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Read External SPI Flash...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0008: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Flash OK
Rust test display

# Second update OK:

Starting Bootloader...
Displaying image...
Image displayed
Button: 0
[INF] Primary image: magic=good, swap_type=0x4, copy_done=0x1, image_ok=0x1
[INF] Scratch: magic=bad, swap_type=0x1, copy_done=0x2, image_ok=0x2
[INF] Boot source: none
[INF] Swap type: test
Button: 0
Button: 0
Bootloader done
TMP create temp_stub_0
NET hwid 4a f8 cf 95 6a be c1 f6 89 ba 12 1a 
NET standalone node 
Testing flash...
Read Internal Flash ROM...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x01 0x20 0xd9 0x00 0x00 0x00 
  0x0008: 0x35 0x01 0x00 0x00 0x37 0x01 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Read External SPI Flash...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0008: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Flash OK
Rust test display

# Confirm image:

+ sudo ./newtmgr image confirm -c pinetime
Error: 1

+ sudo ./newtmgr image list -c pinetime
Images:
 image=0 slot=0
    version: 1.1.0
    bootable: true
    flags: active confirmed
    hash: 66a23f4f8f5766b5150711eb8c7c4be326cebabef37429fd21879f6e0eacffe5
 image=0 slot=1
    version: 1.0.0
    bootable: true
    flags: 
    hash: 703ebbf811458b1fad189e64e3a5e0f809cbe6bad883c76b3dd712791c822fb5
Split status: N/A (0)

# Reboot:

Starting Bootloader...
Displaying image...
Image displayed
Button: 0
[INF] Primary image: magic=good, swap_type=0x2, copy_done=0x1, image_ok=0x1
[INF] Scratch: magic=bad, swap_type=0x1, copy_done=0x2, image_ok=0x2
[INF] Boot source: none
[INF] Swap type: none
Button: 0
Button: 0
Bootloader done
TMP create temp_stub_0
NET hwid 4a f8 cf 95 6a be c1 f6 89 ba 12 1a 
NET standalone node 
Testing flash...
Read Internal Flash ROM...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x01 0x20 0xd9 0x00 0x00 0x00 
  0x0008: 0x35 0x01 0x00 0x00 0x37 0x01 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Read External SPI Flash...
Read 0x0 + 20
  0x0000: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0008: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0010: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
  0x0018: 0x00 0x00 0x00 0x00 0x00 0x00 0x00 0x00 
Flash OK
Rust test display

```

# Further Reading

[Check out the other PineTime articles](https://github.com/lupyuen/pinetime-rust-mynewt/blob/master/README.md)

[RSS Feed](https://lupyuen.github.io/rss.xml)