
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

Architecture diagram:

<img width="1653" height="565" alt="Azure Network architecture" src="https://github.com/user-attachments/assets/346ef665-b92d-4251-8fab-0d29b95bb9c9" />
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

# 5. Resource Group Creation

The Azure resources were organised within:

```text
rg-network-troubleshooting-lab
```

The resource group was created in preparation for the networking and virtual machine resources used throughout the lab.

### Screenshot Evidence

<img width="1912" height="910" alt="01-resource-group-created png" src="https://github.com/user-attachments/assets/98315513-9e69-4cb8-941f-31c6cc294975" />

**Evidence:** `01-resource-group-created.png`

This screenshot documents the successful creation of the project resource group.

---

# 6. Virtual Network and Subnet Configuration

The following Virtual Network was created:

```text
vnet-network-troubleshooting
```

with the address space:

```text
10.10.0.0/16
```

Two subnets were created.

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

### Screenshot Evidence

<img width="1916" height="910" alt="02-vnet-subnets-created png" src="https://github.com/user-attachments/assets/aacf5d05-ec0f-4004-a88f-787b15ffe2b0" />

**Evidence:** `02-vnet-subnets-created.png`

---

# 7. VM Size Availability Check

Before deploying the virtual machines, available VM sizes were reviewed for the selected Azure region.

The selected region for this project was:

```text
West Central US
```

This check was important because Azure VM SKU availability can vary by region and subscription.

### Screenshot Evidence

<img width="1917" height="911" alt="03-available-vm-sizes-west-central-us png" src="https://github.com/user-attachments/assets/8205a42c-a5a3-4f38-bbbb-41c0929475ea" />

**Evidence:** `03-available-vm-sizes-west-central-us.png`

---

# 8. WEB-SERVER-01 Deployment

The first virtual machine was deployed as:

```text
WEB-SERVER-01
```

The VM was configured with:

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

The VM was configured with a Network Security Group:

```text
nsg-web-server
```

RDP access was enabled for administration and configuration of the lab server.

### Screenshot Evidence — Final VM Configuration

<img width="1908" height="916" alt="04-web-server-final-configuration png" src="https://github.com/user-attachments/assets/b2ccc32b-4d46-4353-8b53-eca609fa4795" />

**Evidence:** `04-web-server-final-configuration.png`

### Screenshot Evidence — Deployment

<img width="1919" height="910" alt="05-web-server-deployment-success png" src="https://github.com/user-attachments/assets/7fe03a80-743b-40d3-bb6d-9d66973bd96c" />

**Evidence:** `05-web-server-deployment-success.png`

### Screenshot Evidence — VM Overview

<img width="1919" height="971" alt="06-web-server-overview png" src="https://github.com/user-attachments/assets/e277dce9-93e1-4c3d-a5d4-973e38a97cc3" />

**Evidence:** `06-web-server-overview.png`

---

# 9. Connecting to WEB-SERVER-01

After deployment, Remote Desktop Protocol (RDP) was used to connect to the Windows Server.

The successful RDP connection confirmed that the VM was accessible for administration.

### Screenshot Evidence

<img width="1919" height="1012" alt="07-rdp-successful png" src="https://github.com/user-attachments/assets/371ee10b-ccdc-4e8c-a54e-3db3f72e08be" />

**Evidence:** `07-rdp-successful.png`

---

# 10. Verify WEB-SERVER-01 Network Configuration

The server's network configuration was checked from inside Windows Server.

The following PowerShell command can be used:

```powershell
Get-NetIPConfiguration
```

or:

```powershell
ipconfig
```

The purpose of this step was to identify:

* IPv4 address
* Subnet information
* Default gateway
* DNS configuration
* Network adapter status

### Screenshot Evidence

<img width="1919" height="1019" alt="08-web-server-network-config png" src="https://github.com/user-attachments/assets/582b67b8-0262-4ad4-b083-dd91000991cb" />

**Evidence:** `08-web-server-network-config.png`

---

# 11. IIS Installation

IIS was installed on `WEB-SERVER-01` using PowerShell.

Command:

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

After installation, the IIS role was verified:

```powershell
Get-WindowsFeature Web-Server
```

The expected result was that the Web Server role was installed.

### Screenshot Evidence

<img width="1919" height="1018" alt="09-iis-installed png" src="https://github.com/user-attachments/assets/c030bb81-a801-4a45-a70d-6fcaf5469f39" />

**Evidence:** `09-iis-installed.png`

---

# 12. IIS Local Connectivity Test

Before testing network connectivity from another VM, IIS was tested locally.

