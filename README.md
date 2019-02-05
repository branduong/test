**Copyright (c) 2016, Michael Ogg**
# DoorOpener
## Table of Contents
* [Abstract](#Abstract)
* [Basic Methodology](#Basic-Methodology)
* [Main Components](#Main-Components)
* [Installation](#Installation)
* [Usage](#Usage)
* [Hardware Choices](#Hardware-Choices)
* [Images](#Images)
* [License](#License)

## Abstract

DoorOpener was designed by Dr. Michael Ogg with the intention of creating a hands-free method of entering or exiting one’s house. It was documented by the Ryerson Chapter of Tetra. DoorOpener is built for a user with limited or no limb mobility but has wheelchair access. It is based around the use of a Raspberry Pi, a Z-wave Home Automation Controller, RFID tags and pressure plates. The objective of having the code, as well as additional information, on Github is to allow users to copy and modify Dr. Ogg’s solution for their own purposes.

This project assumes you have a basic understanding of the following concepts: electrical work, Raspberry Pi, Z-wave controllers, and Python. The files are designed so that you can copy and paste this solution almost directly with only minor changes to the config file, given that the hardware is laid out in the same manner. 

We hope that for whatever reason, whether it be as an electronic hobbyist or as someone working in accessibility services, you can make use of Dr. Ogg’s solution. If you’d like to see a video of the DoorOpener, please click here: https://www.youtube.com/watch?v=-B-Js19Npv0


## Basic Methodology
DoorOpener relies on two main controllers: the Raspberry Pi, which controls the majority of the project, and the Vera MiCasaVerde Z-wave controller, which controls the door lock.

The RFID tag readers control if the door is locked or unlocked (based on commands sent from the Pi to the Vera to the Z-wave door lock). Since there is a reader on both the inside and outside of the house, it is possible to change the state of the lock from either side of the door. 

The pushplates activate the door opening by activating the latch strike relay and then door operator relay through the Pi. The door closes automatically after a set period of time (declared in the CONFIG file). In addition to the pushplates, there is a Z-wave relay (commonly referred to as a "Compatabile Contact Closure") which can open the door through a Z-wave app (controlled by a device such as an iPhone or Android). The pushplates and the Z-wave relay are all wired in parallel to the Pi via a 3.5 mono jack and a Swifty USB.

The Pi communicates with the Vera via URL commands (based on API given from Vera) sent via WiFi. This allows the Pi to control the door locker and is the middleman between the Z-wave relay and the Vera itself.

The Pi itself is controlled through threaded Python files, which means all the components are running simultaneously.




## Main Components
* `Raspberry Pi` : Main controller that connects to the Vera MiCasaVerde and all other components.
* `RFID Reader` x2 : Device used to read RFID tags.
* `Latch Strike Relay` : Relay used to open the door latch. (Contained in relay box).
* `Door Operator Relay` : Similar to the strike relay. Used to signal the door operator to begin opening. (Contained in relay box). 
* `Z-wave Door Lock` : A Z-wave controlled lock which locks/unlocks based on the RFID readers.
* `Vera MiCasaVerde` : Z-Wave hub that controls the door lock.
* `Z-Wave Compatible Contact Closure (Z-wave Relay)` : a relay that allows the door to be opened and close via Z-wave.
* `Pushplates` x4 : In this project, four pushplates are used as switches to actuate the door open relay. One is installed at an average height and the other is installed at foot level height both inside and outside the house.
* `Swifty USB Contact Sensor` : a USB switch interface which detects contact closure. Takes a 3.5 mono jack input.

More information on the components can be found under the [Hardware Choices](#Hardware-Choices) section. Pictures of this set up have been included under [Images](#Images).


### Hardware Schematic
![tetra - hw schematc - final v2](https://user-images.githubusercontent.com/20260964/51933115-96198200-23ce-11e9-80db-69c8a9657aa1.png)


* 'PP' refers to pushplate.
* The black icon Z icon implies the devices using Z-wave.



## Installation

DoorOpener is designed to perform on any Pi with Wi-Fi compatibility or an ethernet port and the correct number of USB ports.
It is also designed to be used with a Vera Z-wave controller, but ultimately, any Z-wave controller that can be controlled with HTTP commands can be used.

Although the source code can be implemented on other devices with minor alterations, the following installation guide is intended for use with the aforementioned devices specifically.


1. Go to https://home.getvera.com/users/login and register your Vera controller (your home router must be enabled for port forwarding).

1. Pair your z-wave door contact sensor to your vera hub using the following link: https://support.getvera.com/customer/en/portal/articles/2949040-how-to-pair-z-wave-devices?b_id=712. Connect the sensor terminals to a usb connector and plug it into port ttyUSB3 of your Pi device.

1. Connect the two RFID readers to ports ttyUSB0 and ttyUSB1 on your Pi device.

1. Add the relay device to port ttyUSB2 on your Pi device and wire those relays accordingly to your pre-existing door automation system.

1. Turn on your Raspberry Pi device then download the Door Opener source code and save it into your home directory.

1. To initiate the Door Opener software one must run doormain.py and this can be done by opening a terminal windown on your Raspberry Pi and typing in the following commands:

```
cd /home/DoorOpener
python doormain.py
```

These commands will run the code once. If you would like to run the code continuously without having to retype these commands after each reboot, use the follwing commands instead:

```
cd /home/DoorOpener
python doormain.py
```

followed by:

```
sudo reboot
```

to reboot the device.

## Usage
The DoorOpener software is self contained and complete, thus it doesn't require additional scripts or libraries to function. Depending on your hardware configuration, you may need to adjust several parts of the CONFIG file. As well, you must include your own RFID tag keys in the rfid.db. The following subsections explain the individual parts of the CONFIG file.

  #### Device Ports
 
 Please consult Pi documentation online for specific port names. Change the section of code after /dev/ (e.g. /dev/**ttyUSB0**) as required.

  #### Relay triggers / door timings
  

  In the case where the timing of relay triggers and open delays are not optimal or do not function as desired, the following lines of the CONFIG file can be edited:

  ```
  strikeRelay /dev/ttyUSB2 1 4    # relay 1, hold 4 sec
  doorRelay   /dev/ttyUSB2 2 4    # relay 2, hold 1 sec
  openDelay   1.5         # time delay for door to open
  ```
  The strike relay holds the latch strike for 4 seconds which gives the door relay plenty of time to initiate the door operator sequence.

  ##### Contact Closure
  The contactSensor refers to the Z-wave Contact Closure (Relay). 
  
  ```
  contactSensor	/dev/input/mouse0 9 0 0	# close [9,0,0], open [8,0,0]
  # contactSensor	/dev/hidraw0 1 0 0	# close [1,0,0], open [0,0,0]
  ```
  You'll notice the contact closure is listed as a mouse, despite not being such a device. It so happens that the Pi can easily read the output of the contact closure easily (the 3 number sequences) as such when it is listed as a mouse, keyboard, or HIDRAW (as seen in the commented out line).
  
  HIDRAW refers to human interface device (raw), which could also be used instead of 'mouse'. However, if listing the contact closure as a hidraw, the other two 3-number sequences must be used.

  #### Vera address
  
  ```
  vera		http://192.168.1.13:3480
  ```
 Please consult your Vera documentation for obtaining the URL of your device. 
 
 
  
  #### doorID
  doorID refers to the device number given to the door lock.
  
  ```
  doorId		16
  ```
  
You'll have to access the Vera homepage via a browser to find out your door lock ID. Consult your Vera documentation to find the appropriate address / URL, and there will be a list of devices that can be connected or are already connected. The door lock ID will be listed there with a number.
 
 #### RFID tags
  
All valid RFID key tags are stored in the rfid.db file and it can be opened in any plain text editor to add or remove key tags.

## Hardware Choices
The intention of this section is to explain why the specific pieces of hardware were used, as well as to explain in further depth the particular components.
Please note that although some decisions would not make sense from the perspective of a pure electronic hobbyist, the intention of the project was to make it as easy as possible to set up. This resulted in the purchase of a few extra components (such as the relay box) to minimize soldering / wiring and maximize the amount of 'plug-and-play'. This also resulted in increase portability across platforms.

* `Raspberry Pi 2` 

A Pi 2 was used with a Wi-Fi dongle, although a Pi 3 was used with built in wi-fi compatibility later by Dr. Ogg. Any Pi would do long as it has Wi-Fi or ethernet capabilities. It was used because of it's low-cost and minimal power usage, although in theory, any microprocessor / small computer could be used.


* `RFID Reader`

A 125 kHz RFID reader was used. It had a solid range of 3 inches and a USB interface (via its FTDI connection) which was easily read out into a Linux TTY device. The specific RFID reader was used because it appears as a virtual COM (making it easy to interface with the Pi) and no driver installation was needed.

The following RFID system is composed of the following components:
Reader: https://www.sparkfun.com/products/9963
Chip: https://www.sparkfun.com/products/11828 (this plugs into the above reader).
Tags: https://www.sparkfun.com/products/retired/8310
*The specific Sparkfun tags are retired, but any 125 kHz tag will do. Ensure that the tags you purchase are of the correct frequency.*

* `USB Relay Box`

Although the Pi has GPIO pins that the Latch Strike and Door Operator relays could connect directly, as mentioned previously, the priority was ease of set-up and use. This Relay box which connects via USB to the Pi allows the user to control two devices from the Pi directly. Please note that you still need to wire the Latch Strike and Door Operator into the relay correctly.

https://www.kmtronic.com/usb-relays.html?product_id=55

For information about wiring a device to the relay, we recommend reading the following article:
https://electronics.stackexchange.com/questions/30952/can-you-clarify-what-an-1no1nc-switch-is


* `Latch Strike Relay`

An electric latch strike was required to strike the latch of the door before the door can open. (Otherwise, the door operator would not be able to function as the door would get jammed on the latch.) The electric latch strike allows the door latch to be flush against the surface of the door before the door opens. 
This is wired into the USB Relay Box. The relay activates the latch strike.

Dr. Ogg used a Securitron UNL-24.
https://www.assaabloyesh.com/en/local/assaabloyeshcom/products/electric-strikes/securitron-unl/

* `Door Operator Relay`

The door operator relay sends a signal to the door operator to begin the opening process. This signal does not need to be sustained for the entire operation of the opening sequence.
The door operator is what physically opens the door once it is unlocked and the latch is flush with the door. The following operator was used:
https://www.amazon.com/gp/product/B00QUORLVQ/ref=oh_aui_search_asin_title?ie=UTF8&psc=1

* `Z-wave Door Lock`

Several brands of Z-wave locks are available. The specific brand will not make a difference to the user as the protocols are encapsulated in the Z-wave protocols, not the specific lock.

In DoorOpener, a Kwikset 910 mortise lock was used. For more information on this type of lock, see:
https://www.kwikset.com/products/styles/smartcode-deadbolt-with-home-connect.aspx

* `VeraLite MiCasaVerde`

Though an older model, Dr. Ogg was familiar with the VeraLite. Although newer Z-wave controllers are on the market, ensure that the Z-wave controller can be programmatically controlled via HTTP commands.
More information on the device itself can be found here:
https://getvera.com/controllers/veralite/

Useful API information is available at the following links:
http://wiki.micasaverde.com/index.php/Luup_Device_Categories
http://wiki.micasaverde.com/index.php/Luup_Requests#action

In order to see all Z-wave devices available (such as the door lock), please use the following link format:
http://< IP address of hub>:3480/data_request?id=sdata&output_format=json
*<IP address of hub> must be replaced with the actual IP address of your Z-wave controller.

* `Z-Wave Compatible Contact Closure` 

This Contact Closure, also referred to as a Z-wave relay, allows the user to close a circuit connection through Z-wave. This is not required, but it allows the door to be opened via a Z-wave App. It is wired in parallel with the pushplates to the Swifty USB.
https://www.evolvecontrols.com/wp-content/uploads/2016/03/LFM-20.pdf

The specific app that is used by Dr. Ogg in conjunction with this is called HomeWave:
http://homewave.intvelt.com/

* `Pushplates` 

These are the exact same type of pushplate you find in many buildings with the wheelchair accessibility symbol on them. They are wired in parallel via speaker cable through the walls and connect to the 3.5 mono jack (which, in turn, acts as the input for the Swift USB). When the pushplate is pressed it completes the circuit with the Swifty USB and sends a signal to the Pi.

It is worth noting a contractor was needed to complete the wiring through the walls.

* `Swifty USB Contact Sensor` 

The Swifty USB is a device that allows the detection of a circuit closure. This means that when either a pushplate is pressed or the Z-wave relay is used, a signal is sent to the Pi to initiate door opening. It takes a 3.5 mono jack which is wired to the pushplates and Z-wave relay.

http://orin.com/access/swifty/

* `Power`

For powering this, it will function with either a 12 or 24 VDC supply composed of 12 V SLA batteries. In DoorOpener, this would be used to power the latch strike and, if you desire, the door operator and as the USB power supply for the Pi.  There is also an Uninterrupted Power Supply used so that DoorOpener will function even if there is a power outage.

## Images

These are included within this Readme to give the reader an appreciation of what the final product looks like. Please note that certain components, such as the Vera MiCasaVerde and the latch strike are not visible.


![hw1](https://user-images.githubusercontent.com/20260964/51931965-230f0c00-23cc-11e9-824b-e25a54e095f2.jpg)
1) Door Operator
2) 24V DC Power Supply
3) Uninterrupted Power Supply
4) USB HUb
5) USB Relay Box
6) Raspberry Pi
7) Door lock



![hw2](https://user-images.githubusercontent.com/20260964/51931968-24403900-23cc-11e9-9770-ce22058e3d30.JPG)

8) Upper and lower pushplates 
9) RFID reader

This same setup is also on the outside of Dr. Ogg's house.


## License
This file is part of DoorOpener. DoorOpener is free software: you can
redistribute it and/or modify it under the terms of the GNU General
Public License as published by the Free Software Foundation, either
version 3 of the License, or (at your option) any later version.

DoorOpener is distributed in the hope that it will be useful, but
WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
General Public License for more details.

You should have received a copy of the GNU General Public License
along with DoorOpener. If not, see <http://www.gnu.org/licenses/>.

ogg.michael@gmail.com

http://michaelogg.com
