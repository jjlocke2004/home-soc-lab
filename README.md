# Home SOC Lab

A production-relevant Security Operations Center lab built from scratch
using free tools. Covers Active Directory administration, SIEM deployment,
threat detection, and incident response — mapped to real SOC workflows.

## Lab Status
| Phase | Component | Status |
|-------|-----------|--------|
| 1 | Active Directory (Windows Server 2022) | ✅ Complete |
| 2 | SIEM — Wazuh 4.7.5 on Ubuntu 22.04 | ✅ Complete |
| 3 | Windows 10 Endpoint - Victim Machine | 🔄 In Progress |
| 4 | Network Monitoring — Zeek | 🔄 In Progress |
| 5 | Attack Simulations — Kali Linux | ⏳ Planned |
| 6 | IR Playbooks | ⏳ Planned |

## Lab Network Configuration

All VMs communicate over a VMware host-only network (VMnet2) isolated 
from the host machine's real network. This ensures attack traffic 
never reaches outside the lab environment.

| Setting | Value |
|---------|-------|
| Network | VMnet2 — Host Only |
| Subnet | 192.168.100.0/24 |
| DHCP | Disabled — all VMs use static IPs |

![VMnet2 Configuration](assets/vmnet2-config.png)

## Environment
| VM | OS | Role | IP |
|----|----|------|----|
| WinServer-DC | Windows Server 2022 | Domain Controller | 192.168.100.7 |
| Wazuh SIEM | Ubuntu 22.04 LTS | SIEM / Dashboard | 192.168.100.5 |
| Win10-Victim | Windows 10 | Endpoint / Domain Member | 192.168.100.10 |
| Kali Linux | Kali Rolling | Attacker | 192.168.100.20 |
| REMnux | REMnux 7 | Malware Analysis | 192.168.100.30 |

## Architecture
![Lab Architecture](assets/architecture-diagram.png)

## Key Skills Demonstrated
- Active Directory design — OUs, users, groups, GPOs, audit policies
- SIEM deployment and agent enrollment
- Windows event log collection and centralized monitoring
- Detection of AD events — user creation, deletion, group changes

## Tools Used
`Wazuh` `Windows Server 2025` `Active Directory` `Ubuntu 22.04`
`VMware Workstation` `PowerShell` `Sysmon`