The following URL was opened from the web server:

```text
http://localhost
```

A successful IIS response confirmed that the web application was functioning locally.

This was an important troubleshooting baseline because it established that IIS itself was working before any network incident was introduced.

### Screenshot Evidence

<img width="1917" height="1021" alt="10-iis-local-test-success png" src="https://github.com/user-attachments/assets/9be77448-fc65-494b-89e7-cf4c452329eb" />

**Evidence:** `10-iis-local-test-success.png`

---

# 13. ADMIN-VM Deployment

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

### Screenshot Evidence — Final Configuration

<img width="1913" height="907" alt="11-admin-vm-final-configuration png" src="https://github.com/user-attachments/assets/3b307496-7fff-4da3-b37f-e18699ff2f71" />

**Evidence:** `11-admin-vm-final-configuration.png`

### Screenshot Evidence — Deployment

<img width="1918" height="911" alt="12-admin-vm-deployment-success png" src="https://github.com/user-attachments/assets/fcad87fb-8041-4a0b-9e0a-1981b062aa4d" />

**Evidence:** `12-admin-vm-deployment-success.png`

### Screenshot Evidence — VM Overview

<img width="1919" height="970" alt="13-admin-vm-overview png" src="https://github.com/user-attachments/assets/c7893147-1568-4d98-ac65-4326107fc481" />

**Evidence:** `13-admin-vm-overview.png`

---

# 14. Connecting to ADMIN-VM

RDP was used to connect to the administration VM.

### Screenshot Evidence

<img width="1919" height="1022" alt="14-admin-vm-rdp-successful png" src="https://github.com/user-attachments/assets/25bb018c-5e5c-4bec-bafb-0947765e21b6" />

**Evidence:** `14-admin-vm-rdp-successful.png`

---

# 15. Verify ADMIN-VM Network Configuration

The administration VM network configuration was checked using:

```powershell
Get-NetIPConfiguration
```

or:

```powershell
ipconfig
```

The private IP address was recorded for use during troubleshooting.

### Screenshot Evidence

<img width="1919" height="1021" alt="15-admin-vm-network-configuration png" src="https://github.com/user-attachments/assets/6bb26df0-86ba-474b-b53d-94a1e2859507" />


**Evidence:** `15-admin-vm-network-configuration.png`

---

# 16. Baseline Connectivity Test

Before introducing the simulated fault, connectivity from `ADMIN-VM` to `WEB-SERVER-01` was tested.

The private IP address of `WEB-SERVER-01` was used.

Command:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

Example:

```powershell
Test-NetConnection 10.10.1.X -Port 80
```

The expected result was:

```text
TcpTestSucceeded : True
```

This confirmed that TCP port `80` was reachable.

An HTTP request was also tested:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

The successful response established a working baseline.

### Screenshot Evidence

<img width="1919" height="1021" alt="16-initial-connectivity-success png" src="https://github.com/user-attachments/assets/70b54fa3-b5b2-4011-b236-a2f6d4ded9be" />

**Evidence:** `16-initial-connectivity-success.png`

---

# 17. Configure Initial HTTP NSG Allow Rule

An inbound NSG rule was configured on:

```text
nsg-web-server
```

The initial testing rule was:

| Setting          | Value                       |
| ---------------- | --------------------------- |
| Name             | `Allow-HTTP-Testing`        |
| Priority         | `200`                       |
| Source           | Appropriate internal source |
| Protocol         | TCP                         |
| Destination Port | `80`                        |
| Action           | Allow                       |

This rule permitted HTTP traffic to the IIS web server.

### Screenshot Evidence

<img width="1914" height="909" alt="17-http-allow-nsg-rule png" src="https://github.com/user-attachments/assets/72fe79a6-7bda-4427-932e-a8a00983b737" />

**Evidence:** `17-http-allow-nsg-rule.png`

---

# 18. Verify IIS Service

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

### Screenshot Evidence

<img width="1919" height="1016" alt="18-iis-service-running png" src="https://github.com/user-attachments/assets/74cb1cdb-dfb8-42d7-a5d8-98a791427555" />

**Evidence:** `18-iis-service-running.png`

---

# 19. Simulate the Network Failure

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

### Screenshot Evidence

<img width="1917" height="909" alt="19-nsg-http-deny-rule-created png" src="https://github.com/user-attachments/assets/debb0be6-13f4-4533-be40-6f193be52ac3" />

**Evidence:** `19-nsg-http-deny-rule-created.png`

---

# 20. Reproduce the Connectivity Failure

