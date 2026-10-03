# SPLUNK SOC 1.0

## Objective
Build a centralized SOC monitoring workflow that ingests Windows security telemetry into Splunk, validates detection visibility through controlled attack activity and supports investigation of suspicious authentication events.

## Architecture & Environment
This project uses an isolated VirtualBox SOC lab comprising Active Directory, a Windows endpoint, Splunk Enterprise, and Kali Linux. All security testing was authorized and performed exclusively within the lab environment.

## Skills
**SIEM deployment & telemetry analysis** — deployed Splunk and analyzed centralized Windows Security and Sysmon events.  
**Network configuration & connectivity** — designed and validated the isolated VirtualBox Host-only SOC network.  
**Windows security & authentication analysis** — investigated EventCode `4624` and `4625` authentication activity.  
**Attack simulation & detection investigation** — executed controlled RDP authentication testing from Kali Linux and validated - resulting telemetry in Splunk.  
**Active Directory administration** — configured domain services, user accounts, OUs, domain membership and RDP access.

## Tools
**Splunk Enterprise** — centralized Windows security telemetry, detection, correlation and investigation.  
**Splunk Universal Forwarder** — forwarded WS01 endpoint telemetry to the centralized Splunk instance.  
**Sysmon** — provided detailed endpoint process and system activity telemetry.  
**Windows Server & Windows 10** — supported Active Directory, authentication, domain membership and RDP testing across DC1 and WS01.  
**Ubuntu Server** — hosted Splunk Enterprise and provided the SIEM ingestion infrastructure.  
**Kali Linux** — generated controlled authentication activity for detection validation.  
**Crowbar & NetExec** — simulated RDP authentication attacks and validated resulting detection telemetry.  
**VirtualBox** — provided the isolated SOC lab infrastructure for controlled security testing.

## Steps

### 1. Core Lab Configuration
Configured the VirtualBox Host-only network and validated connectivity across the SOC lab systems.

### 2. Splunk Deployment
Configured SPLUNK-SRV with static IP `192.168.56.130`, deployed Splunk Enterprise and enabled persistent service operation.

### 3. Telemetry Ingestion
Configured the Splunk Universal Forwarder on WS01 to collect Application, Security, System and Sysmon telemetry, created the `splunk_soc` index, enabled TCP `9997` ingestion and verified events from both Windows systems were successfully indexed in Splunk.

### 4. Active Directory Environment
Configured DC1 as the `SplunkSOC.local` domain controller, created the IT and HR OUs and domain accounts and joined WS01 to the domain.

<img src="Screenshots1.0/ADDS-Iinstalled-on DC1.png">

<img src="Screenshots1.0/ADUC.png">

<img src="Screenshots1.0/WS01_Domain_Join_SplunkSOC_DNS_Verification.png">

### 5. Domain Authentication Validation
Validated domain authentication on WS01 with JSmith and confirmed the active domain context.

<img src="Screenshots1.0/whoami.png">

### 6. RDP Access Configuration
Enabled RDP on WS01 and configured authorized domain users for remote authentication testing.

### 7. RDP Brute-Force Simulation
Executed a controlled RDP authentication attack from Kali against WS01 with NetExec to generate realistic failed and successful Windows authentication telemetry for SOC detection validation.

<img src="Screenshots1.0/NetExec_RDP_BruteForce_Success.png">

### 8. SOC Investigation
Queried the `splunk_soc` index to correlate the generated authentication telemetry, confirming repeated `EventCode=4625` failures followed by `EventCode=4624` success (SL) and establishing the baseline authentication-detection capability of the SOC environment.

<img src="Screenshots1.0/bruteforce-eventcode.png">

**Splunk table showing the successful JSmith logon, Logon Type 3, source IP 192.168.56.101, and target WS01.**

<img src="Screenshots1.0/4624.png">

## Challenges & Troubleshooting
**Network addressing:** DC1 DHCP addressing caused inconsistent endpoint connectivity; static addressing was applied to stabilize the lab network.
**Sysmon ingestion format:** `inputs.conf` used XML rendering and the `XmlWinEventLog` sourcetype; corrected to `renderXml = false` with the standard `WinEventLog` Sysmon sourcetype.
**Forwarder destination:** WS01 Universal Forwarder retained the previous Splunk server address; `outputs.conf` was updated to `192.168.56.130:9997`.
**Validation:** Restarted the Universal Forwarder and confirmed fresh Sysmon events were indexed in the expected format.

## Summary

### Investigation Findings
Splunk established a working SIEM detection baseline by capturing and correlating repeated `EventCode=4625` failed logons followed by `EventCode=4624` success, providing visibility into brute-force authentication indicators and the originating source.

### Security Decision & Validation
The validated telemetry pipeline demonstrated that the SOC environment could collect, centralize, and investigate authentication activity across the Windows domain, providing a foundation for detecting brute-force attempts and potential lateral-movement activity.

