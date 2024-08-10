# Steps

# Rasberry pi init setup

```sh
sudo apt install rpi-imager
```

1. Imager>...(other)>lite(64 bit newest), choose drive & click on gear icon bottom right
- Set hostname: x2
- Enable SSH
- Set username & password: rpi/rpi
- Configure wireless LAN



2. this creates `firstrun.sh` script in **bootfs**

```
#!/bin/bash

set +e

CURRENT_HOSTNAME=`cat /etc/hostname | tr -d " \t\n\r"`
echo x2 >/etc/hostname
sed -i "s/127.0.1.1.*$CURRENT_HOSTNAME/127.0.1.1\tx2/g" /etc/hosts
FIRSTUSER=`getent passwd 1000 | cut -d: -f1`
FIRSTUSERHOME=`getent passwd 1000 | cut -d: -f6`
if [ -f /usr/lib/userconf-pi/userconf ]; then
   /usr/lib/userconf-pi/userconf 'rpi' '$5$URF9GEaRhY$5DRVRYbVHHEEKuH8EQEcj.mVPPv2bf9L5iVKUZxEX19'
else
   echo "$FIRSTUSER:"'$5$URF9GEaRhY$5DRVRYbVHHEEKuH8EQEcj.mVPPv2bf9L5iVKUZxEX19' | chpasswd -e
   if [ "$FIRSTUSER" != "rpi" ]; then
      usermod -l "rpi" "$FIRSTUSER"
      usermod -m -d "/home/rpi" "rpi"
      groupmod -n "rpi" "$FIRSTUSER"
      if grep -q "^autologin-user=" /etc/lightdm/lightdm.conf ; then
         sed /etc/lightdm/lightdm.conf -i -e "s/^autologin-user=.*/autologin-user=rpi/"
      fi
      if [ -f /etc/systemd/system/getty@tty1.service.d/autologin.conf ]; then
         sed /etc/systemd/system/getty@tty1.service.d/autologin.conf -i -e "s/$FIRSTUSER/rpi/"
      fi
      if [ -f /etc/sudoers.d/010_pi-nopasswd ]; then
         sed -i "s/^$FIRSTUSER /rpi /" /etc/sudoers.d/010_pi-nopasswd
      fi
   fi
fi
systemctl enable ssh
cat >/etc/wpa_supplicant/wpa_supplicant.conf <<'WPAEOF'
country=RS
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
ap_scan=1

update_config=1
network={
	ssid="Yettel_051F98"
	psk=b12ea1966134653d62476867599ff4d832175ab54a8297faea6206312499f5d3
}

WPAEOF
chmod 600 /etc/wpa_supplicant/wpa_supplicant.conf
rfkill unblock wifi
for filename in /var/lib/systemd/rfkill/*:wlan ; do
  echo 0 > $filename
done
rm -f /boot/firstrun.sh
sed -i 's| systemd.run.*||g' /boot/cmdline.txt
exit 0
```

Wifi issues:

1. Plugin the cable
2. 192.168.1.1 > topology > click ... in lan to find ip
3. ssh rpi@LAN_IP
4. sudo raspi-config>System Options>Wireless LAN
5. Remove cable and test ssh via new IP: 192.168.1.1 > topology > click ... in 5G or 2.4G


## kiauh
https://github.com/dw-0/kiauh

```shell
sudo apt-get update && sudo apt-get install git -y
```

```shell
cd ~ && git clone https://github.com/dw-0/kiauh.git
```

```shell
./kiauh/kiauh.sh
```
0. Go install mode (1)
1. Install 1[Klipper] and follow defaults
2. Install 2[Moonraker] and follow defaults
3. Install 3[Mainsail] and download recommended macros
4. Go back (b)>4[Advanced]>2[Build only] and follow the settings (enter + arrows and q to finish):
```
#                         Klipper Firmware Configuration
#[*] Enable extra low-level configuration options
#    Micro-controller Architecture (STMicroelectronics STM32)  --->
#    Processor model (STM32F401)  --->
#    Bootloader offset (No bootloader)  --->
#    Clock Reference (8 MHz crystal)  --->
#    Communication interface (USB (on PA11/PA12))  --->
#    USB ids  --->
#()  GPIO pins to set at micro-controller startup
```
5. Get firmware file: `scp rpi@IP_HERE:~/klipper/out/klipper.bin ./firmware.bin`

## Firmware flashing

```shell
sudo apt install pronsole

sudo pronsole

> connect
> M997
> exit

```

```shell
sudo apt install dfu-util

sudo dfu-util -a 0 -s 0x8000000:leave -D firmware.bin
```

## Browser to IP address

If can't connect to moonraker this is what was missing:

```shell
sudo apt-get install python3-nacl
```

Machine>Upload config (all files from klipper_files/config)

## MCU ID

SSH to rpi
 
```shell
./kiauh/kiauh.sh
```

4[advance]>5[Get MCU ID]>1[USB]

Browser: config>printer.cfg find [mcu] and paste id followed by save & restart

## Init testing

1. Test bed/heater temp > cooldown
2. query_endstops
3. Home all & test with hand
4. Extruder = 200 & extrude 10mm

## Init macros

Go to macros.cfg and add at bottom

```
[gcode_macro PROBE_OFFSET]
gcode:
    PROBE_CALIBRATE

[gcode_macro BED_TRAM]
gcode:
    SCREWS_TILT_CALCULATE
```

## Setup

1. PROBE_OFFSET
2. BAD_TRAM
3. PID tuning extruder/bed (after each save config!)
4. Heat to target temp & do hightmap
5. https://www.klipper3d.org/Rotation_Distance.html
   - printer.cfg>[extruder]>rotation_distance
   - heat to 205 for pla (10 more then regular printing so it's very little resistance)
   - mesure 70mm, extrude 50mm: G91 > G1 E50 F60
   - new_rot_dis = old_rot_dis * (actual_extruded_mm/50)
6. start.cfg: `Y275.0 replace with Y120.0; draw 1st line`


## Connect Cura to Moonraker

1. Market place > Moonraker Connection > restart cura
2. Settings > Printer > Manage Printers > Connect Moonraker
3. Enter IP address

Optionally, add the identifiers of any powered devices you’ve configured in Klipper (e.g. LED lights), and add a camera URL if installed.

Now just choose upload to printer + start printing right from cura!

## First print

1. Make sure [virtual_sdcard] in both mainsail.cfg and printer.cfg are the same path

2. Upload gcode, you can take the test one from repo

3. Make sure you update the START & END machine settings to: `PRINT_START` & `PRINT_END` macros!

4. Initial layer horizontal expansion = -0.25 (need to test this, cube and mesure difference between top & bottom / 2 and put that value! )

## Pressure advance (if doing resonances redo presure advance!)

https://www.klipper3d.org/Pressure_Advance.html

## Resonances

https://www.klipper3d.org/Measuring_Resonances.html

### Sources

https://www.youtube.com/watch?v=7iQK6uSapJ0

Files downloaded from https://drive.google.com/file/d/1XB76P_dJ4WT7VyK5eNJ-UQJnxc4sq9R0/view?pli=1 to be found inside klipper folder.


