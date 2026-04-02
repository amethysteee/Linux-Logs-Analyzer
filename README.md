# Linux-Logs-Analyzer
Python project to analyze Linux auth logs, hash log files, and detect suspicious login activity and warning clusters.
---

## Project Structure

```text
Linux-Logs-Analyzer/
├── log_analyzer.py        # Python script (main analyzer)
├── collect_logs.sh        # Bash script to create symlinks
├── README.md              # This file
├── .gitignore             # Ignore cache/logs
└── logs/                  # Folder for symlinks to .log files
```

---

## What it does

- Hashes each log file with SHA-256 (integrity check — if a log was tampered with, the hash changes)
- Detects failed SSH login attempts and flags users/IPs that exceed the threshold
- Detects **invalid user enumeration** — when an attacker probes the system with random usernames to find valid accounts before brute forcing
- Ranks IPs by number of failed attempts
- Detects warning clusters — multiple warnings within a 24h window

---

## How to run

```bash
# Step 1 — make the script executable (only needed once)
chmod +x collect_logs.sh

# Step 2 — collect logs into ~/logs using symlinks
./collect_logs.sh

# Step 3 — run the analyzer
python3 log_analyzer.py

Edit the config at the top of `log_analyzer.py` to change thresholds.

---

## Example output

```
Found 2 log file(s)

[hash] auth.log
       sha256: a3f1e9b2c7d045121f...

=======================================================
REPORT
=======================================================

-- Failed Logins --
  [ALERT] root: 47 attempt(s) from ['185.220.101.5', '45.33.32.156']
  [warn]  alice: 2 attempt(s) from ['10.0.0.4']

-- Invalid User Enumeration --
  [enum]  tried username 'admin' from ['185.220.101.5']
  [enum]  tried username 'ubuntu' from ['45.33.32.156']

-- Top IPs by Failed Attempts --
  [ALERT] 185.220.101.5: 47 attempt(s)
          45.33.32.156: 12 attempt(s)

-- Warning Clusters --
  [warn]  syslog: 5 warnings within 24h
```

---

## Log patterns detected

Tested against standard `/var/log/auth.log` format (Debian/Ubuntu):

```
Failed password for root from 1.2.3.4 port 22 ssh2
Failed password for invalid user bob from 1.2.3.4 port 22 ssh2
Invalid user admin from 203.0.113.9
```

---

## Requirements

- Python 3.8+
- Linux 
- Read access to `/var/log`

No third-party libraries needed.

---

## Things I want to add

- [ ] Detect successful logins after multiple failures (possible compromise)
- [ ] Export report to a file instead of just printing
- [ ] Support `journalctl` output format
