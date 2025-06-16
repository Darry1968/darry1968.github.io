---
categories: [Networking, Understanding SNMP]
tags: [snmp]     # TAG names should always be lowercase
author: Darshan
description: Simple Network Management Protocol (SNMP) is a widely used protocol for network management that provides a standardized framework for monitoring and managing network devices.
image:
    path: /assets/img/Banners/snmp.png
pin: false
---

# SNMP

Simple Network Mangement Protocol (SNMP) is a standard protocol used for monitoring, managing, and configuring devices on a network.

It enables network administrators to collect data, monitor performance, and identify issues on devices like routers, switches, servers, printers etc.

## About SNMP:

| Attribute | Description |
| --- | --- |
| OSI Layer | Application Layer (Layer 7) |
| Transport Protocol | UDP |
| Ports | 161/UDP for agents communication, 162/USP for traps |
| Versions | SNMPv1, SNMPv2c, SNMPv3 |
| Common Devices | Router switches, Server, printers, IoT Devices |

## Key components:

### **SNMP Manager**:

- Central system that issues requests to gather data and control network devices.
- It is a centralized system used to monitor the network. It is also known as **Network Management Station (NMS).**

Responsibilities:

- Sends requests to SNMP agents (e.g., `GET`, `SET`, `GETNEXT`).
- Receives responses or TRAPs from SNMP agents.
- Maintains a centralized view of the network status.
- Logs, alerts and performs automated tasks.

Examples:

- SolarWinds
- Manage Engine
- Zabbix

### **SNMP Agent**:

- SNMP agent is a software process running on networked devices.
- Software running on a managed devices that reponds to requests from SNMP manager.
- Collects and stores device specific information.

Responsibilities:

- Listens for SNMP requests on UDP port 161.
- Responds with requested info (e.g., Device uptime, CPU usage).
- Sends SNMP TRAPs or INFORMs to the manager to notify issues/events.
- Pull data from internal device metrics and make it accessible.

Examples:

- `snmpd` on Linux
- Bulit-in agents in Cisco/Juniper devices.

### **MIB**:

- Management Information Base (MIB) is a virtual database (a hierarchical structure) that defines what information an SNMP agent can provide.
- Defines the data objects that can be managed on a device.

Responsibilities:

- Standardize all the device info (uptime, interfaces, errors, etc.)
- Defined Object Identifiers (OIDs): Unique addresses to data points.
- Both manager and agent refer to the MIB to understand each other.

Examples:

- `sysDescr.0`: System description
- OID: .`1.3.6.1.2.1.1.1.0`

## Installation of SNMP agent on Linux:

### Installation:

1. First update your system.
    
    ```bash
    sudo apt update
    ```
    
2. Then install SNMP using apt:
    
    ```bash
    sudo apt install snmp snmpd
    ```
    
3. Check if it is installed or not:
    
    ```bash
    sudo systemctl status snmpd
    ```
    
    ![image.png](/assets/img/SNMP/SNMP%20status.png)
    

### Configuration:

Edit the config file:

```bash
sudo nano /etc/snmp/snmpd.conf
```

Common changes:

```bash
rocommunity public
agentAddress udp:161
sysLocation "Author's machine"
sysContact na5hv4r@na5hv4r.me
```

Reload the SNMP service:

```bash
sudo systemctl restart snmpd
```

### Test SNMP:

```bash
snmpwalk -v2c -c public localhost
```

Output:

![image.png](/assets/img/SNMP/snmp%20test%20cmd.png)

Displaying System Description:

```bash
snmpwalk -v2c -c public localhost 1.3.6.1.2.1.1.1.0
```

![image.png](/assets/img/SNMP/sysDecription.png)

## Common SNMP OIDs and Their Uses:

| OID | Name | MIB | Description / Use |
| --- | --- | --- | --- |
| `.1.3.6.1.2.1.1.1.0` | **sysDescr.0** | `SNMPv2-MIB` | Describes system: OS, version, hardware, etc. |
| `.1.3.6.1.2.1.1.3.0` | **sysUpTime.0** | `SNMPv2-MIB` | Time since the device last rebooted |
| `.1.3.6.1.2.1.1.5.0` | **sysName.0** | `SNMPv2-MIB` | Hostname of the device |
| `.1.3.6.1.2.1.1.4.0` | **sysContact.0** | `SNMPv2-MIB` | Contact person for this device |
| `.1.3.6.1.2.1.1.6.0` | **sysLocation.0** | `SNMPv2-MIB` | Physical location of the device |
| `.1.3.6.1.2.1.2.2.1.2.X` | **ifDescr.X** | `IF-MIB` | Description of interface X |
| `.1.3.6.1.2.1.2.2.1.8.X` | **ifOperStatus.X** | `IF-MIB` | Interface status: up/down/testing |
| `.1.3.6.1.2.1.2.2.1.10.X` | **ifInOctets.X** | `IF-MIB` | Bytes received on interface X |
| `.1.3.6.1.2.1.2.2.1.16.X` | **ifOutOctets.X** | `IF-MIB` | Bytes sent out on interface X |
| `.1.3.6.1.2.1.25.1.1.0` | **hrSystemUptime** | `HOST-RESOURCES-MIB` | System uptime in hundredths of a second |
| `.1.3.6.1.2.1.25.2.3.1.6.X` | **hrStorageUsed.X** | `HOST-RESOURCES-MIB` | Amount of used storage in block units |
| `.1.3.6.1.2.1.25.3.3.1.2.X` | **hrProcessorLoad.X** | `HOST-RESOURCES-MIB` | CPU load of processor X |
| `.1.3.6.1.4.1.2021.4.5.0` | **memTotalReal** | `UCD-SNMP-MIB` | Total physical RAM |
| `.1.3.6.1.4.1.2021.4.6.0` | **memAvailReal** | `UCD-SNMP-MIB` | Available physical RAM |
| `.1.3.6.1.4.1.2021.11.9.0` | **ssCpuIdle** | `UCD-SNMP-MIB` | % of CPU time spent idle |
| `.1.3.6.1.4.1.2021.10.1.3.1` | **laLoad.1** | `UCD-SNMP-MIB` | 1-minute load average |