# Issue with CH340 Driver in Linux


I just got an "ESP8266 D1 mini Dev Broad" by a friend of mine, and tried to play around. If you are not familiar, it is an MCU by [Espressif](https://www.espressif.com/) company, enhanced by extra features to make it a good dev board.

This dev board is really easy to use (as other Espressif products), specially because of Micro USB (or USB-C) port on it. So you can just connect it to your computer and flash the MCU. Attaching a USB port was possible as dev board has a CH340 [USB-to-UART](https://en.wikipedia.org/wiki/USB-to-serial_adapter) chip. The chip is responsible for converting USB to Serial communication.

Most Linux distributions (such as Ubuntu) already have the CH340 driver installed. You can check it like:
```bash
❯ lsmod | grep ch34
ch341                  24576  0
usbserial              57344  1 ch341
```

I connected the board to computer, selected correct board in [Arduino IDE](https://www.arduino.cc/en/software), but then noticed "Port" option is grayed out in the IDE (`Tools` menu). This means device is not connected or not accessible. Quick check showed that there's no USB device:
```bash
❯ ls /dev/ttyUSB*
zsh: no matches found: /dev/ttyUSB*
```
but at least `dmesg` says USB was there but disconnected!
```bash
❯ sudo dmesg| grep ch34
[12988.441725] usb 1-2: ch341-uart converter now attached to ttyUSB0
[13061.347165] ch341-uart ttyUSB0: ch341-uart converter now disconnected from ttyUSB0
[13061.347235] ch341 1-2:1.0: device disconnected
```

More investigations in `dmesg` shows kind of conflict by [brltty](https://brltty.app/) daemon. Easily you can remove it if you don't use it:
```bash
❯ sudo apt remove brltty
```
and if we connect the board again, and check `dmesg`, no disconnect has logged:
```bash
❯ sudo dmesg| grep ch34
[23964.629757] ch341 1-2:1.0: ch341-uart converter detected
[23964.630183] usb 1-2: ch341-uart converter now attached to ttyUSB0
```
and USB device is there:
```bash
❯ ls -l /dev/ttyUSB*
crw-rw---- 1 root dialout 188, 0 Feb 27 21:06 /dev/ttyUSB0
```

Now you can select `/dev/ttyUSB0` in Arduino IDE Port option and flash the MCU.

Enjoy hacking!
