# Project Installation Guide: pyMC_Repeater

This guide provides a step-by-step walkthrough for setting up a Raspberry Pi with a hardened Debian environment to run the pyMC_Repeater service.

---

## 1. Initial OS Setup
1. Use **Raspberry Pi Imager** to flash your SD card.
2. Select **Other OS** > **Debian Bookworm 64-bit Lite** (No Desktop).
3. Once flashed, boot the Pi and run the initial updates:

```bash
sudo apt-get update && sudo apt-get -y install git
```

## 2. OS Hardening (Recommended)
If you are not running on eMMC or an SSD, it is recommended to harden the OS. This moves dynamic and temporary directories into **zram** with persistence to reduce SD card wear.

```bash
# get the dev version of my harden script & install it
git clone -b dev [https://github.com/pinztrek/harden_meshtasticd](https://github.com/pinztrek/harden_meshtasticd) && cd harden_meshtasticd
sudo bash harden.sh
```

> [!IMPORTANT]
> Tell it no to installing meshtasticd for now. Let it run to the end.

## 3. Hardware Configuration (SPI)
Turn on SPI and disable CS on SPI0.

```bash
echo -e "dtparam=spi=onndtoverlay=spi0-0cs" | sudo tee -a /boot/firmware/config.txt
sudo reboot
```

You should be able to see the spi after the reboot:
```bash
ls -l /dev/sp*
```

---

## 4. Install pyMC_Repeater
Log back in and clone the dev repo:

```bash
git clone -b dev [https://github.com/rightup/pyMC_Repeater.git](https://github.com/rightup/pyMC_Repeater.git)
cd pyMC_Repeater
```

Run the install script. For most you will want to select a Zebra (nebrahat) or Waveshare.

```bash
sudo bash ./manage.sh
```

Select Install, and let it process. Exit the menu (arrow down and select Exit).

---

## 5. Web Configuration
Copy the web install link (ex: http://172.16.30.87:8000) when finished. Access the page, and complete the 5 step config to finish your radio.

---

## 6. Nebrahat / Zebra Additional Steps
```bash
sudo nano /etc/pymc_repeater/config.yaml 
```

Look for the radio pin config section and change to the following:
* "busy_pin": 4,
* "reset_pin": 18,
* "rxen_pin": 25,
* "use_dio2_rf": true,

Look for the lat/long section and update your position. Remember longitude is negative (ex: -84.x). Set your Advert period down from 10 to 1 hr during testing.

Save the file and restart the service:
```bash
sudo systemctl restart pymc-repeater
```

---

## 7. Finalizing (Hardened Images Only)
If you are using the hardened pi image, sync the zram before rebooting:

```bash
sudo zram-config sync
```
