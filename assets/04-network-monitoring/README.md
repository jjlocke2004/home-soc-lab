# Phase 4 - Network Monitoring: Zeek NSM

## Objective

Deployed Zeek as a dedicated network security monitoring system to capture and analyze traffic visible to the lab network interface. Zeek provides protocol-level visibility into network connections, DNS queries, HTTP activity, TLS sessions, Kerberos authentication, and other communications.

This network telemetry complements the endpoint and authentication logs collected by Wazuh, providing visibility into both host activity and network behavior.

> **Network visibility note:** Zeek can only analyze traffic delivered to its monitored interface. Connecting Zeek to the same VMware virtual network does not necessarily provide visibility into every unicast connection exchanged between other virtual machines. Full network visibility would require traffic mirroring, bridging, or routing traffic through the Zeek VM.

## Components

- **VM:** Zeek-NSM
- **Operating System:** Ubuntu Server 22.04 LTS
- **Software:** Zeek Network Security Monitor
- **Zeek Version:** 8.0.8
- **Static IP:** 192.168.100.6
- **Lab Network:** 192.168.100.0/24
- **Monitored Interface:** ens33
- **Zeek Installation Path:** `/opt/zeek`
- **Wazuh Agent:** 4.7.5
- **Wazuh Manager:** 192.168.100.5

## Steps Taken

### 1. Built the Ubuntu Server VM

Created a dedicated Ubuntu Server 22.04 virtual machine in VMware with 2 GB of RAM and a 20 GB virtual disk.

The VM was initially configured with two network adapters:

- A VMnet2 adapter for communication with the isolated SOC lab
- A temporary NAT adapter for internet access during package installation

The NAT adapter was only used to install Zeek, the Wazuh agent, and required dependencies. It was removed after installation to preserve the isolation of the SOC lab.

### 2. Assigned a Static IP Address

Configured the VMnet2 interface with the static IP address `192.168.100.6/24` using Netplan.

The interface configuration provided the Zeek VM with a consistent address on the `192.168.100.0/24` SOC lab network.

The temporary secondary interface was configured through DHCP while internet access was required for package installation.

### 3. Added the Zeek Repository

Added the official openSUSE Build Service Zeek repository for Ubuntu 22.04.

The repository was added to:

```text
/etc/apt/sources.list.d/security:zeek.list
```

After adding the repository and its signing key, the APT package index was updated.

### 4. Installed and Verified Zeek

Installed Zeek through APT and verified that the installation completed successfully.

Zeek was installed under:

```text
/opt/zeek
```

The installed version was verified with:

```bash
/opt/zeek/bin/zeek --version
```

The installed package information was also verified using:

```bash
dpkg -l | grep -i zeek
```

### 5. Defined the Local SOC Lab Network

Edited the Zeek network configuration file:

```text
/opt/zeek/etc/networks.cfg
```

Added the SOC lab subnet as a local network:

```text
192.168.100.0/24    VMnet2 Lab Network
```

This allows Zeek scripts to identify systems on the `192.168.100.0/24` subnet as internal lab hosts.

### 6. Configured the Zeek Monitoring Interface

Edited the Zeek node configuration file:

```text
/opt/zeek/etc/node.cfg
```

Configured Zeek as a standalone monitoring node and selected `ens33` as the monitored interface:

```ini
[zeek]
type=standalone
host=localhost
interface=ens33
```

The `ens33` interface is connected to VMware VMnet2, which contains the SOC lab systems.

### 7. Configured Zeek to Generate JSON Logs

Edited the local Zeek policy file:

```text
/opt/zeek/share/zeek/site/local.zeek
```

Loaded Zeek's JSON logging policy:

```zeek
@load policy/tuning/json-logs
```

JSON output was enabled so the Wazuh agent could process the Zeek logs using the `json` log format.

### 8. Deployed Zeek

Applied the configuration and started Zeek using:

```bash
sudo /opt/zeek/bin/zeekctl deploy
```

Verified that the standalone Zeek process was running:

```bash
sudo /opt/zeek/bin/zeekctl status
```

The status output confirmed that the Zeek process was active.

### 9. Verified Zeek Log Generation

Confirmed that Zeek was generating active network logs under:

```text
/opt/zeek/logs/current/
```

