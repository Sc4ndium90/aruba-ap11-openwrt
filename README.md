# Disclosure
This project is for educational purposes only.

The methods described here involve forceful memory rewriting and the use of non-standard firmware. These actions carry a high risk of "bricking" (permanently damaging) your device.
- Proceed at your own risk. I take no responsibility for any hardware damage, data loss, or bricked devices resulting from the use of this information.
- Warranty: Following these instructions will void your manufacturer's warranty.
- Software: The intermediate binary provided is used solely to enable restricted commands for educational demonstration. I do not claim ownership of this software.

## Flashing OpenWRT on an Aruba AP11 Access Point
Aruba AP11 access points are just Aruba AP303 access points, with more restrictions: cloud configuration only, and no "simple" way to bypass the protection for flashing firmware.

To flash OpenWRT, some equipement is required:
- an UART cable with Micro USB termination (3.3V) or make one with a Micro USB cable and an UART USB adapter (must be set to 3.3V, only plug VCC, GND, RXD and TXD, connections must be Key RXD=>Device TXD and vice versa).
- a TFTP server to host the OpenWRT firmware and the binary to bypass firmware protection - the server must be reachable from the AP.
- a RJ45 cable

### Setting up the TFTP server
The server can be created in an Alpine Linux virtual machine like this:
```bash
apk update && apk add tftp-hpa
mkdir -p /var/tftpboot && chmod 777 /var/tftpboot
in.tftpd --listen --address :69 --secure /var/tftpboot &
```

### Prepare the files
The binary `appsbl.bin` must be put on the TFTP root directory. For the OpenWRT binary, it must be retrieved from the official documentation [here](https://openwrt.org/toh/aruba/ap-303). The binary must be renamed `ipq40xx.ari` and put in the TFTP root directory. 
