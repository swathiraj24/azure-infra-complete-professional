# COMPLETE HANDS-ON LABS: Master ALL Azure Networking (Pro-Level, A-to-Z)

## Table of Contents
1. Lab 0: Setup & Prerequisites
2. Lab 1: Foundation - VNets, Subnets, NICs
3. Lab 2: Security - NSGs, ASGs
4. Lab 3: Service Access - Service Endpoints & Private Endpoints (intro)
5. Lab 4: Routing - Route Tables, UDRs, Traffic Control
6. Lab 5: VNet Peering (Multi-VNet Connectivity)
7. Lab 6: Load Balancing - Azure Load Balancer
8. **Lab 7: Hybrid Connectivity Deep Dive - VPN Gateway, Site-to-Site, Point-to-Site, On-Prem Simulation**
9. **Lab 8: DNS Deep Dive - Azure DNS, Private DNS Zones, Conditional Forwarding, DNS Private Resolver, Hybrid Resolution**
10. **Lab 9: Private Link Deep Dive - Private Link Service, Private Endpoints for PaaS, NAT Resolution, DNS Integration**
11. Lab 10: Azure Firewall & DDoS Protection
12. Lab 11: Application Gateway with WAF
13. Lab 12: Monitoring, Flow Logs & Troubleshooting (Network Watcher)
14. Lab 13: Capstone - Complete Real-World Hub-and-Spoke Architecture

**Total Hands-On Time: 12-16 hours (full pro-level depth)**
**All Labs: Copy-Paste Ready, Nothing Skipped**

> **What's new in this edition:** Full hands-on labs for VPN Gateway (Site-to-Site AND Point-to-Site) with a real simulated on-prem network, a complete DNS deep dive (Private DNS Zones, Conditional Forwarding, Azure DNS Private Resolver, split-horizon DNS, hybrid name resolution), a full Private Link Service lab (publishing YOUR OWN service privately, not just consuming PaaS), and the old "roadmap" placeholders for Firewall/WAF/Monitoring/Capstone are now fully scripted, copy-paste hands-on labs.

---

# LAB 0: SETUP & PREREQUISITES

## What You'll Do
- Login to Azure
- Create resource group
- Understand basic commands
- Verify permissions

## Step-by-Step

### Step 1: Login to Azure (Student Account)

```bash
# Open Terminal/PowerShell and run:
az login

# This opens browser for authentication
# Login with your student credentials
# After success, terminal will show subscription info
```

**Expected Output:**
```
[
  {
    "cloudName": "AzureCloud",
    "homeTenantId": "xxxx",
    "id": "your-subscription-id",
    "isDefault": true,
    "name": "Azure for Students",
    "state": "Enabled",
    "tenantId": "xxxx",
    "user": {
      "name": "your-email@school.edu",
      "type": "user"
    }
  }
]
```

### Step 2: Verify Login

```bash
# Confirm you're logged in
az account show

# Output should show your subscription name and ID
```

### Step 3: Set Default Subscription (if multiple)

```bash
# List all subscriptions
az account list --output table

# Set default (replace with your subscription ID)
az account set --subscription "your-subscription-id"
```

### Step 4: Create Resource Group

```bash
# ALL labs will use this resource group
# Replace with your preferred region if not US

az group create \
  --name networking-labs-rg \
  --location eastus

# Verify creation
az group show \
  --name networking-labs-rg \
  --output table
```

**Expected Output:**
```
Name                 Location    ProvisioningState
-------------------  ----------  -------------------
networking-labs-rg   eastus      Succeeded
```

### Step 5: Install Required Tools

```bash
# Azure CLI (already installed)
az version

# Optional: Install extensions
az extension add --name azure-devops
az extension add --name application-insights
```

### Step 6: Create Variables File (for all labs)

```bash
# Create file: ~/.azure-lab-vars.sh
cat > ~/.azure-lab-vars.sh << 'EOF'
#!/bin/bash

# Lab Configuration Variables
export RG="networking-labs-rg"
export LOCATION="eastus"
export LOCATION_SECONDARY="westus"

# VNet Configuration
export VNET_PRIMARY="production-vnet"
export VNET_PRIMARY_SPACE="10.0.0.0/16"
export SUBNET_FRONTEND="frontend-subnet"
export SUBNET_FRONTEND_SPACE="10.0.1.0/24"
export SUBNET_BACKEND="backend-subnet"
export SUBNET_BACKEND_SPACE="10.0.2.0/24"
export SUBNET_DATABASE="database-subnet"
export SUBNET_DATABASE_SPACE="10.0.3.0/24"
export SUBNET_FIREWALL="AzureFirewallSubnet"
export SUBNET_FIREWALL_SPACE="10.0.4.0/24"
export SUBNET_GATEWAY="GatewaySubnet"
export SUBNET_GATEWAY_SPACE="10.0.5.0/24"
export SUBNET_BASTION="AzureBastionSubnet"
export SUBNET_BASTION_SPACE="10.0.254.0/24"

# DR VNet
export VNET_DR="dr-vnet"
export VNET_DR_SPACE="10.1.0.0/16"

# Simulated On-Premises VNet (stands in for a real datacenter for VPN labs)
export VNET_ONPREM="onprem-vnet"
export VNET_ONPREM_SPACE="192.168.0.0/16"
export SUBNET_ONPREM_LAN="onprem-lan-subnet"
export SUBNET_ONPREM_LAN_SPACE="192.168.1.0/24"
export SUBNET_ONPREM_GATEWAY="GatewaySubnet"
export SUBNET_ONPREM_GATEWAY_SPACE="192.168.255.0/27"

# VPN Gateway names
export VPN_GW_AZURE="azure-vpn-gw"
export VPN_GW_ONPREM="onprem-vpn-gw"
export LOCAL_GATEWAY="onprem-local-gateway"
export VPN_CONNECTION="azure-to-onprem-connection"
export VPN_SHARED_KEY="MySuperSecretKey123!"

# Private DNS / Hybrid DNS
export PRIVATE_DNS_ZONE_INTERNAL="corp.internal"
export DNS_RESOLVER_NAME="azure-dns-resolver"
export SUBNET_DNS_INBOUND="dns-inbound-subnet"
export SUBNET_DNS_INBOUND_SPACE="10.0.6.0/28"
export SUBNET_DNS_OUTBOUND="dns-outbound-subnet"
export SUBNET_DNS_OUTBOUND_SPACE="10.0.6.16/28"

# VM Configuration
export VM_ADMIN_USER="azureuser"
export VM_WEB_1="web-server-1"
export VM_WEB_2="web-server-2"
export VM_APP_1="app-server-1"
export VM_DB_1="db-server-1"

# Storage
export STORAGE_ACCOUNT="labstg$(date +%s | tail -c 5)"

# DNS
export DNS_ZONE="labdemo.com"

# Colors for output
export RED='\033[0;31m'
export GREEN='\033[0;32m'
export YELLOW='\033[1;33m'
export BLUE='\033[0;34m'
export NC='\033[0m' # No Color

# Function to print colored output
print_info() {
  echo -e "${BLUE}[INFO]${NC} $1"
}

print_success() {
  echo -e "${GREEN}[SUCCESS]${NC} $1"
}

print_error() {
  echo -e "${RED}[ERROR]${NC} $1"
}

print_warning() {
  echo -e "${YELLOW}[WARNING]${NC} $1"
}

EOF

# Load variables
source ~/.azure-lab-vars.sh

# Verify
print_info "Lab variables loaded successfully"
print_info "Resource Group: $RG"
print_info "Primary VNet: $VNET_PRIMARY ($VNET_PRIMARY_SPACE)"
```

### Step 7: Verify Lab Setup

```bash
# List existing resource groups
az group list --output table

# Check quota
az vm list-skus --resource-group $RG --location $LOCATION | head -20

print_success "Lab 0 Setup Complete!"
```

---

# LAB 1: FOUNDATION - VNets, Subnets, NICs, IPs

## What You'll Learn
- Create Virtual Networks with CIDR planning
- Design subnet strategy
- Create Network Interfaces
- Manage Public/Private IPs
- Understand Azure network architecture

## Deep Dive: VNet Architecture

### Understanding CIDR Notation

```
10.0.0.0/16 = 10.0.0.0 - 10.0.255.255 (65,536 IPs)
  ├─ /24 = 256 IPs per subnet (254 usable)
  ├─ /25 = 128 IPs per subnet
  ├─ /26 = 64 IPs per subnet
  └─ /27 = 32 IPs per subnet

Azure Reserved IPs per Subnet:
  .0   = Network address
  .1   = Default gateway (Azure router)
  .2   = Azure DNS (primary)
  .3   = Azure DNS (secondary)
  .255 = Broadcast address

Usable Range: .4 to .254
```

## Hands-On Lab: Create Production VNet

### Step 1: Design Your Network

```bash
# Define your network architecture (commented in code)

# Network Design:
# Organization: E-Commerce Platform
# Primary Region: East US
# 
# VNet: 10.0.0.0/16
# ├── Frontend Tier: 10.0.1.0/24 (web servers, public)
# ├── App Tier: 10.0.2.0/24 (API servers, private)
# ├── Database Tier: 10.0.3.0/24 (SQL, private)
# ├── Management: 10.0.4.0/24 (Bastion, private)
# ├── Gateway: 10.0.5.0/24 (VPN/ExpressRoute)
# └── Firewall: 10.0.254.0/24 (Azure Firewall)

print_info "Creating VNet with the following design:"
cat << 'DESIGN'
┌────────────────────────────────────────────┐
│ VNet: production-vnet (10.0.0.0/16)        │
├────────────────────────────────────────────┤
│ ┌──────────────────────────────────────┐   │
│ │ Frontend (10.0.1.0/24)               │   │
│ │ ├─ Web Server 1 (10.0.1.10)         │   │
│ │ ├─ Web Server 2 (10.0.1.11)         │   │
│ │ └─ Load Balancer (public IP)        │   │
│ └──────────────────────────────────────┘   │
│                    ↓                         │
│ ┌──────────────────────────────────────┐   │
│ │ Backend (10.0.2.0/24)                │   │
│ │ ├─ API Server 1 (10.0.2.10)         │   │
│ │ ├─ API Server 2 (10.0.2.11)         │   │
│ │ └─ Private (no public IP)           │   │
│ └──────────────────────────────────────┘   │
│                    ↓                         │
│ ┌──────────────────────────────────────┐   │
│ │ Database (10.0.3.0/24)               │   │
│ │ ├─ SQL Server (10.0.3.10)           │   │
│ │ └─ Private (VNet access only)       │   │
│ └──────────────────────────────────────┘   │
│                    ↓                         │
│ ┌──────────────────────────────────────┐   │
│ │ Management (10.0.4.0/24)             │   │
│ │ └─ Bastion Host                      │   │
│ └──────────────────────────────────────┘   │
└────────────────────────────────────────────┘
DESIGN
```

### Step 2: Create Primary VNet

```bash
# Command breakdown:
# --resource-group: Put in our lab RG
# --name: VNet name
# --address-prefix: CIDR range (10.0.0.0/16 = 65,536 IPs)
# --subnet-name: First subnet name
# --subnet-prefix: First subnet CIDR (10.0.1.0/24 = 254 usable IPs)

print_info "Creating VNet: $VNET_PRIMARY"

az network vnet create \
  --resource-group $RG \
  --name $VNET_PRIMARY \
  --address-prefix $VNET_PRIMARY_SPACE \
  --subnet-name $SUBNET_FRONTEND \
  --subnet-prefix $SUBNET_FRONTEND_SPACE \
  --location $LOCATION

print_success "VNet created!"

# Verify
az network vnet show \
  --resource-group $RG \
  --name $VNET_PRIMARY \
  --output table
```

**What This Does:**
- Creates VNet with IP range 10.0.0.0/16
- Automatically creates first subnet (frontend-subnet 10.0.1.0/24)
- Sets up default route table
- Configures DNS to use Azure default

### Step 3: Add Additional Subnets

```bash
# Now add more subnets (backend, database, etc.)

print_info "Creating Backend Subnet..."
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_BACKEND \
  --address-prefix $SUBNET_BACKEND_SPACE

print_success "Backend subnet created!"

print_info "Creating Database Subnet..."
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_DATABASE \
  --address-prefix $SUBNET_DATABASE_SPACE

print_success "Database subnet created!"

# Note: Firewall and Gateway subnets are created later
# (They have specific naming requirements)

# Verify all subnets
print_info "Verifying subnets..."
az network vnet subnet list \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --output table
```

**Expected Output:**
```
Name                       AddressPrefix    ProvisioningState
-----------------------  ----------------  -------------------
frontend-subnet            10.0.1.0/24       Succeeded
backend-subnet             10.0.2.0/24       Succeeded
database-subnet            10.0.3.0/24       Succeeded
```

### Step 4: Understand Subnet IP Allocation

```bash
# Get detailed subnet info
print_info "Detailed subnet information:"

az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_FRONTEND \
  --output json | jq '{
    Name: .name,
    AddressPrefix: .addressPrefix,
    ProvisioningState: .provisioningState,
    IPConfigurations: (.ipConfigurations | length),
    ServiceEndpoints: .serviceEndpoints,
    Delegations: .delegations
  }'

# Explanation of output:
cat << 'EXPLANATION'
Frontend Subnet: 10.0.1.0/24
├── Network Address: 10.0.1.0 (reserved)
├── Gateway: 10.0.1.1 (reserved - Azure router)
├── DNS Primary: 10.0.1.2 (reserved)
├── DNS Secondary: 10.0.1.3 (reserved)
├── Usable IPs: 10.0.1.4 to 10.0.1.254 (251 VMs max)
├── Broadcast: 10.0.1.255 (reserved)
└── Service Endpoints: None yet
EXPLANATION

print_info "Subnet IP capacity:"
echo "Total IPs in /24: 256"
echo "Reserved by Azure: 5 (.0, .1, .2, .3, .255)"
echo "Available for VMs: 251"
```

### Step 5: Create Public IP (Static)

```bash
# Public IPs are used for internet-facing resources
# Important: Public IP in Azure is a separate resource

print_info "Creating Static Public IP for Load Balancer..."

az network public-ip create \
  --resource-group $RG \
  --name lb-public-ip \
  --sku Standard \
  --allocation-method Static \
  --version IPv4 \
  --idle-timeout 30 \
  --dns-name lab-frontend

print_success "Public IP created!"

# Get the actual public IP address
PUBLIC_IP=$(az network public-ip show \
  --resource-group $RG \
  --name lb-public-ip \
  --query ipAddress \
  --output tsv)

print_success "Your Public IP: $PUBLIC_IP"

# View public IP details
az network public-ip show \
  --resource-group $RG \
  --name lb-public-ip \
  --output table
```

**Key Points:**
```
Static IP: Never changes (until you delete)
Dynamic IP: Changes when VM stops/deallocates
Standard SKU: For production (has SLA)
Basic SKU: Deprecated, avoid
```

### Step 6: Create Network Interface Card (NIC)

```bash
# NICs are how VMs connect to VNets
# Each NIC has:
# - Private IP (from subnet)
# - Optional public IP
# - Optional NSG association
# - DNS settings
# - MAC address

print_info "Creating NIC for Web Server 1..."

az network nic create \
  --resource-group $RG \
  --name web-server-1-nic \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_FRONTEND \
  --public-ip-address lb-public-ip \
  --private-ip-address 10.0.1.10 \
  --ip-forwarding false \
  --enable-accelerated-networking false

print_success "NIC created!"

# View NIC details
az network nic show \
  --resource-group $RG \
  --name web-server-1-nic \
  --output json | jq '{
    Name: .name,
    VNet: .ipConfigurations[0].subnet.id,
    PrivateIP: .ipConfigurations[0].privateIPAddress,
    PublicIP: .ipConfigurations[0].publicIPAddress,
    NSG: .networkSecurityGroup,
    EnableAccelerated: .enableAcceleratedNetworking
  }'

# Create NIC for App Server (without public IP - private only)
print_info "Creating NIC for App Server 1 (private only)..."

az network nic create \
  --resource-group $RG \
  --name app-server-1-nic \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_BACKEND \
  --private-ip-address 10.0.2.10 \
  --ip-forwarding false

print_success "App server NIC created!"

# Create NIC for Database Server
print_info "Creating NIC for Database Server 1..."

az network nic create \
  --resource-group $RG \
  --name db-server-1-nic \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_DATABASE \
  --private-ip-address 10.0.3.10 \
  --ip-forwarding false

print_success "Database server NIC created!"
```

### Step 7: Verify All Networking Resources

```bash
# List all VNets
print_info "All VNets in resource group:"
az network vnet list --resource-group $RG --output table

# List all subnets
print_info "All Subnets:"
az network vnet subnet list \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --output table

# List all NICs
print_info "All Network Interfaces:"
az network nic list --resource-group $RG --output table

# List all Public IPs
print_info "All Public IPs:"
az network public-ip list --resource-group $RG --output table
```

### Step 8: Test Network Connectivity (Dry Run)

