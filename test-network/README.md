# Test Network for Proof of Collaborative Learning (PoCL)

This directory contains the Hyperledger Fabric blockchain network configuration and deployment scripts for the Proof of Collaborative Learning (PoCL) system. It provides the blockchain infrastructure that supports the federated learning consensus mechanism.

## Overview

The test network provides:
- **Blockchain Infrastructure**: Hyperledger Fabric network with peers and orderers
- **Channel Management**: Multiple channels for different types of transactions
- **Chaincode Deployment**: Automated deployment of all PoCL chaincodes
- **Network Lifecycle**: Scripts for starting, stopping, and managing the network

## Network Architecture

### Organizations
- **Org1**: Primary organization with peers and certificate authority
- **Org2**: Secondary organization for multi-org scenarios
- **OrdererOrg**: Ordering service organization

### Channels
- **demo**: Channel for demo transactions, model proposals, predictions, and votes
- **main**: Channel for main cryptocurrency transactions and rewards

### Deployed Chaincodes
1. **demoCC**: Demo transaction management (demo channel)
2. **modelCC**: Model proposal handling (demo channel)
3. **predCC**: Prediction management (demo channel)
4. **voteCC**: Voting system (demo channel)
5. **mainCC**: Main wallet operations (main channel)

## Quick Start for PoCL

### Complete System Startup
The `start.sh` script provides one-command deployment of the entire PoCL blockchain infrastructure:

```bash
./start.sh
```

This script automatically:
1. Starts the Hyperledger Fabric network
2. Creates demo and main channels
3. Deploys all 5 PoCL chaincodes
4. Initializes the blockchain for federated learning

### Manual Network Management
You can also use the standard `network.sh` script for individual operations:

```bash
# Start network infrastructure
./network.sh up

# Create channels
./network.sh createChannel -c demo
./network.sh createChannel -c main

# Deploy individual chaincodes
./network.sh deployCC -ccp ../demo-coin-transfer/demo-coin-transfer-chaincode \
  -ccn demoCC -c demo -ccl javascript

# Stop and clean network
./network.sh down
```

## Prerequisites

Before you can deploy the test network, you need to follow the instructions to [Install the Samples, Binaries and Docker Images](https://hyperledger-fabric.readthedocs.io/en/latest/install.html) in the Hyperledger Fabric documentation.

## Using the Peer commands

The `setOrgEnv.sh` script can be used to set up the environment variables for the organizations, this will help to be able to use the `peer` commands directly.

First, ensure that the peer binaries are on your path, and the Fabric Config path is set assuming that you're in the `test-network` directory.

```bash
 export PATH=$PATH:$(realpath ../bin)
 export FABRIC_CFG_PATH=$(realpath ../config)
```

You can then set up the environment variables for each organization. The `./setOrgEnv.sh` command is designed to be run as follows.

```bash
export $(./setOrgEnv.sh Org2 | xargs)
```

(Note bash v4 is required for the scripts.)

You will now be able to run the `peer` commands in the context of Org2. If a different command prompt, you can run the same command with Org1 instead.
The `setOrgEnv` script outputs a series of `<name>=<value>` strings. These can then be fed into the export command for your current shell.

## Chaincode-as-a-service

To learn more about how to use the improvements to the Chaincode-as-a-service please see this [tutorial](./test-network/../CHAINCODE_AS_A_SERVICE_TUTORIAL.md). It is expected that this will move to augment the tutorial in the [Hyperledger Fabric ReadTheDocs](https://hyperledger-fabric.readthedocs.io/en/release-2.4/cc_service.html)


## Podman

*Note - podman support should be considered experimental but the following has been reported to work with podman 4.1.1 on Mac. If you wish to use podman a LinuxVM is recommended.*

Fabric's `install-fabric.sh` script has been enhanced to support using `podman` to pull down images and tag them rather than docker. The images are the same, just pulled differently. Simply specify the 'podman' argument when running the `install-fabric.sh` script. 

Similarly, the `network.sh` script has been enhanced so that it can use `podman` and `podman-compose` instead of docker. Just set the environment variable `CONTAINER_CLI` to `podman` before running the `network.sh` script:

```bash
CONTAINER_CLI=podman ./network.sh up
````

As there is no Docker-Daemon when using podman, only the `./network.sh deployCCAAS` command will work. Following the Chaincode-as-a-service Tutorial above should work. 


