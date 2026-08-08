# Azure Network Troubleshooting & NSG Security Lab

![Microsoft Azure](https://img.shields.io/badge/Microsoft%20Azure-Cloud-blue)
![Networking](https://img.shields.io/badge/Azure-Networking-blue)
![Security](https://img.shields.io/badge/Focus-Cloud%20Security-red)
![Troubleshooting](https://img.shields.io/badge/Focus-Network%20Troubleshooting-green)
![Windows Server](https://img.shields.io/badge/Windows%20Server-2022-blue)

## 📌 Project Overview

This project is a hands-on **Microsoft Azure Network Troubleshooting and Network Security lab** designed to simulate a real-world internal web server connectivity incident.

The environment was built using **Azure Virtual Networks, subnets, Windows Server virtual machines, IIS, and Network Security Groups (NSGs)**.

A working network baseline was established before an intentional NSG misconfiguration was introduced to simulate an HTTP connectivity failure. The incident was then investigated using **PowerShell, Windows networking tools, Azure NSG analysis, effective security rules, and Azure Network Watcher**.

The project demonstrates the complete troubleshooting lifecycle from **network configuration and baseline testing through fault simulation, investigation, remediation, and validation**.

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
* Identify and resolve a network security misconfiguration.
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


<img width="1536" height="1024" alt="Lab Architecture" src="https://github.com/user-attachments/assets/d183b07b-8c89-4c8a-b98a-cb4709166c7e" />


### Network Architecture Diagram

                     MICROSOFT AZURE
                            |
              rg-network-troubleshooting-lab
                            |
             vnet-network-troubleshooting
                     10.10.0.0/16
                            |
             +--------------+--------------+
             |                             |
       Web-Subnet                     Admin-Subnet
       10.10.1.0/24                  10.10.2.0/24
             |                             |
             v                             v
      WEB-SERVER-01                    ADMIN-VM
      Windows Server                  Windows Server
             |
             v
            IIS
          TCP 80

      nsg-web-server
             |
      NSG traffic filtering

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

The web server is protected by an Azure **Network Security Group**, allowing network traffic to be controlled through explicit security rules.

---

# 🚨 Troubleshooting Scenario

The lab simulates an internal web server connectivity incident.

The initial configuration was tested to establish a known-good baseline between `ADMIN-VM` and `WEB-SERVER-01`.

An intentional NSG misconfiguration was then introduced to block inbound HTTP traffic on TCP port `80`.

This caused the administration VM to lose connectivity to the IIS web server.

The incident was investigated using:

* PowerShell network testing
* `Test-NetConnection`
* `Invoke-WebRequest`
* IIS service validation
* Azure NSG rule analysis
* Effective security rules
* Azure Network Watcher

The investigation identified the NSG configuration as the source of the connectivity failure. The configuration was subsequently corrected and connectivity was restored.

For the complete technical investigation, see the detailed troubleshooting documentation.

---

# 📚 Project Documentation

### 🔧 Azure Network Setup

Detailed information about the Azure infrastructure, Windows Server configuration, IIS deployment, baseline testing, simulated incident, and troubleshooting process:

- **[View Azure Network Setup](./azure-network-setup.md)**

### 🔎 Troubleshooting Report

Detailed technical investigation covering the incident, root-cause analysis, remediation, least-privilege improvement, validation, and troubleshooting results:

- **[View Troubleshooting Report](./troubleshooting-report.md)**

---

# 📸 Project Evidence

The project contains **28 screenshots** documenting the complete implementation and troubleshooting lifecycle.

Evidence includes:

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
* HTTP NSG configuration
* IIS service validation

### Troubleshooting

* NSG deny rule
* Connectivity failure
* Server health check
* Local IIS connectivity
* Effective security rules
* Network Watcher failure

### Remediation

* Corrected NSG configuration
* Connectivity restored
* Network Watcher success
* Final HTTP test

📸 **[View All Lab Screenshots](./Screenshots/)**

---

# 📂 Project Structure

```text
Azure-network-troubleshooting-nsg-security-lab/
│
├── README.md
│
├── azure-network-setup.md
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

# 💡 Key Lessons Learned

### Azure Networking

* Azure VNets and subnets provide logical network segmentation.
* Private IP addresses enable communication between resources within the VNet.
* NSGs provide network-level traffic filtering and access control.
* NSG rule priority determines which security rule is evaluated first.

### Network Troubleshooting

* Establish a known-good baseline before investigating an incident.
* Validate the application and service before assuming the network is the problem.
* Compare local and remote connectivity tests.
* Examine effective security rules rather than relying only on configured rules.
* Use Azure Network Watcher to validate connectivity at the Azure network layer.

### Security

* Lower numerical NSG priorities are evaluated before higher numerical priorities.
* Incorrectly prioritised deny rules can unintentionally block legitimate traffic.
* Network access should follow the principle of least privilege.
* Configuration changes should always be followed by validation testing.

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

The project successfully demonstrated a complete **Azure network troubleshooting and security incident lifecycle**:

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

## Project Status

**Status:** ✅ Completed

---

## Author

**Ayodeji Olumide Awe**

*IT Support & Cybersecurity Professional*


**Project Focus:** Azure Networking | Cloud Troubleshooting | Network Security | Windows Server | IT Support | Cybersecurity

