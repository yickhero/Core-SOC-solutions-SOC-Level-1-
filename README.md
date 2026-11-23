# Core-SOC-solutions-SOC-Level-1-
Here is my note after finishing core SOC solutions module in tryhackme
# Introduction to EDR

**Endpoint Detection and Response (EDR)** is a security solution designed to monitor, detect, and respond to advanced threats at the endpoint level.

### EDR Solutions in the Market
* CrowdStrike Falcon
* Symantec EDR
* SentinelOne EDR
* Microsoft Defender for Endpoint

### Core Capabilities & Limitations
* **Visibility:** The level of visibility EDR provides is impressive. It collects detailed data from the endpoints, including process modifications, registry modifications, file and folder modifications, user actions, and much more. It presents this information in a very structured format so the analyst can see the whole process tree with a complete activity timeline of the sequence of actions.
* **Detection:** Incorporates signature-based detections as well as behavior-based detections (such as unexpected user activities). With modern machine learning capabilities, it identifies any deviation from baseline behavior and instantly flags it. It can also detect fileless malware that resides in memory and allows feeding custom IOCs for threat detections.
* **Response:** Empowers analysts to take action on detected threats at any endpoint within the central EDR console. With full-fledged details on when, where, and what happened, an analyst can isolate a complete endpoint, terminate a process, quarantine files, or connect to the host remotely and execute actions independently.
* **Limitation:** It is important to remember that an EDR is a **host-only** security solution and does not detect network-level threats.

### EDR vs. Antivirus (AV)
The Antivirus (AV) may detect some basic threats, but to detect advanced threats that evade normal detections, we need an EDR. Unlike antivirus software's basic signature-based detection, an EDR monitors and records the behaviors of the endpoint. An EDR also provides organization-wide visibility of any activity—for example, if a suspicious file is detected on one endpoint, the EDR will also check it across all the other endpoints.

### Architecture & Telemetry
* **Agents (Sensors):** Multiple endpoints are integrated with the EDR and managed through a centralized console using EDR agents (also referred to as sensors). They are the eyes and ears of the EDR, sitting at the endpoint to monitor all activities.
* **Correlation:** All detailed data sent by the EDR agents is correlated and analyzed through complex logic, machine learning algorithms, and threat intelligence matching. The EDR acts like the brain connecting all the dots to form a detection (often called an alert).
* **Telemetry:** The "black box" of an endpoint containing everything necessary for detection and investigation.

### Collected Telemetry
* **Process Executions and Terminations:** Keeps track of all running and idle processes, helping identify suspicious child-parent process relationships, suspicious executables initiating a process, malware payloads, etc.
* **Network Connections:** Monitors all endpoint network connections to help identify connections to a C2 server, unusual port usage, data exfiltration, or lateral movement within the network.
* **Command Line Activity:** Captures all commands executed on the endpoints in CMD, PowerShell, etc., helping identify malicious command execution and obfuscated PowerShell script executions often missed by traditional antivirus.
* **Files and Folders Modifications:** Tracks modifications made by threat actors during data staging, ransomware executions, and malicious file dropping.
* **Registry Modifications:** The registry is a goldmine of information about configurations in a Windows system. Most registry modifications that occur during malicious activity are monitored by the EDR.

### Detection Techniques
* **Behavioral Detection:** Instead of just matching signatures with known threats, it observes the complete behavior of a file. Advanced threats craft malware to look clean and use legitimate processes to carry out attacks; EDR catches this behavior.
  * *Example:* A `winword.exe` process spawning `PowerShell.exe` will be flagged by the EDR due to an unusual parent-child relationship.
* **Anomaly Detection:** Over time, EDR understands the baseline behavior of endpoints and flags any deviation. While this can sometimes generate false positives, the full context provided allows analysts to identify its legitimacy.
  * *Example:* A process modifies an auto-start registry key, which is not common behavior on that endpoint.
* **IOC Matching:** EDRs have strong threat intelligence feed integrations. Except for zero-day attacks, most attacks have indicators published in threat intelligence feeds, and EDR flags any activity matching a known IOC.
  * *Example:* A user downloads a file that drops an executable; the hash of this executable is matched with the threat intelligence feed and instantly flagged.
* **MITRE ATT&CK Mapping:** Flagged activities are mapped with the specific MITRE Tactic and Technique (attack stage) to assist analysts.
  * *Example:* Creating a scheduled task maps to **Tactic:** Persistence / **Technique:** Scheduled Task/Job.
* **Machine Learning Algorithms:** Modern EDRs use ML models trained on large datasets of normal and malicious behaviors to detect complex attack patterns that may bypass other detection techniques.
  * *Example:* Attacks where individual actions are not inherently malicious, but the ML algorithm identifies the whole chain of activities as malicious (such as fileless attacks and multi-staged intrusions).