The generated logs included:

- `conn.log` for connection metadata
- `dns.log` for DNS requests and responses
- `http.log` for unencrypted HTTP traffic
- `kerberos.log` for Kerberos authentication activity
- `ssl.log` for TLS and certificate information
- `notice.log` for notable network events

Generated test traffic using ping and DNS requests, then reviewed `conn.log` to confirm that Zeek was processing network activity.

### 10. Installed the Wazuh Agent

Installed Wazuh Agent version 4.7.5 on the Zeek VM.

Configured the agent to communicate with the Wazuh manager at:

```text
192.168.100.5
```

The agent communicates with the manager over TCP port 1514.

### 11. Registered Zeek with the Wazuh Manager

Opened the Wazuh agent manager on the Wazuh server:

```bash
sudo /var/ossec/bin/manage_agents
```

Created a new agent with the following information:

- **Agent name:** zeek-nsm
- **IP address:** 192.168.100.6
- **Agent ID:** 003

Extracted the enrollment key from the Wazuh manager and imported it on the Zeek VM.

> The enrollment key must be redacted before the screenshot is published in a public repository.

### 12. Configured Zeek Log Forwarding

Edited the Wazuh agent configuration file:

```text
/var/ossec/etc/ossec.conf
```

Added the active Zeek logs as monitored local files inside the final `<ossec_config>` section:

```xml
<localfile>
  <log_format>json</log_format>
  <location>/opt/zeek/logs/current/conn.log</location>
</localfile>

<localfile>
  <log_format>json</log_format>
  <location>/opt/zeek/logs/current/dns.log</location>
</localfile>

<localfile>
  <log_format>json</log_format>
  <location>/opt/zeek/logs/current/http.log</location>
</localfile>

<localfile>
  <log_format>json</log_format>
  <location>/opt/zeek/logs/current/kerberos.log</location>
</localfile>

<localfile>
  <log_format>json</log_format>
  <location>/opt/zeek/logs/current/ssl.log</location>
</localfile>

<localfile>
  <log_format>json</log_format>
  <location>/opt/zeek/logs/current/notice.log</location>
</localfile>
```

Restarted the Wazuh agent after modifying the configuration:

```bash
sudo systemctl restart wazuh-agent
```

### 13. Verified the Wazuh Agent Service

Checked the local Wazuh agent service using:

```bash
sudo systemctl status wazuh-agent --no-pager
```

Confirmed that the service was active and running without configuration errors.

### 14. Confirmed Three Active Wazuh Agents

Verified that the Wazuh dashboard displayed three active agents:

- Windows Server domain controller
- Win10-Victim
- zeek-nsm

All three systems were running Wazuh Agent version 4.7.5, and the dashboard reported 100% agent coverage.

### 15. Generated Test Network Traffic

Generated controlled traffic between lab systems using ping and DNS queries.

Example traffic generated from the Windows 10 victim endpoint:

```powershell
ping 192.168.100.7
nslookup soclab.local 192.168.100.7
```

Reviewed the active Zeek connection log to determine whether the traffic was visible to the monitored interface:

```bash
sudo grep -E '192\.168\.100\.(7|10)' \
/opt/zeek/logs/current/conn.log | tail -n 10
```

This test helped verify the extent of Zeek's visibility within the VMware virtual network.
## Screenshots

### Static IP Configuration

Configured the `ens33` interface with the static IP address `192.168.100.6/24` on the VMnet2 lab network. The secondary `ens37` interface temporarily used DHCP to provide internet access during package installation.

![Zeek Static IP Configuration](<screenshots/zeek-static-ip-netplan.png>)

### Zeek Repository Configuration

Added the openSUSE Build Service Zeek repository for Ubuntu 22.04 to the system's APT sources.

![Adding the Zeek Repository](<screenshots/adding-zeek-repo.png>)

### Local Network Definition

Configured `192.168.100.0/24` as the local VMnet2 SOC lab network in Zeek's `networks.cfg` configuration file.

![Zeek Local Network Configuration](<screenshots/zeek-network-cfg.png>)

### Zeek Monitoring Interface Configuration

Configured Zeek as a standalone monitoring node using the `ens33` interface connected to the VMnet2 SOC lab network.

