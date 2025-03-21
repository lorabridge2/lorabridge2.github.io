# Software Components

We utilise [Docker](https://docs.docker.com/get-started/overview/) and [Docker Compose](https://docs.docker.com/compose/) to provide all necessary LoRaBridge software components as a multi-container Docker application. The components needed for bridges and gateways vary and are listed below.

## Bridge

![Bridge](../assets/Bridge.drawio.svg)

> [Butix](https://commons.wikimedia.org/wiki/User:Butix), based on works by [Lucasbosch](https://commons.wikimedia.org/wiki/User:Lucasbosch) and [Cmykey](https://commons.wikimedia.org/wiki/User:Cmykey), [Raspberry Pi 3 illustration](https://commons.wikimedia.org/wiki/File:Raspberry_Pi_3_illustration.svg), modified, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/legalcode)

As shown [here](hw_components.md#bridge), a bridge needs a ZigBee dongle for connecting ZigBee devices, a LoRaWAN hat for establishing a link to the gateway as well as optionally a network connection.

### Zigbee2MQTT

[Zigbee2MQTT](https://www.zigbee2mqtt.io/) exchanges device data with the ZigBee dongle as it handles join events, status updates and so on. It keeps a list of devices and publishes the device data via MQTT messages to [Eclipse Mosquitto](#eclipse-mosquitto).

### Eclipse Mosquitto

[Mosquitto](https://mosquitto.org/) is an MQTT message broker, meaning you can publish your own messages as well as subscribe to messages sent by others. In case of the bridge, [Zigbee2MQTT](#zigbee2mqtt) publishes the sensor data to the Mosquitto server, while the [Forwarder](#forwarder) is subscribed to those.

### Forwarder

The [Forwarder](https://github.com/lorabridge2/bridge-forwarder) is a self-provided Python3 program and listens to message on the [Mosquitto](#eclipse-mosquitto) server. It removes disabled attributes (a list that can be modified via the [web interface](#web-interface)), substitutes known attibute keys as well as reformats and compresses the data by using the [MessagePack](https://msgpack.org/) format. The resulting compressed data is pushed to a list in a [Redis](#redis) server.

### Redis

[Redis](https://redis.io/) is an in-memory data store and is used as a cache / message queue in our case. It receives compressed sensor data from the [Forwarder](#forwarder) and holds the data until [LoRaWAN TX](#lorawan-tx) retrieves it.

### Bridge LoRaWAN Interface watcher

[LoRaWAN Interface Watcher](https://github.com/lorabridge2/bridge-lorawan-interface-watcher) is a Python3 program that regularly checks the status of the [LoRaWAN Interface](#bridge-lorawan-interface) and restarts the container, if it crashed. Furthermore, it sends a notification to the Gateway via a system event. It uses the Docker API via a socket.

> Docker should restart containers automatically when they crash using the restart policy we configured. 
> Unfortunately, Docker refuses to do so, when a device needed by the container is missing. 
> As our desired behaviour would be to restart the container as soon as the device is readded, we chose to implement it using our [LoRaWAN Interface Watcher](#bridge-lorawan-interface-watcher).

### Bridge LoRaWAN Interface

[LoRaWAN TX](https://github.com/lorabridge2/bridge-lorawan-interface) is a python program, which acts as an interface between the physical LoRaWAN modem and the software components on the bridge. 

It has three main tasks:

1. Bridge system time synchronization via TimeSyncRequest feature of LoRaWAN 1.0.3. 
2. Listening to downlink data events and forwarding the data towards automation manager 
3. Fetching compressed device data and [user](../additional/data_types.md#user-event)/[system](../additional/data_types.md#system-event) events from [Redis](#redis) and pushing them towards the gateway. 

### NodeRED

[NodeRED](https://github.com/lorabridge2/bridge-node-red) implements user defined automations. MQTT is used for interfacing Zigbee devices and Redis (via NodeRED Redis add-on) for interfacing LoRaBridge events.

### Automation manager

[Automation manager](https://github.com/lorabridge2/bridge-automation-manager) performs decompression of [automation configuration commands](../additional/compression_rules.md), stores automations in an intermediate data structure and ultimately composes NodeRED flows and uploads them to NodeRED.   

### Web Interface

Our [web interface](https://github.com/lorabridge/bridge-device-interface) is a self-provided [SvelteKit](https://svelte.dev/docs/kit/introduction) web application that shows the ZigBee devices, which are retrieved via the [SSE server](#sse-server). It enables you to disable unnecessary sensor attributes, in order to further reduce the transmitted data. Authentication is provided with [basic auth](#basic-auth) via nginx.

!!! info
    The default port via [Basic Auth](#basic-auth)  is `3000`

??? example
    ![Svelte web interface](../assets/svelte_interface.png)

### SSE Server

The [SSE server](https://github.com/lorabridge2/bridge-device-sse) is a self-provided TypeScript application. It retrieves a list of ZigBee devices from the [Zigbee2MQTT](#zigbee2mqtt) server and provides the data per HTTP as well as any updates to the data (e.g. new devices, additional attributes) per [server-sent events (SSE)](https://html.spec.whatwg.org/multipage/server-sent-events.html#server-sent-events) for the [web interface](#web-interface).

### Basic Auth

This component consists of a nginx and provides authentication for the [web interface](#web-interface) via Basic HTTP Authentication.

### Ofelia

[Ofelia](https://github.com/mcuadros/ofelia) is a job scheduler design for docker environments. This container is only connected to the host network and we use Ofelia to periodically retrieve the current IP addresses and store them inside a file. This file is mounted into the [LCD UI](#lcd-ui) container and provides thie IP information. This setup is necessary, because docker does not allow a container to belong to multiple networks at once (per `docker-compose.yml`).

## Gateway

=== "Logic Connections"

    ![Gateway](../assets/Gateway_logic.drawio.svg)

=== "Direct Connections"

    ![Gateway](../assets/Gateway_direct.drawio.svg)

> [Butix](https://commons.wikimedia.org/wiki/User:Butix), based on works by [Lucasbosch](https://commons.wikimedia.org/wiki/User:Lucasbosch) and [Cmykey](https://commons.wikimedia.org/wiki/User:Cmykey), [Raspberry Pi 3 illustration](https://commons.wikimedia.org/wiki/File:Raspberry_Pi_3_illustration.svg), modified, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/legalcode)
>
!!! note
    The <span style="color:light-dark(#8C8C8C, #757575);font-weight:bold;">grey</span> arrows symbolize the logical communication flow.
    The <span style="color:light-dark(#000000, #ffffff);font-weight:bold;">black/white</span> and <span style="color:light-dark(#994C00, #db9958);font-weight:bold;">brownie/orange</span> colored arrows in the gateway diagram above show the direct communation paths. 

As shown [here](hw_components.md#gateway), a gateway needs a LoRaWAN hat for establishing links to the bridges as well as optionally a network connection.

### Gateway Flow UI

The [web interface](https://github.com/lorabridge2/gateway-flow-ui) is based on the [SvelteKit](https://svelte.dev/docs/kit/introduction) web application framework in combination with the [Svelte UI](https://svelte.dev/docs/svelte/overview) component framework. The interface uses [Flowbite Svelte](https://flowbite-svelte.com/) for individual UI components (such as styled buttons) and [Svelte Flow](https://svelteflow.dev/) for flow visualization.

To enable undo functionality, individual versions of a flow are stored in a [CouchDB](https://couchdb.apache.org/) database, which resides locally in the browser. Furthermore, the changes are synchronized with a [CouchDB database on the gateway](#couchdb). This allows flows to be modified offline and uploaded later, ensuring no data is lost if the internet connection is lost.

??? info "Usage"
    We placed special emphasis on the usability of the user interface to make the creation of automations as intuitive as possible. For this purpose, the layout of the interface was built on well-known, commonly used structures.

    At the top, there is a menu bar where settings can be configured, and actions can be executed. The rest of the area is divided into three columns.

    The left column contains the available [automation nodes](automation_nodes.md) (graphical elements for constructing the flow diagram) that can be dragged and dropped into the visualization area.

    The middle column is used for visualizing the flow diagram. Here, the previously added nodes can be connected to define the automation process. Additionally, the properties of each node or automation element can be configured via various input fields. Device-related nodes, such as "Sensor" and "Output Device," also provide a list of available devices and the sensor attributes accessible per device.

    The right column is for managing individual flows. Flows can be thought of as simplified files in this context. Accordingly, new flows can be created, and existing flows can be renamed or deleted. Clicking on a specific flow in the right column loads it and displays it in the middle column.

    Changes to the flow diagram are saved automatically and can be undone or redone via the menu bar. Additionally, the flow can also be transferred to the bridge and activated or deactivated via the menu bar.

    The "Status" button opens a dedicated status area that contains messages from the bridge, such as critical error messages.

!!! info
    The default port is `3000`

### Gateway Flow Manager

To integrate the [web interface](#gateway-flow-ui) into the system, the [Flow Manager](https://github.com/lorabridge2/gateway-flow-manager) was created as a standalone Python3 component. The Flow Manager loads flows from the Redis database, which are stored there by the web interface when deploying the flows. It compares new versions with the previous version, if applicable, and generates the necessary compressed [action commands](../additional/compression_rules.md). These commands are transmitted to the bridge, where they are converted into the actual Node-RED flow by the [Automation Manager](#automation-manager). After transmission, the correctness of the transferred flow is verified on both sides via a checksum comparison.

### CouchDB

[CouchDB](https://couchdb.apache.org/) is a document-oriented NoSQL database with an emphasis on synchronization. It is used by the [Flow UI](#gateway-flow-ui) as an in browser instance for storing each change (version) of a flow created via the webinterface. Furthermore, the changes are synchronized with a CouchDB instance on the gateway. This allows flows to be modified offline and uploaded later, ensuring no data is lost if the internet connection is lost.

> We created our [own CouchDB Docker image](https://github.com/lorabridge2/couchdb-docker), as there is no prebuilt image for armhf (32 bit) systems. This image is only used for the database on gateway.

### Connection Watcher

The [Connection watcher](https://github.com/lorabridge2/gateway-connection-watcher) is a self-provided Python3 application, that uses the [Chirpstack](#chirpstack) API to retrieve the timestamp when the bridge was seen last.

### Packet Forwarder

The [Packet Forwarder](https://github.com/lorabridge2/gateway-forwarder) is a self-provided C application based on [this repository](https://github.com/fhessel/dragino_pi_gateway_fwd) that interacts with the LoRaWAN hat. It receives the LoRaWAN packets and publishes the data to the [ChirpStack Gateway Bridge](#chirpstack-gateway-bridge) on port `1700/udp`.

### Eclipse Mosquitto (Gateway)

[Mosquitto](https://mosquitto.org/) is an MQTT message broker, meaning you can publish your own messages as well as subscribe to messages sent by others. In case of the gateway, the [ChirpStack Gateway Bridge](#chirpstack-gateway-bridge) publishes the forwarded LoRaWAN data to the Mosquitto server. Various other services read/write data from to the Mosquitto server, while modifying the data. The next service in line in the logical processing chain is [ChirpStack](#chirpstack).

### ChirpStack Gateway Bridge

The [ChirpStack Gateway Bridge](https://www.chirpstack.io/docs/chirpstack-gateway-bridge/index.html) converts LoRa Packet Forwarder protocols in a common data format. It receives the data from the [Packet Forwarder](#packet-forwarder), converts it and publishes it to the [Mosquitto](#eclipse-mosquitto-gateway) for the [ChirpStack](#chirpstack).

### ChirpStack

[ChirpStack](https://www.chirpstack.io/docs/chirpstack/changelog.html) handles parts of the LoRaWAN communication like the authentication as well as provides a device inventory as well as a web interface for displaying LoRaWAN connections, data and for configuration. It persists data in a [PostgreSQL](#postgresql) database and uses [Redis](#redis-gateway) as a session store and for non-persistent data. It receives and publishes data via [Mosquitto](#eclipse-mosquitto-gateway).

!!! info
    The default port is `8080`

??? example
    ![ChirpStack web interface](../assets/Chirpstack.png)

### PostgreSQL

[PostgreSQL](https://www.postgresql.org/) is an object-relational database and by the [ChirpStack](#chirpstack) for storing persistent data.

### Redis (Gateway)

[Redis](https://redis.io/) is an in-memory data store and is used for storing session and non-persistent data by the [ChirpStack](#chirpstack).

### Converter

The [Converter](https://github.com/lorabridge2/gateway-converter) is a self-provided Python3 application, which listens for device data and [other types of messages](../additional/data_types.md) published by the [ChirpStack Gateway Bridge](#chirpstack-gateway-bridge) via MQTT message on the [Mosquitto](#eclipse-mosquitto-gateway). For the device data type, it decompresses the data, undoes the key substitution and reformats the data. Afterwards, the data is published back to the [Mosquitto](#eclipse-mosquitto-gateway) server. Other types of message are processed accordingly and passed to other service either via [Redis](#redis-gateway) or [Mosquitto](#eclipse-mosquitto-gateway).

### Device Manager

The [Device Manager](https://github.com/lorabridge2/gateway-device_manager) is a self-provided Python3 application keeping track of the seen devices via the [Redis](#redis-gateway) server. It publishes MQTT messages on the [Mosquitto](#eclipse-mosquitto-gateway) server for device discovery events and status (data) updates. These messages are picked up by the [HA Integration](#ha-integration) service.

### HA Integration

The [HA Integration](https://github.com/lorabridge2/gateway-ha_integration) is a self-provided Python3 application. It translates the MQTT messages sent by the [Device Manager](#device-manager) into messages understood by the MQTT integration of the [Home Assistant](#home-assistant).

!!! tip
    This additional translation step enables easy integration of other services (e.g. a replacement for [Home Assistant](#home-assistant) or an extra web interface).

### Home Assistant

Our [Home Assitant](https://github.com/lorabridge2/gateway-home-assistant) is a preconfigured version of the [official Home Assistant](https://www.home-assistant.io/), a home automation web interface, and is used to diplay the devices and sensor data. It subscribes to MQTT messages via [Mosquitto](#eclipse-mosquitto-gateway).

!!! info
    The default port is `8123`

??? example
    ![Home Assistant web interface](../assets/HomeAssistant.png)

## LoRaWAN module (LilyGo LoRa32 Dongle)

### Firmware

The [firmware](https://github.com/lorabridge2/bridge-lorawan-tx) for the ESP32-based LoRaWAN module (LilyGo LoRa32 Dongle) performs two tasks. First, it retrieves sensor/device data from the [bridge forwarder](#forwarder) over the [LoRaWAN Interface container](#bridge-lorawan-interface) via a serial USB interface and sends it to [ChirpStack](#chirpstack) over a LoRaWAN connection. Second, compressed automation configuration packages are transmitted to the module via LoRaWAN downlink (as part of MAC commands and ACK packets). If no device data is scheduled for uplink transmission, the firmware sends a [heartbeat packet](../additional/data_types.md#heartbeat) to enable periodic downlink communication.