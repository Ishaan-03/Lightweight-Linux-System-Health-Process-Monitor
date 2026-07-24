# 🛡️ Lightweight Linux System Health & Process Monitor

A lightweight, automated Bash-based system monitoring tool designed to track system resource usage, issue real-time threshold alerts, and manage log rotation automatically using the Linux `cron` daemon.

---

## 🎯 Overview

In cloud environments and Linux servers, unmonitored memory leaks or unexpected process spikes can lead to server crashes and downtime , also if a hacker hijacks your linux machine/server running on cloud he might run scripts on your machine maybe to mine crypto in that case your CPU usage will shoot up . This project provides an automated background monitoring solution that periodically tracks memory metrics, appends timestamped records, triggers warning alerts when usage exceeds defined safety limits, and automatically prevents disk space exhaustion via log rotation.

---

## ✨ Features

* ⏱️ **Automated Execution:** Runs background checks at custom intervals using `cron`.
* 📊 **Resource Tracking:** Captures memory usage percentages alongside precise timestamps.
* ⚠️ **Threshold Alerting:** Triggers explicit warning flags when memory usage passes **80%**.
* 🧹 **Automated Log Rotation:** Automatically truncates logs once they exceed 100 lines to protect server disk space.

---

## 📐 Execution Flow

```text
┌──────────────┐     Executes every 2 min     ┌────────────────────────┐
│  Cron Daemon │ ───────────────────────────> │  log_processes.sh      │
└──────────────┘                              └───────────┬────────────┘
                                                          │
                                     ┌────────────────────┴────────────────────┐
                                     ▼                                         ▼
                         ┌───────────────────────┐                 ┌───────────────────────┐
                         │ 1. Check Log Size     │                 │ 2. Check Memory %     │
                         │    (Rotate if >100)   │                 │    (via `free` + `awk`)│
                         └───────────┬───────────┘                 └───────────┬───────────┘
                                     │                                         │
                                     └────────────────────┬────────────────────┘
                                                          ▼
                                            ┌───────────────────────────┐
                                            │ Write to `process_log.txt`│
                                            └───────────────────────────┘`


```
Create the file directory :
```
mkdir security_monitor
```
then :- touch log_processes.sh  

then :- nano log_processes.sh  

paste the script 

🛠️ Script Overview (log_processes.sh)

#!/bin/bash

LOG_FILE="process_log.txt"
MAX_LINES=100
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# Create the log file if it doesn't exist
touch "$LOG_FILE"

# 1. Check Log Size (Log Rotation)
if [ $(wc -l < "$LOG_FILE") -gt $MAX_LINES ]; then
    echo "[$TIMESTAMP] 🧹 Log limit reached ($MAX_LINES lines). Truncating log." > "$LOG_FILE"
fi

# 2. Capture Memory Metrics
MEM_USAGE=$(free | awk '/Mem:/ {printf("%.0f"), $3/$2 * 100}')

# 3. Log Metrics
echo "[$TIMESTAMP] Memory Usage: ${MEM_USAGE}%" >> "$LOG_FILE"

# 4. Threshold Alert
if [ "$MEM_USAGE" -gt 80 ]; then
    echo "[$TIMESTAMP] ⚠️ WARNING: High memory usage at ${MEM_USAGE}%!" >> "$LOG_FILE"
fi


Configure Cron Automation
``` 
crontab -e
```
Add the following entry to execute the script every 2 minutes:
```
*/2 * * * * /home/YOUR_USERNAME/security_monitor/log_processes.sh
```
<img width="860" height="664" alt="Screenshot from 2026-07-24 16-24-14" src="https://github.com/user-attachments/assets/fe91073a-ed87-4595-980b-80d104194a60" />

📜 Sample Log Output (process_log.txt)
```
[2026-07-24 15:59:49] Memory Usage: 66%
[2026-07-24 16:00:36] Memory Usage: 58%
[2026-07-24 16:02:00] Memory Usage: 84%
[2026-07-24 16:02:00] ⚠️ WARNING: High memory usage at 84%!
[2026-07-24 16:04:00] 🧹 Log limit reached (100 lines). Truncating log.

```

