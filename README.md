# Splunk SOC 1.0

## Objective
Build a centralized SOC monitoring workflow that ingests Windows security telemetry into Splunk, validates detection visibility through controlled attack activity and supports investigation of suspicious authentication events.

## Scope & Assumptions
This project uses an isolated VirtualBox SOC lab comprising Active Directory, a Windows endpoint, Splunk Enterprise, and Kali Linux. All security testing was authorized and performed exclusively within the lab environment.

## Skills
SIEM deployment & telemetry analysis | Network configuration & connectivity | Windows security & authentication analysis | Attack simulation & detection investigation | Active Directory administration.

## Tools
- **Splunk Enterprise** — searches, correlates and investigates Windows security telemetry to validate suspicious activity.
- **Splunk Universal Forwarder** — configured and maintained endpoint telemetry forwarding from WS01 to Splunk.
- **Sysmon** — configured endpoint telemetry collection for deeper process and system activity.
- **Windows Server / Windows 10** — managed Active Directory users, authentication, domain membership and RDP access across DC1 and WS01.
- **Ubuntu Server** — deployed and maintained the Splunk server and ingestion infrastructure.
- **Kali Linux** — executed authorized attack simulations and authentication testing.
- **Crowbar / NetExec** — tested the RDP authentication path and validated brute-force detection telemetry.
- **VirtualBox** — built and maintained the isolated SOC lab infrastructure.

## Steps

### 1. Core Lab Configuration
Configured the VirtualBox Host-only network and validated connectivity across the SOC lab systems.

### 2. Splunk Deployment
Configured SPLUNK-SRV with static IP `192.168.56.130`, deployed Splunk Enterprise, and enabled persistent service operation.

### 3. Telemetry Ingestion
Configured the Splunk Universal Forwarder on WS01 to collect Application, Security, System, and Sysmon telemetry, created the `splunk_soc` index, enabled TCP `9997` ingestion, and verified WS01 events were indexed successfully.

### 4. Active Directory Environment
Configured DC1 as the `SplunkSOC.local` domain controller, created the IT and HR OUs and domain accounts, and joined WS01 to the domain.

<img src="Screenshots1.0/ADDS-Iinstalled-on DC1.png">

<img src="Screenshots1.0/ADUC.png">

<img src="Screenshots1.0/WS01_Domain_Join_SplunkSOC_DNS_Verification.png">

### 5. Domain Authentication Validation
Validated domain authentication on WS01 with JSmith and confirmed the active domain context.

<img src="Screenshots1.0/DomainName-Username.png">

<img src="Screenshots1.0/whoami.png">

### 6. RDP Access Configuration
Enabled RDP on WS01 and configured authorized domain users for remote authentication testing.

### 7. RDP Brute-Force Simulation
Executed an authorized RDP password-testing attack from Kali against WS01 with NetExec using the prepared credential list.

<img src="Screenshots1.0/NetExec_RDP_BruteForce_Success.png">

### 8. SOC Investigation
Queried `splunk_soc` for JSmith authentication activity and reviewed the associated Windows security events.

<img src="Screenshots1.0/bruteforce-eventcode.png">

**Splunk table showing the successful JSmith logon, Logon Type 3, source IP 192.168.56.101, and target WS01.**

<img src="Screenshots1.0/4624.png">

## Challenges & Troubleshooting
- **Network addressing:** DC1 DHCP addressing caused inconsistent endpoint connectivity; static addressing was applied to stabilize the lab network.
- **Sysmon ingestion format:** `inputs.conf` used XML rendering and the `XmlWinEventLog` sourcetype; corrected to `renderXml = false` with the standard `WinEventLog` Sysmon sourcetype.
- **Forwarder destination:** WS01 Universal Forwarder retained the previous Splunk server address; `outputs.conf` was updated to `192.168.56.130:9997`.
- **Validation:** Restarted the Universal Forwarder and confirmed fresh Sysmon events were indexed in the expected format.

## Summary

### Investigation Findings
Splunk captured 20 `EventCode=4625` failed logon events for JSmith, followed by a successful `EventCode=4624` authentication. The successful event identified JSmith, Logon Type 3, target WS01, and source IP `192.168.56.101`, establishing the Kali-to-WS01 authentication path.

### Security Decision & Validation
The lab demonstrated an end-to-end SOC workflow by combining WS01 endpoint telemetry, centralized Splunk collection, Active Directory authentication and controlled attack simulation to investigate suspicious authentication activity through source identification.

## Operational Impact
Engineered a SIEM pipeline that collected Windows Security and Sysmon telemetry through the Universal Forwarder into Splunk, enabling raw endpoint events to be correlated into actionable authentication activity and source attribution.