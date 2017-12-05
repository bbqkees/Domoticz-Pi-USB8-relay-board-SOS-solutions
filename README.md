# Domoticz-Pi-USB8-relay-board-SOS-solutions
Integrate 8-channel relay board with ATMEL8 from SOS solutions or Ebay/Ali into Domoticz on Raspberry Pi

## Which board is this about?
I got this board from https://www.sossolutions.nl/8xusbrelais but they are also available on Ebay and Aliexpress.
In the producht description there was a crude Youtube video explaining how to get this board to work.
However, the board in the video had a FTDI 245R controller chip and the board I got just had an Atmel8 microcontroller on board.
It is recognized as vendor=0x16c0 and product=0x05df.

## How to get this board to work at all
Luckily, someone already did most of the work: http://vusb.wikidot.com/project:driver-less-usb-relays-hid-interface and https://github.com/pavel-a/usb-relay-hid

This Github is how to integrate this relay board into Domoticz running on a Raspberry Pi 3. This is more a reminder to myself how I got it working and it may be wholefully incomplete, so your mileage may very.

If you know your Linux just follow the above mentioned Github and build it yourself. Of not, use this description for guidance.

## General control scheme and usage
This board has a certain firmware on board that emulates USB and listens to a number of commands.
What you will do in the end is control the seperate relays on this board via virtual switches in Domoticz.
In the On and Off action of the virtual switch you point to a script while passing parameters.
Example to turn on relay 1 of board id ABCDE:
usbrelay/hidusb-relay-cmd id=ABCDE ON 1

Example to turn off relay 2 of board id ABCDE:
usbrelay/hidusb-relay-cmd id=ABCDE OFF 2

## Step by Step instructions:

### Step 1: plug in the board and check if it is the right one
Plug in the relya board on your Pi and log in with SSH.

type: 'sudo lsusb -v | more'
This will list all devices on USB.
With the spacebar you can check if the board there.
You should see this:
idVendor           0x16c0
idProduct          0x05df
iManufacturer      1 www.dcttech.com
iProduct           2 USBRelay8

### Step 2: Add the USB device to a udev rule so it can be accessed by Domoticz
To grant access to non-root users, define udev rule for the devices:
Example: SUBSYSTEMS=="usb", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="05df", MODE:="0666" 

### Step 3: Non Raspberry PI -> Get and build the source 
### Step 3: Raspberry PI -Copy compiled source to correct folder
Copy the files in the usbrelay folder to the Domoticz scripts folder (/domoticz/scripts).

### Step 4: Set permissions

### Step 5: Find the serial number of the relay board
go to the usbrelay folder and type:
'usbrelay-cmd ENUM'
You will get something like this:
'Board ID=[6EDX8] State: 00 (hex)'
The ID is your boards' serial number. Write this down.

### Step 6: Add the virtual switches in Domoticz

### Step 7: Set the ON/OFF action for every virtual switch