```bash
# We can't test yet without VMs, but we can verify configuration

print_info "Verifying network configuration..."

# Check that frontend NIC has correct settings
FRONTEND_NIC_ID=$(az network nic show \
  --resource-group $RG \
  --name web-server-1-nic \
  --query id -o tsv)

print_info "Frontend NIC: $FRONTEND_NIC_ID"

# Check effective routes (none defined yet, using defaults)
az network nic show-effective-route-table \
  --resource-group $RG \
  --name web-server-1-nic \
  --output table

print_success "Lab 1 Complete!"
print_info "You now have:"
echo "  ✓ VNet: $VNET_PRIMARY ($VNET_PRIMARY_SPACE)"
echo "  ✓ 3 Subnets: Frontend, Backend, Database"
echo "  ✓ 3 NICs: Web, App, Database servers"
echo "  ✓ 1 Public IP: For load balancer"
```

---

# LAB 2: SECURITY - NSGs, ASGs, Firewall Rules

## What You'll Learn
- Deep dive into NSG concepts
- Create security rules with proper ordering
- Understand stateful firewalling
- Implement defense-in-depth
- Application Security Groups

## Deep Dive: NSG Rule Priority & Matching

```
Rules processed in PRIORITY order:
100-1000: Critical production rules
1000-2000: Standard rules
2000-3000: Development/exception rules
3000-4096: Emergency blocks

Flow:
┌─ Packet arrives
├─ Check rules in priority order (100, 101, 102...)
├─ First matching rule wins (ALLOW or DENY)
└─ Default: DENY (implicit)

Example:
Priority 100: Allow HTTP from anywhere → MATCHES → ALLOW (stop checking)
Priority 200: Allow HTTPS from anywhere → SKIPPED (already matched)
```

## Hands-On Lab: Create Security Architecture

### Step 1: Understand NSG Stateful Behavior

```bash
# NSG is STATEFUL:
# If you allow inbound traffic, response is automatically allowed outbound
# If you allow outbound traffic, response is NOT automatically allowed inbound

# Example:
# VM initiates outbound connection on port 8080 (outbound rule allows it)
# Server responds on port 8080 (return traffic auto-allowed, NO inbound rule needed)

print_info "NSG Statefulness Explanation:"
cat << 'STATEFUL'
┌─ Outbound Rule: Allow *:* → *:8080
├─ VM sends: SYN to port 8080
├─ Server responds with: SYN-ACK from port 8080
├─ Is return traffic blocked? NO! (NSG is stateful)
└─ Response is automatically allowed (regardless of inbound rules)

BUT:
┌─ No outbound rule allowing 8080
├─ VM tries to send to port 8080
├─ NSG blocks (default deny outbound not set, allows all)
└─ Still works because of implicit allow all outbound

RESTRICTION:
┌─ Default deny all outbound (set later)
├─ VM tries to send to port 8080 (no outbound rule)
├─ NSG blocks
└─ Connection fails (no automatic return)
STATEFUL
```

### Step 2: Create NSG for Frontend (Web Tier)

```bash
# Frontend NSG strategy:
# - ALLOW: HTTP (80) from Internet
# - ALLOW: HTTPS (443) from Internet
# - ALLOW: SSH (22) from specific IP (office)
# - ALLOW: Outbound to Backend on port 8080
# - DENY: Everything else

print_info "Creating NSG for Frontend Tier..."

az network nsg create \
  --resource-group $RG \
  --name nsg-frontend \
  --location $LOCATION

print_success "Frontend NSG created!"

# Rule 1: Allow HTTP from Internet
print_info "Adding rule: Allow HTTP from Internet..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --name allow-http \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 80

print_success "HTTP rule added!"

# Rule 2: Allow HTTPS from Internet
print_info "Adding rule: Allow HTTPS from Internet..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --name allow-https \
  --priority 101 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443

print_success "HTTPS rule added!"

# Rule 3: Allow SSH from Office (replace with your IP)
# Get your public IP: curl ifconfig.me
print_info "Adding rule: Allow SSH from Office..."
OFFICE_IP="203.0.113.0/24"  # Replace with your IP/CIDR

az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --name allow-ssh-office \
  --priority 102 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes $OFFICE_IP \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22

print_success "SSH rule added!"

# Rule 4: Allow Azure Health Probes
print_info "Adding rule: Allow Azure Health Checks..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --name allow-lb-health \
  --priority 103 \
  --direction Inbound \
  --access Allow \
  --protocol '*' \
  --source-address-prefixes AzureLoadBalancer \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges '*'

print_success "Health check rule added!"

# Rule 5: Allow outbound to Backend API servers (10.0.2.0/24 on port 8080)
print_info "Adding rule: Allow outbound to Backend..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --name allow-backend-outbound \
  --priority 100 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes 10.0.2.0/24 \
  --destination-port-ranges 8080

print_success "Backend outbound rule added!"

# Rule 6: Allow outbound DNS (needed for domain lookups)
print_info "Adding rule: Allow DNS outbound..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --name allow-dns-outbound \
  --priority 101 \
  --direction Outbound \
  --access Allow \
  --protocol Udp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 53

print_success "DNS rule added!"

# Rule 7: Allow outbound HTTPS to Internet (for updates, APIs)
print_info "Adding rule: Allow HTTPS outbound..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --name allow-https-outbound \
  --priority 102 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443

print_success "HTTPS outbound rule added!"

# Verify all rules
print_info "Verifying Frontend NSG rules..."
az network nsg rule list \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --output table
```

**Output Should Show:**
```
Name                        Priority  Direction  Access  Protocol  SourcePrefix  DestinationPrefix  DestinationPort
---------------------------  --------  ---------  ------  --------  ------------- -------------------- ----------------
allow-http                  100       Inbound    Allow   Tcp       *             *                  80
allow-https                 101       Inbound    Allow   Tcp       *             *                  443
allow-ssh-office            102       Inbound    Allow   Tcp       203.0.113.0/24 *                 22
allow-lb-health             103       Inbound    Allow   *         AzureLoadBalancer * *
allow-backend-outbound      100       Outbound   Allow   Tcp       *             10.0.2.0/24        8080
allow-dns-outbound          101       Outbound   Allow   Udp       *             *                  53
allow-https-outbound        102       Outbound   Allow   Tcp       *             *                  443
DenyAllInbound              65000     Inbound    Deny    *         *             *                  *
AllowVnetOutBound           65001     Outbound   Allow   *         VirtualNetwork VirtualNetwork     *
AllowInternetOutBound       65002     Outbound   Allow   *         *             Internet           *
DenyAllOutbound             65003     Outbound   Deny    *         *             *                  *
```

### Step 3: Create NSG for Backend (App Tier)

```bash
# Backend NSG strategy:
# - ALLOW: Inbound ONLY from Frontend subnet (10.0.1.0/24) on port 8080
# - ALLOW: Inbound from Management subnet (10.0.4.0/24) on SSH
# - ALLOW: Outbound to Database subnet (10.0.3.0/24) on port 3306
# - ALLOW: Outbound for DNS and HTTPS
# - DENY: Everything else

print_info "Creating NSG for Backend Tier..."

az network nsg create \
  --resource-group $RG \
  --name nsg-backend \
  --location $LOCATION

# Rule 1: Allow from Frontend on app port
print_info "Adding rule: Allow from Frontend..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-backend \
  --name allow-from-frontend \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes 10.0.1.0/24 \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 8080

# Rule 2: Allow SSH from Management subnet
print_info "Adding rule: Allow SSH from Management..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-backend \
  --name allow-ssh-management \
  --priority 101 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes 10.0.4.0/24 \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22

# Rule 3: Outbound to Database
print_info "Adding rule: Allow to Database..."
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-backend \
  --name allow-database-outbound \
  --priority 100 \
  --direction Outbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes 10.0.3.0/24 \
  --destination-port-ranges 3306

# Rule 4: Outbound DNS
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-backend \
  --name allow-dns-outbound \
  --priority 101 \
  --direction Outbound \
  --access Allow \
  --protocol Udp \
  --source-address-prefixes '*' \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 53

print_success "Backend NSG created!"
```

### Step 4: Create NSG for Database (DB Tier)

```bash
# Database NSG - most restrictive
# - ALLOW: Inbound ONLY from Backend subnet on port 3306 (MySQL)
# - ALLOW: Inbound from Management on SSH (for maintenance)
# - NO outbound (databases don't initiate)

print_info "Creating NSG for Database Tier..."

az network nsg create \
  --resource-group $RG \
  --name nsg-database \
  --location $LOCATION

# Rule 1: Allow from Backend apps
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-database \
  --name allow-from-backend \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes 10.0.2.0/24 \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 3306

# Rule 2: Allow SSH from Management
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-database \
  --name allow-ssh-management \
  --priority 101 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes 10.0.4.0/24 \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 22

# Rule 3: Allow backup connections from Azure
az network nsg rule create \
  --resource-group $RG \
  --nsg-name nsg-database \
  --name allow-backup \
  --priority 102 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --source-address-prefixes AzureBackup \
  --source-port-ranges '*' \
  --destination-address-prefixes '*' \
  --destination-port-ranges 443

print_success "Database NSG created!"
```

### Step 5: Create Application Security Groups (ASGs)

```bash
# ASGs allow dynamic grouping of VMs
# Instead of hardcoding IPs in NSG rules, reference ASGs
# When you add VM to ASG, rules automatically apply

print_info "Creating Application Security Groups..."

az network asg create \
  --resource-group $RG \
  --name asg-web-servers \
  --location $LOCATION

az network asg create \
  --resource-group $RG \
  --name asg-app-servers \
  --location $LOCATION

az network asg create \
  --resource-group $RG \
  --name asg-database-servers \
  --location $LOCATION

print_success "ASGs created!"

# Later, when we create VMs, we'll assign NICs to ASGs
# This way, if we add new web server, it automatically gets web-server rules
```

### Step 6: Associate NSGs with Subnets

```bash
# NSGs are associated with subnets (or individual NICs)
# Subnet-level NSG applies to all NICs in that subnet

print_info "Associating NSGs with subnets..."

# Associate Frontend NSG
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_FRONTEND \
  --network-security-group nsg-frontend

# Associate Backend NSG
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_BACKEND \
  --network-security-group nsg-backend

# Associate Database NSG
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_DATABASE \
  --network-security-group nsg-database

print_success "NSGs associated!"

# Verify
print_info "Verifying subnet-NSG associations..."
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_FRONTEND \
  --query networkSecurityGroup
```

### Step 7: Test NSG Rules (Simulation)

```bash
# We'll test actual connectivity after creating VMs
# For now, let's verify rule logic

print_info "NSG Rule Verification..."

# View all frontend NSG rules
az network nsg rule list \
  --resource-group $RG \
  --nsg-name nsg-frontend \
  --output table

cat << 'ANALYSIS'
┌─ NSG Rule Analysis: Frontend Subnet
├─ Inbound:
│  ├─ Priority 100: HTTP (80) ← INTERNET
│  ├─ Priority 101: HTTPS (443) ← INTERNET
│  ├─ Priority 102: SSH (22) ← OFFICE_IP only
│  ├─ Priority 103: Health checks ← Azure LB
│  └─ Priority 65000: All others → DENY (implicit)
│
├─ Outbound:
│  ├─ Priority 100: TCP/8080 → Backend
│  ├─ Priority 101: UDP/53 (DNS) → Any
│  ├─ Priority 102: TCP/443 (HTTPS) → Any
│  └─ Priority 65002: Internet → Any (implicit allow)
│
└─ Result: Web servers can:
   ✓ Receive HTTP/HTTPS from internet
   ✓ Send to app servers on port 8080
   ✗ Cannot accept SSH except from office
   ✗ Cannot access database directly
ANALYSIS

print_success "Lab 2 Complete!"
print_info "You now have:"
echo "  ✓ 3 NSGs (Frontend, Backend, Database)"
echo "  ✓ 20+ Security rules (organized by priority)"
echo "  ✓ 3 ASGs for dynamic VM grouping"
echo "  ✓ Defense-in-depth security architecture"
```

---

# LAB 3: SERVICE ACCESS - Endpoints & Private Endpoints

## What You'll Learn
- Service Endpoints (VNet-level access control)
- Private Endpoints (private IP access)
- Private DNS zones
- Service-to-VNet integration

## Hands-On Lab: Secure Azure Services Access

### Step 1: Create Storage Account for Testing

```bash
# We'll use storage account to demonstrate service endpoints vs private endpoints

print_info "Creating Storage Account..."

# Generate unique name (storage names must be globally unique)
STORAGE_NAME="labstg$(openssl rand -hex 3)"

az storage account create \
  --resource-group $RG \
  --name $STORAGE_NAME \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2 \
  --https-only true

print_success "Storage Account created: $STORAGE_NAME"

# Get storage account ID (needed for endpoints)
STORAGE_ID=$(az storage account show \
  --resource-group $RG \
  --name $STORAGE_NAME \
  --query id -o tsv)

print_info "Storage ID: $STORAGE_ID"
```

### Step 2: Enable Service Endpoint

```bash
# Service Endpoint: Routes traffic through Azure backbone instead of internet
# Makes VNet-to-Storage faster, cheaper, more secure

print_info "Enabling Service Endpoint for Storage..."

# Update frontend subnet to have Storage service endpoint
az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_FRONTEND \
  --service-endpoints Microsoft.Storage

print_success "Service Endpoint enabled!"

# Verify
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_FRONTEND \
  --query serviceEndpoints
```

### Step 3: Configure Storage Firewall for Service Endpoint

```bash
# Storage account default: Allow public access
# We'll lock it down to only allow from VNet

print_info "Configuring Storage firewall..."

# Block all public access
az storage account update \
  --resource-group $RG \
  --name $STORAGE_NAME \
  --default-action Deny

print_success "Public access blocked!"

# Add allow rule for frontend subnet (via service endpoint)
az storage account network-rule add \
  --resource-group $RG \
  --account-name $STORAGE_NAME \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_FRONTEND \
  --action Allow

print_success "Frontend subnet allowed!"

# View firewall rules
az storage account network-rule list \
  --resource-group $RG \
  --account-name $STORAGE_NAME
```

### Step 4: Create Private Endpoint (Better Alternative)

```bash
# Private Endpoint creates a private IP in VNet for storage access
# Superior to Service Endpoint because:
# - Works across peered VNets
# - Works from on-premises (via VPN)
# - Better monitoring
# - More control

print_info "Creating Private Endpoint for Storage..."

az network private-endpoint create \
  --resource-group $RG \
  --name storage-private-endpoint \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_BACKEND \
  --private-connection-resource-id $STORAGE_ID \
  --group-ids blob \
  --connection-name storage-connection

print_success "Private Endpoint created!"

# Get the private IP assigned to private endpoint
PRIVATE_IP=$(az network private-endpoint show \
  --resource-group $RG \
  --name storage-private-endpoint \
  --query 'networkInterfaces[0].id' -o tsv)

print_info "Private Endpoint NIC: $PRIVATE_IP"
```

### Step 5: Create Private DNS Zone

```bash
# Private DNS Zone allows VMs to resolve storage name to private IP
# Without this, DNS still resolves to public IP

print_info "Creating Private DNS Zone..."

az network private-dns zone create \
  --resource-group $RG \
  --name privatelink.blob.core.windows.net

print_success "Private DNS Zone created!"

# Link Private DNS Zone to VNet
az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name privatelink.blob.core.windows.net \
  --name vnet-link \
  --virtual-network $VNET_PRIMARY \
  --registration-enabled false

print_success "VNet linked to Private DNS Zone!"

# Get the actual private IP of the private endpoint
ENDPOINT_IP=$(az network private-endpoint show \
  --resource-group $RG \
  --name storage-private-endpoint \
  --query 'customDnsConfigs[0].ipAddresses[0]' -o tsv)

print_info "Private Endpoint IP: $ENDPOINT_IP"

# Create A record pointing storage name to private IP
az network private-dns record-set a create \
  --resource-group $RG \
  --zone-name privatelink.blob.core.windows.net \
  --name $STORAGE_NAME \
  --ttl 3600

az network private-dns record-set a add-record \
  --resource-group $RG \
  --zone-name privatelink.blob.core.windows.net \
  --record-set-name $STORAGE_NAME \
  --ipv4-address $ENDPOINT_IP

print_success "DNS A record created!"

cat << 'DNS_EXPLANATION'
┌─ DNS Resolution Flow (with Private Endpoint):
├─ VM queries: nslookup labstg1234.blob.core.windows.net
├─ Private DNS Zone resolves to: 10.0.2.254 (private IP)
├─ VM connects to: 10.0.2.254 (stays within VNet)
└─ Storage traffic never leaves Azure backbone

WITHOUT Private Endpoint:
├─ VM queries: nslookup labstg1234.blob.core.windows.net
├─ Public DNS resolves to: 20.21.22.23 (public IP)
├─ VM connects to: 20.21.22.23 (goes to internet)
└─ Traffic slower, costs egress charges
DNS_EXPLANATION
```

### Step 6: Test Service Endpoint Access (After VMs created)

```bash
# This test requires a VM in the subnet
# We'll document the command here for later execution

cat << 'SERVICE_ENDPOINT_TEST'
# After creating VM in frontend-subnet, SSH and run:

# Test 1: Check if Storage is accessible
az storage blob list --account-name labstg1234 --container-name test

# Test 2: View storage account network rules are working
az storage account show --name labstg1234 --query networkRuleBypass

# Test 3: Try access from different network (should fail)
# From public internet or non-whitelisted IP:
# curl https://labstg1234.blob.core.windows.net/test
# Result: 403 Forbidden (Service Endpoint/Firewall blocking)

SERVICE_ENDPOINT_TEST

print_success "Lab 3 Complete!"
```

---

