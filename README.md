# Open Wallet Stack

Open Wallet Stack is a collection of software forked from [bitcore](https://bitcore.io) and [Copay](https://copay.io), with the intention of creating a working group outside of the control of any one company.  OWS development has been funded by multiple organizations to date. If you are interested in contributing, please feel free to create Github issues, PRs, etc. We welcome any contribution.

## Technology

We are working to make OWS easy to install and operate. For this purpose, we provide Docker images for backend components, and aim to support Docker Compose and Kubernetes for deployments.

All Docker images are publicly available on [Docker Hub](https://hub.docker.com/r/owstack/)

## Goals and Roadmap

One of our original intentions with this project was to formally support BCH. Since then, Bitcore has added native support too. We have taken a slightly different approach in how things are logically separated. We applaud that effort, but will stay the course.

We have just recently added support for LTC. We were able to fork much of the work done by the Litecoin Project towards litecore.io (which seems to have been abandoned in 2017).

We are examining other currencies for inclusion in the future.

## Getting started

The first thing to do is decide which currencies you want to support. Currently the options are BTC, BCH, and LTC.

For each currency you want to support, for now you need to run a special version of the client that has been modified to allow extra indices for addresses, timestamps, and spent transactions.  These client images can be obtained from our docker hub page. (We are working towards removing the need for this special build.)

Note: In Docker or Kubernetes, each time a container is restarted, all data is lost. Therefore, for services that need persistent storage, be sure to create and mount a disk.

An example Kubernetes deployment on GKE for Bitcoin Core:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: btc-mainnet-bitcoin-core-1
  labels:
    app: btc-mainnet-bitcoin-core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: btc-mainnet-bitcoin-core
  template:
    metadata:
      labels:
        app: btc-mainnet-bitcoin-core
    spec:
      containers:
      - image: registry.hub.docker.com/owstack/bitcoin-core:0.14.1-ows
        name: bitcoin-0-14-1-ows
        command: ["bitcoind", "-datadir=/data", "-server=1", "-addressindex=1", "-spentindex=1", "-timestampindex=1", "-txindex=1", "-zmqpubhashtx=tcp://0.0.0.0:28332", "-zmqpubhashblock=tcp://0.0.0.0:28332", "-rpcallowip=0.0.0.0/0", "-rpcuser=someUserName", "-rpcpassword=somePassword", "-dbcache=512", "-rpcthreads=16", "-rest=1", "-printtoconsole"]
        ports:
        - containerPort: 8332
        - containerPort: 8333
        - containerPort: 28332
        volumeMounts:
        - mountPath: /data
          name: btc-mainnet-bitcoin-core-1
      securityContext:
        runAsUser: 1000
        fsGroup: 1000
      volumes:
      - name: btc-mainnet-bitcoin-core-1
        gcePersistentDisk:
          pdName: btc-mainnet-bitcoin-core-1
          fsType: ext4

```

After deploying this, the blockchain sync process will start, and it may be 24-48 hours before this service is ready to use.

When the sync process is complete, we can spin up one or more "btc-node" servers and configure the services we would like to use.

A "btc-node" can run one or all of the following services:

- A Block Explorer API
- A Block Explorer UI
- A Wallet Service

The "btc-node" image on Docker Hub already has all of the components included. There are a few reasons to split things up:

- A Block Explorer API is specific to the currency and network it is configured to use. If you plan to support livenet and testnet, you will need to run one for each.
- A block explorer UI supports many currencies and networks.
- A Wallet Service supports both livenet and testnet for a single currency

Based on these traits, and high-availablity needs, one may decide how best to deploy things for their scenario. For the simplest deployment, all services can be run together.

Example configs can be found in the [configs](https://github.com/owstack/owstack-website/blob/master/configs) directory.

In Kubernetes, these config files need to be converted into "[secrets](https://kubernetes.io/docs/concepts/configuration/secret/)", that can be mounted to appear as a local file. This allows the base image to be reused without rebuilding it for each deployment.

In Docker Compose, the config files can be mounted as a local file.
