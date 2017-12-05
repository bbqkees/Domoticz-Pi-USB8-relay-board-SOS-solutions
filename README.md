# Domoticz-Pi-USB8-relay-board-SOS-solutions
Integrate 2/4/8-channel relay USB board with an ATMEL8 microcontroller from SOS solutions or Ebay/AliExpress into Domoticz on Raspberry Pi (3).

## Which board is this about?
I got this board from https://www.sossolutions.nl/8xusbrelais but they are also available on Ebay and Aliexpress.
In the producht description there was a crude Youtube video explaining how to get this board to work.<br>
However, the board in the video had a FTDI 245R controller chip and the board I got only had an Atmel8 microcontroller on board.<br>
The board is recognized as vendor=0x16c0 and product=0x05df.

## How to get this board to work at all
Luckily, someone already did most of the work: <br>http://vusb.wikidot.com/project:driver-less-usb-relays-hid-interface and <br>https://github.com/pavel-a/usb-relay-hid

This particular Github page is a short guide how to integrate this relay board into Domoticz running on a Raspberry Pi 3. <br>This is more a reminder to myself how I got it working and it may be wholefully incomplete, so your mileage may very.

If you know your Linux stuff just follow the above mentioned Github and build it yourself. <br>If not, you can use this description for guidance.

## General control scheme and usage
What you will do in the end is control the separate relays on this board via virtual switches in Domoticz.<br>
This board has a certain firmware on board that emulates USB and listens to a number of commands.<br>
In the On and Off action of the virtual switch you point to a script while passing parameters.<br>
In turn this script/utility will connect to the Atmel controller via USB and set those relays for you.

Example to turn ON relay 1 of board id ABCDE:<br>
`script://usbrelay/hidusb-relay-cmd id=ABCDE ON 1`<br>

Example to turn OFF relay 2 of board id ABCDE:<br>
`script://usbrelay/hidusb-relay-cmd id=ABCDE OFF 2`<br>

## Step by Step instructions:

### Step 1: plug in the board and check if it is the right one
Plug in the relay board to a free USB port on your Pi and log in to the Pi via a SSH shell (F.i. with Putty).

type: `sudo lsusb -v | more`<br>
This will list all devices on USB.<br>
With the spacebar you can check if the board there.<br>
You should see something like this somewhere in the list:
```
idVendor           0x16c0
idProduct          0x05df
iManufacturer      1 www.dcttech.com
iProduct           2 USBRelay8
```
If not, you cannot use this method. If it is there, you can continue.

### Step 2: Add the USB device to a udev rule so it can be accessed by Domoticz
To grant access to non-root users, define an udev rule for the devices:<br>
Example: `SUBSYSTEMS=="usb", ATTRS{idVendor}=="16c0", ATTRS{idProduct}=="05df", MODE:="0666" `<br>
(you need to put this in a udev file, do not copy and paste it into the terminal. <br>Google it if you don't know how this works)<br>

### Step 3 for Non Raspberry PI -> Get and build the source 
F.i.:
```
git clone https://github.com/pavel-a/usb-relay-hid.git
cd commandline/makemake && make
```
(Keep in mind the actual source is not from my Github page)

### Step 3 for Raspberry PI -Copy compiled source to correct folder
Copy the files from the usbrelay folder on this Github page to a new subfolder in the Domoticz scripts folder (/domoticz/scripts/usbrelay).

### Step 4: Set permissions
you may need to set some file permissions so everything can execute properly.

### Step 5: Find the serial number of the relay board
In the shell go to the usbrelay folder and type:<br>
`usbrelay-cmd ENUM`<br>
You will get something like this:<br>
`Board ID=[6EDX8] State: 00 (hex)`<br>
The ID is your boards' serial number. Write this down.<br>
If you have more of these boards connected, you will see more boards in the list.

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

When you are done, you can use these virtual switches to control the relays as you would do with any other switch in Domoticz.<br>Domoticz will execute the function in the background for every switch action.<br>

Note: If you put the files somewhere else, change the location in the above commands.<br>
Domoticz uses the domoticz scripts folder as root.

## Checking relay status
With the command `usbrelay-cmd STATUS` you will get the status of all relays in HEX format.<br>
So if relay 2 and 4 are ON while the rest is OFF, the response will be `State: 0A (hex)`.<br>
'OA' HEX is 0b00001010, You can see this corresponds with relay 2 and 4.<br>
When Domoticz executes the script, it does not check whether the relay has actually turned on or off.<br>
If you need this, you need to write custom script that checks the relay status after the switch action with the 'STATUS' parameter.

## Additional commands
```
usbrelay-cmd id=ABCDE ON ALL
usbrelay-cmd id=ABCDE OFF ALL
```
If you omit the device ID, the script will execute the command for the first ID it found.<br>
So if you have more of these boards connected, you need to include the device ID.



