# Task 4 — Setup and Use a Firewall on Windows/Linux

## Overview
This hands-on project explores how a firewall filters network traffic. I used Windows Defender Firewall with PowerShell to inspect firewall profiles and existing rules, create and verify a temporary inbound rule blocking TCP port 23 (Telnet), test connectivity, and remove the test rule afterward. Linux UFW commands are included as a reference.

## Objective
Configure and inspect basic firewall rules to understand how incoming and outgoing network traffic can be allowed or blocked.

## Environment
- Operating system: Windows (PowerShell running as Administrator)
- Firewall: Windows Defender Firewall
- Test protocol: TCP
- Test port: 23 (Telnet)
- Linux reference: UFW (Uncomplicated Firewall)

## What I did

### 1. Checked firewall profiles
```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```
**Observed result:** Domain, Private, and Public profiles all showed `Enabled = True`. The default inbound and outbound action fields displayed `NotConfigured` in this command output.

### 2. Listed existing firewall rules
```powershell
Get-NetFirewallRule -PolicyStore ActiveStore |
    Select-Object -First 20 DisplayName, Direction, Action, Enabled
```
**Observed result:** The command displayed the first 20 rules from the ActiveStore. The output included both enabled and disabled rules, with inbound/outbound directions and allow actions.

### 3. Created an inbound block rule for Telnet
```powershell
New-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound" -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block -Profile Any
```
**Observed result:** Windows returned the newly created rule. The output showed `Enabled = True`, `Direction = Inbound`, `Action = Block`, and `Profile = Any`.

### 4. Verified the rule's port filter
```powershell
Get-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound" |
    Get-NetFirewallPortFilter
```
**Observed result:** `Protocol = TCP`, `LocalPort = 23`, `RemotePort = Any`, `IcmpType = Any`, and `DynamicTarget = Any`.

### 5. Checked for a listener and tested connectivity
```powershell
Get-NetTCPConnection -LocalPort 23 -State Listen -ErrorAction SilentlyContinue
Test-NetConnection -ComputerName 127.0.0.1 -Port 23
```
**Observed result:** The listener check returned no output, so no TCP listener was found on local port 23 at the time of testing. `Test-NetConnection` reported `TcpTestSucceeded = False` and warned that the TCP connection to `127.0.0.1:23` failed.

**Important limitation:** This failed connection does not prove that the firewall blocked the traffic. Because no service was listening on port 23, the connection could fail even without the firewall rule. Also, a loopback test to `127.0.0.1` is not a reliable demonstration of inbound firewall enforcement. A controlled listener on a separate test machine and a connection attempt from another device would be needed to demonstrate blocking conclusively.

### 6. Removed the temporary test rule
```powershell
Remove-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound"
Get-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound" -ErrorAction SilentlyContinue
```
**Observed result:** The removal command completed, and the subsequent lookup returned no output, indicating that the named test rule was no longer present.

## Results summary
| Check | Actual result | Status / interpretation |
|---|---|---|
| Firewall profiles | Domain, Private, Public: Enabled = True | Verified |
| Existing rules | First 20 ActiveStore rules displayed | Completed |
| Telnet block rule | Created; Enabled=True, Inbound, Block, Profile=Any | Verified |
| Port filter | TCP; LocalPort=23; RemotePort=Any | Verified |
| Port 23 listener | No output from listener check | No listener found |
| Connectivity test | TcpTestSucceeded=False | Connection failed; firewall enforcement not proven |
| Rule removal | Follow-up lookup returned no output | Verified removed |

## Screenshots
The `screenshots/` folder contains the evidence captured during the practical:
1. `01-firewall-profiles.png` — Firewall profile status.
2. `02-existing-rules.png` — Existing firewall rules.
3. `03-telnet-block-created.png` — Successful creation of the inbound TCP/23 block rule.
4. `04-rule-port-filter.png` — Verified port-filter settings.
5. `05-connectivity-test.png` — Listener check and connection-test output.
6. `06-rule-removed.png` — Removal and follow-up verification.

## Linux UFW reference
These commands are for a Linux system with UFW installed; they were not part of the Windows screenshots in this project.

```bash
sudo ufw status verbose
sudo ufw deny 23/tcp
sudo ufw status numbered
sudo ufw allow 22/tcp
sudo ufw delete deny 23/tcp
```

**Caution:** Before changing UFW on a remote Linux machine, confirm SSH access and any cloud/provider firewall settings so you do not lock yourself out.

## Key concepts learned
- A firewall applies rules to control network traffic.
- Inbound rules govern traffic arriving at a device; outbound rules govern traffic leaving it.
- TCP port 23 is traditionally associated with Telnet, which does not encrypt normal session traffic.
- A rule's direction, protocol, local port, profile, and action all matter.
- A failed connection test is not, by itself, proof that a firewall blocked the connection; the destination service must also be listening.
- Temporary test rules should be removed after the practical to restore the prior configuration.


## Conclusion
I inspected the Windows firewall configuration, created and verified a temporary inbound TCP port 23 block rule, performed a connectivity test, and removed the rule afterward. The rule configuration and cleanup were verified. Since no service was listening on port 23, the failed connection test cannot independently establish that the firewall blocked the connection.
