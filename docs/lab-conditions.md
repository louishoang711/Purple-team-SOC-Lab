# Lab Conditions

Environment facts that every measurement in this project depends on. Update this file whenever something below changes. All results in this repo are lab results, not production data.

Last updated: 2026-11-09 (end of Phase 1 telemetry build).

## Machines

| Tag | Role | OS | RAM | Address | Notes |
|---|---|---|---|---|---|
| `HOST` | VMware Workstation host, git, browser | Windows 11 (i5-1135G7, 4C/8T, 16 GB) | n/a | n/a | Runs nothing from the attack tooling |
| `WAZUH01` | SIEM (Wazuh manager, indexer, dashboard) and network sensor (Suricata) | Ubuntu Server 22.04.5 | 4-6 GB | 192.168.149.131 (DHCP) | Wazuh 4.14.10, Suricata 8.0.8, ET Open rules |
| `DC01` | Domain controller (AD DS, DNS) | Windows Server 2022 Standard Evaluation (Desktop Experience) | 4 GB | 192.168.149.10 (static) | Domain `soclab.local` |
| `WIN10-01` | Victim workstation, domain-joined | Windows 10 Pro 22H2 | 2 GB | DHCP (192.168.149.129 at last check) | Sysmon 15.22, Wazuh agent 4.14.10 |
| `KALI01` | Attacker | Kali Linux | 2 GB | DHCP | Powered off except during attacks |

## Network

- All VMs share the VMware NAT network `192.168.149.0/24` (gateway `192.168.149.2`).
- No isolated host-only network yet. Planned isolation for attack runs: **stop the "VMware NAT Service" on `HOST`** so the VMs keep talking to each other (static IPs, Wazuh logs, dashboard) but lose internet and home-LAN access. Disconnecting a VM's only network adapter would also cut it from the lab, so that is not used. To be verified before Phase 2: with the service stopped, `ping 8.8.8.8` from `WIN10-01` must fail and `ping 192.168.149.10` must still succeed. Residual risk: `KALI01` can still reach `HOST` through the VMware host adapter (192.168.149.1). A dedicated isolated VMnet remains an option.
- `WAZUH01` runs its NIC in promiscuous mode (persistent via a systemd unit) so Suricata can see traffic between other VMs. Verified with ICMP between `WIN10-01` and `DC01`.

## Active Directory

- Domain: `soclab.local` (NetBIOS `SOCLAB`).
- Lab accounts: `alice` (standard user), `svc-sql` (service account with SPN `MSSQLSvc/sql01.soclab.local:1433`, used as the Kerberoasting target). Passwords are lab-only and are never stored in this repository.
- Audit policy (Default Domain Controllers Policy): Kerberos Authentication Service and Kerberos Service Ticket Operations, success and failure.

## Telemetry sources verified in the SIEM

| Source | Evidence |
|---|---|
| `WIN10-01` Sysmon (config: sysmon-modular `sysmonconfig.xml`) | Alerts from rules 92031, 92201, 92021 |
| `WIN10-01` Windows Security | Logon events |
| `DC01` Windows Security (4768, 4769, 4771) | Alerts from rules 60106, 60104, 60112 |
| `DC01` Sysmon | Alerts from rules 92154, 92213, 92151 |
| Suricata on `WAZUH01` | Test rule fired for traffic between `WIN10-01` and `DC01` |

Known gap: Kerberos service ticket events (4769) are mapped by the default Wazuh ruleset to the generic "Windows Logon Success" rule, so Kerberoasting has no dedicated detection yet. This is the Phase 3 starting point.

## Known noise

Alerts produced by the lab build itself (installing Sysmon and the agent, running PowerShell as administrator, changing audit policy) are not attacks. They must be excluded from baseline and false-positive counts.

## Conditions that still have to be recorded before Phase 2

| Item | Status |
|---|---|
| Windows Defender and Tamper Protection state on `WIN10-01` and `DC01` | To be confirmed and fixed before the baseline run |
| Windows Update policy | Not decided |
| Atomic Red Team version pin (commit SHA) | Not decided |
| Attack-run network isolation | Not decided |

## Not done on purpose

- No VM snapshots (disk space). Consequence: a victim VM cannot be reverted to a clean state between runs.
- Linux auditd not deployed: there is no Linux victim in scope.
