# 🔎 Azure Network Troubleshooting Report

## 🖥️ Web Server Configuration

`WEB-SERVER-01` was deployed using **Windows Server 2022 Datacenter** and connected to the `Web-Subnet`.

### Server Configuration

| Configuration    | Value                          |
| ---------------- | ------------------------------ |
| VM Name          | `WEB-SERVER-01`                |
| Operating System | Windows Server 2022 Datacenter |
| Region           | West Central US                |
| Virtual Network  | `vnet-network-troubleshooting` |
| Subnet           | `Web-Subnet`                   |
| Subnet CIDR      | `10.10.1.0/24`                 |
| Web Service      | IIS                            |
| Web Protocol     | HTTP                           |
| Web Port         | TCP `80`                       |

IIS was installed using PowerShell:

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

The IIS service was then verified:

```powershell
Get-Service W3SVC
```

The service was confirmed to be running.

The local IIS website was tested using:

```text
http://localhost
```

A successful response confirmed that the IIS web server was functioning before network troubleshooting began.

### Evidence

![IIS Installed](screenshots/09-iis-installed.png)

![IIS Local Test](screenshots/10-iis-local-test-success.png)

---

# 🧪 Baseline Connectivity Test

Before introducing the simulated network fault, connectivity between `ADMIN-VM` and `WEB-SERVER-01` was tested.

The purpose of the baseline test was to establish that the network and web service were functioning correctly before deliberately changing the NSG configuration.

From `ADMIN-VM`, TCP port `80` was tested using:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

The expected successful result was:

```text
TcpTestSucceeded : True
```

An HTTP request was also performed:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

The successful response confirmed that `ADMIN-VM` could communicate with the IIS web server.

### Baseline Result

```text
ADMIN-VM
    |
    | TCP 80
    | SUCCESS
    v
WEB-SERVER-01
    |
    v
IIS
```

### Evidence

![Initial Connectivity Success](screenshots/16-initial-connectivity-success.png)

---

# 🚨 Simulated Network Incident

To simulate a realistic Azure network security incident, an intentional NSG misconfiguration was introduced.

A deny rule was created on the web server's Network Security Group:

```text
Rule Name:       Deny-HTTP-Troubleshooting
Priority:        100
Protocol:        TCP
Destination:     Port 80
Action:          Deny
```

An existing HTTP allow rule was configured with:

```text
Rule Name:       Allow-HTTP-Testing
Priority:        200
Protocol:        TCP
Destination:     Port 80
Action:          Allow
```

Because the deny rule had the lower numerical priority, it was evaluated first.

```text
Priority 100 → DENY
Priority 200 → ALLOW
```

The deny rule therefore prevented the HTTP traffic from reaching the IIS web server.

### Result

The previously successful connectivity test was repeated from `ADMIN-VM`.

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

The result changed to:

```text
TcpTestSucceeded : False
```

The HTTP request also failed:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

This successfully reproduced the simulated network incident.

### Evidence

![HTTP Deny Rule Created](screenshots/19-nsg-http-deny-rule-created.png)

![Connectivity Failure](screenshots/20-connectivity-failure.png)

---

# 🔎 Troubleshooting Process

The incident was investigated systematically rather than immediately changing the configuration.

The troubleshooting approach followed a layered methodology:

```text
Application
    ↓
Service
    ↓
Operating System
    ↓
Local Connectivity
    ↓
Remote Connectivity
    ↓
Network Security Group
    ↓
Effective Security Rules
    ↓
Azure Network Watcher
```

---

## 1. Verify Web Server Availability

The Azure portal was checked to confirm that `WEB-SERVER-01` was still running and available.

The VM was found to be operational.

### Finding

The virtual machine itself was not stopped or unavailable.

### Evidence

![Web Server Health Check](screenshots/21-web-server-health-check.png)

---

## 2. Verify IIS Service

The IIS service was checked from `WEB-SERVER-01`:

```powershell
Get-Service W3SVC
```

The service was confirmed to be running.

### Finding

```text
IIS / W3SVC → Running
```

This indicated that the web application was still operational.

---

## 3. Test Local Connectivity

A local TCP connectivity test was performed directly on `WEB-SERVER-01`:

```powershell
Test-NetConnection localhost -Port 80
```

The test succeeded.

### Finding

```text
WEB-SERVER-01 → localhost:80
SUCCESS
```