After introducing the deny rule, the same connectivity test was repeated from `ADMIN-VM`.

Command:

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

### Screenshot Evidence

<img width="1916" height="1017" alt="20-connectivity-failure png" src="https://github.com/user-attachments/assets/379026e3-0819-429a-a635-ccd6451d3cbc" />

**Evidence:** `20-connectivity-failure.png`

---

# 21. Troubleshooting Investigation

The incident was investigated using a layered troubleshooting methodology.

The objective was to determine whether the failure was caused by:

* VM availability
* Network configuration
* IIS service
* Windows Firewall
* NSG rules
* Routing
* Azure networking

The investigation started at the server/application layer and moved outward toward the Azure network security layer.

---

# 22. Step 1 — Verify VM Availability

The Azure portal was checked to confirm that:

```text
WEB-SERVER-01
```

was still running and had no obvious VM-level health issue.

### Finding

The virtual machine remained operational.

### Conclusion

The issue was unlikely to be caused by the VM being stopped or unavailable.

### Screenshot Evidence

<img width="1912" height="907" alt="21-web-server-health-check png" src="https://github.com/user-attachments/assets/4f7b27dc-69ad-4dd9-88c4-f513cd32d228" />

**Evidence:** `21-web-server-health-check.png`

---

# 23. Step 2 — Verify IIS Service

The IIS service was checked directly on `WEB-SERVER-01`.

Command:

```powershell
Get-Service W3SVC
```

The service was found to be running.

### Finding

```text
IIS → Running
```

### Conclusion

The web application was still operational.

Therefore, the investigation moved toward network connectivity.

---

# 24. Step 3 — Test Local TCP Connectivity

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

The IIS service was listening locally on TCP port `80`.

This helped eliminate IIS as the primary cause of the remote connectivity failure.

### Screenshot Evidence

<img width="1919" height="1016" alt="22-local-iis-connectivity-success png" src="https://github.com/user-attachments/assets/c3ab7e0e-1ccd-4355-b199-96b014ab5af5" />

**Evidence:** `22-local-iis-connectivity-success.png`

---

# 25. Step 4 — Compare Local and Remote Connectivity

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

This was an important diagnostic finding.

The application was available locally, but remote access was being blocked.

### Troubleshooting Direction

The investigation therefore moved toward:

```text
Network Security
      ↓
NSG
      ↓
Effective Security Rules
      ↓
Azure Network Watcher
```

---

# 26. Step 5 — Analyse Network Security Group Rules

The NSG associated with `WEB-SERVER-01` was reviewed.

The following rules were identified:

### Rule 1 — Deny

```text
Name:       Deny-HTTP-Troubleshooting
Priority:   100
Protocol:   TCP
Port:       80
Action:     Deny
```

### Rule 2 — Allow

```text
Name:       Allow-HTTP-Testing
Priority:   200
Protocol:   TCP
Port:       80
Action:     Allow
```

The numerical priority was critical.

Azure evaluates NSG rules based on priority, with lower numerical values taking precedence.

Therefore:

```text
100 → DENY
200 → ALLOW
```

The deny rule was being evaluated first.

---

# 27. Step 6 — Review Effective Security Rules

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

### Screenshot Evidence

<img width="1915" height="909" alt="23-effective-security-rules-deny-http png" src="https://github.com/user-attachments/assets/1e8779b7-95ef-44f4-aeb5-0c7dad9877ce" />

**Evidence:** `23-effective-security-rules-deny-http.png`

---

# 28. Step 7 — Azure Network Watcher Investigation

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

### Screenshot Evidence

<img width="1919" height="910" alt="24-network-watcher-http-failure png" src="https://github.com/user-attachments/assets/86aa5b7b-5f6f-4c6d-a4ca-f41fc943daaa" />

**Evidence:** `24-network-watcher-http-failure.png`

---

# 29. Root Cause Analysis

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

# 30. Why IIS Was Not the Root Cause

It was important not to immediately assume that the web service was broken.

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

# 31. Remediation

The incorrect NSG deny rule was removed.

Removed rule:

```text
Deny-HTTP-Troubleshooting
Priority: 100
TCP: 80
DENY
```

The valid HTTP allow rule remained.

### Screenshot Evidence

<img width="1917" height="908" alt="25-nsg-corrected png" src="https://github.com/user-attachments/assets/55e163c7-f1a2-4ebb-bd34-f7cc48620cef" />

**Evidence:** `25-nsg-corrected.png`

---

# 32. Post-Remediation Connectivity Test

After removing the deny rule, the connectivity test was repeated from `ADMIN-VM`.

