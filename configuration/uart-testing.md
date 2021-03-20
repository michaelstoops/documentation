## Observations

### Prep

Device used for testing:
```
pi@raspberrypi:~ $ cat /proc/cpuinfo
processor	: 0
model name	: ARMv7 Processor rev 3 (v7l)
BogoMIPS	: 108.00
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd08
CPU revision	: 3

processor	: 1
model name	: ARMv7 Processor rev 3 (v7l)
BogoMIPS	: 108.00
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd08
CPU revision	: 3

processor	: 2
model name	: ARMv7 Processor rev 3 (v7l)
BogoMIPS	: 108.00
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd08
CPU revision	: 3

processor	: 3
model name	: ARMv7 Processor rev 3 (v7l)
BogoMIPS	: 108.00
Features	: half thumb fastmult vfp edsp neon vfpv3 tls vfpv4 idiva idivt vfpd32 lpae evtstrm crc32
CPU implementer	: 0x41
CPU architecture: 7
CPU variant	: 0x0
CPU part	: 0xd08
CPU revision	: 3

Hardware	: BCM2711
Revision	: b03114
Serial		: 10000000afdf7c6c
Model		: Raspberry Pi 4 Model B Rev 1.4
```

SD card imaged from 2021-01-11-raspios-buster-armhf-lite.zip (SHA265: d49d6fab1b8e533f7efc40416e98ec16019b9c034bc89c59b83d0921c2aefeef)

```shell
sudo systemctl enable ssh
sudo systemctl start ssh
```

### Method
```shell
# Show config.txt lines, except block headers, comments, and empty lines
grep "^[^\[\s#]" /boot/config.txt

# output the kernel command line
cat /boot/cmdline.txt 

# list all known serial port character devices
ls -l /dev/serial* /dev/ttyS* /dev/ttyAMA*

# list serial ports in device tree
for DIR in /proc/device-tree/soc/serial*; do  echo $DIR; cat $DIR/compatible; echo; done;
```

### Scenario 1: First boot

SD card imaged from 2021-01-11-raspios-buster-armhf-lite.zip (SHA265: d49d6fab1b8e533f7efc40416e98ec16019b9c034bc89c59b83d0921c2aefeef)

```
pi@raspberrypi:~ $ # Show config.txt lines, except block headers, comments, and empty lines
pi@raspberrypi:~ $ grep "^[^\[\s#]" /boot/config.txt
dtparam=audio=on
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # output the kernel command line
pi@raspberrypi:~ $ cat /boot/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=71606cfa-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list all known serial port character devices
pi@raspberrypi:~ $ ls -l /dev/serial* /dev/ttyS* /dev/ttyAMA*
ls: cannot access '/dev/ttyS*': No such file or directory
lrwxrwxrwx 1 root root          7 Jan 11 13:08  /dev/serial1 -> ttyAMA0
crw-rw---- 1 root dialout 204, 64 Jan 11 13:09  /dev/ttyAMA0
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list serial ports in device tree
pi@raspberrypi:~ $ for DIR in /proc/device-tree/soc/serial*; do  echo $DIR; cat $DIR/compatible; echo; done;
/proc/device-tree/soc/serial@7e201000
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201400
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201600
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201800
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201a00
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e215040
brcm,bcm2835-aux-uart
```

Analysis:
* Serial console would not be functional. Even though `cmdline.txt` shows `console=serial0,115200`, `/dev/serial0` is not linked. Confirmed using a serial terminal.

### Scenario 2: raspi-config variations

This scenario exercises the options offered by raspi-config. It also covers some of the `config.txt` options, since that's mostly how raspi-config configures the UARTs.

Continued from scenario 1--card not reimaged.

```
sudo raspi-config
3
P6
"Would you like a login shell to be accessible over serial?"
Yes
"The serial login shell is enabled"
"The serial interface is enabled"
sudo reboot
```

```
pi@raspberrypi:~ $ # Show config.txt lines, except block headers, comments, and empty lines
pi@raspberrypi:~ $ grep "^[^\[\s#]" /boot/config.txt
dtparam=audio=on
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
enable_uart=1
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # output the kernel command line
pi@raspberrypi:~ $ cat /boot/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=71606cfa-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list all known serial port character devices
pi@raspberrypi:~ $ ls -l /dev/serial* /dev/ttyS* /dev/ttyAMA*
lrwxrwxrwx 1 root root          5 Mar 20 03:24 /dev/serial0 -> ttyS0
lrwxrwxrwx 1 root root          7 Mar 20 03:24 /dev/serial1 -> ttyAMA0
crw-rw---- 1 root dialout 204, 64 Mar 20 03:24 /dev/ttyAMA0
crw--w---- 1 root tty       4, 64 Mar 20 03:25 /dev/ttyS0
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list serial ports in device tree
pi@raspberrypi:~ $ for DIR in /proc/device-tree/soc/serial*; do  echo $DIR; cat $DIR/compatible; echo; done;
/proc/device-tree/soc/serial@7e201000
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201400
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201600
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201800
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201a00
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e215040
brcm,bcm2835-aux-uart
```

Serial console is functional, confirmed using a terminal.