This confirmed that IIS was listening on TCP port `80`.

### Evidence

![Local IIS Connectivity](screenshots/22-local-iis-connectivity-success.png)

---

## 4. Test Remote Connectivity

The same port was tested remotely from `ADMIN-VM`:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

The connection failed.

### Finding

```text
ADMIN-VM → WEB-SERVER-01:80
FAILURE
```

The difference between the successful local test and failed remote test indicated that the issue was likely related to network access rather than IIS itself.

---

## 5. Analyse Network Security Group Rules

The Network Security Group associated with `WEB-SERVER-01` was reviewed.

The following rules were identified:

### Deny Rule

```text
Priority: 100
Name:     Deny-HTTP-Troubleshooting
Protocol: TCP
Port:     80
Action:   DENY
```

### Allow Rule

```text
Priority: 200
Name:     Allow-HTTP-Testing
Protocol: TCP
Port:     80
Action:   ALLOW
```

The deny rule had the higher evaluation priority because it had the lower numerical value.

Therefore:

```text
100 → DENY
200 → ALLOW
```

The deny rule was blocking the HTTP traffic before the allow rule could take effect.

### Evidence

![NSG HTTP Deny Rule](screenshots/19-nsg-http-deny-rule-created.png)

---

## 6. Review Effective Security Rules

The effective security rules applied to the web server's network interface were reviewed.

The results confirmed that inbound TCP port `80` traffic was being denied.

### Finding

```text
Source:
ADMIN-VM / Admin-Subnet

Destination:
WEB-SERVER-01

Protocol:
TCP

Destination Port:
80

Effective Result:
DENY
```

This provided evidence that the NSG configuration was actively blocking the connection.

### Evidence

![Effective Security Rules](screenshots/23-effective-security-rules-deny-http.png)

---

## 7. Use Azure Network Watcher

Azure Network Watcher Connection Troubleshoot was used to independently test the connection.

### Test Configuration

```text
Source:
ADMIN-VM

Destination:
WEB-SERVER-01

Protocol:
TCP

Destination Port:
80
```

The diagnostic returned a failed connection.

This provided Azure-side confirmation that the network path was being blocked.

### Evidence

![Network Watcher HTTP Failure](screenshots/24-network-watcher-http-failure.png)

---

# 🧠 Root Cause

The root cause of the connectivity failure was an incorrectly prioritised **Azure Network Security Group deny rule**.

The faulty rule was:

```text
Deny-HTTP-Troubleshooting
Priority: 100
Protocol: TCP
Port: 80
Action: DENY
```

The existing allow rule was:

```text
Allow-HTTP-Testing
Priority: 200
Protocol: TCP
Port: 80
Action: ALLOW
```

Because Azure evaluates NSG rules according to priority, the rule with priority `100` was evaluated before the rule with priority `200`.

Therefore, the HTTP traffic was denied.

### Root Cause Summary

```text
ADMIN-VM
    |
    | TCP 80
    v
nsg-web-server
    |
    +-- Priority 100 → DENY ❌
    |
    +-- Priority 200 → ALLOW
    |
    X
WEB-SERVER-01
```

---

## Why IIS Was Not the Root Cause

The investigation showed that IIS itself was functioning correctly.

| Test            | Result            |
| --------------- | ----------------- |
| Web Server VM   | Running           |
| IIS Service     | Running           |
| Local TCP 80    | Successful        |
| Remote TCP 80   | Failed            |
| Effective NSG   | TCP 80 Denied     |
| Network Watcher | Connection Failed |

The successful local test proved that the web service was available on the server.

The failed remote test, combined with the effective NSG rule and Network Watcher results, demonstrated that the failure occurred at the network security layer.

### Conclusion

The incident was caused by **NSG configuration**, not by IIS or the Windows Server operating system.

---

# 🔧 Remediation

The incorrectly configured deny rule was removed from:

```text
nsg-web-server
```

Removed rule:

```text
Rule Name:       Deny-HTTP-Troubleshooting
Priority:        100
Protocol:        TCP
Destination:     Port 80
Action:          Deny
```

The valid HTTP allow rule was retained.

After the incorrect rule was removed, the network configuration was tested again.

### Evidence

![Corrected NSG](screenshots/25-nsg-corrected.png)

---

# 🔒 Least-Privilege Improvement

Although removing the deny rule restored connectivity, the final configuration was reviewed from a security perspective.

