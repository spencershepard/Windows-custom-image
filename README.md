# Creating custom windows ISO
You can create a custom Windows installation image that includes your prefered software and settings. This is useful for repetitive Windows installations.  Once you have the 'Technician Machine' (in this case, a Hyper-V VM) set up, it's relatively painless to update and create a new ISO.  If you're looking for the windows equivalent of Kali Live, look into Windows To Go via Rufus (see notes below).

## Prepare Host Machine

#### Download ISO and  copy contents
Download the Windows ISO from the official website.  Mount the ISO as virtual DVD on your host machine, and copy the entire contents of the ISO to a directory on your host (ie. C:\myWindowsISO\). 

#### Install ADK on host machine
```The Windows Assessment and Deployment Kit (Windows ADK) and Windows PE add-on has the tools you need to customize Windows images for large-scale deployment, and to test the quality and performance of your system, its added components, and the applications running on it.```

After installation you will find it under Start > All Apps > Windows Kits

https://learn.microsoft.com/en-us/windows-hardware/get-started/adk-install


## Prepare VM (Windows 'Technician' Machine)

#### Start Windows Install on VM
Create a new VM in Hyper-V using the same official ISO you downloaded earlier. Change the VM settings to use Standard Checkpoints and uncheck automatic checkpoints. Under Security, check 'Enable Trusted Platform Module'. Select the "I don't have a product key" option during installation.

At the sign-in screen of the Windows installation, press Ctrl-Shift-F3 to enter audit mode. You will notice the System Preparation Tool (Sysprep) open.


#### Create unattend.xml file
Copy the text of unattend.xml from this repository.  Create a new file in the VM, paste the text and save it to the C:\Windows\System32\Sysprep\unattend.xml.  This file will be used by Sysprep to apply the settings to the Windows installation.


#### Create partition
Using diskmgmt.msc, create a new partition on the C: disk that will hold the Windows image (10GB should be plenty).  Format the partition with NTFS, assign a drive letter, and label the new partition something like "Windows Image".  Create a new folder on the new partition called "Scratch".


#### Customization Checkpoint
Create a checkpoint in Hyper-V to save the current state of the Windows installation.



## Customize or Update your ISO
Run your VM from the restore point you created. You should already be logged in as the audit user and ready to make changes.

#### Customize Windows

Install software and make changes to the windows installation, but do not run any applications or bother with device drivers (they will not be included in the image).

How to customize start menu for all users:
https://www.techtarget.com/searchenterprisedesktop/tip/How-to-create-a-custom-ISO-for-Windows-10

Run disk cleanup (all options selected), empty downloads folder, etc

Optionally create a new Hyper-V checkpoint to build on your progress later.

#### Run Sysprep
The dialog box should still be open. If not:  %WINDIR%\system32\sysprep\sysprep.exe  Select the "Out of box experience" option and check the "Generalize" box.  Click OK to run Sysprep. This action will read the unattened answer file and apply the settings to the Windows installation.

#### Capture Windows Image
Reboot from the installation media (change boot settings for VM in Hyper-V).  When the Windows installation screen appears, press Shift-F10 to open a command prompt.  Run `diskpart` and then `list volume` to see the drive letters.  Run `exit` to close diskpart.  

Modify and run this command to suit:
 `dism /capture-image /imagefile:D:\install.wim /capturedir:C:\ /ScratchDir:D:\Scratch /name:"Windows 11 Home" /description:"Windows 11 Home" /compress:maximum /checkintegrity /verify /bootable`  

#### Copy the install.wim file to the ISO source
*** Important: mount as read-only! You will not be able to run the VM again without checking the box! *** Open diskmgmt, 'Action' -> 'Attach VHD'. For Hyper-V, VHDs are located in C:\ProgramData\Microsoft\Windows\Virtual Hard Disks.  In the 'Browse Virtual Disk Files' dialog, view 'All files' and select the .adhdx  file that corresponts to the image partition you created.  Check 'Read only'!  Once the VHD is mounted, you can view the contents in Windows Explorer.

Copy the install.wim file from the VM to the host machine.  Copy the install.wim file to the sources directory of the Windows ISO (ie. C:\myWindowsISO\sources\).

Unmount the VHD in diskmgmt.

#### Create a new ISO
On the host machine, open ADK's Deployment Tools as administrator (Start > All Apps > Windows Kits). Run the following command to create a new ISO file: `oscdimg -m -o -u2 -udfver102 -bootdata:2#p0,e,bC:\myWindowsISO\boot\etfsboot.com#pEF,e,bC:\myWindowsISO\efi\microsoft\boot\efisys.bin C:\myWindowsISO C:\myNewWindowsISO.iso`

# Notes

#### Windows To Go
- Rufus can burn an ISO as a bootable 'Windows To Go' USB drive.  If you've followed this guide, the Windows To Go option in Rufus may fail when it attempts to create it's own unattend.xml file (this may depend on the options chosen). See unattend-wintogo.xml in this repository for an example of the unattend file that Rufus creates. For Windows To Go, you probably don't want to create a custom ISO.  Instead, you can use Rufus to burn the standard ISO to a USB drive and then customize the installation on the USB drive.  After burning the ISO on the USB drive, you will still need to go through the install process, but then you will be able to boot directly from the USB and make persistant changes.

# References
https://www.xda-developers.com/how-create-custom-windows-iso/
https://www.techtarget.com/searchenterprisedesktop/tip/How-to-create-a-custom-ISO-for-Windows-10
