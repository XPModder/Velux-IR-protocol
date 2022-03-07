# Velux-IR-protocol
Reverse engineering of IR protocol to control Velux blinds

This project is about reverse engineering of the proprietary protocol used for IR-controlled blinds made by Velux.

Disclaimer: The name Velux and the names of its products are property of the company Velux. This project is created for fun and out of interest to interface with the blinds by different platforms. It is not used for profit in any way and is intended for personal use only.

Currently this project is about the blinds that are controlled by a WLR100 infrared remote control. The goal of the project is to completely reverse engineer the protocol used by this remote to control the blinds.

# General Info about the remote
This is a Velux WLR100 remote.
![IMG_20220307_174344 1](https://user-images.githubusercontent.com/30577392/157082603-7a202523-a616-402a-a29f-25d8eda04005.jpg)
![IMG_20220307_174354 1](https://user-images.githubusercontent.com/30577392/157082891-4d88f4cd-69f5-4bd7-a929-b0a825f8d22f.jpg)

Removing the battery cover reveals 2 sets of DIP-switches above the batteries. The inside of the battery cover has a sticker that shows the use of the switches.
![IMG_20220307_174419 1](https://user-images.githubusercontent.com/30577392/157082800-0d6527d9-7402-4a10-9ffd-8a135f5c6e1b.jpg)

The upper row of 4 switches controls the operating mode of the remote control. For the most part this seems to be purely changing the visual indication on the lcd. 

The only functional change that is controlled by these switches is that in the "AUTO" mode there will be the additional option of controlling all blinds at once. This option simply does not exist in the "MANUAL" mode.

As depicted on the sticker the 4th switch controls the visual indication on the lcd. In the default mode (upper picture) the blinds are organised into 10 sets of 3 motors (blinds) each. This gives a total number of 30 blinds. In the second mode the 30 blinds are simply accesable individually and not organised in these groups. This however is a purely visual change on the lcd of the remote.

The lower row of 10 switches sets a "security code" which has to match with the code set in the wall-mounted ir-receiver by a similar row of switches. Its purpose is to differentiate between multiple identical systems in one building, preventing the remote for the blinds in one room from accidentally controlling the blinds in another room.

This code is transmitted in every packet.

# Reverse engineering the IR-protocol
I have been searching for a way to control these blinds from something like an arduino or esp in order to automate them based on the environment like outdoor light level or time.

While there are many different arduino libraries available to send and receive IR-codes in order to control devices, none of these would recognize and decode the signal from the Velux remote and I was never able to recreate a signal to control the blinds.

It is however possible to find some universal remotes that can correctly replicate the signal once they have learned it.

Since I recently got my first Oscilloscope I decided to try to reverse engineer the seemingly proprietary protocol used by these remotes.

The protocol send a number of pulses all of the same length and duty cycle. Every pulse is a 13.2 us high-time followed by a 19.4us low-time. Thus resulting in a period of about 32 us.

![DS0010](https://user-images.githubusercontent.com/30577392/157088173-ddd1501c-ca0b-49d6-b8ad-591482223c84.PNG)
![DS0009](https://user-images.githubusercontent.com/30577392/157088199-a5ce68df-9bbd-4e1d-a2f0-cba3f4ee4662.PNG)

The information is not encoded in the pulses, but rather by the number of pulses. 

A 1 is encoded by 39 pulses and a 0 is encoded by 13 pulses.

A new bit starts every 1.69 ms.

![DS0011](https://user-images.githubusercontent.com/30577392/157089487-b74ecdf4-23cb-46ef-b15d-3d2afe1abc5c.PNG)
![DS0012](https://user-images.githubusercontent.com/30577392/157089509-18ab9d8f-1797-4ce7-bcc8-b4195806c1a5.PNG)
![DS0013](https://user-images.githubusercontent.com/30577392/157089532-c54ee8a1-6272-405b-83e4-7a0d1f408216.PNG)
![DS0014](https://user-images.githubusercontent.com/30577392/157089550-7ebf32ef-9bca-4795-a740-61d49f030d88.PNG)

Every packet consists of 24 bits.

These contain the command for the blinds, the selected motor, the security code and 4 bits that I could not (yet) find the purpose of.

(In the following images the polarity of the scope probe was reversed, so it looks like the waveform is upside down. You can just ignore this.)

![DS0007](https://user-images.githubusercontent.com/30577392/157090884-05e876db-40d3-4380-903c-65ac24d0def1.PNG)

The following image shows the decoded packet to open all blinds.
![Velux-All-Up-Decode](https://user-images.githubusercontent.com/30577392/157091541-c858e703-8365-4605-a8cd-0621e3a33a85.png)

The first bit is for the stop command. This bit is set (1) when the stop button is pressed.<br>
The second bit signifies the direction. A 0 indicates the up/open direction and a 1 indicates the down/close direction.<br>
The next bit is always 1.<br>
The following three bits (bit 4, 5 and 6) indicate the selected motor (see description of default mode above). When all motors should be controlled all three bits are set.<br>
The next 4 bits indicate the motor set from 1 to 10. When all bits are 0 the command controls all motor sets. The numbers are represented in binary as usual.<br>
The following 10 bits after this directly represent the security code as set by the DIP-switches in the remote.

The last 4 bits are (as mentioned above) still a mystery to me.<br>
They do not seem to encode any additional information (like the mode the remote is set to), as they do not change when the settings of the remote are changed.<br>
They are however different between different commands.<br>
I thought that they might be some form of error detection or correction, however they do not appear to be parity bits, checksums or CRC.<br>
I will continue to work on this and update this file when I found out, what these bits do.<br>
I could use some help with this.<br>
If anyone has any other idea what they might be, please send me a message.<br>
I might go through all the possible commands and write down these last 4 bits for each command.


Maybe I could help someone with this.

I might also write a small arduino library to control these blinds.