# LAB 4: ROUTING - Route Tables, UDRs, Traffic Control

## Deep Dive: Routing Logic

```
When VM sends packet:
1. Check destination IP
2. Look in route table
3. Find matching route (longest prefix match wins)
4. Apply next hop

Routes processed in order:
- System routes (automatic) - lowest priority
- User-defined routes (your rules) - override system
- Service endpoint routes - direct connection
```

## Hands-On Lab: Implement Custom Routing

### Step 1: Create Route Table

```bash
# Route tables control how traffic flows between subnets and networks

print_info "Creating Route Table..."

az network route-table create \
  --resource-group $RG \
  --name route-table-production \
  --location $LOCATION \
  --disable-bgp-route-propagation false

print_success "Route Table created!"

# disable-bgp-route-propagation=false means:
# Routes from VPN/ExpressRoute gateway will auto-appear in table
```

### Step 2: Create User-Defined Routes (UDRs)

```bash
# UDR 1: Route all traffic through firewall
# (We'll use a dummy firewall IP for now)

print_info "Creating UDR: Route to Firewall..."

FIREWALL_IP="10.0.254.5"  # Internal firewall IP (not yet created)

az network route-table route create \
  --resource-group $RG \
  --route-table-name route-table-production \
  --name route-to-firewall \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $FIREWALL_IP

print_success "Firewall route created!"

# UDR 2: Route to on-premises network (via VPN gateway)
print_info "Creating UDR: Route to On-Premises..."

az network route-table route create \
  --resource-group $RG \
  --route-table-name route-table-production \
  --name route-to-onprem \
  --address-prefix 192.168.0.0/16 \
  --next-hop-type VirtualNetworkGateway

print_success "On-Premises route created!"

# UDR 3: Route to peered VNet directly (no firewall)
print_info "Creating UDR: Route to Peered VNet..."

az network route-table route create \
  --resource-group $RG \
  --route-table-name route-table-production \
  --name route-to-dr-vnet \
  --address-prefix 10.1.0.0/16 \
  --next-hop-type VirtualNetworkPeering

print_success "Peering route created!"

# Verify all routes
print_info "Listing all routes..."
az network route-table route list \
  --resource-group $RG \
  --route-table-name route-table-production \
  --output table
```

### Step 3: Associate Route Table with Subnet

```bash
# Routes apply to all VMs in the subnet

print_info "Associating Route Table with Backend Subnet..."

az network vnet subnet update \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_BACKEND \
  --route-table route-table-production

print_success "Route Table associated!"

# Verify
az network vnet subnet show \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_BACKEND \
  --query routeTable
```

### Step 4: View Effective Routes (What Actually Applies)

```bash
# After creating VMs, we can see actual routes in effect

print_info "Route Table Summary:"
cat << 'ROUTE_SUMMARY'
┌─ System Routes (Automatic):
├─ 10.0.0.0/16 → VnetLocal (internal)
├─ 10.0.1.0/24 → VnetLocal (frontend subnet)
├─ 10.0.2.0/24 → VnetLocal (backend subnet)
├─ 10.0.3.0/24 → VnetLocal (database subnet)
└─ 0.0.0.0/0 → Internet (default)

┌─ User-Defined Routes (Your rules):
├─ 0.0.0.0/0 → VirtualAppliance (10.0.254.5 - Firewall)
├─ 192.168.0.0/16 → VirtualNetworkGateway (VPN)
└─ 10.1.0.0/16 → VirtualNetworkPeering (DR VNet)

┌─ Route Selection Logic (longest prefix match):
├─ Destination: 8.8.8.8
│  ├─ Check 10.0.0.0/16 → No match
│  ├─ Check 0.0.0.0/0 → MATCH (sends to firewall)
│  └─ Selected: Firewall (10.0.254.5)
│
├─ Destination: 192.168.1.100
│  ├─ Check 10.0.0.0/16 → No match
│  ├─ Check 192.168.0.0/16 → MATCH (most specific)
│  └─ Selected: VPN Gateway
│
└─ Destination: 10.0.2.10 (same subnet)
   ├─ Check 10.0.2.0/24 → MATCH (most specific)
   └─ Selected: VnetLocal (direct)
ROUTE_SUMMARY

print_success "Lab 4 Complete!"
```

---

# LAB 5: VNET PEERING - Multi-VNet Connectivity

## Hands-On Lab: VNet Peering

> Note: Full VPN Gateway and on-prem hybrid connectivity is covered in depth in **Lab 7**. This lab covers peering between Azure VNets only.

### Step 1: Create DR VNet

```bash
# Disaster Recovery VNet in same region (for testing)

print_info "Creating DR VNet..."

az network vnet create \
  --resource-group $RG \
  --name $VNET_DR \
  --address-prefix $VNET_DR_SPACE \
  --subnet-name dr-frontend \
  --subnet-prefix 10.1.1.0/24 \
  --location $LOCATION

print_success "DR VNet created!"

# Add subnets
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_DR \
  --name dr-backend \
  --address-prefix 10.1.2.0/24

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_DR \
  --name dr-database \
  --address-prefix 10.1.3.0/24

print_success "DR VNet subnets created!"
```

### Step 2: Create VNet Peering

```bash
# Peering allows resources in different VNets to communicate

print_info "Creating VNet Peering..."

# Peer: Production → DR
az network vnet peering create \
  --resource-group $RG \
  --name prod-to-dr-peering \
  --vnet-name $VNET_PRIMARY \
  --remote-vnet $VNET_DR \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --allow-gateway-transit

print_success "Production→DR peering created!"

# Peer: DR → Production (reverse direction)
az network vnet peering create \
  --resource-group $RG \
  --name dr-to-prod-peering \
  --vnet-name $VNET_DR \
  --remote-vnet $VNET_PRIMARY \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --use-remote-gateways

print_success "DR→Production peering created!"

# Verify peering status
print_info "Peering status:"
az network vnet peering list \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --output table
```

---

# LAB 6: LOAD BALANCING - Azure Load Balancer

## Hands-On Lab: Azure Load Balancer

> Note: Application Gateway with WAF is covered in depth in **Lab 11**.

### Step 1: Create Load Balancer

```bash
# Load Balancer distributes traffic among backend VMs

print_info "Creating Azure Load Balancer..."

az network lb create \
  --resource-group $RG \
  --name production-lb \
  --sku Standard \
  --public-ip-address lb-public-ip \
  --frontend-ip-name lb-frontend \
  --backend-pool-name lb-backend-pool \
  --location $LOCATION

print_success "Load Balancer created!"

# Create health probe (checks if VMs are healthy)
print_info "Creating Health Probe..."

az network lb probe create \
  --resource-group $RG \
  --lb-name production-lb \
  --name health-probe-http \
  --protocol HTTP \
  --path / \
  --port 80 \
  --interval 15 \
  --threshold 2

print_success "Health probe created!"

# Create load balancing rule
print_info "Creating Load Balancing Rule..."

az network lb rule create \
  --resource-group $RG \
  --lb-name production-lb \
  --name lb-rule-http \
  --protocol TCP \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name lb-frontend \
  --backend-pool-name lb-backend-pool \
  --probe-name health-probe-http

print_success "Load Balancing rule created!"
```

---

# LAB 7: HYBRID CONNECTIVITY DEEP DIVE — VPN Gateway, Site-to-Site, Point-to-Site, On-Prem Simulation

## What You'll Learn
- What a Virtual Network Gateway actually is, and the SKUs that matter
- The difference between a **Local Network Gateway** (represents on-prem) and a **Virtual Network Gateway** (Azure side)
- How to build a real **Site-to-Site (S2S) VPN** between Azure and a simulated on-prem network (a second VNet standing in for your datacenter — this is the standard way to practice S2S without real hardware)
- How to build a **Point-to-Site (P2S) VPN** so your own laptop can connect into the VNet like a remote employee
- BGP basics over VPN (route propagation instead of static routes)
- Active-active and active-passive gateway HA concepts
- How to actually prove the tunnel works (ping across, trace the path, read the tunnel status)
- Where VPN Gateway fits vs ExpressRoute vs VNet Peering (decision matrix)

This is the lab most people skip and then get stuck on in interviews and real jobs. We are not skipping it.

---

## Deep Dive: How VPN Gateway Actually Works

```
┌─────────────────────────────────────────────────────────────────┐
│                      THE BIG PICTURE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│   ON-PREMISES (your datacenter / office)                         │
│   ┌───────────────────────┐                                     │
│   │  LAN: 192.168.1.0/24   │                                     │
│   │  VPN Device (router)  │── Public IP: 20.x.x.x (example)      │
│   │  e.g. Cisco/Fortinet   │                                     │
│   └───────────┬───────────┘                                     │
│               │                                                  │
│               │  IPsec/IKE Tunnel (encrypted, over the internet) │
│               │                                                  │
│   ┌───────────▼───────────┐                                     │
│   │  Azure VPN Gateway     │   <- lives in GatewaySubnet          │
│   │  Public IP: 4.x.x.x    │                                     │
│   └───────────┬───────────┘                                     │
│               │                                                  │
│   AZURE VNET: 10.0.0.0/16                                        │
│   ┌───────────▼───────────┐                                     │
│   │ frontend / backend /  │                                     │
│   │ database subnets      │                                     │
│   └───────────────────────┘                                     │
└─────────────────────────────────────────────────────────────────┘

KEY OBJECTS YOU NEED, IN ORDER:
1. GatewaySubnet            - special subnet, name MUST be exactly "GatewaySubnet"
2. Public IP for the gateway
3. Virtual Network Gateway  - the actual Azure-side VPN endpoint (takes 30-45 min to deploy!)
4. Local Network Gateway    - a Azure OBJECT that just describes the on-prem side
                              (its public IP + its address space) - it is NOT a real device,
                              it's how Azure knows "this IP + this CIDR = the other end"
5. Connection                - ties the Virtual Network Gateway to the Local Network Gateway
                              with a shared key (PSK) - this is what brings the tunnel UP
```

### Gateway SKU Cheat Sheet (this matters in real design + interviews)

```
SKU            Max S2S Tunnels   Max P2S Conn   Aggregate Throughput   BGP   Zone-Redundant
-------------  ----------------  -------------  ---------------------  ----  --------------
Basic          10                128            100 Mbps               No    No   (legacy, avoid)
VpnGw1         30                250            650 Mbps                Yes   No
VpnGw2         30                500            1 Gbps                  Yes   No
VpnGw3         30                1000           1.25 Gbps                Yes   No
VpnGw1AZ       30                250            650 Mbps                 Yes   Yes  (zone-redundant)
VpnGw2AZ       30                500            1 Gbps                   Yes   Yes
VpnGw3AZ       30                1000           1.25 Gbps                Yes   Yes

Rule of thumb:
- Lab/learning           -> VpnGw1
- Production, no zones   -> VpnGw1/2/3 based on throughput need
- Production, HA region  -> the "AZ" SKUs (deploys gateway instances across Availability Zones)
```

### Local Network Gateway vs Virtual Network Gateway — the #1 confusion point

```
Virtual Network Gateway = the Azure-side VPN device. It lives INSIDE your VNet's GatewaySubnet.
Local Network Gateway   = a description object of the REMOTE side. It is metadata only:
                            - its public IP (or FQDN)
                            - its address space (what CIDR is reachable behind it)
                           It does not run anywhere. It is how the Azure Gateway knows
                           "if someone in the VNet wants to reach 192.168.0.0/16,
                           go talk to public IP 20.x.x.x"

In real life:
  Local Network Gateway's IP = your actual on-prem firewall/router's public IP
In this lab (simulation):
  Local Network Gateway's IP = the public IP of a SECOND Azure VPN Gateway,
  because we're using a second VNet to stand in for "on-prem"
```

---

## Hands-On Lab Part A: Build the Simulated On-Prem Network

### Step 1: Create the "On-Premises" VNet

```bash
source ~/.azure-lab-vars.sh

# This VNet plays the role of your company's datacenter.
# Using 192.168.0.0/16 deliberately - it's the classic "home/office" range,
# and it must NOT overlap with the Azure production VNet (10.0.0.0/16).

print_info "Creating simulated on-premises VNet..."

az network vnet create \
  --resource-group $RG \
  --name $VNET_ONPREM \
  --address-prefix $VNET_ONPREM_SPACE \
  --subnet-name $SUBNET_ONPREM_LAN \
  --subnet-prefix $SUBNET_ONPREM_LAN_SPACE \
  --location $LOCATION

print_success "On-prem VNet created!"

# Add the GatewaySubnet to the on-prem VNet
# CRITICAL: the name must be EXACTLY "GatewaySubnet" - Azure looks for this name specifically
az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_ONPREM \
  --name GatewaySubnet \
  --address-prefix $SUBNET_ONPREM_GATEWAY_SPACE

print_success "On-prem GatewaySubnet created!"
```

### Step 2: Add the GatewaySubnet to the Production VNet (if not already present)

```bash
# The production VNet (10.0.0.0/16) also needs its own GatewaySubnet
# This was reserved in the design from Lab 1 (10.0.5.0/24) - now we actually create it

print_info "Creating GatewaySubnet in production VNet..."

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name GatewaySubnet \
  --address-prefix $SUBNET_GATEWAY_SPACE

print_success "Production GatewaySubnet created!"

# Verify both VNets have a GatewaySubnet
print_info "Verifying GatewaySubnets..."
az network vnet subnet list --resource-group $RG --vnet-name $VNET_PRIMARY --output table
az network vnet subnet list --resource-group $RG --vnet-name $VNET_ONPREM --output table
```

> **Why GatewaySubnet can't have an NSG:** Azure manages traffic in/out of the gateway directly. Attaching an NSG to GatewaySubnet is unsupported and can break the gateway. Leave it bare.

---

## Hands-On Lab Part B: Deploy Both VPN Gateways

> **Heads up — timing:** Each Virtual Network Gateway takes **30-45 minutes** to provision. This is real Azure infrastructure being spun up, not a quick API call. Kick off BOTH gateway creations in the background (`--no-wait`) so they build in parallel, then go get coffee.

### Step 3: Create Public IPs for Both Gateways

```bash
print_info "Creating Public IP for Azure (production) gateway..."

az network public-ip create \
  --resource-group $RG \
  --name azure-vpn-gw-ip \
  --allocation-method Static \
  --sku Standard \
  --location $LOCATION

print_info "Creating Public IP for on-prem (simulated) gateway..."

az network public-ip create \
  --resource-group $RG \
  --name onprem-vpn-gw-ip \
  --allocation-method Static \
  --sku Standard \
  --location $LOCATION

print_success "Both Public IPs created!"
```

### Step 4: Create the Production-Side Virtual Network Gateway

```bash
print_info "Creating Azure (production) Virtual Network Gateway... this takes 30-45 minutes"
print_warning "Running with --no-wait so we can build BOTH gateways in parallel"

az network vnet-gateway create \
  --resource-group $RG \
  --name $VPN_GW_AZURE \
  --vnet $VNET_PRIMARY \
  --public-ip-addresses azure-vpn-gw-ip \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1 \
  --no-wait

print_success "Azure gateway creation started in background"
```

### Step 5: Create the On-Prem-Side Virtual Network Gateway (simulating the remote site)

```bash
print_info "Creating on-prem (simulated) Virtual Network Gateway... this also takes 30-45 minutes"

az network vnet-gateway create \
  --resource-group $RG \
  --name $VPN_GW_ONPREM \
  --vnet $VNET_ONPREM \
  --public-ip-addresses onprem-vpn-gw-ip \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1 \
  --no-wait

print_success "On-prem gateway creation started in background"

print_info "Both gateways are now deploying in parallel."
print_info "Check status with:"
echo '  az network vnet-gateway show --resource-group $RG --name '"$VPN_GW_AZURE"' --query provisioningState'
echo '  az network vnet-gateway show --resource-group $RG --name '"$VPN_GW_ONPREM"' --query provisioningState'
```

### Step 6: Wait for Both Gateways to Finish Provisioning

```bash
# Poll until both gateways report "Succeeded"
# This loop checks every 60 seconds - feel free to just run the az wait commands instead

print_info "Waiting for Azure gateway to finish provisioning (this is the long part)..."
az network vnet-gateway wait \
  --resource-group $RG \
  --name $VPN_GW_AZURE \
  --created

print_success "Azure gateway ready!"

print_info "Waiting for on-prem gateway to finish provisioning..."
az network vnet-gateway wait \
  --resource-group $RG \
  --name $VPN_GW_ONPREM \
  --created

print_success "On-prem gateway ready!"

# Grab their public IPs - we need these for the next step
AZURE_GW_IP=$(az network public-ip show --resource-group $RG --name azure-vpn-gw-ip --query ipAddress -o tsv)
ONPREM_GW_IP=$(az network public-ip show --resource-group $RG --name onprem-vpn-gw-ip --query ipAddress -o tsv)

print_info "Azure Gateway Public IP: $AZURE_GW_IP"
print_info "On-Prem Gateway Public IP: $ONPREM_GW_IP"
```

---

## Hands-On Lab Part C: Connect the Two Sides (Local Network Gateways + Connections)

### Step 7: Create Local Network Gateways (each side describes the OTHER side)

