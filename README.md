# Meshtastic to FreeTAKServer Integration
## Introduction
This repository facilitates the integration of Meshtastic devices with FreeTAKServer (FTS) using Mosquitto MQTT broker and Node-RED. This setup enables the translation of Meshtastic messages into Cursor-on-Target (CoT) format, allowing seamless communication between Meshtastic LoRa mesh networks and Tactical Awareness Kit (TAK) clients.

## Architecture
This integration employs MQTT and Node-RED to bridge Meshtastic mesh networks with FreeTAKServer, an open-source TAK server implementation. This setup enables the transmission of position and chat messages from  Meshtastic devices to TAK clients like ATAK, iTAK, and WinTAK
![image](https://github.com/user-attachments/assets/9e8b7b28-501a-4002-9a11-76f18319c5a2)

## how is different from the Meshtastic Plugin for ATAK?
this integration is suoted  for scenarios necessitating centralized coordination, such as emergency response, search and rescue operations, and large-scale event management. The server-centric approach allows for comprehensive data aggregation and dissemination among multiple clients and systems.

## Installation
By following these steps, you can successfully activate and configure Node rd, MQTT and Meshtastic , enabling seamless communication between your mesh network and FreeTAKServer. 
### pre-requirements
We assume that you have already installed FTS using the ZeroTouch installer, so that you have FTS and Node-RED already running. Also you have at least one meshtastic node that can connect to Wifi
In case you do not have a zero touch installation:
- Ensure that the last version of Node-RED is installed on your system. You can install it following the [Node-RED documentation](https://nodered.org/docs/getting-started/).
- Have FreeTAKServer installed and running. Configure its API and TCP/UDP endpoints.

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
* configure Node red
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
protocol mqtt

log_type error
log_type warning
log_type notice
log_type information
log_type subscribe
```
> **Warning**  
> this is the minimum configuration, for production you need to secure your server.

**Apply Changes**: After saving your changes, restart the Mosquitto service to apply them:
   ```bash
   sudo systemctl restart mosquitto
   ```
## configure a Meshtastic Node to pass messages to Moquitto
To enable MQTT functionality in the Meshtastic Android application, follow these steps:

**Access Radio Configuration:**
   - Open the Meshtastic app on your Android device.
   - Tap the three-dot menu (⋮) in the top-right corner.
   - Select "Radio Configuration."

**Enable the MQTT Module:**
   - Scroll to the "MQTT" section.
   - Toggle the "MQTT Enabled" switch to the "On" position.
   - Tap "Send" to apply the changes.
   - Configure MQTT Settings:
   - **Server Address:** Enter your MQTT broker's address. If left blank, the default public server is used.
   - **JSON Enabled:** Toggle based on your integration needs.
   - **Root Topic:** Define ````msh/US```a root topic to organize MQTT messages
   - **Client Proxy Enabled:** If your LORA radio  lacks WIFI  access, enabling this allows it to use your phone's connection.

**Enable Channel Uplink and Downlink:**
   - In "Radio Configuration," navigate to "LoRa Config."
   - activate "OK to MQTT"
**Enable Channel Uplink and Downlink:**
   - In "Radio Configuration," navigate to "Channels."
   - Select the 'Longfast'.
   - Enable "Uplink" to allow the device to publish mesh packets to MQTT and to FTSS.
     **Note:** Future feature
   - Enable "Downlink" to permit the device to subscribe to MQTT and forward packets to the mesh.
   - Tap "Save" to confirm settings.

**Configure Network Settings:**
   - If your device connects via Wi-Fi:
     - Ensure "Wi-Fi Enabled" is active.
     - Enter the SSID and password of your Wi-Fi network.
     - Tap "Save" to apply.
   - If using your phone's internet connection:
     - Enable "Client Proxy to client enabled" to allow the device to utilize the phone's network for MQTT communication.

**Note:** Proper configuration of network settings and channel permissions is crucial for effective MQTT integration. For detailed information, refer to the [Meshtastic MQTT Module Configuration](https://meshtastic.org/docs/configuration/module/mqtt/).

### configure Node red
 **Import the Flow**:
   - Open your Node-RED editor (usually accessible via `http://localhost:1880`).
   - Click on the menu in the top-right corner and select `Import`.
   - Copy and paste the contents of the provided JSON file or upload the file directly.
   - Click `Import`.
    
**Install Required Node Modules**:
![image](https://github.com/user-attachments/assets/7274d175-3113-43ed-8548-e8997ec33bbd)

   - Install the required Node-RED nodes used in the flow:
     ```bash
     npm install node-red-contrib-mqtt node-red-contrib-influxdb node-red-contrib-config
     ```
   - Add any additional nodes listed in the flow that may not already be installed.

 **Set MQTT Broker Configuration**:
   - Locate the MQTT nodes (e.g., "Longfast topic JSON", "longfast topic Protobuff").
   - Double-click on each MQTT node and configure the broker settings, including:
     - Broker IP/hostname.
     - Port (default is 1883 for non-SSL connections).

 **Verify Node-RED Flow**:
   - Click on the `Deploy` button in the top-right corner of the Node-RED editor.
   - Check the debug nodes to monitor the flow's operations and identify any errors.
  

 **OPTIONAL / Future usage Configure Global Variables**:
   - Locate the `FTH Global Config` node in the flow.
   - Set the following global variables according to your FreeTAKServer setup:
     - `FTH_FTS_URL`: IP address of your FreeTAKServer instance.
     - `FTH_FTS_API_Port`: Port number for the FreeTAKServer API.
     - `FTH_FTS_TCP_Port`: TCP port for streaming.
     - `FTH_FTS_API_Auth`: Authentication token for the FreeTAKServer API.

**Support and Troubleshooting**:
- Use the Node-RED debug nodes (active in the flow) to trace issues.
- Consult the FreeTAKServer and Meshtastic documentation for specific configurations.
- Verify that all required external services (MQTT broker, FreeTAKServer) are operational and correctly networked.

  ## Usage
   - connect a TAK app to FTS
   - ensure that you have a GPS position
   - Send a test message via meshatastic to the topics configured.
   - in the TAK app you will see the position and the  chat message

# Configure meshtastic device
please follow the official page for updated instructions
## Configure Wi-Fi:
![image](https://github.com/user-attachments/assets/d49cfd13-68fe-4cc3-a4a8-e97dbd99722f)

- Go to radio config -> Network > Wi-Fi config.
- Enter the SSID and Password for your network.
- Toggle Enable Wi-Fi to ON.
 -click on Save and Reboot

## configure  MQTT
   - Go to the "module config" or the web UI.
   - Select "MQTT" 
   - Enable MQTT: Toggle the "Enabled" switch to ON.
   - Server Address: Enter your MQTT broker’s address (e.g., mqtt.example.com or 192.168.1.100).
   - no Credentials
   - No  TLS
   - Root topic: msh/US <-- if you change this you will need to change also the nodereed flow

## OK to LORA
   - under Radio config - > loRa, enable the togle "OK to MQTT"
