# Azure Network Troubleshooting & NSG Security Lab

![Microsoft Azure](https://img.shields.io/badge/Microsoft%20Azure-Cloud-blue)
![Networking](https://img.shields.io/badge/Azure-Networking-blue)
![Security](https://img.shields.io/badge/Focus-Cloud%20Security-red)
![Troubleshooting](https://img.shields.io/badge/Focus-Network%20Troubleshooting-green)
![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-blue)

## 📌 Project Overview

This project is a hands-on **Microsoft Azure Network Troubleshooting and Network Security lab** designed to simulate a real-world internal web server connectivity incident.

The environment was built using **Azure Virtual Networks, subnets, Windows Server virtual machines, IIS, and Network Security Groups (NSGs)**.

After establishing a working baseline, a deliberate NSG misconfiguration was introduced to block HTTP traffic to the web server. The resulting connectivity failure was investigated using **PowerShell, Windows networking tools, Azure NSG analysis, effective security rules, and Azure Network Watcher**.

The root cause was identified, the incorrect NSG rule was removed, connectivity was restored, and the final configuration was improved using a **least-privilege network access rule**.

---

## 🎯 Project Objectives

* Build a segmented Azure Virtual Network.
* Deploy Windows Server virtual machines.
* Configure separate administration and web-server subnets.
* Install and configure IIS.
* Configure Azure Network Security Groups.
* Establish baseline network connectivity.
* Simulate an intentional network security failure.
* Troubleshoot TCP/HTTP connectivity issues.
* Analyse effective NSG security rules.
* Use Azure Network Watcher for diagnostics.
* Identify the root cause of the incident.
* Restore service connectivity.
* Apply least-privilege network access controls.
* Document the troubleshooting process and evidence.

---

## 🛠️ Technologies & Tools

| Technology / Tool                 | Purpose                            |
| --------------------------------- | ---------------------------------- |
| **Microsoft Azure**               | Cloud infrastructure               |
| **Azure Virtual Network**         | Network infrastructure             |
| **Azure Subnets**                 | Network segmentation               |
| **Azure Network Security Groups** | Network traffic filtering          |
| **Azure Network Watcher**         | Connectivity diagnostics           |
| **Windows Server 2022**           | Server operating system            |
| **IIS**                           | Internal web server                |
| **PowerShell**                    | Administration and troubleshooting |
| **RDP**                           | Remote administration              |
| `Test-NetConnection`              | TCP connectivity testing           |
| `Invoke-WebRequest`               | HTTP testing                       |
| `ipconfig`                        | IP configuration                   |
| `Get-NetIPConfiguration`          | Network configuration              |
| `Get-Service`                     | Service validation                 |

---

# 🏗️ Lab Architecture

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

![Azure Network Architecture](architecture/azure-network-architecture.png)

---

# ☁️ Azure Resources

| Resource                | Configuration                    |
| ----------------------- | -------------------------------- |
| **Resource Group**      | `rg-network-troubleshooting-lab` |
| **Region**              | West Central US                  |
| **Virtual Network**     | `vnet-network-troubleshooting`   |
| **VNet Address Space**  | `10.10.0.0/16`                   |
| **Web Subnet**          | `Web-Subnet`                     |
| **Web Subnet CIDR**     | `10.10.1.0/24`                   |
| **Admin Subnet**        | `Admin-Subnet`                   |
| **Admin Subnet CIDR**   | `10.10.2.0/24`                   |
| **Web Server**          | `WEB-SERVER-01`                  |
| **Admin VM**            | `ADMIN-VM`                       |
| **Web Server NSG**      | `nsg-web-server`                 |
| **Web Service**         | IIS                              |
| **Network Diagnostics** | Azure Network Watcher            |

---

# 🔐 Network Security Design

The environment uses two separate subnets to provide basic network segmentation.

### Web Subnet

```text
10.10.1.0/24
```

Hosts:

```text
WEB-SERVER-01
```

### Admin Subnet

```text
10.10.2.0/24
```

Hosts:

```text
ADMIN-VM
```

The administration VM communicates with the IIS server using the web server's **private IP address** over TCP port `80`.

---

# 🖥️ Web Server Configuration

`WEB-SERVER-01` was deployed using Windows Server 2022 Datacenter.

IIS was installed using PowerShell:

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

The IIS service was verified with:

```powershell
Get-Service W3SVC
```

The IIS website was also tested locally using:

```text
http://localhost
```

This established that the web server and IIS application were functioning before network troubleshooting began.

---

# 🧪 Baseline Connectivity Test

Before introducing the simulated fault, connectivity between `ADMIN-VM` and `WEB-SERVER-01` was tested.

From `ADMIN-VM`:

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

The initial tests confirmed that the network and IIS service were functioning correctly.

### Evidence

<img width="1919" height="1021" alt="16-initial-connectivity-success png" src="https://github.com/user-attachments/assets/432cf5ab-a7e3-4ba1-b44a-de34579f09bf" />

---

# 🚨 Simulated Network Incident

To simulate a real-world network security incident, an NSG deny rule was intentionally created.

### Faulty Rule

```text
Rule Name:       Deny-HTTP-Troubleshooting
Priority:        100
Protocol:        TCP
Destination:     Port 80
Action:          Deny
```

An existing HTTP allow rule had a lower precedence:

```text
Rule Name:       Allow-HTTP-Testing
Priority:        200
Protocol:        TCP
Destination:     Port 80
Action:          Allow
```

Because the deny rule had priority `100`, it took precedence over the allow rule.

### Result

The administration VM could no longer reach the IIS web server over TCP port `80`.

### Evidence

<img width="1917" height="909" alt="19-nsg-http-deny-rule-created png" src="https://github.com/user-attachments/assets/e24b02d3-1584-4cc4-b1b6-195778d9fa93" />

<img width="1916" height="1017" alt="20-connectivity-failure png" src="https://github.com/user-attachments/assets/6e7d15be-6443-46d0-be0a-59a95e4c0ad6" />

---

# 🔎 Troubleshooting Process

The incident was investigated systematically rather than immediately changing the configuration.

## 1. Verify the Web Server

The Azure portal was checked to confirm that `WEB-SERVER-01` was still running.

<img width="1912" height="907" alt="21-web-server-health-check png" src="https://github.com/user-attachments/assets/06b50dbd-b0ff-4f45-bd08-367c0544c9e9" />

---

## 2. Verify IIS

The IIS service was checked:

```powershell
Get-Service W3SVC
```

The service was confirmed to be running.

---

## 3. Test Local Connectivity

A local connection test was performed on the web server:

```powershell
Test-NetConnection localhost -Port 80
```

The local test succeeded.

This demonstrated that IIS itself was functioning correctly.

<img width="1919" height="1016" alt="22-local-iis-connectivity-success png" src="https://github.com/user-attachments/assets/dc5a73af-62dd-453c-a104-7eb7ad436b1d" />

---

## 4. Test Remote Connectivity

The administration VM attempted to connect to the web server:

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

The connection failed.

This helped isolate the issue to network access between the two VMs.

---

## 5. Analyse NSG Rules

The `nsg-web-server` configuration was reviewed.

The following rules were identified:

```text
Priority 100
Deny-HTTP-Troubleshooting
TCP 80
DENY
```

and:

```text
Priority 200
Allow-HTTP-Testing
TCP 80
ALLOW
```

The higher-priority deny rule was blocking the HTTP connection.

---

## 6. Review Effective Security Rules

The effective security rules applied to the web server's network interface were examined.

The results confirmed that inbound TCP port `80` traffic was being denied.

<img width="1915" height="909" alt="23-effective-security-rules-deny-http png" src="https://github.com/user-attachments/assets/a331d778-0d90-4730-8e6b-aeaca9fb66b3" />

---

## 7. Azure Network Watcher

Azure Network Watcher Connection Troubleshoot was used to test:

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

The diagnostic confirmed the connectivity failure.

<img width="1919" height="910" alt="24-network-watcher-http-failure png" src="https://github.com/user-attachments/assets/3fd5f534-b0c8-4378-b66f-ffa1792f24e4" />

---

# 🧠 Root Cause

The root cause was an incorrectly configured **Azure Network Security Group rule**.

The rule:

```text
Deny-HTTP-Troubleshooting
Priority: 100
TCP: 80
Action: Deny
```

had a higher evaluation priority than the HTTP allow rule:

```text
Allow-HTTP-Testing
Priority: 200
TCP: 80
Action: Allow
```

Therefore, Azure blocked the inbound HTTP traffic before the allow rule could permit it.

### Important Finding

The IIS application was **not** the cause of the incident.

Evidence showed:

```text
IIS Service              → Running
Local TCP 80             → Successful
Remote TCP 80            → Failed
Effective NSG Rule       → Deny
Network Watcher          → Connection Failed
```

This confirmed that the problem was caused by **network security configuration rather than the web server application**.

---

# 🔧 Remediation

The incorrect deny rule was removed from:

```text
nsg-web-server
```

The HTTP allow rule was then retained.

<img width="1917" height="908" alt="25-nsg-corrected png" src="https://github.com/user-attachments/assets/539024be-a19e-4513-86fc-161d3dad16d5" />

---

# 🔒 Least-Privilege Improvement

The final configuration was improved so that HTTP access was restricted to the administration subnet rather than allowing unrestricted internal sources.

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

This means that HTTP traffic is permitted from the `Admin-Subnet` while unnecessary sources are not granted access.

---

# ✅ Validation After Remediation

After correcting the NSG configuration, connectivity was tested again.

```powershell
Test-NetConnection <WEB-SERVER-PRIVATE-IP> -Port 80
```

Result:

```text
TcpTestSucceeded : True
```

An HTTP request was also performed:

```powershell
Invoke-WebRequest http://<WEB-SERVER-PRIVATE-IP>
```

The IIS server successfully responded.

Azure Network Watcher was also used again to confirm that the network path was operational.

### Evidence

<img width="1913" height="1013" alt="26-connectivity-restored png" src="https://github.com/user-attachments/assets/abff51fa-5879-479c-94db-ea61b932af54" />

<img width="1914" height="909" alt="27-network-watcher-success png" src="https://github.com/user-attachments/assets/4062ead1-fb54-4976-bad1-f11e314f72e5" />

<img width="1919" height="1021" alt="28-final-http-test-success png" src="https://github.com/user-attachments/assets/3113267b-9cd1-4721-a428-4ddb4be28ae7" />

---

# 📊 Troubleshooting Results

| Test               | Baseline | During Incident | After Remediation |
| ------------------ | -------: | --------------: | ----------------: |
| Web Server Running |        ✅ |               ✅ |                 ✅ |
| IIS Service        |        ✅ |               ✅ |                 ✅ |
| Local TCP 80       |        ✅ |               ✅ |                 ✅ |
| Remote TCP 80      |        ✅ |               ❌ |                 ✅ |
| HTTP Request       |        ✅ |               ❌ |                 ✅ |
| Effective NSG      |    Allow |          ❌ Deny |           ✅ Allow |
| Network Watcher    |        ✅ |        ❌ Failed |      ✅ Successful |

---

# 📸 Project Evidence

The project contains 28 screenshots documenting the complete lifecycle.

### Deployment

* Resource group creation
* VNet and subnet configuration
* Available VM sizes
* Web server configuration
* VM deployment
* VM overview
* RDP access
* Network configuration
* IIS installation
* IIS local testing

### Network Configuration

* Admin VM deployment
* Admin VM configuration
* Network configuration
* Initial connectivity
* HTTP NSG rule
* IIS service validation

### Troubleshooting

* NSG deny rule
* Connectivity failure
* Server health check
* Local IIS connectivity
* Effective security rules
* Network Watcher failure

### Remediation

* Corrected NSG
* Connectivity restored
* Network Watcher success
* Final HTTP test

---

# 📂 Project Structure

```text
Azure-network-troubleshooting-nsg-security-lab/
│
├── README.md
│
├── troubleshooting-report.md
│
├── screenshots/
│   │
│   ├── 01-resource-group-created.png
│   ├── 02-vnet-subnets-created.png
│   ├── 03-available-vm-sizes-west-central-us.png
│   ├── 04-web-server-final-configuration.png
│   ├── 05-web-server-deployment-success.png
│   ├── 06-web-server-overview.png
│   ├── 07-rdp-successful.png
│   ├── 08-web-server-network-config.png
│   ├── 09-iis-installed.png
│   ├── 10-iis-local-test-success.png
│   ├── 11-admin-vm-final-configuration.png
│   ├── 12-admin-vm-deployment-success.png
│   ├── 13-admin-vm-overview.png
│   ├── 14-admin-vm-rdp-successful.png
│   ├── 15-admin-vm-network-configuration.png
│   ├── 16-initial-connectivity-success.png
│   ├── 17-http-allow-nsg-rule.png
│   ├── 18-iis-service-running.png
│   ├── 19-nsg-http-deny-rule-created.png
│   ├── 20-connectivity-failure.png
│   ├── 21-web-server-health-check.png
│   ├── 22-local-iis-connectivity-success.png
│   ├── 23-effective-security-rules-deny-http.png
│   ├── 24-network-watcher-http-failure.png
│   ├── 25-nsg-corrected.png
│   ├── 26-connectivity-restored.png
│   ├── 27-network-watcher-success.png
│   └── 28-final-http-test-success.png
│
└── architecture/
    └── azure-network-architecture.png
```

---

# 📄 Detailed Troubleshooting Report

For the complete technical investigation, including detailed troubleshooting steps, commands, evidence and root-cause analysis:

➡️ **[View Detailed Troubleshooting Report](./troubleshooting-report.md)**

---

# 💡 Key Lessons Learned

### Azure Networking

* How Azure VNets and subnets provide network segmentation.
* How private IP addresses facilitate internal communication.
* How NSGs control inbound network traffic.

### Network Troubleshooting

* Always establish a working baseline before troubleshooting.
* Test the application locally before investigating network controls.
* Compare local and remote connectivity.
* Use effective security rules to determine what is actually being applied.
* Use Network Watcher to validate Azure network paths.

### Security

* NSG priority directly affects traffic decisions.
* A lower numerical priority is evaluated before a higher numerical priority.
* Least-privilege rules reduce unnecessary network exposure.
* Security controls should be tested after configuration changes.

---

# 🎓 Skills Demonstrated

This project demonstrates practical experience with:

* Microsoft Azure
* Azure Virtual Networks
* Azure Subnets
* Azure Network Security Groups
* Azure Network Watcher
* Windows Server 2022
* IIS
* PowerShell
* TCP/IP troubleshooting
* HTTP troubleshooting
* Network segmentation
* Security rule analysis
* Root-cause analysis
* Incident troubleshooting
* Least-privilege access control
* Technical documentation

---

# 🏁 Project Outcome

The project successfully demonstrated a complete network troubleshooting lifecycle:

```text
Azure Infrastructure
        ↓
Network Configuration
        ↓
Windows Server Deployment
        ↓
IIS Configuration
        ↓
Baseline Connectivity
        ↓
NSG Fault Simulation
        ↓
Connectivity Failure
        ↓
Troubleshooting
        ↓
Root Cause Identification
        ↓
NSG Remediation
        ↓
Connectivity Restored
        ↓
Least-Privilege Improvement
        ↓
Final Validation
```

**Status: ✅ Completed**

**Project Focus:** Azure Networking | Cloud Troubleshooting | Network Security | Windows Server | IT Support | Cybersecurity
