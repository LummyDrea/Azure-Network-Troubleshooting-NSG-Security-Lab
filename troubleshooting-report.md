# Azure Network Troubleshooting & NSG Security Lab — Technical Troubleshooting Report

## 1. Incident Overview

### Incident Title

**Internal IIS Web Server Connectivity Failure Caused by Azure Network Security Group Misconfiguration**

### Environment

| Component          | Configuration                    |
| ------------------ | -------------------------------- |
| Cloud Platform     | Microsoft Azure                  |
| Resource Group     | `rg-network-troubleshooting-lab` |
| Region             | West Central US                  |
| Virtual Network    | `vnet-network-troubleshooting`   |
| VNet Address Space | `10.10.0.0/16`                   |
| Web Subnet         | `Web-Subnet` — `10.10.1.0/24`    |
| Admin Subnet       | `Admin-Subnet` — `10.10.2.0/24`  |
| Web Server         | `WEB-SERVER-01`                  |
| Administration VM  | `ADMIN-VM`                       |
| Web Server OS      | Windows Server 2022 Datacenter   |
| Web Service        | IIS                              |
| Web Protocol       | HTTP                             |
| Web Port           | TCP 80                           |
| Web Server NSG     | `nsg-web-server`                 |
| Diagnostic Tool    | Azure Network Watcher            |

---

# 2. Executive Summary

This lab simulated a real-world Azure network connectivity incident involving an internal IIS web server.

Two Windows Server virtual machines were deployed into separate Azure subnets:

* `WEB-SERVER-01` was deployed into `Web-Subnet`.
* `ADMIN-VM` was deployed into `Admin-Subnet`.

IIS was installed and verified on `WEB-SERVER-01`. Baseline connectivity testing confirmed that `ADMIN-VM` could successfully reach the IIS web server over TCP port `80`.

A deliberate Network Security Group misconfiguration was then introduced by creating a higher-priority deny rule for TCP port `80`.

Following the change, the administration VM could no longer reach the web server.

The investigation followed a layered troubleshooting approach:

1. Verified VM availability.
2. Verified the IIS service.
3. Tested local connectivity on the web server.
4. Tested remote connectivity from the administration VM.
5. Analysed NSG rules.
6. Reviewed effective security rules.
7. Used Azure Network Watcher Connection Troubleshoot.
8. Identified the higher-priority NSG deny rule as the root cause.
9. Removed the incorrect deny rule.
10. Re-tested connectivity.
11. Confirmed successful HTTP communication.
12. Improved the final configuration using a least-privilege source restriction.

---

# 3. Lab Architecture

```text
                         MICROSOFT AZURE
                                |
                                |
                  rg-network-troubleshooting-lab
                                |
                                |
                 vnet-network-troubleshooting
                         10.10.0.0/16
                                |
                 +--------------+--------------+
                 |                             |
                 |                             |
           Web-Subnet                     Admin-Subnet
           10.10.1.0/24                  10.10.2.0/24
                 |                             |
                 |                             |
                 v                             v
          WEB-SERVER-01                    ADMIN-VM
          Windows Server                  Windows Server
                 |                             |
                 |                             |
                 v                             |
                IIS <--------- TCP 80 ----------+
                 |
                 |
        Network Security Group
          nsg-web-server
                 |
                 |
        Azure Network Watcher
```

### Architecture Diagram

![Azure Network Architecture](./architecture/azure-network-architecture.png)

---

# 4. Network Addressing Plan

## Virtual Network

```text
Name:          vnet-network-troubleshooting
Address Space: 10.10.0.0/16
```

## Web Subnet

```text
Name:          Web-Subnet
Address Range: 10.10.1.0/24
Purpose:       Internal web server
```

## Admin Subnet

```text
Name:          Admin-Subnet
Address Range: 10.10.2.0/24
Purpose:       Administration and troubleshooting
```

The actual private IP addresses assigned to the virtual machines were obtained from Azure after deployment.

---

# 5. Infrastructure Deployment

## 5.1 Resource Group Creation

The Azure resources were organised within:

```text
rg-network-troubleshooting-lab
```

