# Domoticz-Pi-USB8-relay-board-SOS-solutions
Integrate 8-channel relay board with ATMEL8 from SOS solutions or Ebay/Ali into Domoticz on Raspberry Pi

## Which board is this about?
I got this board from https://www.sossolutions.nl/8xusbrelais but they are also available on Ebay and Aliexpress.
In the producht description there was a crude Youtube video explaining how to get this board to work.<br>
However, the board in the video had a FTDI 245R controller chip and the board I got just had an Atmel8 microcontroller on board.
It is recognized as vendor=0x16c0 and product=0x05df.

## How to get this board to work at all
Luckily, someone already did most of the work: http://vusb.wikidot.com/project:driver-less-usb-relays-hid-interface and https://github.com/pavel-a/usb-relay-hid

This Github is how to integrate this relay board into Domoticz running on a Raspberry Pi 3. This is more a reminder to myself how I got it working and it may be wholefully incomplete, so your mileage may very.

If you know your Linux stuff just follow the above mentioned Github and build it yourself. If not, use this description for guidance.

## General control scheme and usage
What you will do in the end is control the seperate relays on this board via virtual switches in Domoticz.
This board has a certain firmware on board that emulates USB and listens to a number of commands.
In the On and Off action of the virtual switch you point to a script while passing parameters.

Example to turn on relay 1 of board id ABCDE:<br>
usbrelay/hidusb-relay-cmd id=ABCDE ON 1<br>

Example to turn off relay 2 of board id ABCDE:<br>
usbrelay/hidusb-relay-cmd id=ABCDE OFF 2<br>

## Step by Step instructions:

### Step 1: plug in the board and check if it is the right one
Plug in the relay board on your Pi and log in with a SSH shell (F.i. with Putty).

type: `sudo lsusb -v | more`
This will list all devices on USB.
With the spacebar you can check if the board there.
You should see this:
```
idVendor           0x16c0
idProduct          0x05df
iManufacturer      1 www.dcttech.com
iProduct           2 USBRelay8
```

### Step 2: Add the USB device to a udev rule so it can be accessed by Domoticz
To grant access to non-root users, define udev rule for the devices:<br>
Example: `SUBSYSTEMS=="usb", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="05df", MODE:="0666" `

### Step 3 for Non Raspberry PI -> Get and build the source 
F.i.:
```
git clone https://github.com/pavel-a/usb-relay-hid.git
cd commandline/makemake && make
```

### Step 3 for Raspberry PI -Copy compiled source to correct folder
Copy the files in the usbrelay folder to the Domoticz scripts folder (/domoticz/scripts/usbrelay).

### Step 4: Set permissions
you may need to set some file permissions so everything can execute properly.

### Step 5: Find the serial number of the relay board
In the SSH shell go to the usbrelay folder and type:<br>
`usbrelay-cmd ENUM`<br>
You will get something like this:<br>
`Board ID=[6EDX8] State: 00 (hex)`<br>
The ID is your boards' serial number. Write this down.<br>

### Step 6: Add the virtual switches in Domoticz
In Domoticz you need to create one virtual switch for every relay.<br>
What I normally do is to create a hardware device per board. <br>
'Setup->Hardware->Add Type:Dummy'<br>
When this hardware is added, you can add virtual devices of type 'Switch'.<br>
So one virtual switch per relay. If the board has 8 relays, you create 8 virtual switches.<br>

### Step 7: Set the ON/OFF action for every virtual switch
Now go to the 'Switches' tab in Domoticz and do this for each virtual switch;<br>
Go to 'Edit' and fill in the following for the 'On' and 'Off action';<br>
`script://usbrelay/hidusb-relay-cmd id=[device ID] [ON or OFF action] [relay number 1-8]`<br>
So for relay 1 of device ID 6EDX8;<br>
'On action':<br>
`script://usbrelay/hidusb-relay-cmd id=6EDX8 ON 1`<br>
And for the 'Off' action:<br>
`script://usbrelay/hidusb-relay-cmd id=6EDX8 OFF 1`<br>