## Operational Impact
Engineered a SIEM pipeline that collected Windows Security and Sysmon telemetry through the Universal Forwarder into Splunk, enabling raw endpoint events to be correlated into actionable authentication activity and source attribution.

---


# SPLUNK SOC 2.0 (SOC AUTOMATION WITH AI)


## Objective
Extend the existing Splunk SOC 1.0 environment into an automated SOC pipeline that detects unauthorized account activity, enriches security alerts with threat intelligence, uses AI-driven Tier 1 triage and delivers actionable findings to Slack.

## Architecture & Environment
This project extends the existing Splunk SOC 1.0 environment by integrating n8n, Google Gemini, AbuseIPDB and Slack to automate security-alert analysis, threat-intelligence enrichment and analyst notification.

<img src="Screenshots 2.0/Architecture.png">

## Skills
**SIEM monitoring & detection** — created Splunk authentication detections and routed triggered alerts into the automation pipeline.  
**SOC Tier 1 triage & threat intelligence** — enriched suspicious source IPs with AbuseIPDB and incorporated the results into alert analysis.  
**Security automation & AI-assisted analysis** — built an n8n workflow that passed Splunk alert data through Gemini for structured Tier 1 analysis.  
**Security integration & incident notification** — integrated Splunk, n8n, Gemini, AbuseIPDB and Slack into an automated alert-to-notification workflow.

## Tools

**Draw.io** — designed the SOC architecture and security workflow.<br>
**Splunk Enterprise** — SIEM for detecting Windows authentication activity and generating security alerts.<br>
**n8n** — workflow automation platform connecting Splunk with analysis, enrichment, and notification services.<br>
**Google Gemini** — AI model used to structure and analyze security alerts.<br>
**AbuseIPDB** — threat-intelligence service used to enrich suspicious source IP addresses.<br>
**Slack** — notification platform used to deliver analyzed security alerts.<br>
**Docker & Docker Compose** — used to deploy n8n on the existing SPLUNK-SRV.<br>
**Windows Server, Windows 10 & Sysmon** — supplied endpoint security telemetry to Splunk.

## Steps

### 1. Deploy n8n on the Existing Splunk Server
Deployed n8n on the existing `SPLUNK-SRV` using Docker and Docker Compose to provide the automation layer for processing Splunk security alerts without introducing another VM.

**Docker Compose configuration:**

```text
    services:
      n8n:
        image: n8nio/n8n:latest
        restart: always
        ports:
          - "5678:5678"
        environment:
          - N8N_HOST=192.168.56.130
          - N8N_PORT=5678
          - N8N_PROTOCOL=http
          - N8N_SECURE_COOKIE=false
        volumes:
          - ./n8n_data:/home/node/.n8n
```

### 2. Create the Splunk Brute-Force Detection

<img src="Screenshots 2.0/Alert.png">

Created a scheduled Splunk alert to detect failed Windows authentication activity and forward the resulting security alert into the automation workflow.

**SPL detection query:**

```spl
index=splunk_soc EventCode=4625 | stats count by _time ComputerName user src_ip
```

- Used Windows EventCode `4625` as the detection signal for failed authentication activity.
- Aggregated failed logons by timestamp, host, user, and source IP to expose concentrated authentication activity.
- Configured the `Test-Brute-Force` alert on a 1-minute schedule to evaluate recent authentication activity continuously.
- Routed triggered detections through the Splunk webhook action into the n8n automation pipeline.
- Validated the detection by generating controlled failed RDP authentication attempts against WS01.

### 3. Connect Splunk to n8n

<img src="Screenshots 2.0/n8n-worflow.png">

Connected the Splunk alert to an n8n Webhook so security-alert data could be passed automatically into the analysis workflow.
The webhook received the alert's time, computer name, user, source IP and event count, confirming successful Splunk-to-n8n data transfer.

### 4. Integrate Gemini for SOC Alert Analysis
Integrated Google Gemini into n8n to structure security alerts into analyst-focused findings, including threat-intelligence enrichment, severity assessment, and recommended actions.

**n8n prompt template containing expressions:**

**Values 1**

```text
    Act as a tier one SOC analyst assistant when provided with a security alert. Perform the following steps:
    1. Summarize the alert.
    2. Enrich with threat intelligence.
    3. Assess the severity based on MITRE ATT&CK.
    4. Recommend next actions.

    Keep the response concise and focused. Avoid unnecessary explanations or lengthy detail.

    For any IP enrichment, use the tool named abuse-ipdb-enrichment.
```

**Values 2**

```text
    Format the output clearly. Return findings in a structured format such as Summary, IOC Enrichment, Severity Assessment, and Recommended Actions.
```

**Values 3**

