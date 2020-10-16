scuznet
=======

Open source hardware emulating a Nuvolink SCSI to Ethernet adapter, for use
with vintage Macintosh computers.

# Status

The firmware is at an early alpha stage. There are many things that are known
to not work correctly, including:

* The emulated hard drive does not provide mode information yet and cannot be
  directly formatted by available tools I've tried. Instead, instructions
  [here](http://www.codesrc.com/mediawiki/index.php/HFSFromScratch) have been
  helpful for creating test images.
* Some SD cards have exhibited compatibility issues that have yet to be
  fully identified and corrected.
* MAC address changing at runtime from the driver is not implemented yet.

There are likely a large number of other bugs that have yet to be identified
and/or fixed.

# Compatibility

This implementation has some limitations by design:

1. There must be only one initiator on the bus.
2. The initiator on the bus must be at ID 7.
3. Read parity is not used. Transmission parity is provided but may be disabled
   to slightly improve performance.

These should not be a significant problem for the target computer platform,
which for the most part has similar limitations.

# Device Configuration

See [SETTINGS.html](SETTINGS.html) for a description of firmware configuration
options.

# Firmware programming
The firmware can be programmed with a Raspberry Pi or other avrdude-compatible programmer. The following instructions are for a Raspberry Pi, running Raspberry Pi OS (formerly Raspbian)

1. Install necessary packages and checkout scuznet code
```
sudo apt install avrdude avr-libc binutils-avr gcc-avr
cd ~
git clone https://github.com/saybur/scuznet.git
```
2. If necessary, update the microcontroller device you're using:
```
pi@rascsi-dev:~/scuznet $ more Makefile 
PROGRAMMER := avrispv2
MCU := atxmega128a3u  ## UPDATE THIS LINE
F_CPU := 32000000
OPTIONS := -DHW_V01 -DDEBUGGING
```
3. Compile the software
```
make all
```
4. Connect the following signals from your Raspberry Pi to the programming header on the scuzznet.
----
Pin 

2
| Raspberry Pi Signal | Raspberry Pi Pin | Scuznet Pin | Scuznet Signal |
| -------------- | --------- | ---------- | ---------- |
|3.3v|Pin 1|Pin 2|VCC|
|Ground|Pin 9|Pin 6|GND|
|~~MISO (GPIO09)|Pin 21|Pin 1|MISO~~|
|MOSI (GPIO10)|Pin 19|Pin 4|MOSI|
|~~SCLK (GPIO11)|Pin 23|Pin 3|SCK~~|
|GPIO25|Pin 22|Pin 5|RST|

For the Scuznet: use the 6-pin AVR pinout
<a href="https://telecnatron.com/reference/pinouts/avr-isp/avr-isp-pinout-345x.png"><img src="https://telecnatron.com/reference/pinouts/avr-isp/avr-isp-pinout-345x.png"/></a>

Setup the AVR configuration - /usr/local/etc/avrdude.conf
```
programmer
  id = "linuxspi";
  desc = "Use Linux SPI device in /dev/spidev*";
  type = "linuxspi";
  reset = 25;
;
```

Example output: http://kevincuzner.com/2013/05/27/raspberry-pi-as-an-avr-programmer/
```
kcuzner@tiny-tim:~/avrdude/avrdude$ sudo avrdude -c linuxspi -p m48 -P /dev/spidev0.0 -U flash:w:../blink.hex 
[sudo] password for kcuzner: 

avrdude: AVR device initialized and ready to accept instructions

Reading | ################################################## | 100% 0.00s

avrdude: Device signature = 0x1e9205
avrdude: NOTE: "flash" memory has been specified, an erase cycle will be performed
         To disable this feature, specify the -D option.
avrdude: erasing chip
avrdude: reading input file "../blink.hex"
avrdude: input file ../blink.hex auto detected as Intel Hex
avrdude: writing flash (2282 bytes):

Writing | ################################################## | 100% 0.75s

avrdude: 2282 bytes of flash written
avrdude: verifying flash memory against ../blink.hex:
avrdude: load data flash data from input file ../blink.hex:
avrdude: input file ../blink.hex auto detected as Intel Hex
avrdude: input file ../blink.hex contains 2282 bytes
avrdude: reading on-chip flash data:

Reading | ################################################## | 100% 0.56s

avrdude: verifying ...
avrdude: 2282 bytes of flash verified

avrdude: safemode: Fuses OK

avrdude done.  Thank you.
```

# License

Except where otherwise noted, all files in this repository are available under
the terms of the GNU General Public License, version 3, available in the
LICENSE document. There is NO WARRANTY, not even for MERCHANTABILITY or
FITNESS FOR A PARTICULAR PURPOSE. For details, refer to the license.
