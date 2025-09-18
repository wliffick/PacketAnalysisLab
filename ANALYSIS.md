# Network Traffic Analysis Lab — Analysis

## Environment
- Attacker: Kali Linux (local VM)
- Target: Ubuntu Server (AWS EC2)
- Captures collected on target using `tcpdump`; analysis with `tshark`/`Wireshark`.

---

## Nmap SYN scan (`pcaps/nmap_syn_one_fixed_sanitized.pcap`)
- **Filter:** `tcp.flags.syn == 1 && tcp.flags.ack == 0`  
- **Observed metrics (from raw analysis):**
  - SYN packets: **994**
  - Duration: **~3.16 s**
  - Rate: **~315 SYNs/sec**
  - Example ports targeted: `65`, `443`, `999`, `998`, `997`, ...
- **Behavior:** Rapid SYNs across many destination ports; responses include `SYN/ACK` (open), `RST` (closed) or no reply (filtered).
- **MITRE ATT&CK:** T1046 — Network Service Scanning

**Evidence:**
- ![Nmap SYN Filter](screenshots/nmap_syn_one_fixed_sanitized_filter.png)
- ![Nmap SYN Packet](screenshots/nmap_syn_one_fixed_sanitized_packet.png)
- See also `dumps/nmap_syn_one_fixed_sanitized.txt` (optional)

---

## SSH brute-force (`pcaps/stream_398_sanitized.pcap` & `pcaps/accepted_login_frames_44-52_sanitized.pcap`)
- **Filter:** `tcp.port == 22`
- **Observed metrics (from raw analysis):**
  - Total SSH packets: **16,572**
  - Duration: **~468.8 s**
  - Packets/sec: **~35.35**
  - SYN-only attempts to port 22: **418**
  - Distinct TCP sessions (approx): **838**
- **Behavior:** Repeated connection attempts to port 22; multiple short sessions and many failed auths; one stream (398) shows a successful login.
- **MITRE ATT&CK:** T1110 — Brute Force

**Evidence:**
- Stream 398 (successful login)
  - ![Stream 398 Filter](screenshots/stream_398_sanitized_filter.png)
  - ![Stream 398 Packet](screenshots/stream_398_sanitized_packet.png)
  - `dumps/stream_398_verbose_sanitized.txt`

- Accepted login frames 44–52 (minimal excerpt)
  - ![Accepted Login Filter](screenshots/accepted_login_frames_44-52_sanitized_filter.png)
  - ![Accepted Login Packet](screenshots/accepted_login_frames_44-52_sanitized_packet.png)
  - `dumps/accepted_login_frames_44-52_sanitized.txt`

---

## SSH session example (`pcaps/ssh_one_sanitized.pcap`)
- **Filter:** `tcp.port == 22`
- **Behavior:** Small reference capture of a normal SSH handshake.

**Evidence:**
- ![SSH Filter](screenshots/ssh_one_sanitized_filter.png)
- ![SSH Packet](screenshots/ssh_one_sanitized_packet.png)

---

## Detection & Recommendations
- Alert rule suggestions:
  - Spike detection: alert if >100 SYNs/min from same IP to many dst ports (example threshold: >100 SYNs/min).
  - SSH auth failures: alert when >N failed auths for same account from single IP within X minutes.
- Correlate network detection with host logs (`/var/log/auth.log`) to confirm successful logins and lockouts.
- Re-disable password authentication and remove test user after testing.

---

## Notes
- All PCAPs published here are **sanitized**: payloads truncated, IPs remapped to `10.0.0.x`, MACs overwritten.
- Raw captures remain private.
- This project is for authorized lab/demo purposes only.