```text
    Alert: {{ $json.body.search_name }}

    Alert Details: {{ JSON.stringify($json.body.result, ['_time', 'user', 'ComputerName'], 2) }}

    Source IP: 194.127.117.45
```

### 5. Connect n8n Workflow to Slack

Connected n8n to a dedicated Slack alerts channel to route completed SOC analysis directly to analysts for triage.

- Passed the structured Gemini analysis output into the Slack notification node.
- Formatted the alert payload into an analyst-readable notification containing the alert summary, IOC enrichment, severity assessment, and recommended actions.
- Routed the completed analysis to the designated SOC alerts channel.
- Validated end-to-end delivery from the n8n workflow to Slack.

### 6. Add AbuseIPDB Threat-Intelligence Enrichment

Integrated AbuseIPDB into the n8n workflow as an IOC-enrichment stage, allowing suspicious source IPs to be evaluated before Gemini performs the SOC analysis.

- Passed the detected source IP from the Splunk alert into the AbuseIPDB API enrichment node.
- Queried AbuseIPDB for reputation and abuse-history data associated with the source IP.
- Injected the enrichment results into the workflow alongside the original Splunk alert context.
- Fed the combined alert and threat-intelligence data to Gemini for contextual Tier 1 analysis.
- Validated that the workflow returned abuse confidence, report count, geographic information and threat context for the tested IP.

### 7. Validate the End-to-End SOC Workflow

<img src="Screenshots 2.0/n8n-output.png">


<img src="Screenshots 2.0/Slack-alert.png">

Tested the complete **Splunk → n8n → Gemini → AbuseIPDB → Slack** workflow and verified that a detected brute-force alert was enriched, analyzed, assessed, and delivered to Slack.

## Challenges & Troubleshooting
The n8n Slack integration initially failed because the Docker container could not resolve Slack's hostname; container DNS and HTTPS tests identified the issue, and restarting the n8n container restored connectivity and successful Slack message delivery.

## Summary

**Investigation Findings:** Splunk correlated repeated Windows `EventCode=4625` failed-logon events into a brute-force detection trigger, providing the structured alert payload used by the downstream automation pipeline.

**Security Decision:** n8n, Gemini, AbuseIPDB and Slack were added to the existing Splunk SOC 1.0 environment to automate alert analysis while retaining Splunk as the detection and monitoring layer.

**Validation:** The completed workflow successfully enriched the source IP with AbuseIPDB and delivered a structured SOC analysis to Slack containing the alert summary, IOC enrichment, MITRE ATT&CK severity assessment, and recommended actions.

## Operational Impact
Automated the movement of a detected security alert from SIEM detection through threat-intelligence enrichment and Tier 1 analysis to analyst notification, reducing manual alert-handling steps.

---


# Splunk SOC 3.0 — SOAR/EDR Integration

## Objective
Build an end-to-end SOC automation workflow to detect password-recovery tool activity, generate Slack and email alerts, obtain analyst approval, automatically isolate the affected endpoint, and verify the isolation status and network containment.

## Scope & Assumptions
This project was implemented as a controlled lab simulation using the existing Windows Server DC1 enrolled in LimaCharlie, with Tines, Slack and email used for detection, alerting, analyst approval, automated response and validation.

## Skills
- **EDR Operations** — deployed and validated the LimaCharlie sensor and reviewed endpoint telemetry generated by controlled LaZagne execution.
- **Threat Hunting & Detection Engineering** — analyzed endpoint process telemetry to identify actionable file path, command-line, and SHA-256 indicators, then created and validated a LimaCharlie Detection & Response rule for LaZagne activity.
- **SOAR Automation** — connected LimaCharlie detections to Tines and built automated alerting, analyst approval, endpoint isolation, and verification workflows.
- **Incident Response** — implemented analyst-approved endpoint containment, including a separate response path when isolation was declined.
- **Security Validation** — verified endpoint containment through LimaCharlie isolation status and network connectivity testing, then confirmed network access was restored after rejoining.
- **Security Communications** — delivered detection and response information through Slack and email alerts.

## Tools
- **LimaCharlie** — EDR telemetry, detection engineering, endpoint isolation, and isolation-status verification.
- **Tines** — SOAR orchestration, alert routing, analyst approval, and automated response.
- **Slack** — SOC detection, investigation, and response-status notifications.
- **Email** — secondary detection-alert channel containing investigation details.
- **Windows Server DC1** — controlled endpoint used to generate and validate security telemetry.
- **LaZagne** — controlled password-recovery tool activity used to generate detectable endpoint telemetry.

## Steps

### 1. Deploy and Validate the LimaCharlie EDR Sensor

<img src="Screenshots 3.0/dc1-lc-sensor.png">

Deployed the LimaCharlie Windows sensor on DC1 and verified that the endpoint was enrolled and reporting telemetry for centralized EDR monitoring.

