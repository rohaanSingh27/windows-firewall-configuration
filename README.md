# Task 4 — Windows Firewall Configuration and Traffic Filtering

## Objective
Configure, inspect, test, and remove a temporary Windows Defender Firewall rule that blocks inbound TCP traffic on port 23 (Telnet).

## Environment
- OS: Windows 10/11
- Tool: Windows Defender Firewall with Advanced Security / PowerShell (Administrator)
- Test port: TCP 23

## Safety
Run commands on your own Windows machine. Do not disable the firewall. Do not change default policies or delete unrelated rules. A local `Test-NetConnection` failure by itself does not prove a firewall rule worked: the port might have no listening service.

## Procedure
### 1. Inspect existing firewall state
```powershell
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
Get-NetFirewallRule -PolicyStore ActiveStore | Select-Object -First 20 DisplayName, Direction, Action, Enabled
```

### 2. Check whether Telnet is listening
```powershell
Get-NetTCPConnection -LocalPort 23 -State Listen -ErrorAction SilentlyContinue
```

### 3. Create a temporary inbound block rule
```powershell
New-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound" -Direction Inbound -Protocol TCP -LocalPort 23 -Action Block -Profile Any
```

### 4. Verify the rule and its port filter
```powershell
Get-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound" | Format-List DisplayName, Enabled, Direction, Action, Profile
Get-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound" | Get-NetFirewallPortFilter | Format-List Protocol, LocalPort
```

### 5. Test and interpret results
```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 23
```
A failed test is **not** definitive evidence of firewall enforcement when nothing is listening on port 23. For meaningful verification, use a controlled test listener on a separate machine and test connectivity from another device, comparing results with the rule disabled/enabled. Loopback tests may not reflect inbound firewall enforcement. Record whether a listener exists and distinguish a rule configuration check from a verified blocked connection.

### 6. Remove only the rule created for this task
```powershell
Remove-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound"
Get-NetFirewallRule -DisplayName "Task4-Block-Telnet-Inbound" -ErrorAction SilentlyContinue
```

## Linux UFW equivalent (reference only)
```bash
sudo ufw status verbose
sudo ufw deny 23/tcp
sudo ufw status numbered
# Only if SSH access is actually needed: sudo ufw allow 22/tcp
sudo ufw delete deny 23/tcp
```
**Remote Linux warning:** Check SSH access and provider firewall rules before enabling or modifying UFW to avoid locking yourself out.

## Screenshots
Add your own screenshots to `screenshots/`:
1. `01-firewall-profiles.png`
2. `02-existing-rules.png`
3. `03-telnet-block-created.png`
4. `04-rule-port-filter.png`
5. `05-connectivity-test.png`
6. `06-rule-removed.png`

## Observations
| Check | Result | Interpretation |
|---|---|---|
| Firewall enabled | Pending | Record profile state |
| Port 23 listener present | Pending | Important test prerequisite |
| Block rule created | Pending | Verify Action=Block, Direction=Inbound, TCP/23 |
| Connection test | Pending | Distinguish refusal/no listener from filtering |
| Rule removed | Pending | Restore prior state |


## Conclusion
A temporary inbound TCP/23 block rule can be created, inspected and removed without changing the system's default firewall policy. Actual enforcement should be validated with an appropriate controlled network test.
