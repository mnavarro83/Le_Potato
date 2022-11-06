# Introduction

  Currently this is just a couple of md files with step by step instructions to install Raspbian, Mainsail, Klipper and Moonraker on a AML-S905X-CC SOC (aka Le Potato). This thing: https://libre.computer/products/aml-s905x-cc/ . These instructions are verified working at the moment. Please reach out to me or raise an issue if you find any discrepancies. 

  So far the base install (Mainsail, Klipper, Moonraker) is verified to work with KIAUH (https://www.lpomykal.cz/kiauh-installation-guide/) should you wish to automate the process. Further testing with other KIAUH components is required but there is no reason to believe it will have issues. 
  
# Why this board?

- The CPU is the exact same as a RPi 3B.
- It uses the normal Linux community repositories with a custom bootloader which is opensource. Meaning that beyond some modification to how the SD card is flashed this will behave exactly like a RPi 3B running Raspbian.
- It is cheap, of decent quality and readily available.
- No issues so far using the usual community tools (KIAUH, etc)
- Same form factor as a RPi 3B.

# Cons

- No built in wifi. Dongle testing is underway.
- Only USB 2.0, no 3.0.
- GPIO header is numbered differently from a RPi. Some work will need to be done to hook up a display or do other activities with your GPIO: https://docs.google.com/spreadsheets/d/1U3z0Gb8HUEfCIMkvqzmhMpJfzRqjPXq7mFLC-hvbKlE/edit#gid=0
- Only the base components of the install have been tested. No word yet on Klipper Screen or rPiCam integration but there is no reason to believe it won't work.
