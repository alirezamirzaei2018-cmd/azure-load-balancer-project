# Complete Setup Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Phase 1: Network Foundation](#phase-1-network-foundation)
3. [Phase 2: Load Balancer Setup](#phase-2-load-balancer-setup)
4. [Phase 3: Virtual Machines](#phase-3-virtual-machines)
5. [Phase 4: Web Server Configuration](#phase-4-web-server-configuration)
6. [Phase 5: Bastion Deployment](#phase-5-bastion-deployment)
7. [Phase 6: DNS Configuration](#phase-6-dns-configuration)
8. [Phase 7: Testing](#phase-7-testing)
9. [Screenshots Guide](#screenshots-guide)

---

## Phase 1: Network Foundation

### Step 1.1: Create Resource Group

1. Sign in to [Azure Portal](https://portal.azure.com)
2. Verify you're using your ODL account (top-right corner)
3. Click **Create a resource** → Search "Resource group" → **Create**

**Configuration**:
```
Subscription: Azure for Students
Resource group: MST300-project2-rg
Region: (US) East US
```

4. Click **Review + create** → **Create**

📸 **Screenshot 01**: `screenshots/01-resource-group/resource-group-overview.png`
- Show resource group name and region

---

### Step 1.2: Create Virtual Network 1 (Webservers)

1. Navigate to **Create a resource** → Search "Virtual network" → **Create**

**Basics Tab**:
```
Subscription: Azure for Students
Resource group: MST300-project2-rg
Name: MST300-vnet1
Region: (US) East US
```

**IP Addresses Tab**:
1. Remove default address space
2. Click **Add IPv4 address space**
   - Enter: `10.73.0.0/16`

3. Click **+ Add a subnet**
   - Subnet name: `vnet1-subnet1`
   - Starting address: `10.73.0.0`
   - Subnet size: `/24 (256 addresses)`

4. Click **+ Add a subnet**
   - Subnet name: `AzureBastionSubnet` ⚠️ **EXACT NAME REQUIRED**
   - Starting address: `10.73.1.0`
   - Subnet size: `/24 (256 addresses)`

**Security Tab**: Leave defaults

**Review + create** → **Create**

📸 **Screenshot 02**: `screenshots/02-virtual-networks/vnet1-overview.png`
- Show address space 10.73.0.0/16
- Show both subnets listed

---

### Step 1.3: Create Virtual Network 2 (Client)

1. **Create a resource** → "Virtual network" → **Create**

**Basics Tab**:
```
Subscription: Azure for Students
Resource group: MST300-project2-rg
Name: MST300-vnet2
Region: (US) East US
```

**IP Addresses Tab**:
1. Remove default address space
2. Add IPv4 address space: `10.74.0.0/16`
3. Add subnet:
   - Subnet name: `vnet2-subnet1`
   - Starting address: `10.74.0.0`
   - Subnet size: `/24 (256 addresses)`

**Review + create** → **Create**

📸 **Screenshot 03**: `screenshots/02-virtual-networks/vnet2-overview.png`

---

### Step 1.4: Configure Virtual Network Peering

**From vnet1 to vnet2**:

1. Go to **MST300-vnet1** → **Peerings** (left menu) → **+ Add**

**Configuration**:
```
This virtual network:
  Peering link name: vnet1-to-vnet2
  Traffic to remote virtual network: Allow
  Traffic forwarded from remote virtual network: Allow
  Virtual network gateway or Route Server: None

Remote virtual network:
  Peering link name: vnet2-to-vnet1
  Virtual network deployment model: Resource Manager
  Virtual network: MST300-vnet2
  Traffic to remote virtual network: Allow
  Traffic forwarded from remote virtual network: Allow
  Virtual network gateway or Route Server: None
```

2. Click **Add**
3. Wait for peering status to show **Connected** (both directions)

📸 **Screenshot 04**: `screenshots/02-virtual-networks/vnet-peering.png`
- Show both peering connections with "Connected" status

---

## Phase 2: Load Balancer Setup

### Step 2.1: Create Load Balancer

1. **Create a resource** → Search "Load Balancer" → **Create**

**Basics Tab**:
```
Subscription: Azure for Students
Resource group: MST300-project2-rg
Name: MST300-lb
Region: (US) East US
SKU: Standard
Type: Internal
Tier: Regional
```

**Frontend IP Configuration Tab**:
- Click **+ Add a frontend IP configuration**
```
Name: LoadBalancerFrontEnd
Virtual network: MST300-vnet1
Subnet: vnet1-subnet1 (10.73.0.0/24)
Assignment: Dynamic
Availability zone: Zone-redundant
```

**Review + create** → **Create**

📸 **Screenshot 05**: `screenshots/03-load-balancer/lb-overview.png`

---

### Step 2.2: Create Backend Pool

1. Go to **MST300-lb** → **Backend pools** (left menu) → **+ Add**

**Configuration**:
```
Name: MST300-BackendPool
Virtual network: MST300-vnet1
Backend Pool Configuration: NIC
IP Version: IPv4
```

2. Click **Save** (we'll add VMs later)

📸 **Screenshot 06**: `screenshots/03-load-balancer/backend-pool.png`

---

### Step 2.3: Create Health Probe

1. Go to **MST300-lb** → **Health probes** → **+ Add**

**Configuration**:
```
Name: MST300-HealthProbe
Protocol: HTTP
Port: 80
Path: /
Interval: 15
Unhealthy threshold: 3
```

2. Click **Add**

📸 **Screenshot 07**: `screenshots/03-load-balancer/health-probe.png`

---

### Step 2.4: Create Load Balancing Rule

1. Go to **MST300-lb** → **Load balancing rules** → **+ Add**

**Configuration**:
```
Name: MST300-HTTPRule
IP Version: IPv4
Frontend IP address: LoadBalancerFrontEnd (10.73.0.x)
Protocol: TCP
Port: 80
Backend port: 80
Backend pool: MST300-BackendPool
Health probe: MST300-HealthProbe (HTTP:80)
Session persistence: None
Idle timeout (minutes): 15
TCP reset: Enabled
Floating IP: Disabled
```

2. Click **Save**

📸 **Screenshot 08**: `screenshots/03-load-balancer/lb-rules.png`

---

## Phase 3: Virtual Machines

### Step 3.1: Create Webserver 1

1. **Create a resource** → Search "Virtual machine" → **Create**

**Basics Tab**:
```
Subscription: Azure for Students
Resource group: MST300-project2-rg
Virtual machine name: webserver1-vm
Region: (US) East US
Availability options: Availability zone
Availability zone: Zone 1
Security type: Standard
Image: Windows Server 2019 Datacenter - x64 Gen2
Size: Standard_B1s (1 vcpu, 1 GiB memory)

Administrator account:
  Username: azureuser
  Password: [Create strong password - SAVE THIS!]

Inbound port rules:
  Public inbound ports: None
```

**Disks Tab**:
```
OS disk type: Standard SSD (locally-redundant storage)
Delete with VM: Yes (checked)
```

**Networking Tab**:
```
Virtual network: MST300-vnet1
Subnet: vnet1-subnet1 (10.73.0.0/24)
Public IP: None
NIC network security group: Basic
Public inbound ports: None

Load balancing:
  Load balancing options: Azure load balancer
  Select a load balancer: MST300-lb
  Select a backend pool: MST300-BackendPool
```

**Management Tab**: Leave defaults

**Review + create** → **Create**

⏳ **Wait for deployment** (5-10 minutes)

📸 **Screenshot 09**: `screenshots/04-virtual-machines/webserver1-overview.png`
- Show VM name, zone, and network details

---

### Step 3.2: Create Webserver 2

Repeat Step 3.1 with these changes:
```
Virtual machine name: webserver2-vm
Availability zone: Zone 2
[EVERYTHING ELSE IDENTICAL]
```

📸 **Screenshot 10**: `screenshots/04-virtual-machines/webserver2-overview.png`

---

### Step 3.3: Verify Load Balancer Backend Pool

1. Go to **MST300-lb** → **Backend pools** → **MST300-BackendPool**
2. Verify both VMs are listed

📸 **Screenshot 11**: `screenshots/03-load-balancer/backend-pool-with-vms.png`
- Show both webserver1-vm and webserver2-vm in the pool

---

## Phase 4: Web Server Configuration

### Step 4.1: Deploy Azure Bastion (Needed First)

1. **Create a resource** → Search "Bastion" → **Create**

**Basics Tab**:
```
Subscription: Azure for Students
Resource group: MST300-project2-rg
Name: MST300-BastionHost
Region: (US) East US
Tier: Basic
Virtual network: MST300-vnet1
Subnet: AzureBastionSubnet (10.73.1.0/24)

Public IP address:
  Public IP address name: MST300-BastionIP
  Public IP address SKU: Standard
  Assignment: Static
```

2. **Review + create** → **Create**

⏳ **Wait 10-15 minutes** for deployment

📸 **Screenshot 12**: `screenshots/05-bastion/bastion-overview.png`

---

### Step 4.2: Configure IIS on Webserver 1

1. Go to **webserver1-vm** → **Connect** → **Bastion**
2. Enter credentials:
   - Username: `azureuser`
   - Password: [Your password]
3. Click **Connect**

**In the VM**:

4. Open **PowerShell as Administrator**
   - Right-click Start → Windows PowerShell (Admin)

5. Run these commands:
```powershell
# Install IIS
Install-WindowsFeature -name Web-Server -IncludeManagementTools

# Create custom HTML (REPLACE 'amirzaei7' with YOUR student ID)
$html = @"
<!DOCTYPE html>
<html>
<head>
    <title>amirzaei7's Windows Server</title>
</head>
<body style="background-color: yellow; font-family: Arial, sans-serif; padding: 50px;">
    <h1>This webpage is hosted on webserver1-vm</h1>
    <p>Load Balancer Demo - MST300 Project 2</p>
</body>
</html>
"@

# Write HTML to IIS default page
$html | Out-File -FilePath C:\inetpub\wwwroot\iisstart.htm -Encoding UTF8

# Restart IIS to apply changes
iisreset
```

6. Test locally:
   - Open Internet Explorer
   - Go to `http://localhost`
   - Should see YELLOW page with "webserver1-vm"

📸 **Screenshot 13**: `screenshots/04-virtual-machines/webserver1-iis-local.png`
- Browser showing yellow page

7. Disconnect Bastion session

---

### Step 4.3: Configure IIS on Webserver 2

1. Connect to **webserver2-vm** via Bastion
2. Open **PowerShell as Administrator**
3. Run these commands:
```powershell
# Install IIS
Install-WindowsFeature -name Web-Server -IncludeManagementTools

# Create custom HTML (REPLACE 'amirzaei7' with YOUR student ID)
$html = @"
<!DOCTYPE html>
<html>
<head>
    <title>amirzaei7's WebPage</title>
</head>
<body style="background-color: green; font-family: Arial, sans-serif; padding: 50px;">
    <h1>This webpage is hosted on webserver2-vm</h1>
    <p>Load Balancer Demo - MST300 Project 2</p>
</body>
</html>
"@

# Write HTML to IIS default page
$html | Out-File -FilePath C:\inetpub\wwwroot\iisstart.htm -Encoding UTF8

# Restart IIS
iisreset
```

4. Test locally: `http://localhost` → Should see GREEN page

📸 **Screenshot 14**: `screenshots/04-virtual-machines/webserver2-iis-local.png`

---

## Phase 5: Client VM

### Step 5.1: Create Client VM

1. **Create a resource** → "Virtual machine" → **Create**

**Basics Tab**:
```
Resource group: MST300-project2-rg
Virtual machine name: client-vm
Region: (US) East US
Availability options: No infrastructure redundancy required
Image: Windows 10 Pro (latest version)
Size: Standard_B2s (2 vcpus, 4 GiB memory)

Administrator account:
  Username: azureuser
  Password: [Same or different password]

Inbound port rules:
  Public inbound ports: None
```

**Networking Tab**:
```
Virtual network: MST300-vnet2  ⚠️ DIFFERENT VNET!
Subnet: vnet2-subnet1 (10.74.0.0/24)
Public IP: None
NIC network security group: Basic
Public inbound ports: None
Load balancing: None
```

**Review + create** → **Create**

📸 **Screenshot 15**: `screenshots/04-virtual-machines/client-vm-overview.png`
- Highlight that it's in vnet2, not vnet1

---

## Phase 6: DNS Configuration

### Step 6.1: Create Private DNS Zone

1. **Create a resource** → Search "Private DNS zone" → **Create**

**Configuration**:
```
Subscription: Azure for Students
Resource group: MST300-project2-rg
Name: amirzaei7.mst300.com  ⚠️ Use YOUR student ID!
```

2. **Review + create** → **Create**

📸 **Screenshot 16**: `screenshots/06-private-dns/dns-zone-overview.png`

---

### Step 6.2: Link Virtual Networks to DNS Zone

1. Go to **amirzaei7.mst300.com** → **Virtual network links** → **+ Add**

**Link 1 (vnet1)**:
```
Link name: vnet1-link
Virtual network: MST300-vnet1
Enable auto registration: ❌ UNCHECKED
```

2. Click **OK**

3. **+ Add** again

**Link 2 (vnet2)**:
```
Link name: vnet2-link
Virtual network: MST300-vnet2
Enable auto registration: ❌ UNCHECKED
```

4. Click **OK**

📸 **Screenshot 17**: `screenshots/06-private-dns/virtual-network-links.png`
- Show both vnet1-link and vnet2-link

---

### Step 6.3: Create A Record for Load Balancer

**First, get Load Balancer IP**:
1. Go to **MST300-lb** → **Frontend IP configuration**
2. Copy the **Private IP address** (e.g., 10.73.0.6)

**Create A Record**:
1. Go to **amirzaei7.mst300.com** → **+ Record set**

**Configuration**:
```
Name: www
Type: A - Address record
TTL: 1
TTL unit: Hours
IP address: [Paste Load Balancer IP, e.g., 10.73.0.6]
```

2. Click **OK**

📸 **Screenshot 18**: `screenshots/06-private-dns/a-record.png`
- Show www A record with LB IP

---

## Phase 7: Testing

### Step 7.1: Connect to Client VM

1. Go to **client-vm** → **Connect** → **Bastion**
2. Enter credentials → **Connect**

📸 **Screenshot 19**: `screenshots/07-testing/client-vm-desktop.png`
- Show Windows 10 desktop

---

### Step 7.2: Test DNS Resolution

1. Open **Command Prompt**
2. Run: `nslookup www.amirzaei7.mst300.com`

**Expected output**:
```
Server:  UnKnown
Address:  168.63.129.16

Name:    www.amirzaei7.mst300.com
Address:  10.73.0.6
```

📸 **Screenshot 20**: `screenshots/07-testing/nslookup-result.png`

---

### Step 7.3: Test Load Balancing

1. Open **Microsoft Edge** (or Internet Explorer)
2. **Press Ctrl+Shift+P** to open InPrivate window
3. Navigate to: `http://www.amirzaei7.mst300.com`

**First load** → Should see either YELLOW or GREEN page

📸 **Screenshot 21**: `screenshots/07-testing/webserver1-yellow.png`
OR
📸 **Screenshot 22**: `screenshots/07-testing/webserver2-green.png`

4. **Close the InPrivate window**
5. **Open NEW InPrivate window**
6. Navigate again: `http://www.amirzaei7.mst300.com`

**Second load** → Should see the OTHER color

📸 **Screenshot 23**: `screenshots/07-testing/load-balancing-demo.png`
- Capture showing both pages (you may need to take 2 separate screenshots)

---

### Step 7.4: Verify Load Balancer Health

1. From Azure Portal: **MST300-lb** → **Metrics**
2. Add metric: **Data Path Availability**
3. Should show 100% (both backends healthy)

📸 **Screenshot 24**: `screenshots/03-load-balancer/health-metrics.png`

---

## Screenshots Guide

### Naming Convention
```
screenshots/
├── 01-resource-group/
│   └── resource-group-overview.png
├── 02-virtual-networks/
│   ├── vnet1-overview.png
│   ├── vnet2-overview.png
│   └── vnet-peering.png
├── 03-load-balancer/
│   ├── lb-overview.png
│   ├── backend-pool.png
│   ├── backend-pool-with-vms.png
│   ├── health-probe.png
│   ├── lb-rules.png
│   └── health-metrics.png
├── 04-virtual-machines/
│   ├── webserver1-overview.png
│   ├── webserver1-iis-local.png
│   ├── webserver2-overview.png
│   ├── webserver2-iis-local.png
│   └── client-vm-overview.png
├── 05-bastion/
│   ├── bastion-overview.png
│   └── bastion-connection.png
├── 06-private-dns/
│   ├── dns-zone-overview.png
│   ├── virtual-network-links.png
│   └── a-record.png
└── 07-testing/
    ├── client-vm-desktop.png
    ├── nslookup-result.png
    ├── webserver1-yellow.png
    ├── webserver2-green.png
    └── load-balancing-demo.png
```

### Screenshot Checklist

Before each screenshot, ensure:
- ✅ Resource names are visible
- ✅ Configuration values are clear
- ✅ No sensitive information (passwords, etc.)
- ✅ Azure portal breadcrumb shows location
- ✅ Date/time visible when needed