The resource group was created to contain the networking, virtual machine and security resources used throughout the lab.

### Evidence


**Screenshot:** `01-resource-group-created.png`

---

## 5.2 Virtual Network and Subnet Configuration

The following Virtual Network was created:

```text
vnet-network-troubleshooting
```

with the address space:

```text
10.10.0.0/16
```

Two subnets were created:

### Web Subnet

```text
Web-Subnet
10.10.1.0/24
```

### Admin Subnet

```text
Admin-Subnet
10.10.2.0/24
```

The subnet separation was intentional to demonstrate basic network segmentation.

### Evidence



**Screenshot:** `02-vnet-subnets-created.png`

---

## 5.3 VM Size Availability Check

Before deploying the virtual machines, available VM sizes were reviewed for the selected Azure region.

The selected region was:

```text
West Central US
```

This check was important because VM SKU availability can vary by region and subscription.

### Evidence



**Screenshot:** `03-available-vm-sizes-west-central-us.png`

---

# 6. WEB-SERVER-01 Deployment

The first virtual machine was deployed as:

```text
WEB-SERVER-01
```

Configuration:

```text
Operating System:
Windows Server 2022 Datacenter

Region:
West Central US

Virtual Network:
vnet-network-troubleshooting

Subnet:
Web-Subnet
```

The VM was associated with:

```text
nsg-web-server
```

RDP access was enabled for administration and configuration.

### Final VM Configuration



**Screenshot:** `04-web-server-final-configuration.png`

### Deployment Success



**Screenshot:** `05-web-server-deployment-success.png`

### VM Overview



**Screenshot:** `06-web-server-overview.png`

---

# 7. WEB-SERVER-01 RDP and Network Configuration

After deployment, Remote Desktop Protocol (RDP) was used to connect to the Windows Server.

### Evidence



**Screenshot:** `07-rdp-successful.png`

The server's network configuration was then checked using PowerShell.

```powershell
Get-NetIPConfiguration
```

or:

```powershell
ipconfig
```

The purpose was to identify:

* IPv4 address
* Subnet information
* Default gateway
* DNS configuration
* Network adapter status

### Evidence



**Screenshot:** `08-web-server-network-config.png`

---

# 8. IIS Installation and Local Validation

## 8.1 IIS Installation

IIS was installed on `WEB-SERVER-01` using PowerShell:

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

The Web Server role was then verified:

```powershell
Get-WindowsFeature Web-Server
```

### Evidence



**Screenshot:** `09-iis-installed.png`

---

## 8.2 IIS Local Connectivity Test

Before testing connectivity from another VM, IIS was tested locally.

```text
http://localhost
```

A successful IIS response confirmed that the web service was functioning locally.

This established an important troubleshooting baseline before the network fault was introduced.

### Evidence



**Screenshot:** `10-iis-local-test-success.png`

---

# 9. ADMIN-VM Deployment

A second Windows Server virtual machine was deployed as:

```text
ADMIN-VM
```

The VM was connected to:

```text
vnet-network-troubleshooting
```

and deployed into:

```text
Admin-Subnet
10.10.2.0/24
```

The purpose of this VM was to simulate an administrator workstation from which connectivity to the internal web server could be tested.

### Final VM Configuration



**Screenshot:** `11-admin-vm-final-configuration.png`

### Deployment Success



**Screenshot:** `12-admin-vm-deployment-success.png`

### VM Overview



**Screenshot:** `13-admin-vm-overview.png`

---

# 10. ADMIN-VM RDP and Network Configuration

RDP was used to connect to the administration VM.

### Evidence



**Screenshot:** `14-admin-vm-rdp-successful.png`

The administration VM network configuration was checked using:

```powershell
Get-NetIPConfiguration
```

or:

```powershell
ipconfig
```

The private IP address was recorded for use during connectivity testing.

### Evidence



**Screenshot:** `15-admin-vm-network-configuration.png`

---

# 11. Baseline Connectivity

Before introducing the simulated fault, connectivity from `ADMIN-VM` to `WEB-SERVER-01` was tested.

The private IP address of `WEB-SERVER-01` was used.

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

