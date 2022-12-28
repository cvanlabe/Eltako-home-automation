# Eltako-home-automation
This project documents how I'm working on my home automation using Eltako Teleruptors. If you're looking for a modular and reliable way to automate your home, without paying a fortune, read along.

**Keywords**: Distributed, Eltako, Inexpensive, HomeAssistant

## History of this project
The electricity in my house is centrally switched using Eltako 12-series teleruptors (relays if you will). 
The teleruptors toggle on/off whenever they receive a short low voltage pulse.

Since all switching is happening in the closet, I was able to attach a Raspberry Pi, and send those pulses to play with the lights in the house using software.
It allowed me to automatically turn on or off lights at certain hours of the day, and even do home presence simulation where lights turn on depending on when the sun sets.

Worked great, except when someone toggles a light using a wall switch. The Raspberry Pi can't be informed that a wall switch sent a pulse, while it's connected to that same teleruptor.
Unless, if all wall switches their signal would be routed to the Raspberry Pi, and the Pi is the only one connected to all Teleruptors. 
Not an ideal situation, and not good for the "wife-acceptance-factor" when the Pi would crash/fail. Another solution was needed.

## Evaluation
My requirements are:
1. Being able to use the same wall switches without recabling everything
2. Preferrably no central controller which if it fails, the lights can't be operated properly anymore (wife-acceptance-factor! :) ) If no other option, it should be quickly replaceable without long 'downtime'.
3. Ability to control it through software and do home automation

Requirement 1 and 2 kept me with Eltako: the Series-14 that use the EnOcean Protocol.
Requirement 3 brought me to home assistant.

## Test-bed
Components in use are:

- **WNT12 12VDC-24W/2A**: The powersupply for the wall pulse switches connected to the fts14EM [Specification](https://www.eltako.com/fileadmin/downloads/en/_specifications/17_Technical_Data_Switching_Power_Supply_Units_and_Wide_range_Switching_Power_Supply_Units.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_WNT12-12VDC-24W_2A.pdf)
- **Eltako FAM14**: The powersupply and controller of the EnOcean bus [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FGW14-USB_30014049-1_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FGW14-USB.pdf)
- **FTS14EM**: The input where the existing wall switches are connected to [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FTS14EM_30014060-3_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FTS14EM.pdf)
- **WNT12**: A 12VDC power supply powering the FTS14EM connections to the wall switches [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/WNT12_20000060-1_internet_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_WNT12-24V_DC-24W_1A.pdf)
- **FSR14-4x**: A 4-channel teleruptor to which lights are connected [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FSR14-4x_30014001-1_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FSR14-4x.pdf)
- **FGW14-USB**: A USB connection to let Home Assistant send and receive messages on the EnOcean RS485 bus [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FGW14-USB_30014049-1_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FGW14-USB.pdf)

## Cabling Diagram
Despite some people's claims that the Eltako documentation is clear, I tend to disagree with it. It may be all there, but it's all scattered over the place, and not always easy to understand unless you did it before.

Below you can find my cabling diagram:

![Testbed Diagram](images/Eltako14SeriesDiagram.drawio.png)

Connect the `Hold` connection to the all other `Hold` connections across the entire bus. Connect also the 1st FTS14EM (and every 10th) `Enable` connection to the `Hold`.
Make sure the terminator is at installed on the last component of the bus. 

The 12VDC `+` and `-` are cabled to the FTS14EM. The `+` goes to a wall switch - which is further connected to the `+E` input connectors, and the `-` is connected to the `-E` input. 

When all is ok, connect the WNT12 and FAM14 to the mains.

With the circuit cabled, we can now start programming the FAM14.

## Programming the Eltako Bus

### Erase & Clear whatever may still be in memory
While I was fighting the PCT14 tool, I quickly learned that as you mess around, some settings may get stuck and prevent you from achieving what you originally wanted.
You also can't guarantee that the component you use isn't a return from a previous customer which may still have some settings.

As such, it's recommended to start with a full clear of the components.

#### FSR14
1. Turn the middle rotary switch to CLR, and then ALL.
2. The LED is flashing quickly
3. Within 10 seconds, turn the upper rotary switch 6 times all the way to the left (anti-clockwise) and away again.
4. When successful, the LED will stop flickering and go out after 5 seconds.

#### FGW14
1. Turn the rotary switch 5 times to the right (clockwise) and back again within 10 seconds.
2. The LED lights up for 10 seconds and then goes out
3. All IDs are cleared.

#### Teach-in Sensors
Same procedure as teaching in, except that you turn the middle rotary switch to CLR instead of LRN, and then operate the sensor.

### PCT14
In the absense of a Windows PC, I have installed Virtualbox 6.1 (7.0 didn't work for some reason I didn't further investigate) on an intel MacBook. 
I installed Windows 10 as a VM.

Once the FAM14 is powered on, and connected through USB to the macbook, you can add a USB filter in the VM.
Select the `FTDI FT232R USB UART [0600]`, which is the FAM14. Once added, restart the VM.

On the FAM14, put the BA rotary switch to position 2, and the AUTO rotary switch to 1.

Then in PCT14, you should be able to click `Connect`.

### Device Address Assignment
1. In the left pane of PCT14, right-click and choose `Search device for address assignment`
2. This should bring up the FAM14
3. FSR14: turn the middle rotary switch to LRN on only 1 FSR actuator at a time
4. In the left pane of PCT14, right-click and choose `Search device for address assignment` again
5. This should bring up the FSR14 as well
6. Right-click the FSR14, and select `Modify device address and transmit`. Chose an address that's not in use yet (default is 0, and choose something else now)
7. For each channel you can modify the description
8. Once you're done, change the middle rotary switch to `Auto`
9. Next, teach-in the FGW14, move the rotary switch somewhere and back to 10 so it's blinking green
10. In the left pane of PCT14, right-click and choose `Search device for address assignment` again
11. You should now also see the FGW14 showing up
12. Also change its address

Repeat as needed for extra devices.

=> Now we can write the configuration

Right-click the `Device List` in the left pane, and select `Update device list and read out device memory`.

All devices should be visible in the left pane, and in the color green.

The FTS14EM is a special one. It allows you to connect wired switches and convert those pulses to RS485 bus messages.
You will **not** see it in the PCT14 tool. In fact, programming it is different again.

### Programming FTS14EM inputs
#### Setting the buttons
The FTS14EM has two rotary switches. The bottom one defines to which group an FTS14EM belongs. In total there are 5 groups (1, 101, 201, 301, and 401) in which 100 IDs are available.
It has two halves, labeled `UT` or `RT`. The `UT` side is used for `Universal` Buttons (on/off), the `RT` is for `Directional` Buttons (up/down).

The top one defines the ID of the FTS14EM in the group. As such, there can be 10 FTS14EM's in 1 group.

In our setup, we have set the top one to 0, and the bottom one to 1 in the UT section.

#### Testing the buttons
To test if the electrical wiring is ok, you can close the contact once when the PCT14 tools **NOT** connected to the FAM14.
When the contact is closed, you should see top rotary switch light up briefly.

#### Programming the RS485 ID
Since the FTS14EM does not show up in PCT14, you need to manually go and edit the actuator upon which hex ID it will respond.
The RS485 hex IDs for each FTS14EM input follow a common structure: `00 00 1x xx`
> Hex Id structure of the FTS14EM: 00 00 1x xx

`x xx` depends on the position of the two rotary switches and the relevant +E input:

- `x --`: The first x represents the position of the bottom switch. Eg: 1 => 1 xx
- `- x-`: The second x represents the position of the top switch. Eg: 2 => x 2x
- `- -x`: The third x represends which +E connection you need. Eg: +E5 => x x5

This together would give HexID: `00 00 1125`.

In our example, we assume that we have the following addresses:

| Connector | Hex ID      |
|-----------|-------------|
| +E1       | 00 00 10 01 |
| +E2       | 00 00 10 02 |
| +E3       | 00 00 10 03 |
| +E4       | 00 00 10 04 |
| +E5       | 00 00 10 05 |
| +E6       | 00 00 10 06 |
| +E7       | 00 00 10 07 |
| +E8       | 00 00 10 08 |
| +E9       | 00 00 10 09 |
| +E10      | 00 00 10 10 |

Now, in PCT14, go to the FSR14 actor you want to control using an FTS14EM input.
Select the `ID mapping range` tab, and edit the bottom table (function group 2).

Double click one line, edit as new ID the relevant hex ID, the right function, and swlect which channel on the actor you want to activate.

In the below example you see how I linked the pulse switch connected to FTS14EM's +E1 connector (00 00 10 01) to control my FSR14-4x, channel 1.

![Program the FSR14 to a Hex input from the FTS14EM](images/FTS14EM-hex-id.png)

Since I use mono-stable pulse switches, I use the function `23 universal pushbutton ES`. The key is not important here, but the channel is!
When done, click on the `Add data and transfer to device`.

## Home Assistant Eltako Integration



## References
- [Operating manual for Series 14 DIN tail mounted devices](https://www.eltako.com/fileadmin/downloads/en/_bedienung/Series_14_RS485_Bus_DIN_Rail_Mounted_DevicesSeries_gb.pdf)