```bash
# The Azure side needs a Local Network Gateway object describing "on-prem"
# (its public IP + its address space: 192.168.0.0/16)

print_info "Creating Local Network Gateway representing on-prem (for the Azure side)..."

az network local-gateway create \
  --resource-group $RG \
  --name $LOCAL_GATEWAY \
  --gateway-ip-address $ONPREM_GW_IP \
  --local-address-prefixes $VNET_ONPREM_SPACE \
  --location $LOCATION

print_success "Local Network Gateway (on-prem description) created!"

# And the on-prem side needs one describing "Azure production"
print_info "Creating Local Network Gateway representing Azure production (for the on-prem side)..."

az network local-gateway create \
  --resource-group $RG \
  --name azure-prod-local-gateway \
  --gateway-ip-address $AZURE_GW_IP \
  --local-address-prefixes $VNET_PRIMARY_SPACE \
  --location $LOCATION

print_success "Local Network Gateway (Azure prod description) created!"
```

### Step 8: Create the VPN Connections (this is what actually brings the tunnel UP)

```bash
# A Connection ties a Virtual Network Gateway to a Local Network Gateway
# using a Pre-Shared Key (PSK). BOTH sides must use the EXACT SAME key.

print_info "Creating connection: Azure -> On-Prem..."

az network vpn-connection create \
  --resource-group $RG \
  --name $VPN_CONNECTION \
  --vnet-gateway1 $VPN_GW_AZURE \
  --local-gateway2 $LOCAL_GATEWAY \
  --shared-key "$VPN_SHARED_KEY" \
  --location $LOCATION

print_success "Azure -> On-Prem connection created!"

print_info "Creating connection: On-Prem -> Azure..."

az network vpn-connection create \
  --resource-group $RG \
  --name onprem-to-azure-connection \
  --vnet-gateway1 $VPN_GW_ONPREM \
  --local-gateway2 azure-prod-local-gateway \
  --shared-key "$VPN_SHARED_KEY" \
  --location $LOCATION

print_success "On-Prem -> Azure connection created!"

print_info "Tunnel is establishing now - typically connects within 1-5 minutes"
```

### Step 9: Verify the Tunnel Is Actually Up

```bash
# This is the step people forget - ALWAYS verify connection status, don't assume

print_info "Checking connection status (Azure -> On-Prem)..."

az network vpn-connection show \
  --resource-group $RG \
  --name $VPN_CONNECTION \
  --query "{Status:connectionStatus, IngressBytes:ingressBytesTransferred, EgressBytes:egressBytesTransferred}" \
  --output table

# Expected: Status = "Connected"
# If it shows "NotConnected", wait 2-3 more minutes and re-check

print_info "Checking connection status (On-Prem -> Azure)..."

az network vpn-connection show \
  --resource-group $RG \
  --name onprem-to-azure-connection \
  --query "{Status:connectionStatus, IngressBytes:ingressBytesTransferred, EgressBytes:egressBytesTransferred}" \
  --output table
```

**Expected Output:**
```
Status       IngressBytes    EgressBytes
-----------  --------------  -------------
Connected    1234            5678
```

> If both show `Connected`, your IPsec/IKE tunnel is live. Traffic between 10.0.0.0/16 and 192.168.0.0/16 now flows encrypted over the internet through this tunnel automatically — Azure adds the routes for you.

### Step 10: Test Cross-Network Connectivity (after VMs exist)

```bash
cat << 'VPN_TEST'
# Once you have a VM in the production VNet (e.g. 10.0.1.10) and a VM
# in the on-prem VNet (e.g. 192.168.1.10), test like this:

# From the "on-prem" VM:
ping 10.0.1.10
# This packet leaves the on-prem VNet -> on-prem gateway -> IPsec tunnel
# -> Azure gateway -> production VNet -> destination VM

# From the production VM:
ping 192.168.1.10

# Use Network Watcher to PROVE the path without even needing a live ping:
az network watcher test-connectivity \
  --resource-group $RG \
  --source-resource <prod-vm-name> \
  --dest-address 192.168.1.10 \
  --dest-port 3389 \
  --protocol Tcp
VPN_TEST
```

---

## Hands-On Lab Part D: BGP Over VPN (Dynamic Routing, the Real-World Way)

```bash
# Static routing (what we just did) means YOU manually tell Azure the on-prem CIDR.
# BGP means the routers exchange routes automatically - this is what real enterprises use,
# especially when on-prem has many subnets that change over time.

print_info "BGP Deep Dive..."
cat << 'BGP_EXPLANATION'
WITHOUT BGP (static):
  - You hardcode the on-prem CIDR in the Local Network Gateway (--local-address-prefixes)
  - If on-prem adds a new subnet, you must manually update the Local Network Gateway
  - Fine for simple, stable topologies

WITH BGP (dynamic):
  - Both gateways are assigned a private "BGP peering IP" and an ASN (Autonomous System Number)
  - Routers exchange route advertisements automatically over the tunnel
  - New on-prem subnets appear in Azure automatically (and vice versa)
  - REQUIRED for: multiple connections needing automatic failover,
                  active-active gateway configs, and ExpressRoute+VPN coexistence
BGP_EXPLANATION

# To enable BGP, gateways need an ASN and the connection needs --enable-bgp
# Azure default ASN: 65515 (you cannot use this for on-prem side - pick something like 65001)

print_info "Example: creating a gateway with BGP enabled (reference only, optional extension)"
cat << 'BGP_EXAMPLE'
az network vnet-gateway create \
  --resource-group $RG \
  --name azure-vpn-gw-bgp \
  --vnet $VNET_PRIMARY \
  --public-ip-addresses azure-vpn-gw-ip \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1 \
  --asn 65515 \
  --bgp-peering-address 10.0.5.254

az network vpn-connection create \
  --resource-group $RG \
  --name bgp-connection \
  --vnet-gateway1 azure-vpn-gw-bgp \
  --local-gateway2 $LOCAL_GATEWAY \
  --shared-key "$VPN_SHARED_KEY" \
  --enable-bgp
BGP_EXAMPLE

print_info "We're keeping this lab on static routing for simplicity, but now you know exactly"
print_info "what BGP solves and the flags involved when you need it in production."
```

---

## Hands-On Lab Part E: Point-to-Site (P2S) VPN — Connect YOUR Laptop Directly

```bash
# P2S is different from S2S:
# S2S = network-to-network (office <-> Azure)
# P2S = single device-to-network (YOUR laptop <-> Azure) - used for remote/work-from-home access

print_info "Point-to-Site VPN Deep Dive..."
cat << 'P2S_EXPLANATION'
P2S connects an individual client computer to the VNet, using one of:
  - OpenVPN protocol         (cross-platform: Windows, macOS, Linux)
  - IKEv2                    (Windows, macOS)
  - SSTP                     (Windows only)

Authentication options:
  - Azure certificate authentication (self-signed root + client certs) <- what we'll use
  - Azure AD authentication (OpenVPN only, integrates with Entra ID/MFA)
  - RADIUS authentication (enterprise on-prem AD integration)

P2S needs its OWN address pool (separate from the VNet's address space)
that gets handed out to connecting clients - like a mini DHCP for VPN clients.
P2S_EXPLANATION
```

### Step 11: Generate Self-Signed Root and Client Certificates

```bash
# Requires Linux/Mac (or WSL on Windows). Uses Azure's own cert generation pattern.

print_info "Generating root certificate..."

mkdir -p ~/p2s-certs && cd ~/p2s-certs

# Generate root cert private key
openssl genrsa -out root-key.pem 4096

# Generate root cert (self-signed, valid 5 years)
openssl req -x509 -new -nodes -key root-key.pem -sha256 -days 1825 \
  -out root-cert.pem \
  -subj "/CN=P2SRootCert"

# Convert root cert to base64 (Azure needs this format for upload)
openssl x509 -in root-cert.pem -outform der | base64 -w 0 > root-cert-base64.txt

print_success "Root certificate generated!"

# Generate client certificate (signed by root) - this is what installs on YOUR laptop
print_info "Generating client certificate..."

openssl genrsa -out client-key.pem 4096

openssl req -new -key client-key.pem \
  -out client.csr \
  -subj "/CN=P2SClientCert"

openssl x509 -req -in client.csr -CA root-cert.pem -CAkey root-key.pem \
  -CAcreateserial -out client-cert.pem -days 1825 -sha256

# Bundle as .pfx for easy import into Windows/Mac VPN client
openssl pkcs12 -export -out client-cert.pfx \
  -inkey client-key.pem -in client-cert.pem -certfile root-cert.pem \
  -passout pass:ClientCertPassword123!

print_success "Client certificate generated and bundled as client-cert.pfx"
print_info "Install client-cert.pfx on your laptop before connecting (double-click on Windows/Mac)"
```

### Step 12: Configure P2S on the Azure Gateway

```bash
print_info "Configuring Point-to-Site on the Azure gateway..."

ROOT_CERT_DATA=$(cat ~/p2s-certs/root-cert-base64.txt)

az network vnet-gateway update \
  --resource-group $RG \
  --name $VPN_GW_AZURE \
  --address-prefixes 172.16.201.0/24 \
  --client-protocol OpenVPN \
  --root-cert-name P2SRootCert \
  --root-cert-data "$ROOT_CERT_DATA"

print_success "P2S configured!"
print_warning "172.16.201.0/24 is the VPN client address pool - must NOT overlap with"
print_warning "the VNet (10.0.0.0/16), the on-prem VNet (192.168.0.0/16), or your local LAN"
```

### Step 13: Download and Use the VPN Client Configuration

```bash
print_info "Generating the VPN client configuration package..."

az network vnet-gateway vpn-client generate \
  --resource-group $RG \
  --name $VPN_GW_AZURE \
  --authentication-method EAPTLS \
  --output tsv

# This returns a download URL for a ZIP file containing:
# - The Azure VPN Client profile (azurevpnconfig.xml) - for the official Azure VPN Client app
# - OpenVPN config files (.ovpn) - for OpenVPN-compatible clients

cat << 'P2S_CONNECT_STEPS'
TO CONNECT FROM YOUR LAPTOP:
1. Download the ZIP from the URL returned above
2. Install the client-cert.pfx certificate (double-click, enter the password you set)
3. Install the "Azure VPN Client" app (Windows/macOS) from the Microsoft Store / App Store
4. Import the azurevpnconfig.xml profile into the Azure VPN Client
5. Click Connect
6. Verify: run `ipconfig` (Windows) or `ifconfig` (Mac) - you should see a new adapter
   with an IP from 172.16.201.0/24
7. Try: ping 10.0.1.10 (a VM in the production VNet) - if it responds, P2S works!
P2S_CONNECT_STEPS

print_success "Lab 7 Complete!"
```

---

## Decision Matrix: VPN Gateway vs ExpressRoute vs VNet Peering

```
Connectivity Need                          Best Choice
------------------------------------------  -----------------------------------
Two Azure VNets, same or different regions   VNet Peering (fastest, cheapest, no gateway needed)
Office/datacenter <-> Azure, cost-sensitive  Site-to-Site VPN Gateway (internet-based, encrypted)
Remote employees <-> Azure                   Point-to-Site VPN
Office/datacenter <-> Azure, needs:
  - guaranteed bandwidth & low latency
  - SLA-backed private connection
  - large data volumes (backups, migrations) -> ExpressRoute (private circuit via a connectivity
                                                  provider, NOT over the public internet)
Office <-> Azure, want both speed AND
  a backup path                              ExpressRoute as primary + VPN Gateway as failover
                                              (this is extremely common in real enterprise design)

Cost ordering (cheapest to priciest, roughly):
  VNet Peering < VPN Gateway < ExpressRoute
```

> **Note on ExpressRoute:** ExpressRoute requires a connectivity provider (a telecom partner) and a physical circuit — it can't be fully simulated with `az` commands alone the way VPN can, which is why this lab focuses on VPN Gateway for genuine hands-on practice. The architecture and routing concepts (Local Network Gateway equivalent is the "ExpressRoute Circuit" + "Connection" objects) carry over directly once you have a real circuit.

---
# LAB 8: DNS DEEP DIVE — Azure DNS, Private DNS Zones, Conditional Forwarding, Private Resolver, Hybrid Resolution