Expected result:

```text
TcpTestSucceeded : True
```

An HTTP request was also tested:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

The successful tests established the baseline state.

### Evidence



**Screenshot:** `16-initial-connectivity-success.png`

---

# 12. Initial HTTP NSG Allow Rule

An inbound HTTP rule was configured on:

```text
nsg-web-server
```

Initial testing rule:

| Setting          | Value                |
| ---------------- | -------------------- |
| Name             | `Allow-HTTP-Testing` |
| Priority         | `200`                |
| Protocol         | TCP                  |
| Destination Port | `80`                 |
| Action           | Allow                |

The rule permitted HTTP traffic to the IIS web server.

### Evidence



**Screenshot:** `17-http-allow-nsg-rule.png`

---

# 13. IIS Service Validation

The IIS World Wide Web Publishing Service was checked using:

```powershell
Get-Service W3SVC
```

Expected status:

```text
Status : Running
```

The IIS website could also be inspected using:

```powershell
Get-Website
```

The service was confirmed to be operational before introducing the network fault.

### Evidence



**Screenshot:** `18-iis-service-running.png`

---

# 14. Simulated Network Incident

A deliberate NSG misconfiguration was introduced to simulate a realistic network security incident.

The following deny rule was created:

```text
Name:              Deny-HTTP-Troubleshooting
Priority:          100
Protocol:          TCP
Destination Port:  80
Action:            Deny
```

The existing allow rule had:

```text
Priority: 200
```

Therefore:

```text
Priority 100 → DENY
Priority 200 → ALLOW
```

Because lower numerical priority values are evaluated first, the deny rule took precedence.

### Evidence



**Screenshot:** `19-nsg-http-deny-rule-created.png`

---

# 15. Reproduce the Connectivity Failure

After introducing the deny rule, the same connectivity test was repeated from `ADMIN-VM`:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

The result changed from:

```text
TcpTestSucceeded : True
```

to:

```text
TcpTestSucceeded : False
```

An HTTP request was also expected to fail:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

This reproduced the simulated incident.

### Evidence



**Screenshot:** `20-connectivity-failure.png`

---

# 16. Troubleshooting Investigation

The incident was investigated using a layered troubleshooting methodology.

The investigation considered:

* VM availability
* Network configuration
* IIS service
* Local connectivity
* NSG rules
* Effective security rules
* Azure network diagnostics

The investigation began at the server and application layers before moving toward Azure network security controls.

---

## 16.1 Verify VM Availability

The Azure portal was checked to confirm that:

```text
WEB-SERVER-01
```

was still running.

### Finding

The virtual machine remained operational.

### Conclusion

The issue was unlikely to be caused by the VM being stopped or unavailable.

### Evidence



**Screenshot:** `21-web-server-health-check.png`

---

## 16.2 Verify IIS Service

The IIS service was checked directly on `WEB-SERVER-01`:

```powershell
Get-Service W3SVC
```

The service was found to be running.

### Finding

```text
IIS → Running
```

### Conclusion

The web application remained operational.

---

## 16.3 Test Local TCP Connectivity

A local test was performed from `WEB-SERVER-01`:

```powershell
Test-NetConnection localhost -Port 80
```

The result was successful.

### Finding

```text
WEB-SERVER-01
      |
      └── localhost:80 → SUCCESS
```

### Conclusion

IIS was listening locally on TCP port `80`.

### Evidence



**Screenshot:** `22-local-iis-connectivity-success.png`

---

## 16.4 Compare Local and Remote Connectivity

The results were compared.

### Local Test

```text
WEB-SERVER-01 → localhost:80
SUCCESS
```

### Remote Test

```text
ADMIN-VM → WEB-SERVER-01:80
FAILURE
```

This indicated that the application was functioning locally while remote access was being blocked.

The investigation therefore moved toward network security controls.

---

## 16.5 Analyse NSG Rules

The NSG associated with `WEB-SERVER-01` was reviewed.

### Deny Rule

```text
Name:       Deny-HTTP-Troubleshooting
Priority:   100
Protocol:   TCP
Port:       80
Action:     Deny
```

