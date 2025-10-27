# spinal-organ-connector-astrid
Simple BOS-ASTRID api connector to collect occupancy data

https://exchange.se.com/devportal/?api=bdp-api-spec-v3-bundle&id=122362#section/documentation/building-data-platform-introduction

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
pm2 start index.js --name organ-connector-idposition
```
```