# Hardware components

All components necessary to build LoRaBridge bridge and gateway units are listed below. *Please note: the components listed here are the ones used to build
the first proof-of-concept LoRaBridge implementation with verified functionality*. Some components such as different Raspberry PI models, other Zigbee coordinator sticks or alternative LoRa modems (supporting by LMIC node) might work out-of-the-box. Other components like external LoRaWAN gateways may require adjustments to the LoRaBridge software.

## Table of components

| Device  | Manufacturer (link) | Distributor (link)  |  
|---|---|---|
| Raspberry PI 4B  | [Raspberry PI Foundation](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/) |   |
| LilyGo LoRa32 Dongle  | [LilyGo](https://lilygo.cc/products/lora3)  | [LilyGo](https://lilygo.cc/products/lora3)   |
| CC2652RB Zigbee USB development stick | [slae.sh](https://slae.sh/projects/cc2652/)| [Ewelink](https://ewelinkstore.com/product/cc2652rb-zigbee-usb-development-stick/?v=fa868488740a)|
| Dragino PG1301 LoRaWAN GPS Concentrator | [Dragino](https://www.dragino.com/products/lora/item/149-lora-gps-hat.html) |    |

## Hardware Setup

### Bridge

To set up a bridge you need:

- [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- Power adapter
- Micro SD card  (16+ GB recommended)
- LilyGo LoRa32 Dongle ESP32 433/868/915MHZ 0.96 Inch OLED
- CC2652RB Zigbee USB development stick

![Bridge](../assets/Bridge.png)

___

### Gateway

To set up a gateway you need:

- [Raspberry Pi 4](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- Power adapter
- Micro SD card  (16+ GB recommended)
- Dragino PG1301 LoRaWAN GPS Concentrator

![Gateway](../assets/Gateway.png)