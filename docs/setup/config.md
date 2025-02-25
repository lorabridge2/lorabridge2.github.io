# Configuration

## Environment variables

### Bridge environment variables

```yaml
# username for the mosquitto mqtt server (currently unused)
MQTT_USERNAME
# password for the mosquitto mqtt server (currently unused)
MQTT_PASSWORD
# hostname of the mosquitto mqtt server (docker container name, resolved via docker dns)
MQTT_HOST
# port under which mosquitto mqtt server is reachable
MQTT_PORT
# MQTT main topic (almost always zigbee2mqtt)
MQTT_TOPIC
# hostname of the redis server (docker container name, resolved via docker dns)
REDIS_HOST
# port under which redis server is reachable
REDIS_PORT
# redis db index to use (0 - 15)
REDIS_DB
# list name to use for pushing and pulling data
REDIS_LIST
# lorawan identifier for the bridge device (needs to be unique among bridges), consists of 16 HEX values [0-9A-F]
LORA_DEV_EUI
# key to authenticate to the gateway (needs to be same on bridge and gateway - without "\x"!) consists of 32 HEX values [0-9A-F]
LORA_DEV_KEY
# lorawan identifier for the gateway, consists of 16 HEX values [0-9A-F]
LORA_JOIN_EUI
# contains a credentials string used for .htpasswd files (<username>:<password hash representation>) (you can use the command below)
# valuee needs to be enclosed with quotes e.g. 'user:$2$05$mypwhash'
BASIC_AUTH
# hostname of the nodered server (docker container name, resolved via docker dns)
NODERED_HOST
# port under which nodered server is reachable
NODERED_PORT
# serial device file, most likely /dev/ttyACM0
SERIAL_PORT
# group id of docker group on bridge
GID_DOCKER
```

!!! tip
    Create htpasswd hashes with something like this:

    ```bash
    htpasswd -nbB <user> <passwd>
    # result should be something like admin:$2y$05$7HuXBo2Z6qj1kPSimTNsaebTkKdqfHeQtwMvvWhc7ITZQ.PKIKpda
    ```

    Get docker gid with:

    ```bash
    getent group docker | cut -d : -f 3
    ```

### Gateway environment variables

```yaml
# username for the mosquitto mqtt server (currently unused)
MQTT_USERNAME
# password for the mosquitto mqtt server (currently unused)
MQTT_PASSWORD
# hostname of the mosquitto mqtt server (docker container name, resolved via docker dns)
MQTT_HOST
# port under which mosquitto mqtt server is reachable
MQTT_PORT
# hostname of the redis server (docker container name, resolved via docker dns)
REDIS_HOST
# port under which redis server is reachable
REDIS_PORT
# redis db index to use (0 - 15)
REDIS_DB
# mqtt topic used for the device manager
DEV_MAN_TOPIC
# mqtt topic used for publishing the discovered devices
DEV_DISCOVERY_TOPIC
# mqtt topic used for publishing the device state
DEV_STATE_TOPIC
# lorawan identifier for the first (automatically created) bridge device (needs to be unique among bridges), consists of "\x" and 16 HEX values [0-9A-F] (e.g. \x0000000000000000)
CHIRPSTACK_DEV_EUI
# key to authenticate to the gateway (needs to be same on bridge and gateway) consists of "\x" and 32 HEX values [0-9A-F] (e.g. \x00000000000000000000000000000000)
CHIRPSTACK_DEV_KEY
# chripstack api secret for generating tokens
CHIRPSTACK_API_SECRET
# admin user name for chirpstack
CHIRPSTACK_USER
# admin password for chirpstack
CHIRPSTACK_PASSWORD
# password hash containing (needs to be a PHC string)
# $<id>[$v=<version>][$<param>=<value>(,<param>=<value>)*][$<salt>[$<hash>]]
# e.g. '$pbkdf2-sha512$i=10000,l=32$DMJEl+7IRJrDcd3/VamdSw$TukY3fi2kgtbV+8Oq7eCVuvPAaL7gUqL7IurFCaBMIo'
CHIRPSTACK_HASH
# user for couchdb server
COUCHDB_USER
# password for couchdb server
COUCHDB_PASSWORD
# port under which couchdb server is reachable
COUCHDB_PORT
# name of the couchdb database
COUCHDB_DB
# group id of spi group on gateway
GID_SPI
# group id of gpio group on gateway
GID_GPIO
```

!!! tip 
    You can create the CHIRPSTACK_API_SECRET e.g. with:

    ```bash
    openssl rand -base64 32
    ```
    
    You can create the CHIRPSTACK_HASH e.g. with:
    
    ```bash
    CHIRPSTACK_PASS=mypass python3 -c 'import os;import hashlib;import base64;iterations=210000;dklen=64;salt=os.urandom(32);print(f"$pbkdf2-sha512$i={iterations},l={dklen}${base64.b64encode(salt).decode().replace("=","")}${base64.b64encode(hashlib.pbkdf2_hmac("sha512",os.environ.get("CHIRPSTACK_PASS").encode(), salt,iterations=iterations,dklen=dklen)).decode().replace("=","")}")'
    ```

    Get spi gid with:

    ```bash
    getent group spi | cut -d : -f 3
    ```

    Get gpio gid with:

    ```bash
    getent group gpio | cut -d : -f 3
    ```