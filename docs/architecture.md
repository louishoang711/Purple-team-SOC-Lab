# Architecture

This page explains how the lab is built, how a log becomes an alert, and how detections are managed as code. All diagrams are Mermaid, so GitHub renders them directly.

Machine tags used everywhere in this repository: `HOST`, `WAZUH01`, `DC01`, `WIN10-01`, `KALI01` (and `SOAR01` in Phase 4).

## 1. Lab topology

```mermaid
flowchart LR
    HOST["HOST<br/>Windows 11 laptop<br/>VMware, browser, git"]

    subgraph LAB["Lab network: VMware NAT 192.168.149.0/24"]
        direction LR
        KALI["KALI01<br/>Attacker"]

        subgraph VICTIMS["Victims"]
            direction TB
            WIN["WIN10-01<br/>Windows 10 Pro<br/>Sysmon + Wazuh agent"]
            DC["DC01<br/>Windows Server 2022<br/>Active Directory<br/>Sysmon + Wazuh agent"]
        end

        subgraph SOC["SOC"]
            WAZ["WAZUH01<br/>Wazuh SIEM<br/>+ Suricata sensor"]
        end
    end

    KALI -- "attacks (Phase 2)" --> WIN
    KALI -- "attacks (Phase 2)" --> DC
    WIN -- "Sysmon + Security logs" --> WAZ
    DC -- "Kerberos + Sysmon logs" --> WAZ
    HOST -- "dashboard (HTTPS)" --> WAZ
```

| Machine | Role | Runs |
|---|---|---|
| `HOST` | Operator workstation | VMware Workstation, browser, git |
| `WAZUH01` | SIEM and network sensor | Wazuh 4.14.10 (manager, indexer, dashboard), Suricata 8.0.8 with ET Open rules |
| `DC01` | Domain controller | Active Directory `soclab.local`, DNS, Sysmon, Wazuh agent |
| `WIN10-01` | Victim workstation (domain-joined) | Sysmon, Wazuh agent |
| `KALI01` | Attacker | Kali Linux, Atomic Red Team runs target the victims |

Suricata sees traffic between other machines because the `WAZUH01` network card runs in promiscuous mode (verified with ICMP between `WIN10-01` and `DC01`).

## 2. How a log becomes an alert

```mermaid
flowchart LR
    subgraph SRC["Sources"]
        S1["Sysmon<br/>processes, network, files"]
        S2["Windows Security log<br/>logons, Kerberos 4768/4769/4771"]
        S3["Suricata eve.json<br/>network alerts"]
    end

    AG["Wazuh agent<br/>on WIN10-01 and DC01"]
    MGR["Wazuh manager<br/>decoders and rules"]
    IDX["Wazuh indexer<br/>stores alerts"]
    DASH["Dashboard<br/>search, MITRE views"]

    S1 --> AG
    S2 --> AG
    AG -- "1514/TCP" --> MGR
    S3 -- "read locally on WAZUH01" --> MGR
    MGR --> IDX --> DASH
```

An event only appears in the dashboard as an alert if a rule matches it. Events with no matching rule are collected but not shown. This is why Kerberos service tickets (event 4769) currently show up only as the generic "Windows Logon Success" rule, which is the detection gap Phase 3 closes.

## 3. The core idea: detection as code

```mermaid
flowchart LR
    A["1. Write Sigma rule<br/>YAML in Git"] --> B["2. CI on pull request<br/>lint, schema tests, compile"]
    B --> C["3. Gated deploy<br/>config test and rollback"]
    C --> D["4. Run atomic attack<br/>Atomic Red Team"]
    D --> E["5. Validate<br/>query the indexer, pass or fail JSON"]
    E --> F["6. Coverage heatmap<br/>ATT&CK Navigator"]
    F -- "gap found" --> A
```

Rules are files in Git, so every change is reviewed and testable. Each rule is validated against a real simulated attack, and the result feeds a before/after coverage number. Status: planned for Phase 3.

## 4. Planned attack scenario: Kerberoasting (T1558.003)

```mermaid
sequenceDiagram
    participant K as KALI01 (attacker)
    participant D as DC01 (domain controller)
    participant S as WAZUH01 (SIEM)

    K->>D: As a domain user, request a service ticket for svc-sql
    D-->>K: Ticket encrypted with the svc-sql password
    D->>S: Security event 4769 (Kerberos service ticket)
    Note over K: Cracks the ticket offline, no failed logons are produced
    Note over S: Default rule: generic logon success. Phase 3 adds a dedicated rule
```

## 5. Roadmap

```mermaid
flowchart LR
    P1["Phase 1<br/>Lab and telemetry"]
    P2["Phase 2<br/>Baseline attacks"]
    P3["Phase 3<br/>Detection as code"]
    P4["Phase 4<br/>SOAR and reports"]
    P5["Phase 5<br/>One extension"]
    P6["Phase 6<br/>Packaging"]

    P1 --> P2 --> P3 --> P4 --> P5 --> P6

    classDef wip fill:#EF9F27,stroke:#854F0B,color:#412402
    classDef todo fill:#F1EFE8,stroke:#888780,color:#2C2C2A
    class P1 wip
    class P2,P3,P4,P5,P6 todo
```

Amber means in progress, gray means not started. See the [README](../README.md) for the status table.
