# spinal-organ-connector-astrid
Simple BOS-ASTRID api connector to collect occupancy data

https://exchange.se.com/devportal/?api=bdp-api-spec-v3-bundle&id=122362#section/documentation/building-data-platform-introduction

## Sequence Diagram — Spinalhub <-> BDP ( Building Data Platform )

```mermaid
sequenceDiagram
    participant S as SpinalHub
    participant C as Connector
    participant E as BDP API

    C->>S: Connect to SpinalHub
    C->>S: Create or load configuration file

    Note over C: Ensure BMS network
    C->>S: Check if BMS network exists
    alt Not found
        C->>S: Create BMS network
    else Exists
        C->>S: Use existing network
    end

    Note over C: generate or get BDP API token
    C->>E: get API token with client credentials
    Note over C,E: Scope selection
    C->>E: Fetch Building by ID/criteria
    E-->>C: Building metadata

    Note over C,E: Inventory discovery
    C->>E: List Occupancy related Devices for Building (Occupancy_Count_Sensor, Occupancy_Status)
    E-->>C: Devices[]

    Note over C,E: Measures discovery
    C->>E: List Measures for Building
    E-->>C: Measures[]
    C->>C: Keep measures linked to occupancy devices

    Note over C,S: Create or reuse Devices in SpinalHub
    loop For each filtered device
        C->>S: Find device by deviceId (BDP deviceId)
        alt Not found
            C->>S: Create Device node
            C->>S: Set attributes (BDP IDs, types)
        else Exists
            C->>S: Use existing Device node (idempotent)
        end
    end

    Note over C,S: Create or reuse Endpoints in SpinalHub
    loop For each kept measure
        C->>S: Find endpoint under device by BDP deviceId
        alt Not found
            C->>S: Create Endpoint as child of Device
        else Exists
            C->>S: Use existing Endpoint (no duplicate)
        end
    end

    Note over C: Transition to loop mode (polling)
    loop Every X seconds/minutes
        opt Token may be expired
            C->>E: Refresh/obtain API token
            E-->>C: New token
        end
        C->>E: Fetch latest Measures for Building
        E-->>C: Current values
        C->>C: Map values -> (Device, Endpoint)
        par Batch updates
            C->>S: Update Endpoint values in DB
        and
            C->>S: Update organ last sync timestamp
        end
    end

    Note over C: Idempotency & safety
    C->>C: Skip creation if already exists
    C->>C: Handle missing/unknown devices gracefully


```


## Getting Started

These instructions will guide you on how to install and make use of the spinal-organ-connector-astrid.

### Prerequisites

This module requires a `.env` file in the root directory. Use the `.env.example` file as a template to create your own `.env` file with the necessary configuration.


### Installation

Clone this repository in the directory of your choice. Navigate to the cloned directory and install the dependencies using the following command:
    
```bash
spinalcom-utils i
```

To build the module, run:

```bash
npm run build
```

### Usage

Start the module with:

```bash
npm run start
```

Or using [pm2](https://pm2.keymetrics.io/docs/usage/quick-start/)
```bash
pm2 start index.js --name organ-connector-astrid
```
```