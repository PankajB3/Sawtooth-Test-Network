# Creating a Sawtooth Test Network

In a Sawtooth network, each host system (physical computer, virtual machine, set of Docker containers, or Kubernetes pod) is a Sawtooth node that runs one validator, an optional REST API, a consensus engine, and a set of transaction processors.

The first node creates the genesis block, which specifies the initial on-chain settings for the network. The other nodes access those settings when they join the network.

A Sawtooth network has the following requirements:

* Each node must run the same consensus engine. The procedures in this guide show you how to configure PBFT or PoET consensus. For more information, see About Dynamic Consensus.
* Each node must run the same set of transaction processors as all other nodes in the network.
* Each node must advertise a routable address. The Docker and Kubernetes platforms provide preconfigured settings. For the Ubuntu platform, you must configure network settings before starting the validator.
* The authorization type must be the same on all nodes: either trust (default) or challenge. This application development environment uses trust authorization.
* The first node on the network must create the genesis block, which includes the on-chain configuration settings that will be available to the other nodes when they join the network.

Each node in this Sawtooth network runs a validator, a REST API, a consensus engine, and the following transaction processors:

* Settings (settings-tp): Handles Sawtooth's on-chain configuration settings. The Settings transaction processor (or an equivalent) is required for all Sawtooth networks.
* IntegerKey (intkey-tp-python): Demonstrates basic Sawtooth functionality. The associated intkey client includes shell commands to perform integer-based transactions.
* XO (sawtooth-xo-tp-python): Simple application for playing a game of tic-tac-toe on the blockchain. The associated xo client provides shell commands to define players and play a game. 
* (PoET only) PoET Validator Registry (poet-validator-registry-tp): Configures PoET consensus and handles a network with multiple nodes.

## Step 1: Download the Docker Compose File

Find it <a href="https://github.com/hyperledger/sawtooth-core/blob/main/docker/compose/sawtooth-default-poet.yaml">here</a> for PoET

Find it <a href="https://github.com/hyperledger/sawtooth-core/blob/main/docker/compose/sawtooth-default-pbft.yaml"> here </a>for PBFT

## Step 2 : Start the docker environment

Start the environment using following command, in directory where our yaml file resides.
```
docker-compose -f poet.yaml up
```

This Compose file creates five Sawtooth nodes named validator-# (numbered from 0 to 4). 

**Note** the container names for the Sawtooth components on each node.

## Step 3: Check the REST API Process
Use these commands on one or more nodes to confirm that the REST API is running.

Connect to the REST API container on a node, such as sawtooth-poet-rest-api-default-0.
```
docker exec -it sawtooth-rest-api-default-0 bash
```
Use the following command to verify that this component is running.
```
ps --pid 1 fw
```

## Step 4: Confirm Network Functionality
Connect to the shell container.

``` 
docker exec -it sawtooth-shell-default bash
```
To check whether peering has occurred on the network, submit a peers query to the REST API on the first node. This command specifies the container name and port for the first node's REST API.
```
curl http://sawtooth-rest-api-default-0:8008/peers
```

In the current implementation, the peer data for rest-api-default-0 is **[]**.

Whereas for rest-api-default-1, it shows its peers.

Submit a transaction to the REST API on the first node. This example sets a key named MyKey to the value 999.
```
intkey set --url http://sawtooth-rest-api-default-0:8008 MyKey 999
```
The output should resemble this example:
```
{
  "link": "http://sawtooth-rest-api-default-0:8008/batch_statuses?id=dacefc7c9fe2c8510803f8340...
}
```

Watch for this transaction to appear on a different node. The following command requests the value of MyKey from the REST API on the second node.

```
intkey show --url http://sawtooth-rest-api-default-1:8008 MyKey
```