### Allow Rule

```text
Name:       Allow-HTTP-Testing
Priority:   200
Protocol:   TCP
Port:       80
Action:     Allow
```

The numerical priority was critical.

The lower numerical value has higher precedence:

```text
100 → DENY
200 → ALLOW
```

The deny rule therefore took precedence.

---

## 16.6 Review Effective Security Rules

The effective security rules applied to the network interface of `WEB-SERVER-01` were reviewed.

The results showed that inbound TCP port `80` traffic was being denied.

### Finding

```text
Source:
ADMIN-VM / Admin-Subnet

Destination:
WEB-SERVER-01

Protocol:
TCP

Port:
80

Effective Result:
DENY
```

### Conclusion

The NSG was actively preventing the connection.

### Evidence



**Screenshot:** `23-effective-security-rules-deny-http.png`

---

## 16.7 Azure Network Watcher Investigation

Azure Network Watcher Connection Troubleshoot was used to independently test the network path.

### Test Parameters

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

The connection test returned a failure.

### Purpose

Network Watcher provided Azure-side diagnostic evidence supporting the PowerShell connectivity test.

### Finding

The network path to TCP port `80` was blocked.

### Evidence



**Screenshot:** `24-network-watcher-http-failure.png`

---

# 17. Root Cause Analysis

## Problem Statement

`ADMIN-VM` was unable to access the IIS web server hosted on `WEB-SERVER-01` over TCP port `80`.

## Evidence Collected

| Investigation            | Result            |
| ------------------------ | ----------------- |
| VM availability          | Operational       |
| IIS service              | Running           |
| Local TCP 80             | Successful        |
| Remote TCP 80            | Failed            |
| HTTP request             | Failed            |
| NSG configuration        | Deny rule present |
| Effective security rules | TCP 80 denied     |
| Network Watcher          | Connection failed |

## Root Cause

The root cause was a higher-priority Network Security Group deny rule:

```text
Deny-HTTP-Troubleshooting
Priority: 100
TCP: 80
DENY
```

This rule took precedence over:

```text
Allow-HTTP-Testing
Priority: 200
TCP: 80
ALLOW
```

As a result, Azure blocked inbound HTTP traffic to the IIS server.

---

# 18. Why IIS Was Not the Root Cause

The evidence showed:

```text
IIS Service
    ↓
Running

Local TCP 80
    ↓
Successful

Remote TCP 80
    ↓
Failed

Effective NSG
    ↓
Deny TCP 80
```

This demonstrated that:

* The Windows Server VM was operational.
* IIS was operational.
* TCP port 80 was available locally.
* The failure occurred when traffic crossed the network boundary.
* The NSG was responsible for blocking the remote connection.

---

# 19. Remediation

The incorrect NSG deny rule was removed.

Removed rule:

```text
Deny-HTTP-Troubleshooting
Priority: 100
TCP: 80
DENY
```

The valid HTTP allow rule remained.

### Evidence



**Screenshot:** `25-nsg-corrected.png`

---

# 20. Post-Remediation Connectivity Test

After removing the deny rule, the connectivity test was repeated from `ADMIN-VM`:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

Expected result:

```text
TcpTestSucceeded : True
```

### Finding

TCP port `80` became reachable again.

### Evidence



**Screenshot:** `26-connectivity-restored.png`

---

# 21. Post-Remediation Network Watcher Test

Azure Network Watcher Connection Troubleshoot was run again using the same source, destination, protocol and port.

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

### Result

The connection test completed successfully.

### Evidence



**Screenshot:** `27-network-watcher-success.png`

---

# 22. Final HTTP Test

A final HTTP request was performed from `ADMIN-VM`:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

The IIS server successfully returned an HTTP response.

This confirmed that the application layer was reachable after network remediation.

### Evidence



**Screenshot:** `28-final-http-test-success.png`

---

# 23. Least-Privilege Security Improvement

Although removing the deny rule restored connectivity, the final configuration was reviewed from a security perspective.

A broad HTTP allow rule can expose the service to more sources than necessary.