Command:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

Expected result:

```text
TcpTestSucceeded : True
```

### Finding

TCP port `80` became reachable again.

### Screenshot Evidence

<img width="1913" height="1013" alt="26-connectivity-restored png" src="https://github.com/user-attachments/assets/b550a45c-1fdf-4b9b-98e3-2161b02272c1" />

**Evidence:** `26-connectivity-restored.png`

---

# 33. Post-Remediation Network Watcher Test

Azure Network Watcher Connection Troubleshoot was run again using the same source, destination, protocol and port.

### Test

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

### Screenshot Evidence

<img width="1914" height="909" alt="27-network-watcher-success png" src="https://github.com/user-attachments/assets/8994415e-c85d-4153-b5ee-c33ab41cd064" />

**Evidence:** `27-network-watcher-success.png`

---

# 34. Final HTTP Test

A final HTTP request was performed from `ADMIN-VM`.

Command:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

The IIS server successfully returned an HTTP response.

This confirmed that the application layer was reachable after the network remediation.

### Screenshot Evidence

<img width="1919" height="1021" alt="28-final-http-test-success png" src="https://github.com/user-attachments/assets/19afd490-b714-497a-bed2-e1ba51efbdf6" />

**Evidence:** `28-final-http-test-success.png`

---

# 35. Least-Privilege Security Improvement

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

This means that only systems within the administration subnet are permitted to access the web server over HTTP through this rule.

### Security Principle

This implements the **principle of least privilege** by allowing only the network traffic required for the lab's intended administrative function.

---

# 36. Before vs After

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

# 37. Troubleshooting Methodology Used

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

# 38. Complete Evidence Timeline

| Step | Screenshot                                  | Evidence                    |
| ---- | ------------------------------------------- | --------------------------- |
| 1    | `01-resource-group-created.png`             | Resource group              |
| 2    | `02-vnet-subnets-created.png`               | VNet and subnets            |
| 3    | `03-available-vm-sizes-west-central-us.png` | VM size availability        |
| 4    | `04-web-server-final-configuration.png`     | Web VM configuration        |
| 5    | `05-web-server-deployment-success.png`      | Web VM deployment           |
| 6    | `06-web-server-overview.png`                | Web VM overview             |
| 7    | `07-rdp-successful.png`                     | RDP access                  |
| 8    | `08-web-server-network-config.png`          | Network configuration       |
| 9    | `09-iis-installed.png`                      | IIS installation            |
| 10   | `10-iis-local-test-success.png`             | Local IIS test              |
| 11   | `11-admin-vm-final-configuration.png`       | Admin VM configuration      |
| 12   | `12-admin-vm-deployment-success.png`        | Admin VM deployment         |
| 13   | `13-admin-vm-overview.png`                  | Admin VM overview           |
| 14   | `14-admin-vm-rdp-successful.png`            | Admin VM RDP                |
| 15   | `15-admin-vm-network-configuration.png`     | Admin network configuration |
| 16   | `16-initial-connectivity-success.png`       | Baseline connectivity       |
| 17   | `17-http-allow-nsg-rule.png`                | HTTP allow rule             |
| 18   | `18-iis-service-running.png`                | IIS service                 |
| 19   | `19-nsg-http-deny-rule-created.png`         | Fault introduced            |
| 20   | `20-connectivity-failure.png`               | Connectivity failure        |
| 21   | `21-web-server-health-check.png`            | VM health                   |
| 22   | `22-local-iis-connectivity-success.png`     | Local connectivity          |
| 23   | `23-effective-security-rules-deny-http.png` | Effective NSG denial        |
| 24   | `24-network-watcher-http-failure.png`       | Network Watcher failure     |
| 25   | `25-nsg-corrected.png`                      | NSG remediation             |
| 26   | `26-connectivity-restored.png`              | Connectivity restored       |
| 27   | `27-network-watcher-success.png`            | Network Watcher success     |
| 28   | `28-final-http-test-success.png`            | Final HTTP validation       |

---

# 39. Key Technical Findings

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

# 40. Security Lessons Learned

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

# 41. Lessons Learned

This project demonstrated several important practical concepts.

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

# 42. Final Incident Resolution

The incident was successfully resolved.

### Initial State

```text
ADMIN-VM
     |
     | TCP 80
     X
WEB-SERVER-01
```

### Root Cause

```text
NSG
 |
 +-- Priority 100 → DENY TCP 80
 |
 +-- Priority 200 → ALLOW TCP 80
```

### Remediated State

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

# 43. Final Project Outcome

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
