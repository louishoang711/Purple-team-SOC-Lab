# Purple-Team SOC Lab

A simulated mini enterprise network with a full SOC pipeline: telemetry collection, attack simulation, detection engineering, SOAR automation, and incident reporting. Built as a hands-on portfolio project for a SOC Analyst role.

> **Disclaimer:** This is a lab environment. All results come from simulated attacks against systems I own, not production data.

## Goals
- Build a realistic enterprise-style environment (Active Directory, Linux web server, firewall/IDS).
- Collect and normalize endpoint, network, and identity telemetry in a SIEM.
- Simulate a multi-stage kill chain mapped to MITRE ATT&CK.
- Write, test, and tune detections; measure detection rate before and after tuning.
- Automate triage and case management with SOAR.
- Produce SOC-grade incident reports and KPIs.

## Architecture
_Diagram to be added in `docs/images/architecture.png`._

| Component | Role | Tooling |
|---|---|---|
| Hypervisor | Lab host | Proxmox / VMware |
| Firewall + IDS | Segmentation, network detection | pfSense/OPNsense + Suricata |
| Domain Controller | Identity target | Windows Server (AD DS) |
| Windows clients | Endpoints | Windows 10/11 + Sysmon + Wazuh agent |
| Linux web server | Server target | Ubuntu + Auditd + Wazuh agent |
| SIEM / EDR | Detection and correlation | Wazuh |
| Attacker | Adversary simulation | Kali, Atomic Red Team, MITRE Caldera |
| SOAR / Cases | Automation and IR | TheHive, Cortex, Shuffle |
| Threat intel | Enrichment | VirusTotal, AbuseIPDB (MISP optional) |

## Roadmap
- [ ] **Phase 1 - Infrastructure** (`01-infrastructure/`): build network, AD, log pipeline
- [ ] **Phase 2 - Attack Simulation** (`02-attack-simulation/`): kill chain runbook
- [ ] **Phase 3 - Detection Engineering** (`03-detection-engineering/`): rules, tuning, coverage matrix
- [ ] **Phase 4 - Automation & IR** (`04-automation-soar/`): TheHive + Shuffle playbooks
- [ ] **Phase 5 - Reporting** (`05-incident-reports/`): incident reports and KPI summary
- [ ] **Stretch:** Microsoft Sentinel (KQL) ports, internet-facing honeypot

## Results
_To be filled with measured numbers._

| Metric | Value |
|---|---|
| ATT&CK techniques simulated | TBD |
| Custom detection rules written | TBD |
| Detection rate before tuning | TBD |
| Detection rate after tuning | TBD |
| False-positive reduction | TBD |
| Incident reports | TBD |

See [`docs/attack-coverage-matrix.md`](docs/attack-coverage-matrix.md) and [`docs/incident-report-template.md`](docs/incident-report-template.md).

## Repository layout
```
01-infrastructure/        Build notes, configs, network diagram
02-attack-simulation/     Attack runbooks and scripts
03-detection-engineering/ Wazuh rules, Sigma rules, tuning notes
04-automation-soar/       TheHive, Cortex, Shuffle workflows
05-incident-reports/      Investigation write-ups
docs/                     Templates, coverage matrix, images
```

## License
MIT
"# Purple-team-SOC-Lab" 
