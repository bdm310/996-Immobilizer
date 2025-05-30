# Porsche 996/986 Immobilizer
This is a repository of information about the immobilizer on these 911/Boxster/Cayman cars. Part number of the unit here is 996-618-260-05.

![Immobilizer](<Board Top.jpg>)

There are a few useful bits of information needed when working with these:
- The part number of the immobiliser is located at the beginning of the eeprom from address 0x009 to 0x00E. Checking this allows you to verify that the dump you are using is correct.
- Key fob radio code is 12 8 bit hex characters e.g. 40 01 4F 13 06 1E 41 5F 10 38 77 26. This is a unique ID and some type of initialization/sync info for programming the remote control part of the key. This comes on a tag with new keys and was potentially provided with the car's manual for the original keys. Sometimes this tag got taped to the underside of the hood. You need this to program any remote control. I haven't found a way to reverse engineer it out of a remote yet. See the appropriately named folder for initial efforts.
- Transponder code is a unique ID on the RFID pill in the key head. You can probably re-use these transponders with special tools. They're cheap as of 2023 though, so it's probably not worth getting tools or investing time to reverse engineering them. Most should be an ID48 transponder chip commonly used across a lot of manufacturers and models. I got a set of 4 for $10 from Amazon, UPC 712288935256.
- Key learning code is 3 8 bit hex characters e.g. 4C 02 E3. This is needed to program transponders and remote controls. This code is at the end of EEPROM memory, at 0x1F7 and 0x1EE.
- Immobilizer code is needed for programming new/used modules to different cars. This is probably somewhere in the EEPROM too but I haven't needed to find it yet.

Using PIWIS to try various key learning codes:
 - Response to key input from 0x025 - 29000 - 7F 27 35 - key out of order
 - Response to key input from 0x1ee - 29000 - 7F 27 33 - No access right!
 - Got no access right again on key from 0x025

So, you can get the learning code wrong at least 3 times with no lockout. Though, I am now noticing that the second attempt above actually had the right key and gave a different error than the first, definitely wrong, key attempt. Thus, there may be some time delay after a wrong attempt but thankfully no hard lockout.

## Immobilizer versions
The M534 and M535 are the same module but with 433MHz and 315MHz (USA) antennas for the keyfob, respectively.

There are at least two major versions of the M534/M535 module. The early cars use part 996.618.260.0x (where x varies according to minor revision) and the later cars use 996.618.262.0x. For boxsters, the changeover happend in 2001, when the electronic frunk release was implemented. There are a number of [wiring differences](<./M53x wiring differences.md>) between the major revisions which make them non-interchangable. It may be possible to repin certain wires to get basic functionaly, but I have not found anyone who tried this. 

## EEPROM
There's a 93LC66 EEPROM right next to the main processor. The processor has only ROM, so this EEPROM is the only possible place for configuration and state memory. Conveniently, this means you can easily clone a module or inspect the memory contents. I had no issues using an SOIC-8 chip clip and TL866 programmer in-circuit without removing the EEPROM or lifting any leads. If you intend to do any experimentation, it would be prudent to read out this chip beforehand. Since it's the only non-volatile memory, it seems safe to perform any PIWIS or other action while experimenting so long as you have a backup.

If you've got a water damaged module, you can probably get a cheap junkyard replacement and clone. That is, in fact, what started all this.

![Damaged Module](<Water Damage.jpg>)

The ods spreadsheet details what I've learned about the detailed contents of this chip. There's a couple binary dumps here as well.

## Firmware
This module uses an M37710 microcontroller. Turns out that it looks just like an M5M27C101K when wired up a certain way, the M7700 folder has a PCB that does the right stuff. The binary hopefully will give up some secrets for programming key remotes without the original tags.

## Key Fob Radio Code
Every key head I've seen so far uses a Microchip product. I suspect the cipher is Keeloq but I cannot match the transmitted bitstream to any product datasheet I can find. Most notably, it's missing the seemingly ubiquitous synchronization preamble. You'll find LogicPro raw captures and a PulseView composite of a few captures in the reverse engineering folder.