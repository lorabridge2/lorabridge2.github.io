# Troubleshooting

Here you can find information on how to remedy connectivity issues.

## Sensor values not available at the Gateway

- The bridge unit is possibly placed too far away from the gateway unit and therefore the LoRaWAN packets with sensor data are discarded by the gateway. Although a maximum LoRaWAN reach of several kilometers can be achieved in outdoors with a line-of-sight link, an indoor reach is usually limited to 40m-100m depending on the rooms/floors/walls between the transmitter and the receiver. It is a good idea to test connectivity between the bridge and the gateway unit by placing them first a few meters apart. In such setting the connection between the bridge and the gateway should be accomplished in seconds. To test the connection we recommend to interact with some
sensors or use a device, which reports its status periodically (such as some wireless plugs/lightbulbs).

- Check that the antenna of the LoRAWAN gateway concentrator is connected to the correct antenna input.

- LoRaWAN join procedure might still take a while given that the bridge device is placed far away from the gateway.
During the join procedure, the LoRaWAN transmitter tries to adjust the so called spreading factor setting given that
the initial accept message from the gateway gets dropped. In that case, due to the RF channel access limitations the
next join request message is sent after a certain delay. Thus, the join procedure might in the worst case take several minutes.

- Try to reboot the bridge unit. This forces a reboot of the Zigbee2MQTT container, which might help in case the Zigbee coordinator has become unresponsive. 

## Cannot upload automation setup to the Bridge via LoRaMation

- Check the LoRaWAN connectivity as described above. 

## Uploading automation setups takes a long time

- The automation rules are transmitted via LoRaWAN downlink messages. Since LoRaWAN is originally intended to transfer sporadically small IoT payloads, its wireless channel utilization is regionally regulated and that is why the upload procedure might take up to 15-30 minutes. For simple automations with 2-3 devices and few nodes, an upload duration of ca. 1-2 minutes should be achievable.

## No new sensor values are registered in Home Assistant

- There might be an issue with a used sensor device and the Zigbee2MQTT. Please refer to [Z2M github repository](https://github.com/Koenkk/zigbee2mqtt) for
issues related to device compatibility.

- The firmware of the Zigbee dongle might be outdated. Check [Z2M homepage](https://www.zigbee2mqtt.io/guide/adapters/#recommended) for latest firmwares
and guides how to update a ZigBee adapter.

- Zigbee adapter is not supported by Zigbee2MQTT. Refer to [Z2M homepage](https://www.zigbee2mqtt.io/guide/adapters/#recommended) for recommended Zigbee
adapters.

- LoRaWAN link might be driven to its limits, i.e. the bridge unit is placed too far away from the gateway unit. See [How to use page](How-to-use.md) on 
how to inspect LoRaWAN link quality in Chirpstack.

## Cannot pair a sensor to the bridge device

<em> Despite several tries, the bridge will not accept a pairing procedure of a ZigBee Device.</em>

- A sensor device is possibly not supported by the Zigbee2MQTT coordinator. Please check [Z2M homepage](https://www.zigbee2mqtt.io/supported-devices/) to
see if your device is on the list of supported devices.

- Make sure the distance between the device and the bridge unit is not excessively high (e.g. > 30m).

- Make sure the status of Zigbee2MQTT is online (e.g. via www based GUI)