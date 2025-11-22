# Azure Load Balancing Project - MST300

> **Project 2: Load Balancing Websites using Azure Internal Load Balancer**

![Architecture Diagram](./topology/architecture-diagram.png)

## 📋 Table of Contents
- [Project Overview](#project-overview)
- [Student Information](#student-information)
- [Architecture](#architecture)
- [Technologies Used](#technologies-used)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Documentation](#documentation)
- [Results](#results)
- [Acknowledgments](#acknowledgments)

---

## 🎯 Project Overview

This project implements a highly available web application infrastructure on Microsoft Azure using:
- **Internal Load Balancer** to distribute traffic between two web servers
- **Azure Bastion** for secure VM access without public IPs
- **Virtual Network Peering** to connect isolated networks
- **Private DNS Zone** for internal name resolution
- **Availability Zones** for high availability

### Business Scenario
The architecture simulates a production environment where:
- Web servers are isolated in a private network with no internet access
- Traffic is distributed across multiple availability zones for redundancy
- Client access is controlled through private networking
- All administrative access is secured through Azure Bastion

---

## 👨‍🎓 Student Information

- **Name**: Alireza Mirzaei
- **Student ID**: amirzaei7
- **Course**: MST300 - Azure
- **Semester**: Fall 2025
- **Institution**: Seneca College

---

## 🏗️ Architecture

### Network Topology

| Component | Network | Subnet | Address Space |
|-----------|---------|--------|---------------|
| Webserver VMs | MST300-vnet1 | vnet1-subnet1 | 10.73.0.0/24 |
| Azure Bastion | MST300-vnet1 | AzureBastionSubnet | 10.73.1.0/24 |
| Client VM | MST300-vnet2 | vnet2-subnet1 | 10.74.0.0/24 |

### Key Components

#### 1. **Load Balancer (MST300-lb)**
- Type: Internal
- SKU: Standard
- Frontend IP: Dynamic (10.73.0.x)
- Backend Pool: webserver1-vm, webserver2-vm
- Health Probe: HTTP on port 80
- Load Balancing Rule: TCP port 80

#### 2. **Web Servers**
- **webserver1-vm**: Windows Server 2019, Availability Zone 1, Yellow background
- **webserver2-vm**: Windows Server 2019, Availability Zone 2, Green background

#### 3. **Client VM**
- **client-vm**: Windows 10 Pro, MST300-vnet2

#### 4. **Azure Bastion**
- Secure RDP/SSH access without public IPs
- Connected to AzureBastionSubnet

#### 5. **Private DNS Zone**
- Zone Name: amirzaei7.mst300.com
- A Record: www → Load Balancer IP
- Linked to both VNets

For detailed architecture explanation, see [ARCHITECTURE.md](./ARCHITECTURE.md)

---

## 🛠️ Technologies Used

- **Microsoft Azure**
  - Virtual Machines (Windows Server 2019, Windows 10)
  - Azure Load Balancer (Internal)
  - Azure Bastion
  - Virtual Networks & Subnets
  - Virtual Network Peering
  - Private DNS Zones
  - Availability Zones

- **Web Server**
  - Internet Information Services (IIS)
  - HTML/CSS

- **Tools**
  - Azure Portal
  - PowerShell
  - Microsoft Stream (Video Recording)

---

## 📁 Project Structure
```
📦 MST300-Project2
 ┣ 📂 screenshots/          # Organized screenshots by component
 ┣ 📂 topology/             # Architecture diagrams
 ┣ 📜 README.md             # This file - project overview
 ┣ 📜 ARCHITECTURE.md       # Detailed architecture documentation
 ┣ 📜 SETUP-GUIDE.md        # Step-by-step implementation guide
```

---

## 🚀 Quick Start

### Prerequisites
- Azure for Students subscription
- Assigned IP address space from instructor
- Basic knowledge of networking and Azure

### Implementation Summary

1. **Create Resource Group** → MST300-project2-rg
2. **Setup Virtual Networks** → vnet1 (10.73.0.0/16), vnet2 (10.74.0.0/16)
3. **Configure VNet Peering** → Connect vnet1 ↔ vnet2
4. **Deploy Load Balancer** → Internal LB with backend pool
5. **Create Web Servers** → 2 VMs in different availability zones
6. **Configure IIS** → Custom HTML pages with different colors
7. **Deploy Azure Bastion** → Secure access service
8. **Setup Private DNS** → Internal name resolution
9. **Create Client VM** → Testing environment
10. **Test & Validate** → Verify load balancing functionality

For detailed implementation steps, see [SETUP-GUIDE.md](./SETUP-GUIDE.md)

---

## 📚 Documentation

### Detailed Guides

| Document | Description |
|----------|-------------|
| [ARCHITECTURE.md](./ARCHITECTURE.md) | In-depth architecture explanation, design decisions, and network diagrams |
| [SETUP-GUIDE.md](./SETUP-GUIDE.md) | Complete step-by-step implementation instructions |

### Screenshots

All screenshots are organized in the `screenshots/` directory by component:
- Resource Group configuration
- Virtual Network setup and peering
- Load Balancer configuration (backend pool, health probe, rules)
- Virtual Machine deployments
- Azure Bastion setup
- Private DNS Zone configuration
- Testing results and load balancing demonstration

---

## ✅ Results

### Successfully Implemented:

- ✅ **High Availability**: Web servers deployed across 2 availability zones
- ✅ **Load Distribution**: Traffic successfully alternates between servers
- ✅ **Secure Access**: No public IPs on VMs, all access via Azure Bastion
- ✅ **Private Networking**: Virtual network peering enables cross-VNet communication
- ✅ **Name Resolution**: Private DNS zone resolves FQDN to load balancer
- ✅ **Health Monitoring**: Load balancer health probe detects server status

### Test Results:

| Test Case | Expected Result | Actual Result | Status |
|-----------|----------------|---------------|--------|
| Access website via FQDN from client VM | Load balancer distributes requests | Pages alternate between yellow and green | ✅ Pass |
| DNS resolution | www.amirzaei7.mst300.com resolves to LB IP | Correctly resolves to 10.73.0.x | ✅ Pass |
| VNet peering connectivity | Client VM can reach webservers | Successfully connected | ✅ Pass |
| Azure Bastion access | Can connect to all VMs securely | All connections successful | ✅ Pass |
| Health probe monitoring | Both servers show healthy status | Both servers healthy | ✅ Pass |

---

## 📊 Project Statistics

- **Total Azure Resources**: 11
- **Virtual Networks**: 2
- **Subnets**: 3
- **Virtual Machines**: 3
- **Load Balancers**: 1
- **DNS Zones**: 1
- **Implementation Time**: ~3 hours
- **Total Cost**: ~$0.50/hour (Azure for Students credit)

---

## 🎓 Learning Outcomes

Through this project, I gained hands-on experience with:

1. **Load Balancing Concepts**
   - Understanding internal vs. external load balancers
   - Configuring backend pools and health probes
   - Setting up load balancing rules

2. **Azure Networking**
   - Designing multi-VNet architectures
   - Implementing virtual network peering
   - Managing private IP addressing schemes

3. **High Availability**
   - Deploying resources across availability zones
   - Understanding fault tolerance and redundancy

4. **Security Best Practices**
   - Eliminating public IP exposure
   - Using Azure Bastion for secure access
   - Implementing private DNS for internal resolution

5. **Azure IaaS Services**
   - VM deployment and configuration
   - Windows Server administration
   - IIS web server setup

---

## ⚠️ Important Notes

### Cost Management
- **Azure Bastion**: Cannot be stopped; incurs continuous charges (~$0.19/hour)
- **Virtual Machines**: Auto-stop when not in use
- **Recommendation**: Delete all resources when not testing to minimize costs

### Naming Conventions
All resources must follow the specified naming convention:
- Resource Group: `MST300-project2-rg`
- Virtual Networks: `MST300-vnet1`, `MST300-vnet2`
- Load Balancer: `MST300-lb`
- VMs: `webserver1-vm`, `webserver2-vm`, `client-vm`

### Network Requirements
- Must use assigned IP address space (10.73.0.0/16, 10.74.0.0/16)
- No overlapping address spaces between VNets
- AzureBastionSubnet must be exactly named

---

## 🔗 References

- [Azure Load Balancer Documentation](https://docs.microsoft.com/azure/load-balancer/)
- [Azure Bastion Documentation](https://docs.microsoft.com/azure/bastion/)
- [Virtual Network Peering](https://docs.microsoft.com/azure/virtual-network/virtual-network-peering-overview)
- [Azure Private DNS](https://docs.microsoft.com/azure/dns/private-dns-overview)

---

## 📝 Project Rubric Compliance

| Requirement | Status | Evidence |
|-------------|--------|----------|
| Azure Bastion connectivity | ✅ Complete | Video demo + screenshots |
| Load Balancer configuration | ✅ Complete | Configuration screenshots |
| Modified webserver pages | ✅ Complete | Yellow/Green pages captured |
| Virtual Network Peering | ✅ Complete | Peering status shown |
| Private DNS Zone | ✅ Complete | DNS records and resolution |
| Proper naming conventions | ✅ Complete | All resources named correctly |
| Assigned IP addresses used | ✅ Complete | 10.73.0.0/16, 10.74.0.0/16 |

---

## 👤 Author

**Alireza Mirzaei**
- Email: alirezamirzaei2018@gmail.com
- LinkedIn:(https://www.linkedin.com/in/alireza-mirzaei-b2a5a31a4/)
- GitHub: (https://github.com/alirezamirzaei2018-cmd)

---

## 📄 License

This project is submitted as academic work for MST300 course at Seneca College.

---

<div align="center">

**⭐ If you found this project helpful, please consider giving it a star! ⭐**

</div>
