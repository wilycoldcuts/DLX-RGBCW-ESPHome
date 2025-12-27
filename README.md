# DLX-RGBCW-ESPHome
A repo of notes and sample configs for using the Tuya "Smart WiFi Controller" DLX-RGBCW BK7231N with ESPHome.

Last updated December 2025

# Background
I like LED strips. I used to control mine with a series of ESP32-based controllers from [Electrodragon](https://www.electrodragon.com/product/esp-led-strip-board/), but they've gotten flaky over time and later ESPHome updates broke their compatibility (early revisions only had 2MB of storage, but I don't feel like writing new partition tables after an update ate mine.  Lesson learned: I need to keep better notes for this stuff!)

I'm replacing them with these ["Smart WiFi Controller"](https://www.aliexpress.com/item/1005006450606459.html) devices from AliExpress which are smaller and include handy toggle buttons.

# Tuya Smart WiFi Controller
I'm using two different variants: one with 4-wire RGB and one with a single-channel for controlling strips of white LEDs.

<img src="/images/1_4wire.png" width="200"> <img src="/images/2_singlechannel.png" width="200">

After prying open the case with a flat spudger, both variants use the same PCB. Mine are silkscreened as follows:

```
DLX-RGBCW BK7231N-B 5-24V
2022/03/01 Rev 2.6
FR-4 Size 1.2mm
```

## Replacing the stock Tuya firmware with OpenBeken
[OpenBK7231T/OpenBeken](https://github.com/openshwprojects/OpenBK7231T_App) is a project to provide a Tasmota-like alternative firmware for Tuya-based devices like these. They build upon the great work of the Cloudcutter project, although I couldn't Cloudcutter to work because these specific Tuya devices may have been patched. That's okay though - it's pretty easy to flash these manually:

1. Open the device by removing the bottom cover with a spudger or flat shim.
2. Carefully remove the PCB and turn it over.
3. Add a bit of soldering flux and then temporarily solder jumper wires to the following points:
   ```
   GND = GND
   3.3V = 3.3V
   TP4 = TX
   TP5 = RX
   ```
   <img src="/images/3_pcb.png" width="400">
4. Connect the jumper wires to a 3.3V FTDI cable as follows (remember that with FTDI cables, TX on one device must connect to RX on the other, and vice-versa):
   ```
   FTDI Cable     DLX-RGBCW
   GND  <------>  GND
   3V3  <------>  3V3
   TX   <------>  TP5 (RX)
   RX   <------>  TP4 (TX)
   ```
   <img src="/images/4_ftdi.png" width="400">
5.  Download the [BK7231GUIFlashTool](https://github.com/openshwprojects/BK7231GUIFlashTool) and run it.
6.  Check the box **"Automatically configure OBK on flash write"**. Click the "Change OBK settings for flash write" button and input your wifi SSID and password.
    * You can optionally provide your MQTT server and credentials here if you want to use the firmware as-is.
    * If you forget to do this step: after flashing is finished, join the broadcast SSID from your phone, then browse to 192.168.4.1 to configure the wifi credentials manually.
   <img src="/images/5_checkobk.png" width="400">
   <img src="/images/6_obksettings.png" width="400">
7.  Close the OBK settings window and run the flasher with the following settings:

    * Select UART port: (pick whatever shows up when you connect your FTDI cable)
    
    * Select chip type: BK7231N
    
    * Select firmware: (Click "Download latest from Web")
    
    * Set baud rate: 230400
    
    * Click "Backup and flash new"
    
8.  Click **"Backup and flash new"**. When prompted for a name, you can leave it blank and click **OK**.
    <img src="/images/7_flashersettings.png" width="400">
9.  When you see "Getting bus... (now, please do reboot by CEN or by power off/on)", carefully disconnect the 3V3 jumper from your FTDI cable and then reconnect it. (The reboots the device without resetting the serial connection.)
10.  When finished, it'll display the pin assignments as reported by the stock firmware. Keep this info because it might come in handy later.
    <img src="/images/8_gpioreport.png" width="400">
11.  Disconnect the FTDI cable and connect the device to mains power to boot normally.

## Replacing OpenBeken firmware with ESPHome
(Tested with ESPHome 2025.12)
1.  In your local ESPHome Builder, create a new device using the **"New Device Setup"** guided process.
2.  Pick a hostname. I like to preface mine with "esphome-" to keep things tidy and organized in my DNS table. Click **Next**.
3.  Select the **"BK72xx"** device type.
4.  Select **"Generic - BK7231N (Tuya QFN32)"**. Click **Next**.
5.  It'll show your ESPHome encryption key, but you don't need this right now. Click **Skip**.
6.  Find your new device config and click **"Edit"**.
7.  Update your YAML config using one of the examples as a guide. The key is to make sure your pin assignments are correct:

    **4-wire RGB:**
    ```
    Blue pin: GPIO07
    Red pin: GPIO08
    Green pin: GPIO24
    Button pin: GPIO14
    ```
    **Single-channel power switch:**
    ```
    Output/white pin: GPIO08
    Button pin: GPIO14
    ```
9.  Click **Save**, then click **Install**.
10.  Click "Manual Download", then wait a few minutes for your ESPHome firmware to compile.
11.  When the compilation is finished, click "Beken OTA Image". It'll download a file named [hostname]-ota.rbl.
    <img src="/images/9_download.png" width="400">
13.  Find this file in download folder and rename it to `OpenBK7231N_esphome.rbl`. (The OTA interface won't accept ESPHome firmware unless you rename it!)
14.  Open a web browser and browse to the IP address of your device. (You may need to check the admin interface of your wifi network or router to find its IP address.)
15.  Click "Launch Web Application".
     <img src="/images/10_webapp.png" width="400">
17.  Click the "OTA" tab.
18.  Click Browse... and select OpenBK7231N_esphome.rbl from your downloads folder.
19.  Click Start OTA.
     <img src="/images/11_ota.png" width="400">
21.  When finished, check to make sure it appears "online" in ESPHome, then add it to your Home Assistant instance.
22.  If everything looks good, carefully de-solder the jumper wires and reassemble the case.

## 3D-printed case
I'm sticking mine to a metal shelf, so here's a basic mounting adapter I made. You can press-fit 6mm x 2mm magnets into the bottom.
<img src="/images/12_holder.png" width="400">