```text
cd Downloads
dir
.\hcp_win_x64_release_5.3.12.exe -i <sensor key>
```

### 2. Create and Validate the LaZagne Detection

Created a LimaCharlie Detection & Response rule that identifies LaZagne through Windows process telemetry, file path, command line and file hash indicators.

```yaml
events:
  - NEW_PROCESS
  - EXISTING_PROCESS
op: and
rules:
  - op: is windows
  - op: or
    rules:
    - case sensitive: false
      op: ends with
      path: event/FILE_PATH
      value: LaZagne.exe
    - case sensitive: false
      op: contains
      path: event/COMMAND_LINE
      value: LaZagne
    - case sensitive: false
      op: is
      path: event/HASH
      value: "dc06d62ee95062e714f2566c95b8edaabfd387023b1bf98a09078b84007d5268"
```

```yaml
- action: report
  metadata:
    author: DeleDFIR
    description: TEST - Detects Lazagne 
    falsepositives:
    - None
    level: high
    tags:
    - attack.credential_access
  name: DeleDFIR - HackTool - Lazagne (SOAR-EDR)
```

<img src="Screenshots 3.0/lazagne-detection-validation.png">

Executed `LaZagne.exe all` on DC1 and verified that LimaCharlie generated a detection containing the endpoint and process telemetry required for downstream SOAR automation.

### 3. Integrate LimaCharlie with Tines and Slack
Configured the existing Slack workspace and dedicated `soar-edr-alerts` channel, connected LimaCharlie detections to a Tines webhook, and verified that detection alerts were received by Tines and delivered successfully to Slack for downstream SOC alerting.

### 4. Configure Email Alerting and Analyst Isolation Decision
Configured Tines to send LimaCharlie detection alerts by email and built a user prompt that presents the detection details and captures the analyst’s Yes/No decision on whether to isolate the affected endpoint.

<img src="Screenshots 3.0/no-response.png">

### 5. Configure the No-Isolation Response
Configured the `isolate = false` branch to send a Slack notification when the analyst chooses not to isolate the affected endpoint.

<img src="Screenshots 3.0/tines-no-isolation-slack-alert.png">

### 6. Automate Endpoint Isolation

Connected the `isolate = true` decision to LimaCharlie `Isolate Sensor`, configured the sensor ID and API credential and verified that the isolation action returned HTTP 200 and isolated DC1.

<img src="Screenshots 3.0/tines-yes-lima-isolation.png">

<img src="Screenshots 3.0/limacharlie-endpoint-isolated.png">

Confirmed in LimaCharlie that DC1 was isolated after the approved response.

### 7. Validate Endpoint Isolation and Network Recovery
Validated endpoint containment by confirming network connectivity failed with `General failure` while DC1 was isolated and was restored after rejoining the network.

<img src="Screenshots 3.0/end-to-end-isolation-validation.png">

### 8. Verify Isolation Status in Slack

<img src="Screenshots 3.0/YES-slack-isolation-status-confirmation.png">

Added the LimaCharlie `Get Isolation Status` action and verified that Slack reported `Isolation status: true` after the endpoint was isolated.

<img src="Screenshots 3.0/limacharlie-endpoint-rejoined.png">

Confirmed in LimaCharlie that network access was restored after the endpoint was rejoined.

<img src="Screenshots 3.0/soar-edr-playbook.png">

Completed the SOAR/EDR playbook covering detection, alerting, analyst approval, endpoint isolation, isolation-status verification, and notification.

## Challenges & Troubleshooting
The LimaCharlie `Isolate Sensor` action initially returned HTTP 401, indicating an authentication issue with the API credential used by Tines.
Created and connected the correct LimaCharlie REST API credential with the `*.limacharlie.io` domain restriction, after which the isolation action returned HTTP 200 and successfully isolated DC1.

## Summary

**Investigation Findings:** LimaCharlie telemetry from controlled LaZagne execution provided the file path, command line, hash, hostname, source IP and detection link used to identify credential-access activity and drive the downstream response workflow.

**Security Decision:** LimaCharlie was selected for endpoint detection and containment, while Tines provided orchestration and an explicit analyst approval gate before automated response.

**Validation:** The completed workflow generated the detection, delivered Slack and email alerts, captured the isolation decision, isolated DC1 with HTTP 200, confirmed `Isolation status: true`, blocked network connectivity, and verified connectivity restoration after rejoining.

## Operational Impact
The workflow demonstrated how a SOC can reduce response time to endpoint threats by connecting EDR detection, SOC alerting, analyst approval, automated endpoint containment, isolation verification, and network recovery validation into a single response process. The lab achieved automated endpoint isolation in less than 1 minute from detection, reducing the time an analyst would otherwise spend manually coordinating containment and verification.