```
sudo raspi-config
3
P6
"Would you like a login shell to be accessible over serial?"
No
"Would you like the serial port hardware to be enabled?"
Yes
"The serial login shell is disabled"
"The serial interface is enabled"
# automatically reboots
```

```
pi@raspberrypi:~ $ # Show config.txt lines, except block headers, comments, and empty lines
pi@raspberrypi:~ $ grep "^[^\[\s#]" /boot/config.txt
dtparam=audio=on
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
enable_uart=1
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # output the kernel command line
pi@raspberrypi:~ $ cat /boot/cmdline.txt
console=tty1 root=PARTUUID=71606cfa-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list all known serial port character devices
pi@raspberrypi:~ $ ls -l /dev/serial* /dev/ttyS* /dev/ttyAMA*
lrwxrwxrwx 1 root root          5 Mar 20 03:27 /dev/serial0 -> ttyS0
lrwxrwxrwx 1 root root          7 Mar 20 03:27 /dev/serial1 -> ttyAMA0
crw-rw---- 1 root dialout 204, 64 Mar 20 03:27 /dev/ttyAMA0
crw-rw---- 1 root dialout   4, 64 Mar 20 03:27 /dev/ttyS0
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list serial ports in device tree
pi@raspberrypi:~ $ for DIR in /proc/device-tree/soc/serial*; do  echo $DIR; cat $DIR/compatible; echo; done;
/proc/device-tree/soc/serial@7e201000
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201400
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201600
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201800
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201a00
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e215040
brcm,bcm2835-aux-uart
```

```
sudo raspi-config
3
P6
"Would you like a login shell to be accessible over serial?"
No
"Would you like the serial port hardware to be enabled?"
No
"The serial login shell is disabled"
"The serial interface is disabled"
# automatically reboots
```

```
pi@raspberrypi:~ $ # Show config.txt lines, except block headers, comments, and empty lines
pi@raspberrypi:~ $ grep "^[^\[\s#]" /boot/config.txt
dtparam=audio=on
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
enable_uart=0
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # output the kernel command line
pi@raspberrypi:~ $ cat /boot/cmdline.txt
console=tty1 root=PARTUUID=71606cfa-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list all known serial port character devices
pi@raspberrypi:~ $ ls -l /dev/serial* /dev/ttyS* /dev/ttyAMA*
ls: cannot access '/dev/ttyS*': No such file or directory
lrwxrwxrwx 1 root root          7 Mar 20 03:28  /dev/serial1 -> ttyAMA0
crw-rw---- 1 root dialout 204, 64 Mar 20 03:28  /dev/ttyAMA0
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list serial ports in device tree
pi@raspberrypi:~ $ for DIR in /proc/device-tree/soc/serial*; do  echo $DIR; cat $DIR/compatible; echo; done;
/proc/device-tree/soc/serial@7e201000
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201400
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201600
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201800
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201a00
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e215040
brcm,bcm2835-aux-uart
```

### Scenario 3: Device-tree overlays

Freshly-prepared SD card.

```shell
echo "dtoverlay=uart2
dtoverlay=uart3
dtoverlay=uart4
dtoverlay=uart5
" | sudo tee -a /boot/config.txt
sudo reboot
```

```
pi@raspberrypi:~ $ # Show config.txt lines, except block headers, comments, and empty lines
pi@raspberrypi:~ $ grep "^[^\[\s#]" /boot/config.txt
dtparam=audio=on
dtoverlay=vc4-fkms-v3d
max_framebuffers=2
dtoverlay=uart2
dtoverlay=uart3
dtoverlay=uart4
dtoverlay=uart5
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # output the kernel command line
pi@raspberrypi:~ $ cat /boot/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=a093ebfc-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list all known serial port character devices
pi@raspberrypi:~ $ ls -l /dev/serial* /dev/ttyS* /dev/ttyAMA*
ls: cannot access '/dev/ttyS*': No such file or directory
lrwxrwxrwx 1 root root          7 Mar 20 03:48  /dev/serial1 -> ttyAMA0
crw-rw---- 1 root dialout 204, 64 Mar 20 03:48  /dev/ttyAMA0
crw-rw---- 1 root dialout 204, 65 Mar 20 03:48  /dev/ttyAMA1
crw-rw---- 1 root dialout 204, 66 Mar 20 03:48  /dev/ttyAMA2
crw-rw---- 1 root dialout 204, 67 Mar 20 03:48  /dev/ttyAMA3
crw-rw---- 1 root dialout 204, 68 Mar 20 03:48  /dev/ttyAMA4
pi@raspberrypi:~ $
pi@raspberrypi:~ $ # list serial ports in device tree
pi@raspberrypi:~ $ for DIR in /proc/device-tree/soc/serial*; do  echo $DIR; cat $DIR/compatible; echo; done;
/proc/device-tree/soc/serial@7e201000
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201400
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201600
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201800
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e201a00
arm,pl011arm,primecell
/proc/device-tree/soc/serial@7e215040
brcm,bcm2835-aux-uart
```

I'm not going to test disable-bt and miniuart-bt, because I think I know what they do.