![Zeek Node Configuration](<screenshots/zeek-node-config.png>)

### Zeek Deployed and Running

Deployed the Zeek configuration using `zeekctl deploy` and verified that the standalone Zeek process was running successfully.

![Zeek Deployment Status](<screenshots/zeek-deploy-status.png>)

### Adding Zeek to the Wazuh Agent Manager

Created a new Wazuh agent named `zeek-nsm` and assigned it the IP address `192.168.100.6`. The Wazuh manager assigned the agent ID `003`.

![Adding Zeek to the Wazuh Agent Manager](<screenshots/adding-agent-in-agent-manager.png>)

### Configuring the Wazuh Manager Connection

Configured the Wazuh agent on the Zeek VM to communicate with the Wazuh manager at `192.168.100.5` over TCP port `1514`.

![Zeek Wazuh Manager Configuration](<screenshots/zeek-ossec-config.png>)

### All Three Agents Active in Wazuh

Verified that the domain controller, Windows 10 victim endpoint, and Zeek network monitor were all connected and reporting as active agents in the Wazuh dashboard.

The dashboard showed:

- Agent `001` — Windows Server domain controller
- Agent `002` — `Win10-Victim`
- Agent `003` — `zeek-nsm`
- Three active agents
- Zero disconnected agents
- 100% agent coverage

![All Three Wazuh Agents Active](<screenshots/all-three-agents-active-wazuh.png>)

## Outcome

Zeek was successfully deployed as the network security monitoring component of the SOC lab.

The Zeek service is running on a dedicated Ubuntu Server VM and is generating protocol-level network logs from traffic visible to the `ens33` VMnet2 interface. Zeek was configured to output its active logs in JSON format for collection by Wazuh.

The Zeek VM was registered with the Wazuh manager as `zeek-nsm`. The Wazuh agent was configured to monitor and forward Zeek's connection, DNS, HTTP, Kerberos, TLS, and notice logs.

The Wazuh dashboard now shows three active monitored systems:

- Windows Server domain controller
- Windows 10 victim endpoint
- Zeek network security monitor

The lab now combines:

- Domain authentication and Windows security events from the domain controller
- Endpoint and Sysmon telemetry from Win10-Victim
- Network connection and protocol telemetry from Zeek

This provides a foundation for correlating host-based and network-based evidence during future attack simulations.

## Lessons Learned

- **Selecting the correct interface is critical:** Zeek must monitor the interface connected to the SOC lab network. Monitoring the temporary NAT adapter would primarily capture package downloads and internet traffic instead of the intended lab activity.

- **The local network must be defined separately:** Adding `192.168.100.0/24` to `networks.cfg` allows Zeek to identify the SOC lab systems as internal hosts.

- **Zeek logs required additional Wazuh configuration:** Installing and connecting the Wazuh agent did not automatically forward Zeek logs. Each log file had to be added as a `<localfile>` source in `ossec.conf`.

- **A connected agent does not prove log collection:** The Wazuh dashboard showing `zeek-nsm` as active only confirms communication between the agent and manager. Log forwarding must be verified by searching for Zeek events in the Wazuh dashboard.

- **JSON formatting simplifies ingestion:** Zeek's JSON logging policy provides structured fields that can be processed more effectively by Wazuh than the default tab-separated Zeek log format.

- **Zeek visibility depends on the network design:** Connecting Zeek to the same VMware network does not guarantee visibility into all unicast traffic exchanged between the other virtual machines. Full visibility requires traffic mirroring, bridging, or routing traffic through the Zeek system.

- **Not every Zeek log is created immediately:** Files such as `http.log`, `kerberos.log`, and `ssl.log` may not appear until matching protocol traffic is generated.

- **Zeek uses `ssl.log` for TLS traffic:** HTTPS and other TLS-encrypted session metadata is recorded in `ssl.log`, not a file named `https.log`.

- **Zeek logs rotate automatically:** Active logs remain under `/opt/zeek/logs/current/`, while older logs are moved into dated directories.

- **Temporary network adapters should be removed:** The secondary internet-facing adapter was only required during installation. Removing it afterward preserved the isolation of the SOC lab.

- **Enrollment keys should not be published:** The Wazuh enrollment key displayed during registration must be cropped or redacted before the screenshot is committed to a public repository.