### Responses After Detection
* **Isolate Host:** Isolates an endpoint from the network through EDR to contain malicious activity and stop lateral movement before it compromises the whole network.
* **Terminate Process:** Used when host isolation would disrupt core business operations and cause more loss than the malicious activity. Analysts can terminate any process at any time, though this must be done consciously to avoid disrupting legitimate processes.
* **Quarantine:** Moves a malicious file to an isolated location where it cannot be executed, allowing analysts to review, restore, or permanently remove it.
* **Remote Access:** Allows analysts to remotely access the shell of an endpoint when built-in responses are not enough, providing deeper visibility to run custom actions, execute scripts, or collect data (e.g., CrowdStrike Falcon EDR's Real Time Response (RTR) console).
* **Artefacts Collection:** Allows analysts to extract important data from endpoints for detailed forensic investigation or legal reporting without physically accessing the device. Commonly extracted artefacts include:
  * Memory Dump
  * Event Logs
  * Specific Folder Contents
  * Registry Hives

---

# Splunk: The Basics

**Splunk** is one of the leading SIEM solutions in the market. It allows users to collect, analyze, and correlate network and machine logs in real time.

### Three Main Components
1. **Splunk Forwarder:** A lightweight agent installed on the endpoint to be monitored. Its main task is to collect data and send it to the Splunk instance without affecting endpoint performance. Key data sources include:
   * Web servers generating web traffic.
   * Windows machines generating Windows Event Logs, PowerShell, and Sysmon data.
   * Linux hosts generating host-centric logs.
   * Databases generating DB connection requests, responses, and errors.
2. **Splunk Indexer:** Plays the main role in processing data received from forwarders. It parses and normalizes the data into field-value pairs, categorizes it, and stores the results as events, making the processed data easy to search and analyze.
3. **Splunk Search Head:** The place within the Search & Reporting App where users search indexed logs using **SPL (Search Processing Language)**. When a user performs a search, the request is sent to the indexer, and relevant events are returned as field-value pairs. It also transforms results into presentable tables and visualizations such as pie, bar, and column charts.

---

# Elastic Stack: The Basics

### Core Components
* **Elasticsearch:** A full-text search and analytics engine for JSON-formatted documents. It acts as a database that stores, analyzes, and correlates data, and supports a RESTful API for interacting with it.
* **Logstash:** A data processing engine that takes data from different sources (beats, ports, or files), filters/normalizes it into field-value pairs, and sends it to a destination. A Logstash configuration file is divided into three parts:
  1. **Input:** Defines the source from which the data is being ingested.
  2. **Filter:** Specifies the filter options to normalize the ingested log.
  3. **Output:** Defines where the filtered data should be sent (a listening port, Kibana interface, Elasticsearch database, or file).
* **Beats:** Host-based agents known as data-shippers that transfer data from endpoints to Elasticsearch. Each beat is a single-purpose agent (e.g., **Winlogbeat** collects Windows event logs, and **Packetbeat** collects network traffic flows).
* **Kibana:** A web-based data visualization tool that works with Elasticsearch to analyze, investigate, and visualize data streams in real time, shaping data into visualizations, time charts, infographics, and dashboards.

### KQL (Kibana Query Language)
KQL is a search query language used to search the ingested logs/documents in Elasticsearch.
* **Free Text Search:** Searches for logs based on text only, regardless of the field (e.g., searching `security` or `United States` returns all documents containing that term in any field).
* **Wildcards & Operators:** KQL looks for the whole term/word in documents, allows the wildcard `*` to match parts of a word, and supports logical operators.
* **Field-Based Search:** Uses the syntax `Field: Value` (with a colon as a separator) to specify the exact field name and value to search for in the logs.

---

# Introduction to SOAR

### Challenges Faced by SOCs
1. **Alert Fatigue**
2. **Too Many Disconnected Tools:** Security tools are often deployed without integration, forcing teams to navigate firewall logs and endpoint security logs independently, leading to tool overload.
3. **Manual Processes:** SOC investigation procedures are often undocumented, leading to inefficient ways of addressing threats.
4. **Talent Shortage**

### What is SOAR?
**Security Orchestration, Automation, and Response (SOAR)** is a way to combine and coordinate all security tools together.
* **Unified Interface:** SOC analysts do not need to switch between SIEM, EDR, Firewall, and other tools for their investigations—they can operate all of them within a single SOAR interface.
* **Case Management:** Provides ticketing and case management features so analysts can document, track, and resolve incidents in a structured way.
* **Automation:** SOAR can follow playbooks automatically.
