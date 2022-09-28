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

- **Eltako FAM14**: The powersupply and controller of the EnOcean bus [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FGW14-USB_30014049-1_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FGW14-USB.pdf)
- **FTS14EM**: The input where the existing wall switches are connected to [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FTS14EM_30014060-3_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FTS14EM.pdf)
- **WNT12**: A 12VDC power supply powering the FTS14EM connections to the wall switches [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/WNT12_20000060-1_internet_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_WNT12-24V_DC-24W_1A.pdf)
- **FSR14-4x**: A 4-channel teleruptor to which lights are connected [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FSR14-4x_30014001-1_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FSR14-4x.pdf)
- **FGW14-USB**: A USB connection to let Home Assistant send and receive messages on the EnOcean RS485 bus [Manual](https://www.eltako.com/fileadmin/downloads/en/_bedienung/FGW14-USB_30014049-1_gb.pdf) [Datasheet](https://www.eltako.com/fileadmin/downloads/en/_datasheets/Datasheet_FGW14-USB.pdf)

## Cabling Diagram

## Programming the Eltako Bus

## Home Assistant Eltako Integration

