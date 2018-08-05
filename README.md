# cli for the imdisk ram disk driver
A reduced version of the original client, intended to be used through meterpreter or other backdoor setups.  
Mostly written to learn about loading drivers and communicating with a loaded driver.  
Since this is a POC it has been made incident response / forensic friendly friendly, by having tons of strings and not optimizing or clearing in memory variables.
If you want to use this during a red team, make sure you adjust the source accordingly :)

## files needed
cliramdisk.exe (x32)  
imdisk.sys (x64)

## usage
```
c:\>cliramdisk.exe
DiabloHorn - https://diablohorn.com
   Install   - cliramdisk.exe i
   Uninstall - cliramdisk.exe u
   Create    - cliramdisk.exe c <size in bytes> <drive|folder to mount> <device number>
   Delete    - cliramdisk.exe d <devicenumber>
   List      - cliramdisk.exe l
```
### loading the driver
The driver is loaded by using the undocumented NT api NtLoadDriver().  
For this to work a couple registry keys need to be created like this:
```
reg import imdiskdriver.reg
```
The content of the imdiskdriver.reg file can be found below:
```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ImDisk]
"DisplayName"="ImDisk Virtual Disk Driver"
"Description"="Disk emulation driver"
"Type"=dword:00000001
"Start"=dword:00000004
"ErrorControl"=dword:00000000
"ImagePath"=hex(2):5c,00,53,00,79,00,73,00,74,00,65,00,6d,00,52,00,6f,00,6f,00,\
  74,00,5c,00,73,00,79,00,73,00,74,00,65,00,6d,00,33,00,32,00,5c,00,44,00,52,\
  00,49,00,56,00,45,00,52,00,53,00,5c,00,69,00,6d,00,64,00,69,00,73,00,6b,00,\
  2e,00,73,00,79,00,73,00,00,00
"DeleteFlag"=dword:00000001
```
Please note that the location in the above file for the imdisk.sys file is:  
```
%systemroot%\system32\DRIVERS\
```
After creating the above registry keys the driver can be loaded like this:
```
cliramdisk.exe i
```
### listing current ram disk devices
To list the devices in use you can issue the command:
```
cliramdisk.exe l
```
Please note that the command outputs the underlying devices and not the mounted driver or folder.

### creating a ram disk device
Creating a ram disk can only be performed after the driver has been loaded.  
List the current devices to know which device number is free to use.
```
cliramdisk.exe c 209715200 R: 3
```

The above command creates a ramdisk of 200MB in size, with drive letter R: pointing to it and the ram disk is supported by the underlying device with number 3.  
After creating the ramdisk you can format it like this:
```
format R: /FS:NTFS /Q /y
```
Your ram disk is now ready to be used.

### deleting a ram disk device
To delete a ram disk device issue the following command, taking into account the output from the list command:
```
cliramdisk.exe d 3
```

## todo items
   * support mounted folders
   * add error handling

## tested on
   * Windows 10 x64

## References
   * http://www.ltr-data.se/opencode.html/
   * https://github.com/killswitch-GUI/HotLoad-Driver/