## What You'll Learn
- Azure Public DNS Zones (hosting your domain) vs Azure Private DNS Zones (internal name resolution)
- How Azure's built-in DNS (168.63.129.16) actually works under the hood
- Auto-registration vs manual records in Private DNS Zones
- VNet linking — and why a zone with no VNet link resolves nothing
- The `privatelink.*` zone naming convention used by every PaaS Private Endpoint
- **Azure DNS Private Resolver** — the service that bridges on-prem DNS and Azure DNS (this is the #1 missing piece in most tutorials)
- Conditional forwarding — both directions: on-prem → Azure, and Azure → on-prem
- Split-horizon DNS (same name, different answer depending on where you ask from)
- End-to-end hybrid resolution test using the VPN tunnel from Lab 7

---

## Deep Dive: The Whole DNS Picture in Azure

```
┌──────────────────────────────────────────────────────────────────────┐
│  THREE SEPARATE DNS WORLDS YOU MUST NOT CONFUSE                       │
├──────────────────────────────────────────────────────────────────────┤
│                                                                        │
│  1) AZURE-PROVIDED DNS (168.63.129.16)                                │
│     - The default resolver every VM gets automatically                │
│     - Resolves: Azure internal hostnames, Private DNS Zones linked    │
│       to the VNet, and public internet names (via recursive lookup)   │
│     - You don't manage this server, it's a platform service           │
│                                                                        │
│  2) AZURE PUBLIC DNS ZONES                                            │
│     - Hosts a real public domain, e.g. "labdemo.com"                  │
│     - Anyone on the internet can query it                             │
│     - Used for: company websites, public APIs                        │
│     - Has nothing to do with your VNet directly                       │
│                                                                        │
│  3) AZURE PRIVATE DNS ZONES                                          │
│     - Only resolvable by VNets you explicitly LINK to the zone        │
│     - Used for: internal app names (e.g. "api.corp.internal")         │
│       AND for Private Endpoint resolution (privatelink.*.azure.com)  │
│     - Completely invisible to the public internet                     │
└──────────────────────────────────────────────────────────────────────┘
```

### How a VM Actually Resolves a Name (the real lookup order)

```
VM in a VNet queries "storageacct.blob.core.windows.net":

1. VM sends DNS query to its configured DNS server
   (default = 168.63.129.16, Azure's recursive resolver, UNLESS you set custom DNS on the VNet)

2. Azure's resolver checks: "Is there a Private DNS Zone linked to THIS VNet
   that matches this name (or is authoritative for a parent zone)?"
   - privatelink.blob.core.windows.net IS linked -> check for a record -> found -> return PRIVATE IP
   - Not linked / no record -> fall through to public DNS -> return PUBLIC IP

3. This is exactly why Private Endpoints "just work" without touching the VM's hosts file -
   the Private DNS Zone silently overrides what would otherwise be a public answer
```

---

## Hands-On Lab Part A: Azure Public DNS Zone

### Step 1: Create a Public DNS Zone

```bash
source ~/.azure-lab-vars.sh

print_info "Creating Public DNS Zone for $DNS_ZONE..."

az network dns zone create \
  --resource-group $RG \
  --name $DNS_ZONE

print_success "Public DNS Zone created!"

# Azure assigns 4 name servers - in real life you'd update these at your domain registrar
print_info "Name servers assigned to this zone (you'd configure these at GoDaddy/Namecheap/etc):"
az network dns zone show \
  --resource-group $RG \
  --name $DNS_ZONE \
  --query nameServers \
  --output table
```

### Step 2: Add Common Record Types

```bash
print_info "Adding A record (www -> a public IP)..."

az network dns record-set a add-record \
  --resource-group $RG \
  --zone-name $DNS_ZONE \
  --record-set-name www \
  --ipv4-address 20.21.22.23

print_info "Adding CNAME record (alias)..."

az network dns record-set cname set-record \
  --resource-group $RG \
  --zone-name $DNS_ZONE \
  --record-set-name shop \
  --cname www.$DNS_ZONE

print_info "Adding MX record (mail routing)..."

az network dns record-set mx add-record \
  --resource-group $RG \
  --zone-name $DNS_ZONE \
  --record-set-name "@" \
  --exchange mail.$DNS_ZONE \
  --preference 10

print_info "Adding TXT record (SPF/verification, very common in real domains)..."

az network dns record-set txt add-record \
  --resource-group $RG \
  --zone-name $DNS_ZONE \
  --record-set-name "@" \
  --value "v=spf1 include:_spf.example.com ~all"

print_success "Public DNS Zone fully populated!"

# View everything
az network dns record-set list \
  --resource-group $RG \
  --zone-name $DNS_ZONE \
  --output table
```

> Public DNS Zones are for the *internet-facing* identity of your domain. They have zero relationship to what happens inside your VNet — that's what Private DNS Zones are for, which we cover next.

---

## Hands-On Lab Part B: Private DNS Zone — Auto-Registration (Internal App Names)

```bash
# This is DIFFERENT from the Private DNS Zone you made in Lab 3 for Private Endpoints.
# This one is for YOUR OWN internal naming, like "webserver1.corp.internal"
```

### Step 3: Create an Internal Private DNS Zone

```bash
print_info "Creating internal Private DNS Zone: $PRIVATE_DNS_ZONE_INTERNAL..."

az network private-dns zone create \
  --resource-group $RG \
  --name $PRIVATE_DNS_ZONE_INTERNAL

print_success "Internal Private DNS Zone created!"
```

### Step 4: Link the VNet WITH Auto-Registration Enabled

```bash
# registration-enabled=true means: every VM's NIC in this VNet AUTOMATICALLY
# gets an A record created for it in this zone (vm-name.corp.internal -> its private IP)
# This is huge for real environments - no manual DNS record management for VMs!

print_info "Linking production VNet with auto-registration enabled..."

az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name $PRIVATE_DNS_ZONE_INTERNAL \
  --name prod-vnet-link-autoreg \
  --virtual-network $VNET_PRIMARY \
  --registration-enabled true

print_success "VNet linked with auto-registration!"

cat << 'AUTOREG_EXPLANATION'
WHAT HAPPENS NOW:
- Create a VM named "web-server-1" in the production VNet
- Azure automatically creates: web-server-1.corp.internal -> 10.0.1.10 (its private IP)
- Delete the VM -> the record is automatically cleaned up
- This ONLY happens on the ONE VNet link that has registration-enabled=true
  (only one link per zone can have auto-registration on)
AUTOREG_EXPLANATION
```

### Step 5: Link a Second VNet WITHOUT Auto-Registration (Resolution Only)

```bash
# Other VNets can still RESOLVE names from this zone, just not auto-register into it

print_info "Linking DR VNet for resolution only (no auto-registration)..."

az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name $PRIVATE_DNS_ZONE_INTERNAL \
  --name dr-vnet-link-resolveonly \
  --virtual-network $VNET_DR \
  --registration-enabled false

print_success "DR VNet linked (resolve-only)!"

# View all links
az network private-dns link vnet list \
  --resource-group $RG \
  --zone-name $PRIVATE_DNS_ZONE_INTERNAL \
  --output table
```

### Step 6: Add a Manual Record (for things that aren't VMs, like a load balancer)

```bash
print_info "Adding manual A record for the internal load balancer..."

az network private-dns record-set a create \
  --resource-group $RG \
  --zone-name $PRIVATE_DNS_ZONE_INTERNAL \
  --name internal-lb

az network private-dns record-set a add-record \
  --resource-group $RG \
  --zone-name $PRIVATE_DNS_ZONE_INTERNAL \
  --record-set-name internal-lb \
  --ipv4-address 10.0.1.100

print_success "Manual record added: internal-lb.corp.internal -> 10.0.1.100"

# Test resolution from inside a VM (after VMs exist):
cat << 'DNS_TEST'
# SSH into any VM in production VNet or DR VNet, then run:
nslookup internal-lb.corp.internal
nslookup web-server-1.corp.internal    # the auto-registered one

# Expected: both resolve to their private IPs, working ONLY inside linked VNets
# Try the same lookup from your laptop over the public internet - it will FAIL
# (this is by design - Private DNS Zones are invisible outside linked VNets)
DNS_TEST
```

---

## Hands-On Lab Part C: Azure DNS Private Resolver (The Hybrid DNS Bridge)

```bash
cat << 'RESOLVER_EXPLANATION'
THE PROBLEM THIS SOLVES:
Your on-prem network has its own DNS server (e.g. Active Directory DNS) resolving
"*.corp.internal" names. Azure VMs need to resolve THOSE names too, AND on-prem
machines need to resolve Azure Private DNS Zone names (like privatelink endpoints).

Before DNS Private Resolver, people ran a VM with BIND/Windows DNS Server just to forward
queries - extra infrastructure to patch and maintain. DNS Private Resolver replaces that
with a managed, no-VM service that sits inside your VNet and forwards DNS both ways.

IT HAS TWO ENDPOINT TYPES:
- INBOUND endpoint  -> gives on-prem a private IP IN Azure to send DNS queries TO
                       (so on-prem's DNS server forwards "*.corp.internal" queries here,
                        and Azure resolves them using its linked Private DNS Zones)
- OUTBOUND endpoint -> lets Azure VMs send DNS queries OUT to on-prem DNS servers
                       (via Forwarding Rulesets, e.g. forward "*.ad.corp.local" to the
                        on-prem AD DNS server's IP, reached over the VPN tunnel)
RESOLVER_EXPLANATION
```

### Step 7: Create Subnets for the DNS Resolver Endpoints

```bash
# Inbound and outbound endpoints each need their OWN dedicated, EMPTY subnet
# (cannot share with VMs, NSGs not supported on these subnets, delegated specifically
# to Microsoft.Network/dnsResolvers)

print_info "Creating subnet for DNS Resolver inbound endpoint..."

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_DNS_INBOUND \
  --address-prefix $SUBNET_DNS_INBOUND_SPACE \
  --delegations Microsoft.Network/dnsResolvers

print_info "Creating subnet for DNS Resolver outbound endpoint..."

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name $SUBNET_DNS_OUTBOUND \
  --address-prefix $SUBNET_DNS_OUTBOUND_SPACE \
  --delegations Microsoft.Network/dnsResolvers

print_success "Both delegated subnets created!"
```

### Step 8: Create the DNS Private Resolver Resource

```bash
print_info "Creating Azure DNS Private Resolver..."

az dns-resolver create \
  --resource-group $RG \
  --name $DNS_RESOLVER_NAME \
  --location $LOCATION \
  --id-vnet $(az network vnet show --resource-group $RG --name $VNET_PRIMARY --query id -o tsv)

print_success "DNS Private Resolver created!"
```

### Step 9: Create the Inbound Endpoint (lets on-prem query Azure)

```bash
print_info "Creating inbound endpoint..."

INBOUND_SUBNET_ID=$(az network vnet subnet show \
  --resource-group $RG --vnet-name $VNET_PRIMARY \
  --name $SUBNET_DNS_INBOUND --query id -o tsv)

az dns-resolver inbound-endpoint create \
  --resource-group $RG \
  --dns-resolver-name $DNS_RESOLVER_NAME \
  --name inbound-endpoint \
  --location $LOCATION \
  --ip-configurations "[{\"private-ip-allocation-method\":\"Dynamic\",\"id\":\"$INBOUND_SUBNET_ID\"}]"

print_success "Inbound endpoint created!"

# Get its IP - THIS is the IP your on-prem DNS server will forward "*.corp.internal" to
INBOUND_IP=$(az dns-resolver inbound-endpoint show \
  --resource-group $RG \
  --dns-resolver-name $DNS_RESOLVER_NAME \
  --name inbound-endpoint \
  --query "ipConfigurations[0].privateIpAddress" -o tsv)

print_info "Inbound Endpoint IP (give this to your on-prem DNS admin): $INBOUND_IP"

cat << 'ONPREM_CONFIG'
ON-PREM SIDE (what your on-prem DNS admin/server config would look like):
  In Windows DNS Server / BIND, create a Conditional Forwarder:
    Zone: corp.internal
    Forward to: <INBOUND_IP printed above>  (reached via the VPN tunnel from Lab 7)

  Now any on-prem machine asking for *.corp.internal gets routed through the tunnel
  to Azure's DNS Private Resolver, which resolves it using the linked Private DNS Zone.
ONPREM_CONFIG
```

### Step 10: Create the Outbound Endpoint + Forwarding Ruleset (lets Azure query on-prem)

```bash
print_info "Creating outbound endpoint..."

OUTBOUND_SUBNET_ID=$(az network vnet subnet show \
  --resource-group $RG --vnet-name $VNET_PRIMARY \
  --name $SUBNET_DNS_OUTBOUND --query id -o tsv)

az dns-resolver outbound-endpoint create \
  --resource-group $RG \
  --dns-resolver-name $DNS_RESOLVER_NAME \
  --name outbound-endpoint \
  --location $LOCATION \
  --subnet $OUTBOUND_SUBNET_ID

print_success "Outbound endpoint created!"

# Create a Forwarding Ruleset - the actual "if domain matches X, forward to Y" rules
print_info "Creating DNS Forwarding Ruleset..."

az dns-resolver forwarding-ruleset create \
  --resource-group $RG \
  --name onprem-forwarding-ruleset \
  --location $LOCATION \
  --outbound-endpoints "[{\"id\":\"$(az dns-resolver outbound-endpoint show --resource-group $RG --dns-resolver-name $DNS_RESOLVER_NAME --name outbound-endpoint --query id -o tsv)\"}]"

print_success "Forwarding Ruleset created!"

# Add the actual rule: forward queries for the on-prem AD domain to the on-prem DNS server
# (reachable through the VPN tunnel from Lab 7, e.g. 192.168.1.10)
print_info "Adding forwarding rule for on-prem domain..."

az dns-resolver forwarding-rule create \
  --resource-group $RG \
  --dns-forwarding-ruleset-name onprem-forwarding-ruleset \
  --name forward-to-onprem-ad \
  --domain-name "ad.corplocal." \
  --target-dns-servers "[{\"ip-address\":\"192.168.1.10\",\"port\":53}]"

print_success "Forwarding rule added: *.ad.corplocal -> 192.168.1.10 (on-prem DNS server)"

# Finally, LINK the ruleset to the VNet(s) that should use these forwarding rules
print_info "Linking the ruleset to the production VNet..."

az dns-resolver vnet-link create \
  --resource-group $RG \
  --dns-forwarding-ruleset-name onprem-forwarding-ruleset \
  --name prod-vnet-ruleset-link \
  --id-vnet $(az network vnet show --resource-group $RG --name $VNET_PRIMARY --query id -o tsv)

print_success "Ruleset linked! Azure VMs in production VNet can now resolve *.ad.corplocal via on-prem DNS"
```

---

## Hands-On Lab Part D: Split-Horizon DNS

```bash
cat << 'SPLIT_HORIZON'
SPLIT-HORIZON DNS = the same hostname resolves to DIFFERENT answers depending on
WHERE the query comes from. Extremely common in real enterprises:

  api.company.com queried from the PUBLIC internet  -> public IP of API Gateway/Front Door
  api.company.com queried from INSIDE the VNet      -> private IP via Private Endpoint

HOW THIS WORKS IN AZURE (you've actually already built the pieces for this):
  1. Public DNS Zone "company.com" has an A/CNAME record -> public IP (App Gateway, Front Door, etc.)
  2. Private DNS Zone, ALSO named "company.com" (or using the privatelink.* convention),
     linked ONLY to your VNets, has an A record for the SAME name -> private IP

  Because Private DNS Zones linked to a VNet are checked BEFORE falling through
  to public DNS, any VM inside the VNet gets the PRIVATE answer automatically,
  while anyone on the public internet only ever sees the PUBLIC zone.

THIS IS EXACTLY HOW PRIVATE ENDPOINTS WORK (Lab 3 and Lab 9):
  privatelink.blob.core.windows.net is a private zone overriding the public
  storage account FQDN ONLY for VNets it's linked to - textbook split-horizon DNS.
SPLIT_HORIZON

print_success "Lab 8 Complete!"
print_info "You now understand:"
echo "  ✓ Public DNS Zones (internet-facing domain hosting)"
echo "  ✓ Private DNS Zones with auto-registration vs manual records"
echo "  ✓ VNet linking and why it gates resolution"
echo "  ✓ DNS Private Resolver - inbound (on-prem -> Azure) and outbound (Azure -> on-prem)"
echo "  ✓ Conditional forwarding rules"
echo "  ✓ Split-horizon DNS concept and why it's the basis for Private Endpoints"
```

### Step 11: End-to-End Hybrid DNS Test (ties Lab 7 + Lab 8 together)

```bash
cat << 'E2E_DNS_TEST'
FULL HYBRID DNS TEST (after VMs exist on both sides, using the VPN tunnel from Lab 7):

FROM AN AZURE VM (10.0.1.x):
  nslookup web-server-1.corp.internal     # resolves via Private DNS Zone, auto-registered
  nslookup somehost.ad.corplocal          # resolves via Outbound Endpoint -> forwarded
                                           # over the VPN tunnel -> on-prem DNS server -> answer

FROM AN ON-PREM VM (192.168.1.x), after configuring the conditional forwarder:
  nslookup web-server-1.corp.internal     # query goes over VPN tunnel -> Inbound Endpoint IP
                                           # -> Azure resolves using linked Private DNS Zone
                                           # -> answer flows back through the tunnel

If both directions work, you have built a genuine hybrid DNS architecture -
this is precisely what's deployed in real enterprises connecting on-prem AD
environments to Azure.
E2E_DNS_TEST
```

---
# LAB 9: PRIVATE LINK DEEP DIVE — Private Link Service, Private Endpoints, NAT & DNS Integration

## What You'll Learn
- The crucial distinction: **Private Endpoint** (you consuming someone else's service privately) vs **Private Link Service** (you publishing YOUR OWN service for others to consume privately)
- How to put your own internal Load Balancer behind a Private Link Service and let another VNet (even another subscription/tenant) connect to it privately
- NAT IP mapping and why the consumer never sees the provider's real address space
- Connection approval workflows (Auto vs Manual)
- The complete DNS chain for Private Endpoints to ANY PaaS service (not just storage)
- Private Link vs Service Endpoint vs VNet Peering — when each one is the right call
- How to lock down a PaaS resource to DENY all public access, Private Link only

This is the lab most courses gloss over because Lab 3 only showed you *half* of Private Link — consuming. Real production environments (especially SaaS providers, or large orgs publishing internal APIs to partner VNets) need the *publishing* side too.

---

## Deep Dive: Private Endpoint vs Private Link Service

```
┌────────────────────────────────────────────────────────────────────────┐
│  PRIVATE ENDPOINT                          PRIVATE LINK SERVICE          │
│  (you're the CONSUMER)                     (you're the PROVIDER)        │
├────────────────────────────────────────────────────────────────────────┤
│  Lives in YOUR VNet                        Lives in the PROVIDER's VNet │
│  Gets a private IP in your subnet          Sits in FRONT OF a Standard  │
│  Points AT a service                       Load Balancer in your VNet   │
│  (Storage, SQL, Key Vault, or another      Other VNets/tenants create   │
│  team's Private Link Service)              Private Endpoints pointing   │
│                                             AT your Private Link Service │
│  You did this in Lab 3 (storage)           THIS IS THE NEW PART         │
└────────────────────────────────────────────────────────────────────────┘

REAL-WORLD ANALOGY:
  Private Link Service = a company puts a private "service counter" inside
                          a secure courtyard, reachable only via a private hallway
  Private Endpoint      = the private hallway YOU build from YOUR building
                          straight to that counter, never touching the public street

A SaaS vendor exposes their app via Private Link Service.
Their CUSTOMERS create Private Endpoints in their own VNets pointing at it.
Customer traffic never crosses the public internet, and the SaaS vendor never
sees the customer's internal network - they only see a NAT IP.
```

---

## Hands-On Lab Part A: Build the Provider Side (Your Own App Behind Private Link Service)

### Step 1: Deploy a Standard Internal Load Balancer (the prerequisite)

```bash
source ~/.azure-lab-vars.sh

# Private Link Service REQUIRES a Standard SKU Internal Load Balancer in front of it.
# This LB fronts whatever app you're publishing (here, simulating an internal API).

print_info "Creating internal Standard Load Balancer for the published service..."

az network lb create \
  --resource-group $RG \
  --name internal-api-lb \
  --sku Standard \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_BACKEND \
  --frontend-ip-name internal-lb-frontend \
  --backend-pool-name internal-lb-backend-pool \
  --location $LOCATION

print_success "Internal Load Balancer created!"

# Health probe + rule, same pattern as Lab 6
az network lb probe create \
  --resource-group $RG \
  --lb-name internal-api-lb \
  --name health-probe-http \
  --protocol HTTP \
  --path / \
  --port 8080

az network lb rule create \
  --resource-group $RG \
  --lb-name internal-api-lb \
  --name api-rule \
  --protocol TCP \
  --frontend-port 8080 \
  --backend-port 8080 \
  --frontend-ip-name internal-lb-frontend \
  --backend-pool-name internal-lb-backend-pool \
  --probe-name health-probe-http

print_success "Internal LB rules configured! (Add backend VMs to internal-lb-backend-pool later)"
```

### Step 2: Create the Private Link Service, Pointing at That Load Balancer

```bash
# This is the actual "publish my service privately" step.
# nat-ip-count > 1 gives the provider more headroom for simultaneous consumer connections.

print_info "Creating Private Link Service..."

az network private-link-service create \
  --resource-group $RG \
  --name internal-api-pls \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_BACKEND \
  --lb-name internal-api-lb \
  --lb-frontend-ip-configs internal-lb-frontend \
  --location $LOCATION

print_success "Private Link Service created!"

# View it - note the alias, this is what you SHARE with consumers
PLS_ALIAS=$(az network private-link-service show \
  --resource-group $RG \
  --name internal-api-pls \
  --query alias -o tsv)

print_info "Private Link Service Alias (share this with consumers): $PLS_ALIAS"

cat << 'PLS_EXPLANATION'
THE ALIAS (looks like: internal-api-pls.abc123-de45.eastus.azure.privatelinkservice)
is what you give to a consumer. They use it to create a Private Endpoint pointing
at YOUR service WITHOUT needing access to your subscription or resource IDs.

This is exactly how SaaS marketplace Private Link offerings work - the vendor gives
you an alias (or you select it from the Azure "Private Link Center"), and you
create a Private Endpoint using only that alias.
PLS_EXPLANATION
```

### Step 3: Configure Visibility and Auto-Approval (Access Control)

```bash
# By default, a Private Link Service requires MANUAL approval of each
# connection request - you see who wants to connect and approve/reject them.
# You can instead auto-approve specific subscriptions.

print_info "Restricting visibility to specific subscriptions..."

# Visibility: which subscriptions can even SEE this PLS exists
az network private-link-service update \
  --resource-group $RG \
  --name internal-api-pls \
  --visibility "<consumer-subscription-id-1>" "<consumer-subscription-id-2>"

# Auto-approval: which subscriptions get connected WITHOUT you manually approving
az network private-link-service update \
  --resource-group $RG \
  --name internal-api-pls \
  --auto-approval "<trusted-partner-subscription-id>"

print_success "Access control configured!"

cat << 'ACCESS_MODEL'
THREE VISIBILITY MODES:
  1. Restricted (default) - only subscriptions you explicitly list can see/request it
  2. Specific subscriptions with auto-approval - listed subs connect instantly, no manual step
  3. Everyone ("*") - ANY subscription can request a connection (still needs YOUR approval
     unless also in the auto-approval list) - use very carefully, mainly for public SaaS offerings
ACCESS_MODEL
```

---

## Hands-On Lab Part B: Build the Consumer Side (DR VNet Connects via Private Endpoint)

### Step 4: Create a Private Endpoint in the Consumer VNet, Targeting the PLS

```bash
# The DR VNet plays the role of "another team" or "another company" consuming our service

print_info "Creating Private Endpoint in DR VNet, pointing at our Private Link Service..."

PLS_ID=$(az network private-link-service show \
  --resource-group $RG \
  --name internal-api-pls \
  --query id -o tsv)

az network private-endpoint create \
  --resource-group $RG \
  --name consumer-to-api-pe \
  --vnet-name $VNET_DR \
  --subnet dr-backend \
  --private-connection-resource-id $PLS_ID \
  --connection-name consumer-api-connection \
  --manual-request false

print_success "Private Endpoint created! (connection request sent to the provider)"
```

### Step 5: Approve the Connection (Provider Side)

```bash
# If auto-approval wasn't configured for this subscription, the PROVIDER must
# manually approve the pending connection request

print_info "Checking pending connections on the Private Link Service..."

az network private-link-service show \
  --resource-group $RG \
  --name internal-api-pls \
  --query "privateEndpointConnections[].{Name:name, Status:privateLinkServiceConnectionState.status, Description:privateLinkServiceConnectionState.description}" \
  --output table

# Approve it (replace <connection-name> with what you see above)
print_info "Approving the pending connection..."

az network private-endpoint-connection approve \
  --resource-group $RG \
  --service-name internal-api-pls \
  --name "<connection-name-from-above>" \
  --type Microsoft.Network/privateLinkServices \
  --description "Approved for partner team integration"

print_success "Connection approved!"
```

### Step 6: Verify End-to-End — Consumer's Private IP and NAT Behavior

```bash
print_info "Checking the consumer-side Private Endpoint's assigned IP..."

az network private-endpoint show \
  --resource-group $RG \
  --name consumer-to-api-pe \
  --query "customDnsConfigs" \
  --output table

cat << 'NAT_EXPLANATION'
WHAT JUST HAPPENED UNDER THE HOOD:
1. The DR VNet (consumer) got a private IP in its OWN subnet (e.g. 10.1.2.50)
   for the Private Endpoint
2. Traffic to 10.1.2.50:8080 tunnels through Azure's private backbone to the
   Private Link Service
3. The Private Link Service NATs that traffic onto the production VNet's
   internal Load Balancer
4. The consumer NEVER sees any IP from the production VNet (10.0.0.0/16) -
   total network isolation, even though traffic is flowing between them
5. From the provider's perspective, inbound connections appear to originate
   from a NAT IP, not the consumer's real source IP
NAT_EXPLANATION

print_success "Provider <-> Consumer Private Link connection fully working!"
```

---

## Hands-On Lab Part C: Private Endpoints for PaaS — The Full DNS Chain (Every Service)

```bash
# Lab 3 did this for Storage (blob). The PATTERN is identical for every PaaS service -
# only the privatelink.* zone name and --group-ids change. Reference table:

cat << 'PRIVATELINK_ZONES_TABLE'
SERVICE                  GROUP ID          PRIVATE DNS ZONE NAME
------------------------  ----------------  --------------------------------------------
Storage (Blob)            blob              privatelink.blob.core.windows.net
Storage (File)            file              privatelink.file.core.windows.net
Storage (Queue)           queue             privatelink.queue.core.windows.net
Azure SQL Database        sqlServer         privatelink.database.windows.net
Azure Key Vault           vault             privatelink.vaultcore.azure.net
Azure Cosmos DB            Sql               privatelink.documents.azure.com
App Service / Web App      sites             privatelink.azurewebsites.net
Azure Container Registry   registry          privatelink.azurecr.io
Azure Cache for Redis      redisCache        privatelink.redis.cache.windows.net
Event Hubs                 namespace         privatelink.servicebus.windows.net
Synapse Analytics           Sql               privatelink.sql.azuresynapse.net
PRIVATELINK_ZONES_TABLE
```

### Step 7: Private Endpoint for Azure Key Vault (a Second Worked Example)

```bash
print_info "Creating a Key Vault to demonstrate the pattern on a different service..."

KV_NAME="lab-kv-$(openssl rand -hex 3)"

az keyvault create \
  --resource-group $RG \
  --name $KV_NAME \
  --location $LOCATION \
  --sku standard

print_success "Key Vault created: $KV_NAME"

# Lock down public access FIRST - this is the real production pattern
print_info "Disabling public network access on Key Vault..."

az keyvault update \
  --resource-group $RG \
  --name $KV_NAME \
  --default-action Deny \
  --public-network-access Disabled

print_success "Public access disabled!"

# Create the Private Endpoint
KV_ID=$(az keyvault show --resource-group $RG --name $KV_NAME --query id -o tsv)

print_info "Creating Private Endpoint for Key Vault..."

az network private-endpoint create \
  --resource-group $RG \
  --name kv-private-endpoint \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_BACKEND \
  --private-connection-resource-id $KV_ID \
  --group-ids vault \
  --connection-name kv-connection

print_success "Key Vault Private Endpoint created!"

# Create the matching Private DNS Zone (note: DIFFERENT zone name than storage)
print_info "Creating Private DNS Zone for Key Vault..."

az network private-dns zone create \
  --resource-group $RG \
  --name privatelink.vaultcore.azure.net

az network private-dns link vnet create \
  --resource-group $RG \
  --zone-name privatelink.vaultcore.azure.net \
  --name kv-vnet-link \
  --virtual-network $VNET_PRIMARY \
  --registration-enabled false

print_success "DNS Zone created and linked!"

# Hook the Private Endpoint's DNS records into the zone automatically
# This is the cleaner way vs manually adding A records (what Lab 3 did manually)
print_info "Creating zone group to auto-manage DNS records..."

az network private-endpoint dns-zone-group create \
  --resource-group $RG \
  --endpoint-name kv-private-endpoint \
  --name kv-zone-group \
  --private-dns-zone privatelink.vaultcore.azure.net \
  --zone-name vault

print_success "DNS zone group created! A records now auto-managed by Azure."

cat << 'ZONE_GROUP_NOTE'
NICE UPGRADE FROM LAB 3:
Instead of manually fetching the endpoint IP and creating an A record
(what we did with Storage in Lab 3), "dns-zone-group" tells Azure to
automatically create/update/remove the DNS record whenever the Private
Endpoint's IP changes. This is the recommended production pattern -
use it for every new Private Endpoint going forward.
ZONE_GROUP_NOTE
```

### Step 8: Test and Prove Public Access Is Actually Blocked

```bash
cat << 'PROOF_TEST'
PROOF THE LOCKDOWN WORKS (after a VM exists in the backend subnet):

FROM A VM INSIDE THE VNET (backend subnet):
  nslookup <kv-name>.vault.azure.net
  -> Expected: resolves to a PRIVATE IP (10.0.2.x) because of the Private DNS Zone
  az keyvault secret list --vault-name <kv-name>
  -> Expected: SUCCEEDS (reaches Key Vault via the private path)

FROM YOUR LAPTOP / THE PUBLIC INTERNET:
  nslookup <kv-name>.vault.azure.net
  -> Expected: resolves to the PUBLIC IP (since you're not using the private zone)
  curl https://<kv-name>.vault.azure.net/secrets?api-version=7.4
  -> Expected: connection REFUSED or 403 - public-network-access is Disabled

This is the real compliance-grade pattern: "PaaS resource reachable ONLY
from inside my private network" - exactly what auditors ask for.
PROOF_TEST

print_success "Lab 9 Complete!"
```

---

## Decision Matrix: Service Endpoint vs Private Endpoint vs Private Link Service vs VNet Peering

```
Scenario                                              Best Choice
-----------------------------------------------------  --------------------------------
Cheapest way to secure traffic to Storage/SQL          Service Endpoint
  from within the SAME VNet/region, no cross-VNet
  or on-prem need
Need PaaS reachable from on-prem, peered VNets,         Private Endpoint
  or want a real private IP + DNS-integrated access
You're PUBLISHING your own app/API for other VNets      Private Link Service
  or other tenants/customers to consume privately
Two of YOUR OWN VNets just need to talk to each          VNet Peering
  other directly (no NAT, full bidirectional access,
  cheapest, no PaaS involved)
Publishing a SaaS product to many customer tenants       Private Link Service +
  without giving them network-level trust                NAT isolation + manual/auto approval workflow

KEY DIFFERENCE TO REMEMBER FOR INTERVIEWS:
  Service Endpoint  = still uses the PaaS service's public IP, just stays on Azure backbone,
                       and adds your VNet's identity to the resource firewall
  Private Endpoint  = gets an actual PRIVATE IP in your subnet - the public IP path doesn't
                       enter the picture at all, can be combined with disabling public access entirely
```
# LAB 10: AZURE FIREWALL & DDOS PROTECTION

## What You'll Learn
- Deploying Azure Firewall (Standard) into a hub VNet
- Application rules (FQDN-based filtering) vs Network rules (IP/port-based) vs NAT rules
- Forcing all outbound traffic through the firewall using UDRs (tying back to Lab 4)
- Firewall Policy (the modern, reusable way to manage rules)
- DDoS Protection Standard vs the always-on Basic tier

---

## Deep Dive: Azure Firewall Rule Processing Order

```
Azure Firewall evaluates rules in this FIXED order (regardless of how you create them):
  1. NAT rules        (DNAT - translate a public IP:port to an internal IP:port)
  2. Network rules     (Layer 3/4 - IP, port, protocol)
  3. Application rules (Layer 7 - FQDN/domain filtering, HTTP/HTTPS/SQL only)

Within each category, rules are processed top-down by priority within Rule Collections,
and Rule Collections are processed by priority within Rule Collection Groups.

Default action if NOTHING matches: DENY (Azure Firewall is default-deny, unlike NSGs'
implicit allow-outbound)
```

## Hands-On Lab: Deploy and Configure Azure Firewall

### Step 1: Create the Firewall Subnet (must be named exactly `AzureFirewallSubnet`)

```bash
source ~/.azure-lab-vars.sh

print_info "Creating AzureFirewallSubnet..."

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name AzureFirewallSubnet \
  --address-prefix $SUBNET_FIREWALL_SPACE

print_success "Firewall subnet created!"
```

### Step 2: Create a Public IP and Deploy the Firewall

```bash
print_info "Creating Public IP for the firewall..."

az network public-ip create \
  --resource-group $RG \
  --name firewall-public-ip \
  --sku Standard \
  --allocation-method Static \
  --location $LOCATION

print_info "Deploying Azure Firewall (takes 5-10 minutes)..."

az network firewall create \
  --resource-group $RG \
  --name production-firewall \
  --location $LOCATION \
  --sku AZFW_VNet \
  --tier Standard

az network firewall ip-config create \
  --resource-group $RG \
  --firewall-name production-firewall \
  --name fw-ip-config \
  --public-ip-address firewall-public-ip \
  --vnet-name $VNET_PRIMARY

print_success "Firewall deployed!"

# Grab its private IP - all UDRs will point here
FIREWALL_PRIVATE_IP=$(az network firewall show \
  --resource-group $RG \
  --name production-firewall \
  --query "ipConfigurations[0].privateIPAddress" -o tsv)

print_info "Firewall Private IP: $FIREWALL_PRIVATE_IP"
```

### Step 3: Create a Firewall Policy and Application Rules

```bash
print_info "Creating Firewall Policy..."

az network firewall policy create \
  --resource-group $RG \
  --name production-fw-policy \
  --location $LOCATION

# Associate the policy with the firewall
az network firewall update \
  --resource-group $RG \
  --name production-firewall \
  --firewall-policy production-fw-policy

print_info "Creating Application Rule Collection Group..."

az network firewall policy rule-collection-group create \
  --resource-group $RG \
  --policy-name production-fw-policy \
  --name app-rules-group \
  --priority 200

print_info "Adding Application Rule: allow outbound to Windows Update / Ubuntu repos..."

az network firewall policy rule-collection-group collection add-filter-collection \
  --resource-group $RG \
  --policy-name production-fw-policy \
  --rule-collection-group-name app-rules-group \
  --name allow-os-updates \
  --collection-priority 100 \
  --action Allow \
  --rule-name allow-ubuntu-updates \
  --rule-type ApplicationRule \
  --source-addresses 10.0.0.0/16 \
  --target-fqdns "*.ubuntu.com" "security.ubuntu.com" \
  --protocols Http=80 Https=443

print_success "Application rule added!"
```

### Step 4: Add Network Rules (Layer 3/4)

```bash
print_info "Creating Network Rule Collection Group..."

az network firewall policy rule-collection-group create \
  --resource-group $RG \
  --policy-name production-fw-policy \
  --name network-rules-group \
  --priority 100

print_info "Adding Network Rule: allow DNS outbound..."

az network firewall policy rule-collection-group collection add-filter-collection \
  --resource-group $RG \
  --policy-name production-fw-policy \
  --rule-collection-group-name network-rules-group \
  --name allow-dns \
  --collection-priority 100 \
  --action Allow \
  --rule-name allow-dns-outbound \
  --rule-type NetworkRule \
  --source-addresses 10.0.0.0/16 \
  --destination-addresses '*' \
  --destination-ports 53 \
  --ip-protocols UDP

print_success "Network rule added!"
```

### Step 5: Add a DNAT Rule (Expose an Internal VM Through the Firewall's Public IP)

```bash
print_info "Creating DNAT Rule Collection Group..."

az network firewall policy rule-collection-group create \
  --resource-group $RG \
  --policy-name production-fw-policy \
  --name nat-rules-group \
  --priority 50

FIREWALL_PUBLIC_IP=$(az network public-ip show --resource-group $RG --name firewall-public-ip --query ipAddress -o tsv)

print_info "Adding DNAT rule: firewall public IP:3389 -> internal RDP server..."

az network firewall policy rule-collection-group collection add-nat-collection \
  --resource-group $RG \
  --policy-name production-fw-policy \
  --rule-collection-group-name nat-rules-group \
  --name rdp-dnat \
  --collection-priority 100 \
  --action DNAT \
  --rule-name allow-rdp-to-jumpbox \
  --source-addresses '*' \
  --destination-addresses $FIREWALL_PUBLIC_IP \
  --destination-ports 3389 \
  --translated-address 10.0.4.10 \
  --translated-port 3389 \
  --ip-protocols TCP

print_success "DNAT rule added: connect to $FIREWALL_PUBLIC_IP:3389 -> reaches 10.0.4.10:3389"
```

### Step 6: Force Outbound Traffic Through the Firewall (UDR)

```bash
# Tying back to Lab 4 - this is the route that was created as a placeholder there.
# Now the firewall actually exists, so this route is fully functional.

print_info "Updating route table to point default route at the real firewall IP..."

az network route-table route update \
  --resource-group $RG \
  --route-table-name route-table-production \
  --name route-to-firewall \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address $FIREWALL_PRIVATE_IP

print_success "All outbound traffic from the backend subnet now routes through the firewall!"

# Verify
az network route-table route show \
  --resource-group $RG \
  --route-table-name route-table-production \
  --name route-to-firewall \
  --output table
```

### Step 7: Test and Verify Firewall Logs

```bash
cat << 'FIREWALL_TEST'
TEST FROM A VM IN THE BACKEND SUBNET (after route table is associated):
  curl https://security.ubuntu.com   -> should SUCCEED (matches application rule)
  curl https://some-random-blocked-site.com -> should FAIL (no matching rule, default deny)

VIEW FIREWALL LOGS (after enabling diagnostics - see Lab 12):
  az monitor log-analytics query \
    --workspace <workspace-id> \
    --analytics-query "AzureDiagnostics | where Category == 'AzureFirewallApplicationRule' | take 20"
FIREWALL_TEST

print_success "Lab 10, Part A (Azure Firewall) Complete!"
```

---

## Hands-On Lab Part B: DDoS Protection

### Step 8: Understand the Two Tiers

```bash
cat << 'DDOS_TIERS'
DDOS PROTECTION BASIC (always on, free, automatic):
  - Built into the Azure network platform for EVERY public IP, no setup needed
  - Protects against common volumetric/network-layer attacks
  - No visibility/metrics, no SLA, no cost

DDOS PROTECTION STANDARD (paid, must enable explicitly):
  - Tuned mitigation policies based on YOUR resource's actual traffic patterns
  - Real-time attack metrics, alerts, and post-attack mitigation reports
  - Cost protection guarantee (Azure credits you for scale-out costs during an attack)
  - SLA-backed
  - Works at the VNET level - protects every Standard public IP in the protected VNet
DDOS_TIERS
```

### Step 9: Enable DDoS Protection Standard

```bash
print_info "Creating a DDoS Protection Plan..."

az network ddos-protection create \
  --resource-group $RG \
  --name production-ddos-plan \
  --location $LOCATION

print_success "DDoS Protection Plan created!"

print_info "Associating the plan with the production VNet..."

az network vnet update \
  --resource-group $RG \
  --name $VNET_PRIMARY \
  --ddos-protection-plan production-ddos-plan \
  --ddos-protection true

print_success "DDoS Protection Standard enabled on production VNet!"

# Verify
az network vnet show \
  --resource-group $RG \
  --name $VNET_PRIMARY \
  --query "enableDdosProtection"

print_success "Lab 10 Complete!"
print_info "You now have:"
echo "  ✓ Azure Firewall with Application, Network, and DNAT rules"
echo "  ✓ Outbound traffic forced through the firewall via UDR"
echo "  ✓ DDoS Protection Standard protecting all public IPs in the VNet"
```

---
# LAB 11: APPLICATION GATEWAY WITH WAF

## What You'll Learn
- Application Gateway architecture: Layer 7 load balancing, path-based routing, SSL termination
- Deploying Application Gateway v2 with a Web Application Firewall (WAF)
- WAF modes: Detection vs Prevention, and OWASP rule sets
- Path-based routing rules (route `/api/*` to one backend, `/images/*` to another)
- Testing WAF blocking with a real attack pattern (SQL injection string)
- Application Gateway vs Azure Load Balancer vs Front Door — where each fits

---

## Deep Dive: Why Application Gateway Is Different From Azure Load Balancer

```
Azure Load Balancer (Lab 6)        Application Gateway (this lab)
----------------------------------  ----------------------------------------
Layer 4 (TCP/UDP)                  Layer 7 (HTTP/HTTPS)
Doesn't see URLs or headers         Reads URL paths, headers, cookies
No SSL termination                  Can terminate SSL (decrypt, inspect, re-encrypt)
No WAF                              Built-in Web Application Firewall option
Routes by IP:port only              Routes by URL path, hostname (multi-site), etc.
Good for: raw TCP/UDP distribution  Good for: web apps, APIs, anything HTTP(S)
```

## Hands-On Lab: Deploy Application Gateway v2 + WAF

### Step 1: Create the Application Gateway Subnet

```bash
source ~/.azure-lab-vars.sh

print_info "Creating Application Gateway subnet..."

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name appgw-subnet \
  --address-prefix 10.0.7.0/24

print_success "AppGW subnet created!"
```

### Step 2: Create a Public IP and the WAF Policy

```bash
print_info "Creating Public IP for Application Gateway..."

az network public-ip create \
  --resource-group $RG \
  --name appgw-public-ip \
  --sku Standard \
  --allocation-method Static \
  --location $LOCATION

print_info "Creating WAF Policy with OWASP rule set..."

az network application-gateway waf-policy create \
  --resource-group $RG \
  --name production-waf-policy \
  --location $LOCATION

# Set the policy to Prevention mode using the OWASP 3.2 managed rule set
az network application-gateway waf-policy managed-rule rule-set add \
  --resource-group $RG \
  --policy-name production-waf-policy \
  --type OWASP \
  --version 3.2

az network application-gateway waf-policy policy-setting update \
  --resource-group $RG \
  --policy-name production-waf-policy \
  --state Enabled \
  --mode Prevention

print_success "WAF Policy created in Prevention mode with OWASP 3.2 rules!"

cat << 'WAF_MODES'
DETECTION MODE: WAF logs what it WOULD have blocked but lets all traffic through
                 - use this first to tune rules and avoid blocking real users
PREVENTION MODE: WAF actively BLOCKS matched requests and logs them
                 - use this in production after you've validated Detection mode logs
WAF_MODES
```

### Step 3: Deploy Application Gateway with WAF

```bash
print_info "Deploying Application Gateway v2 (WAF_v2 SKU)... takes 10-20 minutes"

az network application-gateway create \
  --resource-group $RG \
  --name production-appgw \
  --location $LOCATION \
  --sku WAF_v2 \
  --capacity 2 \
  --vnet-name $VNET_PRIMARY \
  --subnet appgw-subnet \
  --public-ip-address appgw-public-ip \
  --frontend-port 80 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --servers 10.0.1.10 10.0.1.11 \
  --priority 100

print_success "Application Gateway deployed!"

# Attach the WAF policy
print_info "Attaching WAF policy to the gateway..."

az network application-gateway update \
  --resource-group $RG \
  --name production-appgw \
  --set firewallPolicy.id="$(az network application-gateway waf-policy show --resource-group $RG --name production-waf-policy --query id -o tsv)"

print_success "WAF policy attached!"
```

### Step 4: Add Path-Based Routing (Route by URL Path)

```bash
# Real-world pattern: /api/* -> backend API pool, /images/* -> backend static content pool,
# everything else -> default web pool

print_info "Creating a second backend pool for the API tier..."

az network application-gateway address-pool create \
  --resource-group $RG \
  --gateway-name production-appgw \
  --name api-backend-pool \
  --servers 10.0.2.10 10.0.2.11

print_info "Creating URL Path Map for path-based routing..."

az network application-gateway url-path-map create \
  --resource-group $RG \
  --gateway-name production-appgw \
  --name path-routing-map \
  --paths "/api/*" \
  --address-pool api-backend-pool \
  --default-address-pool appGatewayBackendPool \
  --http-settings appGatewayBackendHttpSettings \
  --default-http-settings appGatewayBackendHttpSettings

print_success "Path-based routing configured: /api/* -> API pool, everything else -> web pool"
```

### Step 5: Test WAF Blocking With a Real Attack Pattern

```bash
print_info "Getting Application Gateway public IP..."

APPGW_IP=$(az network public-ip show --resource-group $RG --name appgw-public-ip --query ipAddress -o tsv)

print_info "Application Gateway Public IP: $APPGW_IP"

cat << 'WAF_TEST'
TEST 1: Normal request (should succeed and reach the backend)
  curl http://<APPGW_IP>/

TEST 2: SQL Injection attempt (should be BLOCKED by WAF in Prevention mode)
  curl "http://<APPGW_IP>/?id=1' OR '1'='1"
  Expected: HTTP 403 Forbidden (blocked by OWASP rule matching SQLi pattern)

TEST 3: XSS attempt (should also be BLOCKED)
  curl "http://<APPGW_IP>/?search=<script>alert(1)</script>"
  Expected: HTTP 403 Forbidden

CHECK WHAT WAS BLOCKED AND WHY (after enabling diagnostics in Lab 12):
  az monitor log-analytics query \
    --workspace <workspace-id> \
    --analytics-query "AzureDiagnostics | where Category == 'ApplicationGatewayFirewallLog' | take 20"
WAF_TEST

print_success "Lab 11 Complete!"
print_info "You now have:"
echo "  ✓ Application Gateway v2 with WAF in Prevention mode"
echo "  ✓ Path-based routing (/api/* vs default)"
echo "  ✓ OWASP 3.2 managed rules actively blocking SQLi/XSS"
```

---

## Decision Matrix: AppGW vs Load Balancer vs Front Door

```
Need                                                    Best Choice
-------------------------------------------------------  ------------------------
Raw TCP/UDP load balancing, regional, no HTTP awareness   Azure Load Balancer
HTTP(S) load balancing within ONE region, need WAF,        Application Gateway
  path-based routing, SSL offload
Global load balancing across MULTIPLE regions,             Azure Front Door
  CDN caching, anycast entry point, global WAF
Internal-only HTTP load balancing inside a VNet             Application Gateway
  (set as Internal, no public IP)                            (Internal mode)
```
# LAB 12: MONITORING, FLOW LOGS & TROUBLESHOOTING (Network Watcher)

## What You'll Learn
- Setting up a Log Analytics Workspace as the central destination for all network logs
- NSG Flow Logs v2 (who's talking to whom, accepted/denied, at the flow level)
- Diagnostic settings for Firewall, App Gateway, VPN Gateway, and DNS Resolver
- Network Watcher's troubleshooting toolkit: effective routes, effective NSG rules, IP flow verify, connection troubleshoot, packet capture
- Building one practical alert rule

---

## Hands-On Lab: Centralized Logging

### Step 1: Create the Log Analytics Workspace

```bash
source ~/.azure-lab-vars.sh

print_info "Creating Log Analytics Workspace..."

az monitor log-analytics workspace create \
  --resource-group $RG \
  --workspace-name networking-logs-workspace \
  --location $LOCATION

WORKSPACE_ID=$(az monitor log-analytics workspace show \
  --resource-group $RG \
  --workspace-name networking-logs-workspace \
  --query id -o tsv)

print_success "Workspace created!"
print_info "Workspace ID: $WORKSPACE_ID"
```

### Step 2: Create a Storage Account for Flow Logs

```bash
print_info "Creating storage account for NSG Flow Logs..."

FLOWLOG_STORAGE="flowlogs$(openssl rand -hex 3)"

az storage account create \
  --resource-group $RG \
  --name $FLOWLOG_STORAGE \
  --location $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2

print_success "Flow log storage account created: $FLOWLOG_STORAGE"
```

### Step 3: Enable NSG Flow Logs (v2, with Traffic Analytics)

```bash
# Network Watcher must be enabled in the region first (usually automatic, but verify)
print_info "Verifying Network Watcher is enabled..."

az network watcher list --output table

print_info "Enabling Flow Logs on the Frontend NSG..."

NSG_ID=$(az network nsg show --resource-group $RG --name nsg-frontend --query id -o tsv)
STORAGE_ID=$(az storage account show --resource-group $RG --name $FLOWLOG_STORAGE --query id -o tsv)

az network watcher flow-log create \
  --resource-group $RG \
  --location $LOCATION \
  --name frontend-nsg-flowlog \
  --nsg $NSG_ID \
  --storage-account $STORAGE_ID \
  --enabled true \
  --format JSON \
  --log-version 2 \
  --retention 30 \
  --workspace $WORKSPACE_ID \
  --interval 10

print_success "Flow logs enabled with Traffic Analytics (10-min processing interval)!"

cat << 'FLOWLOG_EXPLANATION'
WHAT FLOW LOGS RECORD (per 5-tuple flow): source IP, destination IP, source port,
destination port, protocol, direction, and whether the NSG rule ALLOWED or DENIED it -
captured every few minutes, written as JSON to blob storage.

TRAFFIC ANALYTICS (the workspace integration) processes these raw logs into
visual dashboards: top talkers, malicious IP flags, traffic between subnets, etc.
This is what you'd actually look at day-to-day instead of raw JSON blobs.
FLOWLOG_EXPLANATION
```

### Step 4: Enable Diagnostic Settings on Key Resources

```bash
# Repeat this pattern for ANY networking resource - firewall, app gateway, VPN gateway, etc.

print_info "Enabling diagnostics on Azure Firewall..."

FIREWALL_ID=$(az network firewall show --resource-group $RG --name production-firewall --query id -o tsv)

az monitor diagnostic-settings create \
  --name firewall-diagnostics \
  --resource $FIREWALL_ID \
  --workspace $WORKSPACE_ID \
  --logs '[{"category": "AzureFirewallApplicationRule", "enabled": true}, {"category": "AzureFirewallNetworkRule", "enabled": true}, {"category": "AzureFirewallDnsProxy", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'

print_success "Firewall diagnostics enabled!"

print_info "Enabling diagnostics on Application Gateway..."

APPGW_ID=$(az network application-gateway show --resource-group $RG --name production-appgw --query id -o tsv)

az monitor diagnostic-settings create \
  --name appgw-diagnostics \
  --resource $APPGW_ID \
  --workspace $WORKSPACE_ID \
  --logs '[{"category": "ApplicationGatewayAccessLog", "enabled": true}, {"category": "ApplicationGatewayFirewallLog", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'

print_success "Application Gateway diagnostics enabled!"

print_info "Enabling diagnostics on VPN Gateway..."

VPNGW_ID=$(az network vnet-gateway show --resource-group $RG --name $VPN_GW_AZURE --query id -o tsv)

az monitor diagnostic-settings create \
  --name vpngw-diagnostics \
  --resource $VPNGW_ID \
  --workspace $WORKSPACE_ID \
  --logs '[{"category": "GatewayDiagnosticLog", "enabled": true}, {"category": "TunnelDiagnosticLog", "enabled": true}, {"category": "IKEDiagnosticLog", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]'

print_success "VPN Gateway diagnostics enabled!"
```

---

## Hands-On Lab Part B: Network Watcher Troubleshooting Toolkit

### Step 5: Effective NSG Rules (What ACTUALLY Applies to a NIC)

```bash
# A NIC can have NSG rules from BOTH the subnet AND the NIC itself - this command
# shows you the merged, ACTUAL effective rule set, which is what really matters

print_info "Checking effective NSG rules on a NIC..."

az network nic list-effective-nsg \
  --resource-group $RG \
  --name <vm-nic-name> \
  --output table
```

### Step 6: Effective Routes (What ACTUALLY Applies, System + UDR Combined)

```bash
print_info "Checking effective routes on a NIC..."

az network nic show-effective-route-table \
  --resource-group $RG \
  --name <vm-nic-name> \
  --output table

cat << 'EFFECTIVE_ROUTES_NOTE'
This merges System Routes + your UDRs + BGP-learned routes (if any) into the
FINAL routing decision table - exactly what the VM's traffic will actually do.
Always check this BEFORE assuming a UDR "isn't working."
EFFECTIVE_ROUTES_NOTE
```

### Step 7: IP Flow Verify (Will This Specific Traffic Be Allowed or Denied?)

```bash
print_info "Testing if traffic from frontend to backend on port 8080 is allowed..."

az network watcher test-ip-flow \
  --resource-group $RG \
  --vm <vm-name> \
  --direction Outbound \
  --protocol TCP \
  --local 10.0.1.10:50000 \
  --remote 10.0.2.10:8080

# Returns: Access = Allow/Deny, and WHICH rule caused that decision
# This instantly answers "why can't my VM reach X" without manually reading 20 NSG rules
```

### Step 8: Connection Troubleshoot (End-to-End Path Test)

```bash
print_info "Testing full connectivity between two VMs..."

az network watcher test-connectivity \
  --resource-group $RG \
  --source-resource <source-vm-name> \
  --dest-resource <dest-vm-name> \
  --dest-port 8080 \
  --protocol Tcp

# Returns: connection status, latency, and a HOP-BY-HOP breakdown of the path,
# flagging exactly which hop is causing failure (NSG, route, firewall, etc.)
```

### Step 9: Packet Capture (When You Need the Actual Bytes)

```bash
print_info "Starting a packet capture on a VM..."

az network watcher packet-capture create \
  --resource-group $RG \
  --vm <vm-name> \
  --name troubleshoot-capture \
  --storage-account $FLOWLOG_STORAGE \
  --time-limit 60 \
  --filters '[{"protocol":"TCP","localPort":"8080"}]'

print_info "Capture runs for 60 seconds, saved to storage account, downloadable as .cap"
print_info "Open with Wireshark for full packet-level analysis"
```

### Step 10: Create a Practical Alert

```bash
print_info "Creating an alert for VPN tunnel disconnection..."

az monitor metrics alert create \
  --resource-group $RG \
  --name vpn-tunnel-down-alert \
  --scopes $VPNGW_ID \
  --condition "avg TunnelAverageBandwidth < 1" \
  --description "Alerts when VPN tunnel bandwidth drops near zero (likely disconnected)" \
  --evaluation-frequency 5m \
  --window-size 15m \
  --severity 1

print_success "Alert created!"

print_success "Lab 12 Complete!"
print_info "You now have:"
echo "  ✓ Centralized Log Analytics Workspace"
echo "  ✓ NSG Flow Logs v2 with Traffic Analytics"
echo "  ✓ Diagnostics on Firewall, App Gateway, and VPN Gateway"
echo "  ✓ Full Network Watcher troubleshooting toolkit hands-on (effective rules,"
echo "    effective routes, IP flow verify, connection troubleshoot, packet capture)"
echo "  ✓ A working alert rule"
```

---
# LAB 13: CAPSTONE — COMPLETE REAL-WORLD HUB-AND-SPOKE ARCHITECTURE

## What You'll Build

A production-pattern **hub-and-spoke topology** — the architecture you will see in almost every real Azure enterprise environment and almost every Azure networking interview. Everything from Labs 0-12 comes together here.

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                         SIMULATED ON-PREMISES                                 │
│                         (192.168.0.0/16)                                      │
│                              │                                                │
│                    Site-to-Site VPN (Lab 7)                                   │
│                              │                                                │
└──────────────────────────────┼────────────────────────────────────────────────┘
                                │
┌───────────────────────────────▼───────────────────────────────────────────────┐
│  HUB VNET (10.0.0.0/16) - shared services, NOT app workloads                  │
│  ┌────────────────┐  ┌──────────────┐  ┌───────────────┐  ┌────────────────┐ │
│  │ GatewaySubnet   │  │ AzureFirewall │  │ DNS Resolver  │  │ AzureBastion   │ │
│  │ VPN Gateway     │  │ Subnet (Lab10)│  │ subnets(Lab8) │  │ Subnet         │ │
│  │ (Lab 7)         │  │               │  │               │  │                │ │
│  └────────────────┘  └──────────────┘  └───────────────┘  └────────────────┘ │
└───────────────┬───────────────────────────────────┬──────────────────────────┘
                │ VNet Peering                       │ VNet Peering
                │ (gateway transit)                  │ (gateway transit)
┌───────────────▼───────────────┐   ┌────────────────▼───────────────────────┐
│  SPOKE 1: Production VNet      │   │  SPOKE 2: DR / Secondary VNet           │
│  (existing $VNET_PRIMARY)      │   │  (existing $VNET_DR)                    │
│  Frontend / Backend / Database │   │  Consumes Private Link Service (Lab 9)  │
│  App Gateway + WAF (Lab 11)    │   │                                          │
│  Private Endpoints (Lab 3,9)   │   │                                          │
└─────────────────────────────────┘   └──────────────────────────────────────────┘

ALL outbound/inter-spoke traffic routes through the HUB firewall via UDRs (Lab 4 + 10).
ALL of it is logged centrally (Lab 12). ALL DNS resolves through the hub's resolver (Lab 8).
```

This is exactly the **hub-and-spoke** model Microsoft's own Cloud Adoption Framework recommends, and it's why every lab you did had a purpose beyond itself.

---

## Step 1: Re-Peer Everything With Gateway Transit (the Missing Piece)

```bash
source ~/.azure-lab-vars.sh

# In Lab 5 we peered production <-> DR directly. In a TRUE hub-and-spoke,
# spokes peer to the HUB, and the hub shares its VPN Gateway via "gateway transit" -
# so spokes reach on-prem WITHOUT each spoke needing its own gateway.

print_info "Re-establishing peering: Production (spoke) <-> treating it as hub-attached..."

cat << 'GATEWAY_TRANSIT_NOTE'
NOTE: In this lab, $VNET_PRIMARY is ALSO playing double-duty as the hub
(it holds the VPN Gateway from Lab 7, the Firewall from Lab 10, and the DNS
Resolver from Lab 8) - which is a perfectly valid small-scale pattern.
For a "pure" hub-and-spoke at larger scale, you would create a SEPARATE
hub-vnet and peer production-vnet + dr-vnet to it as spokes, each using
--use-remote-gateways on the spoke side and --allow-gateway-transit on the hub side.
This is the exact pattern, just decoupled into its own dedicated hub VNet:
GATEWAY_TRANSIT_NOTE

# Reference: create a dedicated hub and attach spokes this way -
az network vnet create \
  --resource-group $RG \
  --name hub-vnet \
  --address-prefix 10.99.0.0/16 \
  --location $LOCATION

az network vnet peering create \
  --resource-group $RG \
  --name spoke-prod-to-hub \
  --vnet-name $VNET_PRIMARY \
  --remote-vnet hub-vnet \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --use-remote-gateways

az network vnet peering create \
  --resource-group $RG \
  --name hub-to-spoke-prod \
  --vnet-name hub-vnet \
  --remote-vnet $VNET_PRIMARY \
  --allow-vnet-access \
  --allow-forwarded-traffic \
  --allow-gateway-transit

print_success "Hub-spoke peering pattern demonstrated (reference architecture)!"
```

---

## Step 2: Deploy the Full VM Fleet (3-Tier + Management)

```bash
print_info "Creating SSH key pair for VM access..."

az sshkey create \
  --resource-group $RG \
  --name lab-ssh-key \
  --location $LOCATION 2>/dev/null || print_info "SSH key may already exist, continuing..."

print_info "Deploying Web Server 1 (Frontend)..."

az vm create \
  --resource-group $RG \
  --name $VM_WEB_1 \
  --image Ubuntu2204 \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_FRONTEND \
  --admin-username $VM_ADMIN_USER \
  --generate-ssh-keys \
  --size Standard_B1s \
  --public-ip-address "" \
  --nsg ""

print_info "Deploying Web Server 2 (Frontend)..."

az vm create \
  --resource-group $RG \
  --name $VM_WEB_2 \
  --image Ubuntu2204 \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_FRONTEND \
  --admin-username $VM_ADMIN_USER \
  --generate-ssh-keys \
  --size Standard_B1s \
  --public-ip-address "" \
  --nsg ""

print_info "Deploying App Server (Backend)..."

az vm create \
  --resource-group $RG \
  --name $VM_APP_1 \
  --image Ubuntu2204 \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_BACKEND \
  --admin-username $VM_ADMIN_USER \
  --generate-ssh-keys \
  --size Standard_B1s \
  --public-ip-address "" \
  --nsg ""

print_info "Deploying Database Server..."

az vm create \
  --resource-group $RG \
  --name $VM_DB_1 \
  --image Ubuntu2204 \
  --vnet-name $VNET_PRIMARY \
  --subnet $SUBNET_DATABASE \
  --admin-username $VM_ADMIN_USER \
  --generate-ssh-keys \
  --size Standard_B1s \
  --public-ip-address "" \
  --nsg ""

print_success "All 4 VMs deployed (no public IPs - access via Bastion only)!"
```

### Step 3: Deploy Azure Bastion for Secure Management Access

```bash
print_info "Creating AzureBastionSubnet..."

az network vnet subnet create \
  --resource-group $RG \
  --vnet-name $VNET_PRIMARY \
  --name AzureBastionSubnet \
  --address-prefix $SUBNET_BASTION_SPACE

print_info "Creating Public IP for Bastion..."

az network public-ip create \
  --resource-group $RG \
  --name bastion-public-ip \
  --sku Standard \
  --allocation-method Static \
  --location $LOCATION

print_info "Deploying Azure Bastion (takes 5-10 minutes)..."

az network bastion create \
  --resource-group $RG \
  --name production-bastion \
  --public-ip-address bastion-public-ip \
  --vnet-name $VNET_PRIMARY \
  --location $LOCATION \
  --sku Standard

print_success "Bastion deployed! Connect to any VM via the Azure Portal -> VM -> Connect -> Bastion"
print_info "No VM needs a public IP, no NSG rule for SSH from the internet needed - zero exposed surface"
```

### Step 4: Attach Backend VMs to the Internal Load Balancer Pools (ties Lab 6 + Lab 9)

```bash
print_info "Adding Web Servers to the production load balancer backend pool..."

az network nic ip-config address-pool add \
  --resource-group $RG \
  --nic-name "${VM_WEB_1}VMNic" \
  --ip-config-name ipconfig1 \
  --lb-name production-lb \
  --address-pool lb-backend-pool

az network nic ip-config address-pool add \
  --resource-group $RG \
  --nic-name "${VM_WEB_2}VMNic" \
  --ip-config-name ipconfig1 \
  --lb-name production-lb \
  --address-pool lb-backend-pool

print_info "Adding App Server to the internal API load balancer (Private Link Service backend)..."

az network nic ip-config address-pool add \
  --resource-group $RG \
  --nic-name "${VM_APP_1}VMNic" \
  --ip-config-name ipconfig1 \
  --lb-name internal-api-lb \
  --address-pool internal-lb-backend-pool

print_success "All backend pools populated with real VMs!"
```

---

## Step 5: Full End-to-End Verification Checklist

```bash
cat << 'FULL_E2E_CHECKLIST'
RUN THROUGH THIS CHECKLIST TO PROVE THE WHOLE ARCHITECTURE WORKS:

[ ] 1. CONNECTIVITY (Lab 5, 7)
      - Connect via Bastion to web-server-1
      - ping 10.0.2.10 (app server)               -> should succeed (same VNet)
      - ping 192.168.1.10 (simulated on-prem VM)   -> should succeed (via VPN tunnel)
      - ping 10.1.x.x (DR VNet VM)                  -> should succeed (via peering)

[ ] 2. SECURITY (Lab 2, 10)
      - From web-server-1, curl http://10.0.3.10:3306 directly -> should be BLOCKED
        (database NSG only allows from backend subnet, not frontend)
      - From web-server-1, curl https://security.ubuntu.com -> routes through firewall,
        ALLOWED (matches application rule from Lab 10)
      - From web-server-1, curl https://random-untrusted-site.com -> BLOCKED by firewall
        default-deny

[ ] 3. DNS (Lab 8)
      - nslookup web-server-1.corp.internal  -> resolves via auto-registered Private DNS Zone
      - nslookup <storage-account>.blob.core.windows.net -> resolves to PRIVATE IP
        (Private Endpoint + DNS zone group from Lab 3/9)
      - From simulated on-prem VM, nslookup web-server-1.corp.internal -> resolves via
        DNS Private Resolver inbound endpoint, tunneled over VPN

[ ] 4. PRIVATE LINK (Lab 9)
      - From a VM in DR VNet, curl http://<consumer-pe-private-ip>:8080 -> reaches the
        production internal API through the Private Link Service, fully isolated

[ ] 5. LOAD BALANCING + WAF (Lab 6, 11)
      - curl http://<production-lb-public-ip> -> distributes across web-server-1/2
      - curl http://<appgw-public-ip>/api/orders -> path-routed to API backend pool
      - curl "http://<appgw-public-ip>/?id=1' OR '1'='1" -> BLOCKED by WAF (403)

[ ] 6. MONITORING (Lab 12)
      - Check Log Analytics workspace for NSG flow log entries
      - Check Firewall logs for the allow/deny decisions from test #2
      - Check the alert rule status for the VPN tunnel

If every box checks out, you have personally built, secured, connected, and
monitored a complete enterprise-grade Azure network from first principles.
FULL_E2E_CHECKLIST

print_success "CAPSTONE COMPLETE!"
```

---

## What You've Mastered (Full Curriculum Recap)

```
✅ Lab 0  - Azure CLI setup, environment, resource groups
✅ Lab 1  - VNets, Subnets, CIDR planning, NICs, Public/Private IPs
✅ Lab 2  - NSGs (stateful rules), ASGs, defense-in-depth, multi-tier security
✅ Lab 3  - Service Endpoints vs Private Endpoints (intro), Private DNS basics
✅ Lab 4  - Route Tables, UDRs, longest-prefix-match routing logic
✅ Lab 5  - VNet Peering, bidirectional config, gateway transit flags
✅ Lab 6  - Azure Load Balancer (Standard SKU), health probes, LB rules
✅ Lab 7  - VPN Gateway DEEP DIVE: Site-to-Site with simulated on-prem,
            Point-to-Site with certificate auth, BGP fundamentals, gateway SKUs
✅ Lab 8  - DNS DEEP DIVE: Public DNS Zones, Private DNS Zones (auto-reg vs manual),
            Azure DNS Private Resolver (inbound/outbound endpoints), conditional
            forwarding, split-horizon DNS, full hybrid resolution testing
✅ Lab 9  - PRIVATE LINK DEEP DIVE: Private Link Service (publishing your OWN app),
            NAT isolation, connection approval workflows, full PaaS Private Endpoint
            DNS chain (Key Vault worked example), public access lockdown + proof
✅ Lab 10 - Azure Firewall (App/Network/NAT rules, Firewall Policy), DDoS Protection Standard
✅ Lab 11 - Application Gateway v2 + WAF, path-based routing, OWASP rule blocking
✅ Lab 12 - Log Analytics, NSG Flow Logs v2 + Traffic Analytics, diagnostics settings,
            full Network Watcher toolkit (effective rules/routes, IP flow verify,
            connection troubleshoot, packet capture), alerting
✅ Lab 13 - Capstone: real hub-and-spoke architecture, end-to-end verification

YOU NOW HAVE PRO-LEVEL, INTERVIEW-READY, PRODUCTION-PATTERN KNOWLEDGE OF AZURE NETWORKING.
```

---

## QUICK COMMAND REFERENCE — NEW SERVICES (Labs 7-13)

```bash
# ===== VPN GATEWAY (Lab 7) =====
az network vnet-gateway create --resource-group $RG --name <gw> --vnet <vnet> --public-ip-addresses <ip> --gateway-type Vpn --vpn-type RouteBased --sku VpnGw1
az network local-gateway create --resource-group $RG --name <lgw> --gateway-ip-address <remote-public-ip> --local-address-prefixes <remote-cidr>
az network vpn-connection create --resource-group $RG --name <conn> --vnet-gateway1 <gw> --local-gateway2 <lgw> --shared-key <key>
az network vpn-connection show --resource-group $RG --name <conn> --query connectionStatus
az network vnet-gateway update --resource-group $RG --name <gw> --address-prefixes <p2s-pool-cidr> --client-protocol OpenVPN --root-cert-name <name> --root-cert-data <base64>

# ===== DNS (Lab 8) =====
az network dns zone create --resource-group $RG --name <public-domain>
az network private-dns zone create --resource-group $RG --name <zone>
az network private-dns link vnet create --resource-group $RG --zone-name <zone> --name <link> --virtual-network <vnet> --registration-enabled true|false
az dns-resolver create --resource-group $RG --name <resolver> --id-vnet <vnet-id>
az dns-resolver inbound-endpoint create --resource-group $RG --dns-resolver-name <resolver> --name <name> --ip-configurations <json>
az dns-resolver outbound-endpoint create --resource-group $RG --dns-resolver-name <resolver> --name <name> --subnet <subnet-id>
az dns-resolver forwarding-rule create --resource-group $RG --dns-forwarding-ruleset-name <ruleset> --name <rule> --domain-name <domain> --target-dns-servers <json>

# ===== PRIVATE LINK SERVICE (Lab 9) =====
az network private-link-service create --resource-group $RG --name <pls> --vnet-name <vnet> --subnet <subnet> --lb-name <lb> --lb-frontend-ip-configs <frontend>
az network private-endpoint create --resource-group $RG --name <pe> --vnet-name <vnet> --subnet <subnet> --private-connection-resource-id <pls-id> --connection-name <name>
az network private-endpoint-connection approve --resource-group $RG --service-name <pls> --name <conn> --type Microsoft.Network/privateLinkServices
az network private-endpoint dns-zone-group create --resource-group $RG --endpoint-name <pe> --name <group> --private-dns-zone <zone> --zone-name <zone-alias>

# ===== AZURE FIREWALL (Lab 10) =====
az network firewall create --resource-group $RG --name <fw> --sku AZFW_VNet --tier Standard
az network firewall policy create --resource-group $RG --name <policy>
az network firewall policy rule-collection-group create --resource-group $RG --policy-name <policy> --name <group> --priority <n>
az network firewall policy rule-collection-group collection add-filter-collection --resource-group $RG --policy-name <policy> --rule-collection-group-name <group> --name <coll> --collection-priority <n> --action Allow --rule-name <rule> --rule-type ApplicationRule|NetworkRule --target-fqdns <fqdn>

# ===== APPLICATION GATEWAY + WAF (Lab 11) =====
az network application-gateway waf-policy create --resource-group $RG --name <policy>
az network application-gateway create --resource-group $RG --name <appgw> --sku WAF_v2 --vnet-name <vnet> --subnet <subnet> --public-ip-address <ip>
az network application-gateway url-path-map create --resource-group $RG --gateway-name <appgw> --name <map> --paths <path> --address-pool <pool>

# ===== MONITORING (Lab 12) =====
az monitor log-analytics workspace create --resource-group $RG --workspace-name <ws>
az network watcher flow-log create --resource-group $RG --name <name> --nsg <nsg-id> --storage-account <storage-id> --workspace <ws-id> --enabled true
az monitor diagnostic-settings create --name <name> --resource <resource-id> --workspace <ws-id> --logs <json>
az network nic list-effective-nsg --resource-group $RG --name <nic>
az network nic show-effective-route-table --resource-group $RG --name <nic>
az network watcher test-ip-flow --resource-group $RG --vm <vm> --direction Outbound --protocol TCP --local <ip:port> --remote <ip:port>
az network watcher test-connectivity --resource-group $RG --source-resource <vm> --dest-resource <vm> --dest-port <port> --protocol Tcp
```

---

## FINAL SUMMARY: Complete Pro-Level Curriculum

```
✅ Lab 0  - Setup & Prerequisites
✅ Lab 1  - VNets, Subnets, NICs, IP planning
✅ Lab 2  - NSGs, ASGs, defense-in-depth security
✅ Lab 3  - Service Endpoints & Private Endpoints (intro)
✅ Lab 4  - Route Tables, UDRs
✅ Lab 5  - VNet Peering
✅ Lab 6  - Azure Load Balancer
✅ Lab 7  - VPN Gateway DEEP DIVE (S2S + P2S + on-prem simulation + BGP)
✅ Lab 8  - DNS DEEP DIVE (Public/Private zones + DNS Private Resolver + hybrid resolution)
✅ Lab 9  - Private Link DEEP DIVE (Private Link Service + full PaaS Private Endpoint pattern)
✅ Lab 10 - Azure Firewall & DDoS Protection
✅ Lab 11 - Application Gateway with WAF
✅ Lab 12 - Monitoring, Flow Logs, Network Watcher
✅ Lab 13 - Capstone: real hub-and-spoke, end-to-end verified

Nothing skipped. Every "roadmap" placeholder is now a real, runnable lab.
```

---

```bash
# VPN Gateways, Application Gateway, Azure Firewall, and Bastion are NOT cheap
# when left running. When you're done practicing, delete everything in one shot:

print_warning "This deletes EVERYTHING in the lab resource group. Confirm before running."

az group delete \
  --name $RG \
  --yes \
  --no-wait

print_info "Deletion started in background - this can take 20-30 minutes for gateways/firewall"
print_info "Verify with: az group exists --name $RG"
```
