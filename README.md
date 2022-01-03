# Qualcomm Sahara / Firehose Attack Client / Diag Tools
(c) B. Kerler 2018-2021

## Why

- Because we'd like to flexible dump smartphones
- Because attacking firehose is kewl
- Because memory dumping helps to find issues :)

### Use Re LiveDVD (everything ready to go, based on Ubuntu):
[Download Re Live DVD V2](https://drive.google.com/file/d/1VUK0toC6I0jhJ4ZTFPREhSXFTkSe9_gl/view?usp=sharing)
User: livedvd, Password:livedvd

## Installation

### Linux (Debian/Ubuntu/Mint/etc): 
```bash
# Debian/Ubuntu/Mint/etc
sudo apt install adb fastboot python3-dev python3-pip liblzma-dev git
sudo apt purge modemmanager
# Fedora/CentOS/etc
sudo dnf install adb fastboot python3-devel python3-pip xz-devel git

sudo systemctl stop ModemManager
sudo systemctl disable ModemManager
sudo apt purge ModemManager

git clone https://github.com/bkerler/edl.git
cd edl
git submodule update --init --recursive
sudo cp Drivers/51-edl.rules /etc/udev/rules.d
sudo cp Drivers/50-android.rules /etc/udev/rules.d
python setup.py build
sudo python setup.py install
```

### macOS:
```bash
brew install libusb git

git clone https://github.com/bkerler/edl.git
cd edl
git submodule update --init --recursive
python setup.py build
sudo python setup.py install
```

### Windows:
#### Install python + git
- Install python 3.9 and git
- If you install python from microsoft store, "python setup.py install" will fail, but that step isn't required.
- WIN+R ```cmd```

#### Grab files and install
```
git clone https://github.com/bkerler/edl
cd edl
git submodule update --init --recursive
pip3 install -r requirements.txt
```

#### Get latest UsbDk 64-Bit
- Install normal QC 9008 Serial Port driver (or use default Windows COM Port one, make sure no exclamation is seen)
- Get usbdk installer (.msi) from [here](https://github.com/daynix/UsbDk/releases/) and install it
- Test on device connect using "UsbDkController -n" if you see a device with pid 0x9008
- Works fine under Windows 10 and 11 :D

------------------------------------------------------------------------------------------------------------------------------------
## Get Loaders
You should get these automatically if you do a ``` git submodule update --init --recursive ```
or from [here](https://github.com/bkerler/Loaders)

## Convert own EDL loaders for automatic usage

- Make a subdirectory "newstuff", copy your edl loaders to this subdirectory
- ```fhloaderparse newstuff Loaders```

- or sniff existing edl tools using Totalphase Beagle 480, set filter to ```filter({'inputs': False, 'usb3': False, 'chirps': False, 'dev': 26, 'usb2resets': False, 'sofs': False, 'ep': 1})```, export to binary file as "sniffeddata.bin" and then use ```beagle_to_loader sniffeddata.bin```


## Install EDL loaders

- ```mkdir examples```
- Copy all your loaders into the examples directory
- ```fhloaderparse examples Loaders``` -> will autodetect and rename loader structure and copy them to the "Loaders" directory
- Or rename Loaders manually as "msmid_pkhash[8 bytes].bin" and put them into the Loaders directory

------------------------------------------------------------------------------------------------------------------------------------

## Run EDL (examples)

Your device needs to have a usb pid of 0x9008 in order to make the edl tool work.
If your device is semi bricked and entered the usb pid 0x900E, there are several options
to get back the 0x9008 mode :

1. Use a edl cable (Short D+ with GND) and force reboot the phone (either vol up + power pressing for more than 20 seconds or disconnect battery), works with emmc + ufs flash (this will only work if XBL/SBL isn't broken)

2. If emmc flash is used, remove battery, short DAT0 with gnd, connect battery, then remove short.

3. If a ufs flash is used, things are very much more complicated. You will need to open the ufs die and short the clk line on boot, some boards have special test points for that.

4. Some devices have boot config resistors, if you find the right ones you may enforce booting to sdcard instead of flash.


### Generic

- ```edl -h``` -> to see help with all options
- ```edl server --memory=ufs --tcpport=1340``` -> Run TCP/IP server on port 1340, see tcpclient.py for an example client
- ```edl xml run.xml``` -> To send a xml file run.xml via firehose
- ```edl reset``` -> To reboot the phone
- ```edl rawxml <xmlstring>``` -> To send own xml string, example :
   ```edl rawxml "<?xml version=\"1.0\" encoding=\"UTF-8\" ?><data><response value=\"ACK\" /></data>```
- ```edl [anycommand] --debugmode``` -> enables Verbose. Only do that is REALLY needed as it will print out everything happening !

### For EMMC Flash

- ```edl printgpt``` -> to print gpt on device with emmc
- ```edl rf flash.bin``` -> to dump whole flash for device with emmc
- ```edl rl dumps --skip=userdata --genxml``` -> to dump all partitions to directory dumps for device with emmc and skipping userdata partition, write rawprogram0.xml
- ```edl rs 0 15 data.bin``` -> to dump 15 sectors from starting sector 0 to file data.bin for device with emmc
- ```edl rs 0 15 data.bin --skipresponse``` -> to dump 15 sectors from starting sector 0 to file data.bin for device with emmc, ignores missing ACK from phones
- ```edl r boot_a boot.img``` -> to dump the partition "boot_a" to the filename boot.img for device with emmc
- ```edl r boot_a,boot_b boot_a.img,boot_b.img``` -> to dump multiple partitions to multiple filenames
- ```edl footer footer.bin``` -> to dump the crypto footer for Androids with emmc flash
- ```edl w boot_a boot.img``` -> to write boot.img to the "boot" partition on lun 0 on the device with emmc flash
- ```edl w gpt gpt.img``` -> to write gpt partition table from gpt.img to the first sector on the device with emmc flash
- ```edl wl dumps``` -> to write all files from "dumps" folder to according partitions to flash
- ```edl wf dump.bin``` -> to write the rawimage dump.bin to flash
- ```edl e misc``` -> to erase the partition misc on emmc flash
- ```edl gpt . --genxml``` -> dump gpt_main0.bin/gpt_backup0.bin and write rawpartition0.xml to current directory (".")


### For UFS Flash

- ```edl printgpt --memory=ufs --lun=0``` -> to print gpt on lun 0
- ```edl printgpt --memory=ufs``` -> to print gpt of all lun
- ```edl rf lun0.bin --memory=ufs --lun=0``` -> to dump whole lun 0
- ```edl rf flash.bin --memory=ufs``` -> to dump all luns as lun0_flash.bin, lun1_flash.bin, ...
- ```edl rl dumps --memory=ufs --lun=0 --skip=userdata,vendor_a``` -> to dump all partitions from lun0 to directory dumps for device with ufs and skip userdata and vendor_a partition
- ```edl rl dumps --memory=ufs --genxml``` -> to dump all partitions from all lun to directory dumps and write rawprogram[lun].xml
- ```edl rs 0 15 data.bin --memory=ufs --lun=0``` -> to dump 15 sectors from starting sector 0 from lun 0 to file data.bin
- ```edl r boot_a boot.img --memory=ufs --lun=4``` -> to dump the partition "boot_a" from lun 4 to the filename boot.img
- ```edl r boot_a boot.img --memory=ufs``` -> to dump the partition "boot_a" to the filename boot.img using lun autodetection
- ```edl r boot_a,boot_b boot_a.img,boot_b.img --memory=ufs``` -> to dump multiple partitions to multiple filenames
- ```edl footer footer.bin --memory=ufs``` -> to dump the crypto footer
- ```edl w boot boot.img --memory=ufs --lun=4``` -> to write boot.img to the "boot" partition on lun 4 on the device with ufs flash
- ```edl w gpt gpt.img --memory=ufs --lun=4``` -> to write gpt partition table from gpt.img to the lun 4 on the device with ufs flash
- ```edl wl dumps --memory=ufs --lun=0``` -> to write all files from "dumps" folder to according partitions to flash lun 0
- ```edl wl dumps --memory=ufs``` -> to write all files from "dumps" folder to according partitions to flash and try to autodetect lun
- ```edl wf dump.bin --memory=ufs --lun=0``` -> to write the rawimage dump.bin to flash lun 0
- ```edl e misc --memory=ufs --lun=0``` -> to erase the partition misc on lun 0
- ```edl gpt . --genxml --memory=ufs``` -> dump gpt_main[lun].bin/gpt_backup[lun].bin and write rawpartition[lun].xml to current directory (".")

### QFIL emulation (credits to LyuOnLine):

- For flashing full image:
   ```
   edl qfil rawprogram0.xml patch0.xml image_dir
   ```
------------------------------------------------------------------------------------------------------------------------------------

### For devices with peek/poke command

- ```edl peek 0x200000 0x10 mem.bin``` -> To dump 0x10 bytes from offset 0x200000 to file mem.bin from memory
- ```edl peekhex 0x200000 0x10``` -> To dump 0x10 bytes from offset 0x200000 as hex string from memory
- ```edl peekqword 0x200000``` -> To display a qword (8-bytes) at offset 0x200000 from memory
- ```edl pokeqword 0x200000 0x400000``` -> To write the q-word value 0x400000 to offset 0x200000 in memory
- ```edl poke 0x200000 mem.bin``` -> To write the binary file mem.bin to offset 0x200000 in memory
- ```edl secureboot``` -> To display secureboot fuses (only on EL3 loaders)
- ```edl pbl pbl.bin``` -> To dump pbl (only on EL3 loaders)
- ```edl qfp qfp.bin``` -> To dump qfprom fuses (only on EL3 loaders)

------------------------------------------------------------------------------------------------------------------------------------

### For generic unlocking
- ```edl modules oemunlock enable``` -> Unlocks OEM if partition "config" exists, fastboot oem unlock is still needed afterwards

#### Dump memory (0x900E mode)
- ```edl memorydump```
- 
------------------------------------------------------------------------------------------------------------------------------------
### Streaming mode (credits to forth32)

#### Enter streaming mode

##### Sierra Wireless Modem
- Send AT!BOOTHOLD and AT!QPSTDLOAD to modem port or use ```modem/boottodwnload.py``` script
- Send AT!ENTERCND="A710" and then AT!EROPTION=0 for memory dump
- ```edl --vid 1199 --pid 9070 --loader=loaders/NPRG9x35p.bin printgpt``` -> To show the partition table

##### Netgear MR1100
- run ```boottodownload```, device will enter download mode (0x900E pid)
- ```edl printgpt --loader=Loaders/qualcomm/patched/mdm9x5x/NPRG9x55p.bin```, device will reboot to 0x9008
- now use edl regulary such as ```edl printgpt``` (do not use loader option)

##### ZTE MF920V, Quectel, Telit, etc.. Modem
- run ```enableadb```, or send to at port "AT+ZCDRUN=E", or send via ```qc_diag -sahara```
- ```adb reboot edl```
- ```edl printgpt``` -> To show the partition table


## Run Diag port tools (examples)

For Oneplus 6T, enter *#801#* on dialpad, set Engineer Mode and Serial to on and try :

- ```qc_diag -vid 0x05c6 -pid 0x676c -interface 0 -info```

### Usage

- ```qc_diag -vid 0x1234 -pid 0x5678 -interface 0 -info``` -> Send cmd "00" and return info
- ```qc_diag -vid 0x1234 -pid 0x5678 -interface 0 -spc 303030303030``` -> Send spc "303030303030"
- ```qc_diag -vid 0x1234 -pid 0x5678 -interface 0 -cmd 00``` -> Send cmd "00" (hexstring)
- ```qc_diag -vid 0x1234 -pid 0x5678 -interface 0 -nvread 0x55``` -> Display nvitem 0x55
- ```qc_diag -vid 0x1234 -pid 0x5678 -interface 0 -nvbackup backup.json``` -> Backup all nvitems to a json structured file
- ```qc_diag -vid 0x1234 -pid 0x5678 -interface 0 -efsread efs.bin``` -> Dump the EFS Modem partition to file efs.bin
- ```qc_diag -vid 0x1234 -pid 0x5678 -interface 0 -efslistdir /``` -> Display / directory listing of EFS


## Issues

- Secure loader with SDM660 on Xiaomi not yet supported (EDL authentification)
- VIP Programming not supported (Contributions are welcome !)
- EFS directory write and file read has to be added (Contributions are welcome !)


## Tested with

- Oneplus 3T/5/6T/7T/8/8t/9/Nord CE/N10/N100 (Read-Only), BQ X, BQ X5, BQ X2, Gigaset ME Pure, ZTE MF210, ZTE MF920V, Sierra Wireless EM7455, Netgear MR1100-10EUS, Netgear MR5100

Published under MIT license
Additional license limitations: No use in commercial products without prior permit.

Enjoy !
