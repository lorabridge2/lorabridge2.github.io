# Compression rules

LoRaBridge automation manager uses compression rules to minimize the LoRaWAN downlink communication required to transfer automations to a bridge unit. Data fields (in bytes) of each instruction accompanied with brief descriptions are given below.

## Add flow
| Action  | FlowID  |
|---|---|
|  9  | B  |

*Description*:
Creates empty LoRaBridge flow with a given FlowID. 

## Remove flow
| Action  | FlowID  |
|---|---|
|  11  | B  |

*Description*:
Removes a flow with given FlowID. 

## Flow complete
| Action  | FlowID  |
|---|---|
|  10  | B  |

*Description*:
Triggers Nodered flow generation 

## Upload flow
| Action  | FlowID  |
|---|---|
|  12  | B  |

*Description*:
Generated Nodered flow is uploaded to Nodered. (Only effective after "flow complete" command).

## Enable flow
| Action  | FlowID  |
|---|---|
|  6  | B  |

*Description*:
Associated Nodered flow is enabled. (Only effective after "flow complete" command).

## Disable flow
| Action  | FlowID  |
|---|---|
|  7  | B  |

*Description*:
Associated Nodered flow is disabled. (Only effective after "flow complete" command).

## Add node
| Action  | FlowID  | NodeID | NodeType |
|---|---|---|---|
|  1  | B  | B | B |

*Description*:
Add an automation node of NodeType to a flow. 

## Remove node
| Action  | FlowID  | NodeID |
|---|---|---|
|  0  | B  | B |

*Description*:
Removes an automation node of NodeType from a flow. 

## Add device
| Action  | FlowID  | NodeID | NodeType | LBDevice | LBAttribute |
|---|---|---|---|---|---|
|  2  | B  | B | B | B | B |

*Description*:
Add an automation (Zigbee2MQTT) device with LBDevice identifier and LBAttribute measurement attribute to a flow.

## Connect node
| Action  | FlowID  | OutputNode | Output | InputNode | Input |
|---|---|---|---|---|---|
|  4  | B  | B | B | B | B |

*Description*:
Connects Output (list index to output attributes in LoRaBridge automation JSON file) of a node with OutputNode ID to
an InputNode Input (list index to input attributes)

## Parameter update
| Action  | FlowID  | NodeID | ParameterID | NumBytes | Type | Content |
|---|---|---|---|---|---|---|
|  3  | B  | B | B | B | B | NumBytes |

*Description*:
Provides update to a node parameter (ParameterID is a list index to parameter list in LoRaBridge automation JSON file). NumBytes
defines the amount of bytes contained in the Content field. Type field defines the content data type (boolean,integer, float, string).

## Get Devices

| Action  | Unused  |
|---|---|
|  13  | B  |

*Description*:
Requests the bridge to send information about all known devices.