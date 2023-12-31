
# M4ROM

Virtual 27C256 rom x 4 to fit Molex 78805 socket

![](ref/M4ROM.jpg)

This is a version of [Teeprom and Meeprom](https://github.com/bkw777/Teeprom) that uses a 128k 29F010 flash instead of a 32k 28C256 eeprom.

The advantages over the original Teeprom are:  
* The flash chip is cheaper and more readily available than the eeprom.  
* The programming adapter is cheaper to buy and simpler to use than the soic test clip.
* Holds four 32k rom images instead of one.  

The disadvantages are:  
* The board has more parts and is more difficult to solder.

There are 2 versions so far,  
'''M4ROM_TANDY''' is only for TANDY 100, 102, & 200, same as Teeprom.

'''M4ROM_27256''' is for everything else, Same as Meeprom.  
Examples: TANDY 600, Epson PX-4 & PX-8, general industrial applications, most anywhere the Molex 78805 socket is found.

All parts other than the PCB are the same for both versions.  
The difference is only in the pinout of the edge connectors. TANDY 100, 102, & 200 have a non-standard pinout. The 27256 version provides a standard 27256 pinout.  

The same programming adapter is used for both.

![M4ROM_TANDY schematic](PCB/out/M4ROM_TANDY.svg)
![M4ROM_TANDY render](PCB/out/M4ROM_TANDY.jpg)
<!-- ![M4ROM_TANDY on Programming Adapter render](PCB/out/M4ROM_TANDY.programming.jpg) -->

![M4ROM_27256 schematic](PCB/out/M4ROM_27256.svg)
![M4ROM_27256 render](PCB/out/M4ROM_27256.jpg)

![M4ROM_27256 on Programming Adapter render](PCB/out/M4ROM_27256.programming.jpg)

![M4ROM Programming Adapter](PCB/out/M4ROM_programming_adapter.jpg)


### TANDY PCB  
[PCBWAY](https://www.pcbway.com/project/shareproject/4ROM_100_multi_option_rom_module_for_TRS_80_Model_100_102_200_93cfa6c8.html)

### 27256 PCB  
[PCBWAY](https://www.pcbway.com/project/shareproject/4ROM_78802_714ecf32.html)

### BOM  
<!-- [Mouser](https://www.mouser.com/ProjectManager/ProjectDetail.aspx?AccessID=66e12c3f20)  -->
[DigiKey](https://www.digikey.com/short/rzj0j0wr)

### Carrier  
[Shapeways](http://shpws.me/SGGB)

### Programming Adapter  
PCB https://www.pcbway.com/project/shareproject/4ROM_Programming_Adapter_fc156337.html  
BOM https://www.digikey.com/short/f3jhw9v1  
<!-- https://www.mouser.com/ProjectManager/ProjectDetail.aspx?AccessID=a770931c82 -->

When ordering the PCB (not the programming adapter PCB):  
Select ENIG copper finish so the castellated edge contacts and programming adapter contacts are gold plated.  
Change the min tacks/spaces option to 6/6mils. The PCBWAY web site automatically selects 5/5 for this board for some reason, but there are no such thin traces or spaces.

# Programming the chip:  
* Put the programming adapter into a programmer.  
* Remove the M4ROM PCB from the carrier and connect it to the programming adapter by the center pins. You don't need to push the pcb all the way down. Just get the pins into the holes at all and that is good. It should be stiff.  
* Select the desired bank number with the slide switch on the M4ROM.  
* Configure the programmer:  
  * device "SST39SF010A"  
  * ignore size mismatch  
  * do not automatically erase the whole chip before writing  
* Write a single 32K rom image.

The following is using a TL-866II+ programmer and the open source [minipro](https://gitlab.com/DavidGriffith/minipro) software.  
(If buying a new programmer, be aware minipro does not support the new T48 or T56 programmers, only TL-866II+ and older.)

### Test the pin connections  
Just to verify that the board is soldered correctly and all of the programming adapter pins are making a good connection.  
It should say pins 2 and 3 are bad, and nothing else.  
```
$ minipro -p 'SST39SF010A' -z
Found TL866II+ 04.2.132 (0x284)
Bad contact on pin:2
Bad contact on pin:3
$
```

"Bad contact" on pins 2 & 3 is expected. Those are the 2 highest address bits A15 and A16, which aren't connected to the outside world. They are only connected to the bank-select logic on the board.

(If you see pins 1 & 2 instead of 2 & 3, then update [minipro](https://gitlab.com/DavidGriffith/minipro) to get this [fix](https://gitlab.com/DavidGriffith/minipro/-/merge_requests/220).  

If you see anything else, inspect the pin connections and solder work. Try pushing the pcb futher down onto the programming adapter to make the pins bind up a little tighter. The holes in the 4ROM are intentionally a little bit closer together than the exact 2.0mm of the pins on the programming adapter, so the pins bind up tighter the further down you push the 4ROM. It should be essentially impossible to push it all the way down, and don't try, but the further you go the stronger the pin contacts. Test the solder work by touching a sharp needle tip probe to the tops of the chip legs right where they enter the plastic body, and the other probe to one of the programmer adapter holes.

### Erase the whole chip
Normally the entire chip would always be erased before each write,  
but this must be supressed in this case when writing the individual banks,  
because we don't want to erase the other 3 banks every time we write one bank.

But the chip must still be erased once before writing the individual banks.

So, erase the chip as a separate step before writing any of the individual banks.

This must also be done before re-writing any bank that isn't currently blank.  
If a bank currently has data, and you want to over-write it, you must erase the whole chip and re-write all 4 banks.

```
$ minipro -p 'SST39SF010A' -u -E
Found TL866II+ 04.2.132 (0x284)
Chip ID: 0xBFB5  OK
Erasing... 0.40Sec OK
$
```

### Write one bank  
Select position "1" on the slide switch, and write one 32K rom image.  
The command line flags used below mean:  
**-e** do not erase the chip before writing  
**-u** un-protect the chip before writing  
**-P** protect the chip after writing  
**-s** non-fatal warning for the size mis-match from writing only 32K when 128K is expected  
```
$ minipro -p 'SST39SF010A' -u -P -s -e -w MULTIPLAN.rom
Found TL866II+ 04.2.132 (0x284)
Chip ID: 0xBFB5  OK
Warning: Incorrect file size: 32768 (needed 131072)
Writing Code...  1.65Sec  OK
Reading Code...  0.25Sec  OK
Verification OK
$
```

### Write another bank  
Select position "2" on the slide switch, and repeat to write another rom.  
```
$ minipro -p 'SST39SF010A' -u -P -s -e -w BASIC.rom
Found TL866II+ 04.2.132 (0x284)
Chip ID: 0xBFB5  OK
Warning: Incorrect file size: 32768 (needed 131072)
Writing Code...  1.64Sec  OK
Reading Code...  0.25Sec  OK
Verification OK
$
```



# References
[Molex78802_Module](https://github.com/bkw777/Molex78802_Module)  
