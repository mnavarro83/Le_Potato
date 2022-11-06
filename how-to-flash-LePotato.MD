# Things to note

- LePotato functions almost identically to a RPi 3B with the exception that it needs a custom boot loader in order to hand off to GRUB and boot Linux. Because of this the flashing process is different from the RPi and will either require you build their custom bootloader into an image of your choice or that you use their provided images (see step 1 below).
- This board does not include wifi, dongle compatibilty testing is in progress and I will share results when I have data.
- There are three status lights on the board near the power port. Normal activity is red and green steady with blinking blue. Anything else probably means you flashed the SD card incorrectly

# Process

1) Download the relevant OS, pay attention to the chip version. The lite version is preferred usually unless you need a GUI:

```
https://distro.libre.computer/ci/raspbian/11/
```

2) Un-xz the file (replace <file> with the name of the file you downloaded:

```
unxz ~/Downloads/<file>
```

Or 7zip, or WinRAR for the PC/Mac users. Documentation available at the libre.computer website detailing preferred methods and the reason why.

3) Flash the SD card. Documentation from https://hub.libre.computer/t/raspbian-11-bullseye-for-libre-computer-boards/82 says that flashing this particular image with RPi Imager is acceptable and also provides other acceptable methods. Bear in mind that advanced options such as setting the initial user, networking and hostname (amongst other things) do not function as of yet but it appears it may be supported in the near future.

4) Insert SD card into LePotato and boot it up. You will need the HDMI and a keyboard/mouse connected for initial setup and to enable SSH and setup your wifi dongle if you will be using one.

5) The installer will open a wizard to create your user (with sudo privileges), password, locale and then reboots to complete the installation.
  
That should be all. You may now proceed to do normal Linux things or continue on with a manual Klipper install.
