# Automated SSH Brute Force Attack Detection and Mitigation using Wazuh SIEM

This project demonstrates the implementation of an **Active Cyber Defense** system within a simulated corporate laboratory environment. Utilizing **Wazuh SIEM/XDR**, an automated response mechanism (**Active Response**) was configured to detect coordinated brute-force attacks in real-time and autonomously mitigate the threat by dropping network traffic from the attacker's IP address.

## 📊 Laboratory Environment Architecture
The lab environment consists of three virtual machines interconnected within a local network:
*   **Wazuh Central Manager:** The centralized server responsible for log correlation, signature analysis, and response orchestration (Ubuntu Server).
*   **Victim Machine (Agent):** A supervised Linux server running the Wazuh Agent (**IP: 192.168.100.90**).
*   **Attacker Machine:** A tactical workstation used to simulate offensive operations (**Kali Linux**).

---

## 🚀 Project Execution Phases

### Phase 1: Offensive Attack Simulation
An aggressive brute-force campaign targeting the SSH service of the victim agent was launched from the Kali Linux machine. The `Hydra` tool was combined with a custom password dictionary to generate a massive burst of failed authentication attempts:

```bash
hydra -l wilf -P pass.txt ssh://192.168.100.90 -t 4 -V
```

### Phase 2: SIEM Detection and Correlation
The Wazuh Agent collected authentication events from the victim's `/var/log/auth.log` system file in real-time and forwarded them to the Manager.

When processing the high-frequency event stream, the SIEM rules engine triggered the following rule IDs:
*   **Rule ID 5503 / 5760:** Identification of individual SSH authentication failures and PAM discrepancies.
*   **Rule ID 40111 / 2502 (High Severity - Level 10):** Multiple authentication failures correlation. The SIEM consolidated the separate events and officially declared an **Active Brute Force Attack**.

### Phase 3: Active Response Implementation (Automated Mitigation)
To transition the SIEM from a passive monitor to a proactive mitigation asset, the centralized `ossec.conf` configuration file on the Manager was modified. Logical directives were defined to orchestrate a dynamic network ban on the affected agent's local firewall when a brute-force pattern is matched:

```xml
<!-- Network mitigation executable definition -->
<command>
  <name>ufw</name>
  <executable>ufw</executable>
  <timeout_allowed>yes</timeout_allowed>
</command>

<!-- Trigger logic for specific rule matches -->
<active-response>
  <command>ufw</command>
  <location>local</location>
  <rules_id>5716,5720,5760,5503,40111,2502</rules_id>
  <timeout>180</timeout>
</active-response>
```
*Action Mechanism:* Upon matching the specified rule IDs, the Manager instructs the agent to invoke the `ufw` script, effectively isolating the aggressor's IP address for a **180-second** cooldown penalty.

---

## 🎯 Results and Mitigation Validation
Following the deployment of defensive policies, hostile activity from Kali Linux was resumed. The system delivered the following successful outcomes:

1.  **Attacker Isolation:** Within seconds of initiating the password spray, the victim's firewall successfully dropped all incoming traffic from the rogue IP, causing the Hydra console to abort operations with the following fatal network error:  
    `[ERROR] could not connect to ssh://192.168.100.90:22 - Socket error: disconnected`.
2.  **Response Audit Trails:** The automated containment execution was verified by auditing the agent's internal log file (`active-responses.log`), capturing the precise `add` JSON command executed by the SIEM core to block the adversary.

## 💡 Demonstrated Skills
*   Installation, deployment, and maintenance of SIEM/XDR architectures (Wazuh).
*   Advanced configuration and debugging of XML rules and active response policies (`ossec.conf`).
*   Technical analysis, parsing, and log ingestion of Linux system events (`auth.log`).
*   Threat simulation using offensive security auditing tools (`Hydra`).
*   Defensive hardening via firewall automation scripts (`ufw` / `iptables`).
