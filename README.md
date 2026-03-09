# Project Installation Guide: pyMC_Repeater

This guide provides a step-by-step walkthrough for setting up a Raspberry Pi with a hardened Debian environment to run the pyMC_Repeater service.

This approach is recommended for standalone (non-docker) repeaters and is optimized for remote operation. 

Whenever possible code blocks have been provided which can be cut and pasted to execute the commands.

As of ~3/1/26 docker capability has been added by ***Miketastic***. This is an excellent option, but beyond the scope of this document. Over time I may leverage Michael's work to create a Balena image which will further simplify provisioning and managing remote nodes. 

***One final caveat: pymc_repeater is for pi based meshcore repeater nodes or similar. If you are looking to setup an esp32/nrf device this is not for you! :-)***

---

## 1. Initial OS Setup
1. Use **Raspberry Pi Imager** to flash your SD card.
2. Select **Other OS** > **Debian Bookworm 64-bit Lite** (No Desktop).
3. Once flashed, boot the Pi and ssh in, run the initial updates and install git:

***Note: if you need wifi credentials you can add it during the imager step. Same for default user, and even ssh certificates. Much easier to do now than later.***

Find your IP address to ssh in, or use the one of the *systemname.local* options debian normally sets up.

```bash
sudo apt-get update && sudo apt-get -y install git
```

## 2. OS Hardening (Recommended)
If you are not running on eMMC or an SSD, it is recommended to harden the OS. This moves dynamic and temporary directories into **zram** with persistence to reduce SD card wear.

This only impacts /tmp, /var/log and /var/lib/pymc_repeater and meshtasticd which are the key dynamic dirs.

It installs key required subsystems. We also install the cloudflared & tailscale daemons as they are often required behind firewalls / CGNAT. They are idle unless you enable them. Likewise they can be removed after the fact if desired. If behind a security layer like cloudflared with authentication, cockpit can be added to allow full admin via web.

```bash
# get the dev version of my harden script & install it
git clone -b dev [https://github.com/pinztrek/harden_meshtasticd](https://github.com/pinztrek/harden_meshtasticd) && cd harden_meshtasticd
sudo bash harden.sh
```

> [!IMPORTANT]
> Tell it no to installing meshtasticd for now. Let it run to the end.

## 3. Hardware Configuration (SPI)
### for SPI based hats- Turn on SPI and disable CS on SPI0.

```bash
echo -e "dtparam=spi=onndtoverlay=spi0-0cs" | sudo tee -a /boot/firmware/config.txt
sudo reboot
```

You should be able to see the spi after the reboot:
```bash
ls -l /dev/sp*
```

### for USB based dongles- get the device ID's for use in configuration

```bash
command to be documented later
```

---

## 4. Install pyMC_Repeater
Log back in via ssh and clone the appropriate deb branch from the repo so it can be installed:

```bash
git clone -b dev [https://github.com/rightup/pyMC_Repeater.git](https://github.com/rightup/pyMC_Repeater.git)
cd pyMC_Repeater
```

```bash
git clone -b feat/newRadios [https://github.com/rightup/pyMC_Repeater.git](https://github.com/rightup/pyMC_Repeater.git)
cd pyMC_Repeater
```

***Note: as of 3/1/26, use the dev or newradios branch. Use only one of the commands above, typically newRadios.***


Run the install script. This actually installs the code, service, etc. This script is also used to reset or update after install. 

```bash
sudo bash ./manage.sh
```
Select *Install*, and let it process. Exit the menu (*arrow down and select Exit*).

It will install pymc_repeater, services, etc, and start it up in web configure mode

---

## 5. Web Configuration
Copy the web install link (ex: http://172.16.30.87:8000) when finished. Access the page, and complete the 5 step config to finish your radio.

***As of 3/1/26 the "submit" button is hard to see and has no label. It's the white box outline in the lower right of the window.***

Work through all steps. It will do the basic config and restart the service. If you have a supported board you are running!

---
## 6. Additional Configuration

Most configuration items managing behavior are located in /etc/pymc_repeater/config.yaml

Use your favorite editor to edit this file to adjust a few items

```bash
sudo nano /etc/pymc_repeater/config.yaml
```
### Update the letsmesh section:

Update the broker index to 1, email, owner and iota code as needed.

```bash
letsmesh:
  broker_index: 1
  disallowed_packet_types: []
  email: 'me@myemail.com'
  enabled: true
  iata_code: XXX
  owner: 'Alan Barrow KM4BA'
  status_interval: 300
```
### Update the txdelay if a rooftop or mobile node
For rooftop or mobile nodes increase the default tx_delay_factor to 1.25 or 1.5 to give high site repeaters a chance to repeat first:

```
delays:
  direct_tx_delay_factor: 1.0
  tx_delay_factor: 1.5
```
### If using a usb device set the ID's for that particular device. Typically found by lsusb or similar. 

```
ch341:
  pid: 21778
  vid: 6790
```

***Note: use your specific ID's in the above section***

---

### Save the edited file, then restart the service:

```bash
sudo systemctl restart pymc-repeater
```

***Note it is hyphen and not underscore in the above command!***

---

## 7. Finalizing (Hardened Images Only)
If you are using the hardened pi image, sync the zram before rebooting:

```bash
sudo zram-config sync
```
This will copy any logs and */var/lib* files to static storage and restore them to zram upon boot. This is done automatically if you use the reboot command to reboot. But it's a good practice to sync before a reboot. 

Most of the dynamic files in /var/lib/pymc_repeater are things like stats and heard nodes which will be recreated if the server is powered down without sync. Or restored from prior copy if it has run in the past. The system will auto sync these dirs hourly. 

## Todo as of 3/9/26:
* Confirm correct branch (have *feat/newRadios* been incorporated into *dev*?)
* update correct lsusb command and defaults for meshtoads
* add links to ***Miketastic*** github
* add links to main meshcore site for firmware nodes
* add links to docker and Balena info
