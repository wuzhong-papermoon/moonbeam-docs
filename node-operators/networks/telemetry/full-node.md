---
title: Full Node
description: How to run telemetry for a full Parachain node for the Moonbeam Network
---

# Telemetry for a Full Node

## Introduction

With the release of Moonbase Alpha v5, you can spin up a full node that connects to the Moonbase Alpha TestNet. You can check those steps in [this tutorial](/node-operators/networks/full-node/).

In this guide we'll provide the necessary steps to enable the telemetry server for your Moonbase Alpha node.

## Telemetry Exporter

Moonbeam will run a telemetry server that collects Prometheus metrics from all the Moonbeam parachain nodes on the network. Running this will be a great help to us during our development phase.  

The metrics exporter can run either as a kubernetes sidecar, or as a local binary if you are running a VM. It will push data out to our servers, so you do not have to enable any incoming ports for this service.

We are using a service called [Gantree Node Watchdog](https://github.com/gantree-io/gantree-node-watchdog) to automatically upload telemetry.  Once you enable telemetry, you can also access a Prometheus/Grafana server from the [Gantree App] (https://app.gantree.io/).  There are detailed instructions on the github repo if you need more info, here is a quick start. 

For now we need to run two node watchdogs, one for the relay chain and one for the parachain.  This will be updated in a future release. 

For help, contact the Moonbeam Discord or the [Gantree Discord](https://discord.gg/4Ep2NKrz). 
 
### Gathering Information

1. Login to https://app.gantree.io and create an account.  Navigate to API keys and copy your API key. 
2. Request a PCK key in our [Discord channel](https://discord.gg/PfpUATX)
   
### Docker

We will run two instances of the gantree node watchdog; one for the relay chain and one for the parachain.  
Required Configuration Information
* GANTREE_NODE_WATCHDOG_API_KEY
* GANTREE_NODE_WATCHDOG_PROJECT_ID
* GANTREE_NODE_WATCHDOG_CLIENT_ID
* GANTREE_NODE_WATCHDOG_PCKRC
* GANTREE_NODE_WATCHDOG_METRICS_HOST

Instructions
1. Download or compile the docker image ([TODO]: no compiled image yet, but soon, need to update instructions below for "IMAGE-NAME", needs persistant storage for .gnw config file )
2. Run the relay Gantree node watchdog
  ```
  docker run -it --network="host" -e GANTREE_NODE_WATCHDOG_API_KEY="YOUR-API-KEY" -e GANTREE_NODE_WATCHDOG_PROJECT_ID="moonbase-alpha" -e GANTREE_NODE_WATCHDOG_CLIENT_ID="YOUR-SERVER-NAME-relay" -e GANTREE_NODE_WATCHDOG_PCKRC="YOUR-PCK-KEY" -e GANTREE_NODE_WATCHDOG_METRICS_HOST="http://172.0.0.1:9615"  --name gantree_watchdog_relay IMAGE-NAME
  ```
3. Run the parachain Gantree node watchdog
  ```
  docker run -it --network="host" -e GANTREE_NODE_WATCHDOG_API_KEY="YOUR-API-KEY" -e GANTREE_NODE_WATCHDOG_PROJECT_ID="moonbase-alpha" -e GANTREE_NODE_WATCHDOG_CLIENT_ID="YOUR-SERVER-NAME-parachain" -e GANTREE_NODE_WATCHDOG_PCKRC="YOUR-PCK-KEY" -e GANTREE_NODE_WATCHDOG_METRICS_HOST="http://172.0.0.1:9616" --name gantree_watchdog_relay IMAGE-NAME
  ```
4.  You should see waiting for provisioning in the logs.  Log into the https://app.gantree.io, select networks. Click the network and then `Provision Dashboard`.  This step may take a few minutes.  Once it completes, return to the network and you will see a `View Monitoring Dashboard` link to your custom Prometheus / Grafana dashboard. 
5.  Once things are workign well, you can update the commands to run in daemon mode.  

### Systemd
  
Instructions

1. Download the gantree node watchdog binary from the release page and extract to a folder.  e.g. `/usr/local/bin` 
2. Create 2 folders for the configuration files
  ```
  mkdir -p /var/lib/gantree/relay
  mkdir -p /var/lib/gantree/parachain
  ```
3. Generate your config files, place each in the folders created in the step above. 
  ```
  # Contents of /var/lib/gantree/relay/.gnw_config.json
  {
    "api_key": "YOUR-API-KEY",
    "project_id": "moonbase-alpha",
    "client_id": "YOUR-SERVER-NAME-relay",
    "pckrc": "YOUR-PCK-KEY",
    "metrics_host": "http://127.0.0.1:9615"
  }
  ```
  ```
  # Contents of /var/lib/gantree/parachain/.gnw_config.json
  {
    "api_key": "YOUR-API-KEY",
    "project_id": "moonbase-alpha",
    "client_id": "YOUR-SERVER-NAME-parachain",
    "pckrc": "YOUR-PCK-KEY",
    "metrics_host": "http://127.0.0.1:9616"
  }
  ```
4. Generate your systemd config
  ```
  # Contents of /etc/systemd/system/gantree-relay.service

  [Unit]
  Description=Gantree Node Watchdog Relay
  After=network.target

  [Service]
  WorkingDirectory=/var/lib/gantree/relay
  Type=simple
  Restart=always
  ExecStart=/usr/local/bin/gantree_node_watchdog

  [Install]
  WantedBy=multi-user.target
  ```
  ```
  # Contents of /etc/systemd/system/gantree-parachain.service

  [Unit]
  Description=Gantree Node Watchdog Parachain
  After=network.target

  [Service]
  WorkingDirectory=/var/lib/gantree/parachain
  Type=simple
  Restart=always
  ExecStart=/usr/local/bin/gantree_node_watchdog

  [Install]
  WantedBy=multi-user.target

  ```
5. Enable and start the systemd services, monitor logs for errors
  ```
  sudo systemctl enable gantree-relay
  sudo systemctl enable gantree-parachain
  sudo systemctl start gantree-relay && journalctl -f -u gantree-relay
  sudo systemctl start gantree-parachain && journalctl -f -u gantree-parachain
  ```
6. You should see the logs waiting for provisioning.  Log into the https://app.gantree.io, select networks. Click the network and then `Provision Dashboard`.  This step may take a few minutes.  Once it completes, return to the network and you will see a `View Monitoring Dashboard` link to your custom Prometheus / Grafana dashboard. 