The final configuration therefore restricts HTTP access to the administration subnet.

## Final Rule

```text
Name:
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

This means that systems within the administration subnet are permitted to access the web server over HTTP through this rule.

### Security Principle

This implements the **principle of least privilege** by allowing only the network traffic required for the lab's intended administrative function.

---

# 24. Before vs After

| Area             | During Incident | After Remediation |
| ---------------- | --------------- | ----------------- |
| Web Server       | Running         | Running           |
| IIS              | Running         | Running           |
| Local Port 80    | Accessible      | Accessible        |
| Remote Port 80   | Blocked         | Accessible        |
| NSG              | HTTP Denied     | HTTP Allowed      |
| Network Watcher  | Failed          | Successful        |
| HTTP Request     | Failed          | Successful        |
| Security Posture | Misconfigured   | Restricted        |

---

# 25. Troubleshooting Methodology Used

The troubleshooting process followed a structured approach:

```text
1. Identify the symptom
        ↓
2. Confirm the affected service
        ↓
3. Verify VM availability
        ↓
4. Verify IIS
        ↓
5. Test local connectivity
        ↓
6. Test remote connectivity
        ↓
7. Inspect NSG configuration
        ↓
8. Review effective security rules
        ↓
9. Use Azure Network Watcher
        ↓
10. Identify root cause
        ↓
11. Apply remediation
        ↓
12. Retest connectivity
        ↓
13. Validate application access
        ↓
14. Improve security configuration
```

This approach prevented unnecessary changes to the server or application before the actual cause was identified.

---

# 26. Complete Evidence Timeline

| Step | Screenshot                                  | Evidence                    |
| ---: | ------------------------------------------- | --------------------------- |
|    1 | `01-resource-group-created.png`             | Resource group              |
|    2 | `02-vnet-subnets-created.png`               | VNet and subnets            |
|    3 | `03-available-vm-sizes-west-central-us.png` | VM size availability        |
|    4 | `04-web-server-final-configuration.png`     | Web VM configuration        |
|    5 | `05-web-server-deployment-success.png`      | Web VM deployment           |
|    6 | `06-web-server-overview.png`                | Web VM overview             |
|    7 | `07-rdp-successful.png`                     | RDP access                  |
|    8 | `08-web-server-network-config.png`          | Network configuration       |
|    9 | `09-iis-installed.png`                      | IIS installation            |
|   10 | `10-iis-local-test-success.png`             | Local IIS test              |
|   11 | `11-admin-vm-final-configuration.png`       | Admin VM configuration      |
|   12 | `12-admin-vm-deployment-success.png`        | Admin VM deployment         |
|   13 | `13-admin-vm-overview.png`                  | Admin VM overview           |
|   14 | `14-admin-vm-rdp-successful.png`            | Admin VM RDP                |
|   15 | `15-admin-vm-network-configuration.png`     | Admin network configuration |
|   16 | `16-initial-connectivity-success.png`       | Baseline connectivity       |
|   17 | `17-http-allow-nsg-rule.png`                | HTTP allow rule             |
|   18 | `18-iis-service-running.png`                | IIS service                 |
|   19 | `19-nsg-http-deny-rule-created.png`         | Fault introduced            |
|   20 | `20-connectivity-failure.png`               | Connectivity failure        |
|   21 | `21-web-server-health-check.png`            | VM health                   |
|   22 | `22-local-iis-connectivity-success.png`     | Local connectivity          |
|   23 | `23-effective-security-rules-deny-http.png` | Effective NSG denial        |
|   24 | `24-network-watcher-http-failure.png`       | Network Watcher failure     |
|   25 | `25-nsg-corrected.png`                      | NSG remediation             |
|   26 | `26-connectivity-restored.png`              | Connectivity restored       |
|   27 | `27-network-watcher-success.png`            | Network Watcher success     |
|   28 | `28-final-http-test-success.png`            | Final HTTP validation       |

---

# 27. Key Technical Findings

### Finding 1 — The VM Was Healthy

`WEB-SERVER-01` remained operational throughout the incident.

### Finding 2 — IIS Was Healthy

The IIS service remained running and responded successfully to local requests.

### Finding 3 — The Failure Was Network-Specific

Local TCP port `80` access succeeded while remote access failed.

### Finding 4 — NSG Rules Were Affecting Traffic

The effective security rules showed that TCP port `80` was being denied.

### Finding 5 — Rule Priority Caused the Failure

The deny rule had priority `100`, while the allow rule had priority `200`.

### Finding 6 — Network Watcher Confirmed the Failure

Azure Network Watcher independently confirmed that the network connection was blocked.

### Finding 7 — Removing the Incorrect Rule Restored Connectivity

After remediation, TCP port `80` and HTTP requests succeeded again.

### Finding 8 — Least Privilege Improved the Final Configuration

HTTP access was restricted to the administration subnet.

---

# 28. Security Lessons Learned

## NSG Rule Priority

NSG rules use numerical priorities.

A lower number has higher priority.

For example:

```text
100 → DENY
200 → ALLOW
```

The `DENY` rule wins.

---

## Layered Troubleshooting

A connectivity problem should not automatically be assumed to be an application problem.

The troubleshooting process should consider:

```text
Application
    ↓
