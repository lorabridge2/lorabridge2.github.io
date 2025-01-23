# Data Types

LoRaBridge [LoRaWAN Interface](../system_overview/sw_components.md#bridge-lorawan-interface) and [Gateway Converter](../system_overview/sw_components.md#converter) know different types of messages that are send from the bridge to the gateway. The different data types accompanied with brief descriptions are given below.

```python
class lbdata_types(IntEnum):
    data = 7
    timesync_req = 1
    system_event = 2
    user_event = 3
    lbflow_digest = 4
    lbdevice_join = 5
    heartbeat = 6
    lbdevice_name = 8
```

## Data
| Type  | Data  |
|---|---|
|  7  | Variable size, [MessagePack](https://msgpack.org/) payload   |

*Description*:
This contains the compressed sensor data and is used by [Forwarder](../system_overview/sw_components.md#forwarder). Known attribute names are replaced with a number and used as keys for the msgpack object. The key `-1` is used for the topic, which will usually be the ieee number of the sensor.
If it is the ieee number, we convert it into an integer to save memory, otherwise we store it as string.

## Timesync Request
| Type  | 
|---|
|  1  |

*Description*:
Used by [LoRaWAN Interface](../system_overview/sw_components.md#bridge-lorawan-interface) and [LoRaWAN module firmware](../system_overview/sw_components.md#firmware) for initiating [`LoRaWAN Application Layer Clock Synchronization`](https://lora-alliance.org/wp-content/uploads/2020/11/application_layer_clock_synchronization_v1.0.0.pdf) with [ChirpStack](../system_overview/sw_components.md#chirpstack). This synchronizes the time between the gateway and the bridge.

## System Event
| Type  | Data  |
|---|---|
|  2  | Variable size, String   |

*Description*:
This can be used to send messages about the system status to the gateway.
For instance, this type is used to send messages about failures of the serial port connection between the bridge and their ESP32 LoRaWAN module.

## User Event
| Type  | Data  |
|---|---|
|  3  | Variable size, String   |

*Description*:
This type is used by the [LoRaWAN Interface](../system_overview/sw_components.md#bridge-lorawan-interface) to send user-defined messages to the gateway, parse them and pass them to the [Flow UI](../system_overview/sw_components.md#gateway-flow-ui). Such messages can be defined in flows via the `Notification` node in the [Flow UI](../system_overview/sw_components.md#gateway-flow-ui), are triggered during the execution of the Node-RED flow and are displayed inside the `Status` overlay of the [Flow UI](../system_overview/sw_components.md#gateway-flow-ui).

## LBFlow Digest
| Type  | Data  |
|---|---|
|  4  | Variable size, Binary String (Hash) |

*Description*:
This is used by the [Automation Manager](../system_overview/sw_components.md#automation-manager) to verify the correctness of the transmitted flow commands.

## LBDevice Join
| Type  | LB_ID | Attributes  |
|---|---|---|
|  5  |  B  | Variable number of Bytes |

*Description*:
Used by the [Automation Manager](../system_overview/sw_components.md#automation-manager) to send information about a known device to the gateway. `LB_ID` contains the internal used number of the device. `Attributes` contains a variable amount of bytes, each representing the number of a known sensor attribute.

## LBDevice Name
| Type  | LB_ID | IEEE ID | Name |
|---|---|---|---|
|  8  | B | 8 Bytes | Variable size, String | 

*Description*:
Used by the [Automation Manager](../system_overview/sw_components.md#automation-manager) to send the ieee number and the name of a known device to the gateway. `LB_ID` contains the internal used number of the device. `IEEE ID` contains the IEEE ID of the device as an 8 Bit integer. `Name` contains the device's value of the `modelId` attribute of zigbee2mqtt. 

## Heartbeat
| Type  |
|---|
|  6  |

*Description*:
[LoRaWAN Interface](../system_overview/sw_components.md#bridge-lorawan-interface) send a heartbeat message to the gateway.