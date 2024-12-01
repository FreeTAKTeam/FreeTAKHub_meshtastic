# Meshtastic to FreeTAKServer Integration
## Introduction
This repository facilitates the integration of Meshtastic devices with FreeTAKServer (FTS) using Mosquitto MQTT broker and Node-RED. This setup enables the translation of Meshtastic messages into Cursor-on-Target (CoT) format, allowing seamless communication between Meshtastic LoRa mesh networks and Tactical Awareness Kit (TAK) clients.
![image](https://github.com/user-attachments/assets/9e8b7b28-501a-4002-9a11-76f18319c5a2)

## Installation
### pre-requirements
we assume that you have already installed FTS using the ZeroTouch installer, so that you have FTS and Node Red already running. Also you have at least one meshtastic node that can connect to Wifi

### Install mosquitto
To install the Mosquitto MQTT broker on an Ubuntu server without using Docker, follow these steps:

1. **Update System Packages**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```
   This ensures all existing packages are up-to-date.

2. **Add the Mosquitto PPA Repository**:
   ```bash
   sudo add-apt-repository ppa:mosquitto-dev/mosquitto-ppa
   sudo apt update
   ```
   Adding the Mosquitto PPA provides access to the latest versions of Mosquitto. 

3. **Install Mosquitto and Client Utilities**:
   ```bash
   sudo apt install mosquitto mosquitto-clients -y
   ```
   This command installs both the Mosquitto broker and client utilities. 

4. **Enable and Start the Mosquitto Service**:
   ```bash
   sudo systemctl enable mosquitto
   sudo systemctl start mosquitto
   ```
   Enabling the service ensures Mosquitto starts on boot. 

5. **Verify the Installation**:
   ```bash
   mosquitto -v
   ```
   This command displays the installed Mosquitto version, confirming a successful installation. 

## Configuration
you will need to:
* Configure mosquitto
* configure a Meshtastic Node to pass messages to Moquitto
### Configure mosquitto
On Ubuntu , the primary configuration file for the Mosquitto MQTT broker is located at:

```
/etc/mosquitto/mosquitto.conf
```

This file contains the main settings for the Mosquitto service. Additionally, configuration snippets can be placed in the `/etc/mosquitto/conf.d/` directory. Files in this directory are included in the main configuration and can be used to modularize settings.


**Edit the Configuration File**: Use a text editor with elevated privileges to edit the file. For example:
   ```bash
   sudo nano /etc/mosquitto/conf.d/meshtasticIntegration.conf
   ```
   
   add the following
   ``` conf
allow_zero_length_clientid true
listener 1883 0.0.0.0
allow_anonymous true
```

**Apply Changes**: After saving your changes, restart the Mosquitto service to apply them:
   ```bash
   sudo systemctl restart mosquitto
   ```



  ## Usage
  
