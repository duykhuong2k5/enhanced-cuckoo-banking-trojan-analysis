# Enhanced Cuckoo Banking Trojan Analysis

A customized Cuckoo Sandbox reporting interface designed for **Banking Trojan** and **Financial Trojan** malware analysis.

This project enhances the default **Cuckoo Sandbox 2.0.7** report interface by adding an ANY.RUN-style layout, dashboard charts, Banking Trojan classification, MITRE ATT&CK mapping, IOC summaries, behavior graph visualization, Banking Fraud Kill Chain, and a custom Banking Trojan heuristic signature.

---

## Overview

`Enhanced Cuckoo Banking Trojan Analysis` improves the readability and analytical value of Cuckoo Sandbox reports. Instead of only showing raw sandbox output, the project reorganizes analysis results into analyst-friendly sections that help users quickly understand:

- What type of malware was analyzed
- Whether the sample behaves like a Banking Trojan
- Which suspicious or malicious behaviors were triggered
- What files were dropped during execution
- What network indicators were observed
- Which MITRE ATT&CK techniques are related to the behavior
- How severe the financial fraud risk is

The project is intended for malware analysts, students, and security researchers working on Windows malware and financial malware analysis.

---

## Key Features

### ANY.RUN-style Summary Report

The default Cuckoo Summary page is redesigned into a more modern malware analysis report layout.

Main improvements:

- Improved file information panel
- Malware score card
- Behavior activity grouping
- Malware configuration panel
- Network and DNS overview
- Dropped file summary
- IOC-focused presentation
- Analyst verdict sections

### Dashboard & Charts

The report includes four visual dashboard charts:

1. **Signature Severity Distribution**  
   Shows how many triggered signatures are malicious, suspicious, or informational.

2. **Behavior Category Overview**  
   Groups malware behavior into process injection, persistence, network/C2, anti-AV, file/payload activity, and banking behavior.

3. **Network & File Activity**  
   Compares DNS requests, contacted hosts, TCP traffic, UDP traffic, and dropped files.

4. **Banking Trojan Risk View**  
   Calculates a Banking Trojan risk score from 0 to 100 based on observed behavior.

### Banking Trojan Classification

Recommended classification format:

```text
Primary classification: Banking Trojan / Financial Trojan
Family: Unknown / Unconfirmed
Family hypothesis: Zeus/Zbot-like behavior, Low confidence
```

The project does not automatically claim a specific malware family such as Zeus/Zbot unless stronger evidence exists, such as:

- YARA match
- Antivirus family label
- Extracted malware configuration
- Webinject or formgrab artifacts
- Known family mutex or string patterns
- Threat intelligence confirmation

### Banking Fraud Kill Chain

The report models the malware behavior as a financial attack flow:

```text
Fake invoice → User execution → Payload dropping → Process injection → Persistence → C2 communication → Financial fraud risk
```

This helps analysts understand the malware as a complete attack chain instead of isolated technical events.

### MITRE ATT&CK Mapping

Observed behaviors are mapped to MITRE ATT&CK techniques, including:

```text
T1204.002 - User Execution: Malicious File
T1036.008 - Masquerading: Masquerade File Type
T1055 - Process Injection
T1547.001 - Registry Run Keys / Startup Folder
T1059.003 - Windows Command Shell
T1071 - Application Layer Protocol
T1095 - Non-Application Layer Protocol
T1489 - Service Stop
T1497 - Virtualization / Sandbox Evasion
```

### Custom Banking Trojan Heuristic Signature

The project includes a custom Cuckoo signature:

```text
banking_trojan_heuristic.py
```

The signature detects Banking Trojan-like behavior based on indicators such as:

- Fake invoice or financial document filename
- Double extension such as `.pdf.exe`
- Process injection behavior
- Autorun persistence
- Dropped executable payloads
- C2 or external network communication
- Anti-AV or service tampering
- Credential or browser-related API usage

---

## Project Structure

```text
enhanced-cuckoo-banking-trojan-analysis/
├── README.md
├── .gitignore
├── templates/
│   └── analysis/
│       └── pages/
│           ├── summary/
│           │   ├── index.html
│           │   ├── _file.html
│           │   ├── _signatures.html
│           │   ├── _dashboard_charts.html
│           │   ├── _behavior_graph.html
│           │   ├── _dropped_files.html
│           │   ├── _threats.html
│           │   ├── _ioc_summary.html
│           │   ├── _malware_mitre.html
│           │   └── _banking_killchain.html
│           ├── behavior/
│           ├── network/
│           └── dropped/
└── signatures/
    └── windows/
        └── banking_trojan_heuristic.py
```

---

## Tested Environment

