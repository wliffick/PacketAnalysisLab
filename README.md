# Packet Analysis Lab — SSH Brute-Force Detection

> SSH brute-force detection & packet analysis lab (with sanitized artifacts).

---

## Overview
This project demonstrates how to capture, analyze, sanitize, and document an SSH brute-force attack against an Ubuntu server. Artifacts include sanitized PCAPs, extracted TCP streams, verbose packet dumps, and Wireshark screenshots.

The goal was to simulate a real-world attack scenario and perform network-level and log-level analysis to identify, validate, and document a successful SSH brute-force login.

**Outcome**: This investigation idetified and isolated the exact TCP stream associated with a successful brute-force login and produced sanitized forensic artifacts.

---

## What This Project Demonstrates

- Network traffic capture with `tcpdump`
- Brute-force attack detection and profiling
- Packet-level forensic analysis with `tshark` / Wireshark
- Log and network evidence correlation
- TCP stream extraction and inspection
- Evidence sanitization for public reporting
- Mapping activity to MITRE ATT&CK techniques

---

## Lab / Environment
- **Attacker**: Kali Linux
- **Target**: AWS EC2 Ubuntu server (SSH enabled)
- **Tools**: `tcpdump`, `tshark` (Wireshark CLI), `hydra`, `ssh`, `journalctl`, `Wireshark`
- Repo layout:
  ```
  pcaps/
    stream_398_sanitized.pcap
    accepted_login_frames_44-52_sanitized.pcap
    nmap_syn_one_fixed_sanitized.pcap
    ssh_one_sanitized.pcap
    (optional) baseline_sanitized.pcap
  screenshots/
    stream_398_sanitized_filter.png
    stream_398_sanitized_packet.png
    accepted_login_frames_44-52_sanitized_filter.png
    accepted_login_frames_44-52_sanitized_packet.png
    nmap_syn_one_fixed_sanitized_filter.png
    nmap_syn_one_fixed_sanitized_packet.png
    ssh_one_sanitized_filter.png
    ssh_one_sanitized_packet.png
  dumps/
    stream_398_verbose_sanitized.txt
    accepted_login_frames_44-52_sanitized.txt
  README.md
  analysis.md
  LICENSE
  .gitignore
  ```

---

## Key Artifacts (sanitized)
- `pcaps/stream_398_sanitized.pcap` — extracted TCP stream for the successful login (headers only, anonymized).
- `pcaps/accepted_login_frames_44-52_sanitized.pcap` — minimal excerpt around the accepted login.
- `pcaps/nmap_syn_one_fixed_sanitized.pcap` — SYN scan traffic sample.
- `pcaps/ssh_one_sanitized.pcap` — small SSH session example.
- `dumps/stream_398_verbose_sanitized.txt` — verbose dump of stream 398.
- `dumps/accepted_login_frames_44-52_sanitized.txt` — verbose dump of accepted-login frames.

> **Security Note:** All artifacts are sanitized. IP addresses are anonymized, MAC addresses replaced, and payloads and credentials removed.

---

## Commands Used (for raw analysis)

```bash
# total SSH-related packets
tshark -r brute_force.pcap -Y 'tcp.port==22' | wc -l

# extract stream 398
tshark -r brute_force.pcap -Y "tcp.stream==398" -w stream_398.pcap

# extract frames 44–52 from stream
tshark -r stream_398.pcap -Y 'frame.number >= 44 && frame.number <= 52' \
  -w accepted_login_frames_44-52.pcap

# verbose dumps (later sanitized)
tshark -r stream_398.pcap -V | tee stream_398_verbose.txt
tshark -r accepted_login_frames_44-52.pcap -V | tee accepted_login_frames_44-52.txt
```

---

## Findings / Metrics (from raw analysis)
- SSH packet count: **16,572**
- Capture duration (SSH packets): **~469 seconds**
- Packets/sec: **~35**
- SYN-only attempts to port 22: **418**
- Distinct sessions: **~838**
- Top source IPs (before anonymization): Traffic consisted of attacker-to-server and server-to-attacker SSH sessions at roughly equal volume..
- Hydra recorded one valid credential (redacted in public repo).
- Logs showed one successful login event aligned with **stream 398**.

---

## Evidence & Screenshots
Each sanitized artifact is paired with screenshots:

- **Stream 398**
  - ![Stream 398 Filter](screenshots/stream_398_sanitized_filter.png)
  - ![Stream 398 Packet](screenshots/stream_398_sanitized_packet.png)

- **Accepted login frames 44–52**
  - ![Accepted Login Filter](screenshots/accepted_login_frames_44-52_sanitized_filter.png)
  - ![Accepted Login Packet](screenshots/accepted_login_frames_44-52_sanitized_packet.png)

- **Nmap SYN scan**
  - ![Nmap SYN Filter](screenshots/nmap_syn_one_fixed_sanitized_filter.png)
  - ![Nmap SYN Packet](screenshots/nmap_syn_one_fixed_sanitized_packet.png)

- **SSH session example**
  - ![SSH Filter](screenshots/ssh_one_sanitized_filter.png)
  - ![SSH Packet](screenshots/ssh_one_sanitized_packet.png)

---

### MITRE ATT&CK Techniques Observed
- T1110 – Brute Force (SSH password guessing)
- T1046 – Network Service Scanning (TCP port probes)
- T1021 – Remote Services (SSH)
- T1078 – Valid Accounts (successful credential use)
- T1573 – Encrypted Channel (SSH encrypted tunnel)

---

## Recommendations / Mitigations
- Disable SSH password authentication
- Disable root login
- Use fail2ban or rate-limiting controls
- Require strong, unique credentials and enforce MFA
- Restrict SSH access via firewall/security groups

---

## Notes
- Only sanitized pcaps are published
- Timestamps are preserved but anonymized captures may alter flow slightly.
- All testing was performed in a controlled lab environment

---

## Author
Wade Liffick — `wliffick`  


