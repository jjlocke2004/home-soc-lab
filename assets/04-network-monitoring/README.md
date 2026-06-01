# Phase 4 - Network Monitoring: Zeek NSM

## Objective 
Deployed Zeek as a dedicated network security monitor to capture and log all traffic on the lab network. Zeek provides protocol-level visibility into DNS, Kerberos, HTTP, and connection data that complements the endpoint logs collected by Wazuh -gviing a full picture of both host and network activity.

## Components
- **VM:** Zeek-NSM
- **OS:** Ubuntu 22.04 LTS
- **Software:** Zeek NSM LTS
- **IP:** 192.168.100.6
- **Wazuh Agent:** 4.7.5

## Steps Taken

**1. Built the Ubuntu 22.04 VM**

Created a new VM in VMware with 2GB ram 20 GB disk, and two network adapters - VMnet 2 for lab communitication and a bridged adapter for internet access during package installation.

**2. Assigned static IP**

Configured a static IP of 192.168.100.6 using netplan. I set up a secondary network adapter using NAT, just to download the needed packages. I then removed the NAT adpater.

**3. Added Zeek repository and installed**

Added the official openSUSE Zeek repository, imported the GPG key, and installed Zeek via apt. Added Zeek to PATH for command line access.

**4. Configured Zeek to monitor the lab interface**

Edited node.cfg to set the monitoring interface to ens33, pointing Zeek at the VMnet2 network where all the lab traffic flows.

**5. Deployed Zeek and verified log generation**

Ran zeekctl deploy to start Zeek and confirmed all log files were generatingin /opt/zeek/logs/current/ including conn.log, dns.log, http.log. kerberos.log, and ssl.log.

**6. Intalled Wazuh aggent and configured log fowarding**

Installed the Wazuh 4.7.5 agent, pointed it at the manager at 192.168.100.5 and added conn.log, dns.log, kerberos.log, and https log as monitored log sources in ossec.conf.

**7. Confirmed 3 active agents in Wazuh**

Verified Zeek-NSM appeared as the third live agent in the Wazuh dashboard alongside WinServer-DC amd Win10-Victim, all running version 4.7.5 with 100% agent coverage.

## Screenshots

### Static IP Configuration

### Zeek Installation

### Node Configuration

### Zeek Deployed and Running

### Log Files Generating

### Wazuh Agent Installed on Zeek VM

### All 3 Agents Active in Wazuh

## Outcome

## Lessons Learned
