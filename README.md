# End-to-End SIEM and EDR Implementation: L1 SOC Operations using Wazuh

This repository documents the deployment and operational validation of a unified **Security Information and Event Management (SIEM)** and **Endpoint Detection and Response (EDR)** infrastructure using **Wazuh**, an open-source counterpart to enterprise SIEM and EDR platforms such as **Splunk** and **IBM QRadar**. The architecture utilises a **star network topology** secured with **AES-256 data encryption** to connect a central **Wazuh Manager** to distributed **Windows and Linux agents** to ensure **cross-platform functionality**. Testing conducted via an external **red team node** validates **threat detection logic**, as well as **active incident response** against **SSH brute-force** and **malware vectors**. Security operations also included automated **vulnerability assessment** for **CIS-benchmarked** reports, identifying critical **CVEs** to ensure **compliance**.

## Technical Stack
* **SIEM/EDR:** Wazuh v4.14.1
* **Hypervisor:** VMware Workstation Pro 17
* **Infrastructure (Manager):** Ubuntu Server 24.04 LTS (Virtual Machine)
* **Monitored Endpoints (Agents):** Windows 11 (Physical Host), Lubuntu 22.04 LTS (Virtual Machine)
* **Offensive Node:** External Lubuntu 22.04 Virtual Machine (Red Team Node)

## Network Architecture
The environment is structured to **centralise security management** and **monitoring**, forcing all endpoint telemetry through **encrypted AES-256 tunnels**.
* **Key Generation:** Unique **authentication keys** were generated for the **Linux** and **Windows agents** using the `manage_agents` tool in the manager's terminal.
* **Encryption Standard:** All log data is encapsulated using **AES-256** to prevent potential **Man-in-the-Middle (MitM)** attacks.
* **Deployment:** The **authentication keys** were **imported** into the **Linux CLI** and **Windows GUI** to authenticate the endpoints and initiate the connection.

## Configured Security Controls
The SIEM is configured to mitigate specific threat profiles using **correlation engines** and **automated policy enforcement**.

**Threat Detection Engineering**
* **Correlation Rules:** Custom **XML-based rules** within the `ossec.conf` file maps incoming events and analyse them according to the **MITRE ATT&CK framework**, ensuring accurate classification of potential **Brute-force** and **Privilege Escalation** vectors.
* **Endpoint Protection (EPP):** The **Windows agent** integrates with **Microsoft Defender** to capture real-time **malicious file write interceptions**.

**Automated Incident Response**
* **Defensive Logic:** An **Active Response module** is configured in the **Manager's** `ossec.conf` file to trigger on **Rule 5712 (SSH brute force)**.
* **Countermeasure:** Executes a `host-deny` **command** to instruct the **SSH daemon** to drop traffic from the malicious **source IP**.

**Vulnerability Assessment**
* **Inventory Scanning:** The configured **Vulnerability Detector module** in the **Manager's** `ossec.conf` file cross-references installed packages against **Canonical OVAL definitions** (Canonical's rulebook used by security tools to automatically detect known CVEs on Ubuntu systems).
* **Compliance:** Audits system baselines against **CIS benchmarks** to identify unpatched **Common Vulnerabilities and Exposures (CVEs)**.

## Attack Simulation & Validation
Validation was executed from the Lubuntu **Red Team node** and **Agent/Manager CLIs** to ensure **telemetry ingestion on the Manager Dashboard** and **response mechanisms** functioned correctly.

* **Privilege Escalation Testing:** Execution of the `runas` **command** on the **Windows host** triggered `Rule 60122` (Logon Failure), which correctly escalated to `Rule 60204` (Multiple Windows Logon Failures) after repeatedly executing the command.
* **Malware Defence Testing:** Implantation of an **EICAR test string** resulted in real-time **Quarantine** by **Windows Defender**, with the event successfully forwarded and visualised on the **Wazuh Manager dashboard** as a **Level 12 alert**.
* **Vulnerability Detection:** Continuous scanning identified **36 Critical CVEs** resulting from an experimental kernel version `linux-image-6.14.0-2` on the Linux agent.
* **Active Incident Response Testing:** An **Automated SSH brute-force Bash script** rapidly triggered the `Rule 5712` threshold. The Manager responded by programmatically executing the `host-deny` protocol, resulting in **100% packet loss** during subsequent **ICMP reachability tests**.

## Disclaimer
This environment was constructed exclusively for **educational purposes** and **local security validation**. All attack simulations were performed within an **isolated laboratory network**.