Service
    ↓
Operating System
    ↓
Local Network
    ↓
NSG
    ↓
Routing
    ↓
Azure Network
```

---

## Baseline Testing

Establishing a working baseline before introducing a fault makes troubleshooting significantly easier.

In this lab:

```text
Before fault:
TCP 80 → SUCCESS
```

After the fault:

```text
TCP 80 → FAILURE
```

After remediation:

```text
TCP 80 → SUCCESS
```

This provided clear evidence of cause and effect.

---

# 29. Lessons Learned

### Azure Networking

* VNets provide the foundation for Azure network communication.
* Subnets provide logical segmentation.
* NSGs provide traffic filtering.
* Effective security rules help identify the actual access controls affecting a network interface.

### Windows Administration

* Windows Server services can be verified using PowerShell.
* IIS can be installed and managed through PowerShell.
* Local service testing is useful for isolating network problems.

### Network Troubleshooting

* `Test-NetConnection` is useful for TCP port testing.
* `Invoke-WebRequest` can validate HTTP application access.
* Local-versus-remote testing helps determine whether an issue is application or network related.
* Network Watcher provides Azure-side connectivity diagnostics.

### Cybersecurity

* Security controls can unintentionally cause availability problems.
* NSG rules should be reviewed carefully for priority conflicts.
* Least-privilege access reduces unnecessary network exposure.
* Troubleshooting should preserve security rather than simply opening ports broadly.

---

# 30. Final Incident Resolution

## Initial State

```text
ADMIN-VM
     |
     | TCP 80
     X
WEB-SERVER-01
```

## Root Cause

```text
NSG
 |
 +-- Priority 100 → DENY TCP 80
 |
 +-- Priority 200 → ALLOW TCP 80
```

## Remediated State

```text
ADMIN-VM
10.10.2.0/24
     |
     | TCP 80
     | ALLOW
     v
WEB-SERVER-01
10.10.1.0/24
     |
     v
IIS
```

The service was restored and successfully validated.

---

# 31. Final Project Outcome

The project successfully demonstrated a complete cloud network troubleshooting lifecycle:

```text
Azure Infrastructure
        ↓
VNet & Subnet Configuration
        ↓
Windows Server Deployment
        ↓
IIS Installation
        ↓
Baseline Connectivity
        ↓
NSG Fault Simulation
        ↓
Connectivity Failure
        ↓
Layered Investigation
        ↓
Root Cause Identification
        ↓
NSG Remediation
        ↓
Connectivity Restored
        ↓
Network Watcher Validation
        ↓
Least-Privilege Improvement
        ↓
Final Documentation
```

## Final Status

**Incident:** Resolved

**Service:** IIS / HTTP

**Affected Port:** TCP 80

**Root Cause:** Higher-priority NSG deny rule

**Resolution:** Removed incorrect deny rule and restricted final HTTP access to the administration subnet

**Validation:** Successful TCP, HTTP and Azure Network Watcher tests

**Project Status:** Completed