Rather than allowing HTTP traffic from unnecessarily broad sources, access was restricted to the administration subnet.

### Final HTTP Rule

```text
Rule Name:
Allow-HTTP-AdminSubnet

Source:
10.10.2.0/24

Protocol:
TCP

Destination Port:
80

Action:
Allow
```

The `10.10.2.0/24` address range corresponds to:

```text
Admin-Subnet
```

This configuration allows the administration VM to access the IIS web server while limiting HTTP access to the intended internal network.

### Security Principle

This follows the **principle of least privilege** by allowing only the network traffic required for the lab's intended purpose.

Instead of broadly allowing HTTP traffic, the rule restricts the permitted source to the administration subnet.

---

# ✅ Validation After Remediation

After the NSG configuration was corrected, connectivity was tested again from `ADMIN-VM`.

## 1. TCP Connectivity Test

The following command was executed:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

The result was:

```text
TcpTestSucceeded : True
```

This confirmed that TCP port `80` was reachable again.

### Evidence

![Connectivity Restored](screenshots/26-connectivity-restored.png)

---

## 2. HTTP Connectivity Test

An HTTP request was performed:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

The IIS web server successfully returned an HTTP response.

This confirmed that application-level connectivity had also been restored.

### Evidence

![Final HTTP Test](screenshots/28-final-http-test-success.png)

---

## 3. Azure Network Watcher Validation

Azure Network Watcher Connection Troubleshoot was executed again using the same source, destination, protocol and port.

```text
Source:
ADMIN-VM

Destination:
WEB-SERVER-01

Protocol:
TCP

Port:
80
```

The connection test was successful.

### Evidence

![Network Watcher Success](screenshots/27-network-watcher-success.png)

---

# 📊 Troubleshooting Results

| Test               | Baseline     | During Incident | After Remediation |
| ------------------ | ------------ | --------------- | ----------------- |
| Web Server Running | ✅            | ✅               | ✅                 |
| IIS Service        | ✅            | ✅               | ✅                 |
| Local TCP 80       | ✅            | ✅               | ✅                 |
| Remote TCP 80      | ✅            | ❌               | ✅                 |
| HTTP Request       | ✅            | ❌               | ✅                 |
| Effective NSG      | Allow        | ❌ Deny          | ✅ Allow           |
| Network Watcher    | ✅ Successful | ❌ Failed        | ✅ Successful      |

---

## Incident Timeline

```text
1. Azure infrastructure deployed
        ↓
2. IIS installed
        ↓
3. Local IIS test successful
        ↓
4. ADMIN-VM → WEB-SERVER-01 TCP 80 successful
        ↓
5. NSG deny rule intentionally created
        ↓
6. Remote TCP 80 test failed
        ↓
7. IIS service verified as running
        ↓
8. Local TCP 80 test remained successful
        ↓
9. NSG rules analysed
        ↓
10. Effective security rules confirmed DENY
        ↓
11. Network Watcher confirmed failure
        ↓
12. Root cause identified
        ↓
13. Incorrect deny rule removed
        ↓
14. TCP 80 connectivity restored
        ↓
15. HTTP request successful
        ↓
16. Network Watcher successful
        ↓
17. Least-privilege rule implemented
        ↓
18. Incident resolved
```

---

# 🏁 Final Outcome

The Azure network connectivity incident was successfully diagnosed and resolved.

The investigation demonstrated that:

* The Windows Server VM remained operational.
* IIS remained operational.
* Local TCP port `80` connectivity remained successful.
* Remote TCP port `80` connectivity failed.
* The effective NSG rules showed that HTTP traffic was being denied.
* Azure Network Watcher confirmed the network failure.
* The higher-priority NSG deny rule was identified as the root cause.
* Removing the incorrect rule restored connectivity.
* Final HTTP testing confirmed application-level connectivity.
* The final configuration was improved using a least-privilege source restriction.

### Final State

```text
ADMIN-VM
10.10.2.0/24
      |
      | TCP 80
      | ALLOW
      v
nsg-web-server
      |
      v
WEB-SERVER-01
10.10.1.0/24
      |
      v
IIS
      |
      v
HTTP 200 / Successful Response
```

**Incident Status: ✅ Resolved**

**Root Cause: NSG priority/configuration**

**Service: IIS / HTTP**

**Affected Port: TCP 80**

**Final Security Model: Least-privilege internal access**