```text
Cuckoo Sandbox: 2.0.7
Host OS: Ubuntu 18.04 LTS
Guest OS: Windows 7 Ultimate x64
Python: Python 2.7
Virtualization: VirtualBox
Database: MongoDB
Network capture: TCPDump
Report type: Django HTML templates
```

Main Cuckoo template path:

```bash
/usr/local/lib/python2.7/dist-packages/cuckoo/web/templates/analysis/pages/
```

Main Cuckoo signature path:

```bash
/opt/cuckoo/signatures/windows/
```

---

## Cuckoo Sandbox Setup Guide

This section describes the recommended environment used to deploy Cuckoo Sandbox for Banking Trojan and Financial Trojan malware analysis.

---

## System Requirements

| Component | Requirement |
|---|---|
| Host OS | Ubuntu 18.04 LTS |
| Guest OS | Windows 7 Ultimate x64 |
| Virtualization | VirtualBox |
| Python | Python 2.7 |
| Database | MongoDB |
| Network Capture | TCPDump |
| Sandbox | Cuckoo Sandbox 2.0.7 |

---

## Host Environment Setup

### Update the system and install dependencies

```bash
sudo apt update && sudo apt upgrade -y

sudo apt install -y virtualbox python python-pip python-dev \
libffi-dev libssl-dev python-virtualenv python-setuptools \
libjpeg-dev zlib1g-dev swig tcpdump apparmor-utils
```

### Configure TCPDump permissions

TCPDump is required by Cuckoo to capture malware network traffic during analysis.

```bash
sudo aa-disable /usr/sbin/tcpdump
sudo groupadd pcap
sudo usermod -a -G pcap cuckoo
sudo chgrp pcap /usr/sbin/tcpdump
sudo setcap cap_net_raw,cap_net_admin=eip /usr/sbin/tcpdump
```

After adding the user to the `pcap` group, log out and log in again or restart the system to apply the group permission.

---

## VirtualBox Network Configuration

### Create a Host-Only Network

Cuckoo uses a host-only network to isolate the malware analysis virtual machine from the public network while still allowing communication between the host and the guest VM.

```bash
VBoxManage hostonlyif create

VBoxManage hostonlyif ipconfig vboxnet0 \
--ip 192.168.56.1 \
--netmask 255.255.255.0
```

Recommended network configuration:

```text
Host-only interface: vboxnet0
Host IP: 192.168.56.1
Guest IP: 192.168.56.101
Netmask: 255.255.255.0
```

---

## Windows 7 Guest VM Setup

### Basic VM Configuration

```text
Guest OS: Windows 7 Ultimate x64
Network mode: Host-only Adapter
Adapter: vboxnet0
Static IP: 192.168.56.101
```

### Disable Windows Security Features

To allow malware behavior to execute inside the sandbox, disable:

```text
UAC
Windows Firewall
Windows Update
Automatic sleep mode
Screen lock
```

This configuration increases the chance that malware behavior will be triggered during sandbox execution.

---

## Cuckoo Agent Installation

Cuckoo requires an agent running inside the Windows guest VM. The agent receives files from the host and executes the submitted malware sample.

### Install Python 2.7 on Windows

Install Python 2.7 inside the Windows guest machine.

### Transfer `agent.py` to the Windows guest

On the Ubuntu host, start a temporary HTTP server from the directory containing the Cuckoo agent:

```bash
python -m SimpleHTTPServer 8000
```

From the Windows guest, download `agent.py` using the browser:

```text
http://192.168.56.1:8000/agent.py
```

### Configure the agent to start automatically

Copy the agent into the Windows Startup folder:

```text
C:\Users\<User>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup
```

Rename the file:

```text
agent.pyw
```

The `.pyw` extension allows the agent to run without opening a visible console window.

---

## VM Snapshot

After Windows 7 is fully configured and the Cuckoo agent is running, create a clean snapshot.

Recommended snapshot state:

```text
Windows 7 installed
Static IP configured
Cuckoo agent running
Security features disabled
No malware executed yet
System is clean
```

This snapshot is used by Cuckoo to restore the virtual machine after each malware analysis task.

---

## Cuckoo Sandbox Installation

### Create a Python virtual environment

```bash
virtualenv venv
source venv/bin/activate
```

### Install Cuckoo Sandbox

```bash
pip install -U pip setuptools
pip install -U cuckoo
```

### Initialize Cuckoo

```bash
cuckoo init
```

The default Cuckoo working directory will be created under:

```bash
~/.cuckoo
```

In this project, the working directory used for analysis is:

```bash
/opt/cuckoo
```

---

## Important Cuckoo Configuration Files

### `virtualbox.conf`

Configure the Windows guest VM information:

```text
VM name
Snapshot name
Network interface: vboxnet0
Result server IP: 192.168.56.1
```

Example configuration items:

```text
machines = win7x64
interface = vboxnet0
ip = 192.168.56.101
snapshot = clean
```

### `cuckoo.conf`

Configure the result server IP:

```text
ip = 192.168.56.1
```

This IP must match the Host-Only interface address.

### `reporting.conf`

Enable the required reporting modules:

```text
HTML report: enabled
MongoDB: enabled
```

MongoDB is useful for storing and querying analysis results from the Cuckoo web interface.

---

## Installation of Enhanced Report Templates

### 1. Copy report templates

Copy the customized templates into the Cuckoo web template directory:

```bash
sudo cp -r templates/analysis/pages/summary \
/usr/local/lib/python2.7/dist-packages/cuckoo/web/templates/analysis/pages/

sudo cp -r templates/analysis/pages/behavior \
/usr/local/lib/python2.7/dist-packages/cuckoo/web/templates/analysis/pages/

sudo cp -r templates/analysis/pages/network \
/usr/local/lib/python2.7/dist-packages/cuckoo/web/templates/analysis/pages/

sudo cp -r templates/analysis/pages/dropped \
/usr/local/lib/python2.7/dist-packages/cuckoo/web/templates/analysis/pages/
```

### 2. Copy the Banking Trojan signature

```bash
sudo cp signatures/windows/banking_trojan_heuristic.py \
/opt/cuckoo/signatures/windows/
```

### 3. Compile the signature

```bash
python -m py_compile /opt/cuckoo/signatures/windows/banking_trojan_heuristic.py
```

### 4. Reprocess an existing analysis task

```bash
cd /opt/cuckoo
cuckoo --cwd /opt/cuckoo process -r 7
```

### 5. Restart the Cuckoo web service

```bash
sudo systemctl restart cuckoo-web
```

or:

```bash
sudo supervisorctl restart cuckoo-web
```

---

## Running Malware Analysis

### Step 1: Start Cuckoo Core

```bash
cuckoo -d
```

### Step 2: Start the Web Interface

```bash
cuckoo web runserver 0.0.0.0:8000
```

### Step 3: Access the Web Interface

```text
http://<HOST-IP>:8000
```

Example:

```text
http://192.168.56.1:8000
```

or locally:

```text
http://127.0.0.1:8000
```

---

## Usage

After installation, open a Cuckoo analysis report:

```text
http://127.0.0.1:8000/analysis/<TASK_ID>/summary/
```

Example:

```text
http://127.0.0.1:8000/analysis/7/summary/
```

The Summary page should now include the enhanced Banking Trojan analysis sections and visual dashboard.

---

## Analysis Output

After submitting a sample, Cuckoo generates:

```text
HTML malware analysis report
Behavior logs
API call logs
Dropped files
Network traffic PCAP
DNS and host indicators
Triggered signatures
Screenshots
MongoDB analysis records
```

The analysis data is stored under:

```bash
/opt/cuckoo/storage/analyses/
```

---

## Example Analysis Focus

This project is especially useful for samples that show behaviors such as:

- Fake invoice or payment-themed executable
- Double extension file masquerading
- Dropped executable payloads
- Process injection
- Autorun persistence
- DNS or external host communication
- Anti-AV or service tampering
- Banking-related strings or behavior
- Credential or browser-related activity

---

## Security Notice

Do not upload the following Cuckoo runtime data to a public GitHub repository:

```text
/opt/cuckoo/storage/
/opt/cuckoo/storage/analyses/
/opt/cuckoo/storage/binaries/
/opt/cuckoo/cuckoo.db
/opt/cuckoo/log/
*.pcap
*.exe
*.dll
*.scr
*.bin
report.json
```

These files may contain malware samples, PCAP traffic, hashes, C2 infrastructure, database records, or other sensitive analysis data.

This repository should only contain templates, signatures, documentation, and safe configuration examples.

---

## Disclaimer

This project is intended for malware analysis research, cybersecurity education, and defensive security purposes only.

It does not distribute malware samples and should not be used to upload or share malicious binaries.

The Banking Trojan classification is heuristic-based. A specific malware family such as Zeus/Zbot, Dridex, Ursnif, Gozi, or IcedID should only be assigned when confirmed by strong evidence such as YARA matches, AV detection labels, extracted configuration, or threat intelligence.

---

## Future Improvements

- Export IOC as TXT, CSV, or JSON
- Add Sigma rule generation
- Add YARA rule draft generation
- Add Suricata/Snort alert correlation
- Add credential/browser artifact detection panel
- Add multi-sample dashboard
- Add family hypothesis scoring
- Add report export to PDF/Word
- Add dark mode report theme

---

## Author

Developed as an enhanced malware analysis reporting interface for Cuckoo Sandbox, focused on Banking Trojan and Financial Trojan behavior analysis.
