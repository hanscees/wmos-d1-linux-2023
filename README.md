# wmos-d1-linux-2023
Short guide howto get wmos d1 or lillygo boards with ch341 working on linux ubuntu 22 jammy or later

After spending too much time troubleshooting some IOT esp8266 boards with Linux and arduino IDE here's a short write-up what worked for me. 

1. Two cheap Esp8266 Boards I have tested are: 
  A. LilyGO TTGO T-OI - ESP8266  see https://www.tinytronics.nl/shop/en/development-boards/microcontroller-boards/with-wi-fi/lilygo-ttgo-t-oi-esp8266-d1-mini-compatible?sort=p.price&order=ASC
  B. LilyGO TTGO T-Base ESP8266  see https://www.tinytronics.nl/shop/en/development-boards/microcontroller-boards/with-wi-fi/lilygo-ttgo-t-base-esp8266?sort=p.price&order=ASC

  For both boards you can get a new driver for Linux, but on Ubuntu they work out of the box IF YOU ADJUST SystemD! 
  Newest drivers are here: but I could not get them to work (they are not properly signed I think) https://github.com/WCHSoftGroup/ch341ser_linux

2. Most important: use a micro-usb DATA cable: unfortunately you cant see if a micro-usb cable has 2 wires (only for charging) or 4 wires. If you plug in the lillygo board and nothing happens: get a data cable.

3. How can you tell if Linux sees the board? Ok, so you first plug in a micro-usb cable into the lillygo esp8266 board and the other normal end of the usb-cable into a usb-connector of your linux laptop/ desktop. 
You can see if something happens like this:
  A: in a terminal type
  lsusb
  It should among other things show you a CH340 line: 
  Bus 002 Device 005: ID 1a86:7523 QinHeng Electronics CH340 serial converter

  B: lsmod should show a loaded driver
   In  a terminal type 
   sudo lsmod | egrep ch
   It should show something like this: 

   ch341                  24576  0
   usbserial              57344  1 ch341

   C: Probably this will not be alright: 
   In a terminal type:
   sudo ls /dev/ttyUS*

   should show: 
   /dev/ttyUSB0

4. If 3C above is alright you can go on and work with the arduino IDE
If not here is a solution I found here : 
https://askubuntu.com/questions/1431678/ch341-uart-converter-now-attached-to-ttyusb0-but-no-such-file-or-directory-de

Apparantly some braille service in systemd interferes here as you can see in dmesg:
If you type this in  a terminal:
sudo dmesg | egrep -v snap | egrep usb

you see these lines:
[  251.298918] usb 2-3.3: ch341-uart converter now attached to ttyUSB0
[  252.048615] usb 2-3.3: usbfs: interface 0 claimed by ch341 while 'brltty' sets config #1

Solution is to disable the brltty service like this:
Type this in a terminal: 

sudo systemctl disable brltty.service
sudo systemctl mask brltty-udev.service

It should react something like this: 
Created symlink /etc/systemd/system/brltty-udev.service → /dev/null.

Reboot and 3c should give you a good answer. Now you can follow any guide on starting with wmos d1 and arduino 

I like this one, since it told me to chack the Usb cable:
https://www.iotstarters.com/coding-the-wemos-d1-mini-using-arduino-ide/
