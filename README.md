# Disclosure
This project is for educational purposes only.

The methods described here involve forceful memory rewriting and the use of non-standard firmware. These actions carry a high risk of "bricking" (permanently damaging) your device.
- Proceed at your own risk. I take no responsibility for any hardware damage, data loss, or bricked devices resulting from the use of this information.
- Warranty: Following these instructions will void your manufacturer's warranty.
- Software: The intermediate binary provided is used solely to enable restricted commands for educational demonstration. I do not claim ownership of this software.

## Flashing OpenWRT on an Aruba AP11 Access Point
Aruba AP11 access points are just Aruba AP303 access points, with more restrictions: cloud configuration only, and no "simple" way to bypass the protection for flashing firmware.

To flash OpenWRT, some equipement is required:
- an UART cable with Micro USB termination (3.3V) or make one with a Micro USB cable and an UART USB adapter (must be set to 3.3V, only plug GND, RXD and TXD, connections must be Key RXD=>Device TXD and vice versa). **NEVER PLUG VCC/POWER**
- a TFTP server to host the OpenWRT firmware and the binary to bypass firmware protection - the server must be reachable from the AP.
- a RJ45 cable

```
  +------------------------+
  |      (UART ADAPTER)    |
  |  O   O   O   O   O   O |
  +--^---^---^---^---------+
         |   |   |
        GND TXD RXD

  TXD : Transmit (To AP RX)
  RXD : Receive (To AP TX)
  GND : Ground
```

The connection was made with Putty with the following settings:
- Baud Rate: 115200
- Data bits: 8
- Stop bits: 1
- Parity: None
- Flow Control: None

### Setting up the TFTP server
The server can be created in an Alpine Linux virtual machine like this:
```bash
apk update && apk add tftp-hpa
mkdir -p /var/tftpboot && chmod 777 /var/tftpboot
in.tftpd --listen --address :69 --secure /var/tftpboot &
```

### Prepare the files
The binary `appsbl.bin` must be put on the TFTP root directory. For the OpenWRT binary, it must be retrieved from the official documentation [here](https://openwrt.org/toh/aruba/ap-303). The binary must be renamed `ipq40xx.ari` and put in the TFTP root directory. 

### Memory rewrite
The memory must be rewritten to get full access.

To access apboot, the boot process must be intercepted by pressing "Enter" when "Hit <Enter> to stop autoboot:" is shown.
```
apboot> dhcp

apboot> setenv serverip XXX.XXX.XXX.XXX # Alpine Linux IP
apboot> setenv ipaddr XXX.XXX.XXX.XXX   # AP IP (if not set with DHCP)
apboot> saveenv                         # Save envs

# Load the bypass bootloader into RAM
apboot> tftp appsbl.bin

# Flash the new bootloader (DANGEROUS ZONE)
apboot> sf probe 0
apboot> sf protect off
apboot> sf erase 0x0000000f0000 0xf0000
apboot> sf write 0x84000000 0x0000000f0000 0xf0000

# Restart the device
apboot> reset
```
