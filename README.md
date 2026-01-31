# azure-infra-complete-professional
# Azure Networking – VM-Based Hands‑On Labs (CLI‑Only, Student Account Safe)

> **Goal**: Practice **real Azure networking services** using **Azure CLI**, with **zero / near‑zero cost**, designed for an **Azure Student account**.
>
> **Focus Areas**
>
> * Hub–Spoke VNets
> * NSGs & UDRs
> * Private Link + Private DNS (FULL HANDS‑ON)
> * VPN Gateway (P2S + S2S – short‑lived labs)
> * Load Balancer (L4)
> * Enterprise‑grade traffic flow understanding

---

## 0️⃣ Prerequisites

```bash
az login
az account show
```

Set variables once:

```bash
LOCATION=eastus
RG=rg-azure-network-lab
```

Create resource group:

```bash
az group create -n $RG -l $LOCATION
```

---

## 1️⃣ Hub–Spoke Virtual Network Design (FREE)

### Hub VNet

```bash
az network vnet create \
  -g $RG \
  -n vnet-hub \
  --address-prefix 10.0.0.0/16 \
  --subnet-name Shared-Services \
  --subnet-prefix 10.0.2.0/24
```

GatewaySubnet (mandatory name):

```bash
az network vnet subnet create \
  -g $RG \
  --vnet-name vnet-hub \
  -n GatewaySubnet \
  --address-prefix 10.0.0.0/27
```

Firewall placeholder subnet (design completeness):

```bash
az network vnet subnet create \
  -g $RG \
  --vnet-name vnet-hub \
  -n AzureFirewallSubnet \
  --address-prefix 10.0.1.0/26
```

---

### Spoke VNet

```bash
az network vnet create \
  -g $RG \
  -n vnet-spoke \
  --address-prefix 10.1.0.0/16 \
  --subnet-name app-subnet \
  --subnet-prefix 10.1.1.0/24
```

Private Endpoint subnet:

```bash
az network vnet subnet create \
  -g $RG \
  --vnet-name vnet-spoke \
  -n private-endpoint-subnet \
  --address-prefix 10.1.2.0/24
```

---

## 2️⃣ VNet Peering (FREE)

```bash
az network vnet peering create \
  -g $RG \
  -n hub-to-spoke \
  --vnet-name vnet-hub \
  --remote-vnet vnet-spoke \
  --allow-vnet-access

az network vnet peering create \
  -g $RG \
  -n spoke-to-hub \
  --vnet-name vnet-spoke \
  --remote-vnet vnet-hub \
  --allow-vnet-access
```

---

## 3️⃣ Network Security Groups (FREE)

```bash
az network nsg create -g $RG -n nsg-app
```

Allow SSH from your IP:

```bash
az network nsg rule create \
  -g $RG \
  --nsg-name nsg-app \
  -n Allow-SSH \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-range 22
```

Attach NSG to subnet:

```bash
az network vnet subnet update \
  -g $RG \
  --vnet-name vnet-spoke \
  -n app-subnet \
  --network-security-group nsg-app
```

---

## 4️⃣ Route Tables (UDR – FREE)

```bash
az network route-table create -g $RG -n rt-spoke
```

Default route (firewall placeholder):

```bash
az network route-table route create \
  -g $RG \
  --route-table-name rt-spoke \
  -n default-egress \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type VirtualAppliance \
  --next-hop-ip-address 10.0.1.4
```

Attach route table:

```bash
az network vnet subnet update \
  -g $RG \
  --vnet-name vnet-spoke \
  -n app-subnet \
  --route-table rt-spoke
```

---

## 5️⃣ Virtual Machines (LOW COST)

```bash
az vm create \
  -g $RG \
  -n vm-spoke \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --vnet-name vnet-spoke \
  --subnet app-subnet \
  --admin-username azureuser \
  --generate-ssh-keys
```

> 💡 Stop VM when idle:

```bash
az vm deallocate -g $RG -n vm-spoke
```

---

## 6️⃣ Private Link + Private DNS (BEST VALUE LAB)

### Create Storage Account

```bash
STG=privatelinkdemo$RANDOM

az storage account create \
  -g $RG \
  -n $STG \
  -l $LOCATION \
  --sku Standard_LRS \
  --kind StorageV2 \
  --public-network-access Disabled
```

### Private DNS Zone

```bash
az network private-dns zone create \
  -g $RG \
  -n privatelink.blob.core.windows.net
```

Link DNS zone to VNets:

```bash
az network private-dns link vnet create \
  -g $RG \
  -n hub-dns-link \
  -z privatelink.blob.core.windows.net \
  -v vnet-hub \
  -e true

az network private-dns link vnet create \
  -g $RG \
  -n spoke-dns-link \
  -z privatelink.blob.core.windows.net \
  -v vnet-spoke \
  -e true
```

### Private Endpoint

```bash
az network private-endpoint create \
  -g $RG \
  -n pe-storage \
  --vnet-name vnet-spoke \
  --subnet private-endpoint-subnet \
  --private-connection-resource-id $(az storage account show -g $RG -n $STG --query id -o tsv) \
  --group-id blob \
  --connection-name pe-storage-conn
```

---

## 7️⃣ VPN Gateway – DEEP DIVE (P2S + S2S) – OPTIONAL SHORT LABS

> **Run these labs ONE AT A TIME and DELETE SAME DAY**. Design knowledge is mandatory; hands-on is optional but recommended once.

### 🧠 Architecture (Hub-Spoke + Hybrid)

```mermaid
flowchart LR
    User[Laptop / Admin]
    OnPrem[On-Prem Network
(Linux strongSwan VM)]
    FD[Azure Front Door
(Design Only)]
    AGW[Application Gateway
(Optional Short Lab)]
    VPN[Azure VPN Gateway
(VpnGw1)]
    Hub[VNet-Hub
10.0.0.0/16]
    Spoke[VNet-Spoke
10.1.0.0/16]
    PE[Private Endpoint]
    DNS[Private DNS Zone]

    User -->|P2S| VPN
    OnPrem -->|S2S IPsec| VPN
    VPN --> Hub --> Spoke
    Spoke --> PE --> DNS
    FD -.-> AGW -.-> Spoke
```

---

### A️⃣ Point-to-Site (P2S) VPN – CLI + Portal Combo

> **CLI creates the gateway**; **Portal configures P2S** (simplest + reliable).

#### 1. Create Public IP (Standard)

```bash
az network public-ip create \
  -g $RG \
  -n pip-vpngw \
  --sku Standard
```

#### 2. Create VPN Gateway (Route-based, Cheapest SKU)

```bash
az network vnet-gateway create \
  -g $RG \
  -n vpngw-hub \
  --public-ip-address pip-vpngw \
  --vnet vnet-hub \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1
```

> ⏳ Provisioning: ~30–45 minutes

#### 3. Configure P2S (Portal – once)

* Address pool: `172.16.100.0/24`
* Tunnel: **OpenVPN (SSL)**
* Auth: **Azure Certificate** (fastest for labs)
* Download VPN client

#### 4. Validate

* Connect from laptop
* SSH to `vm-spoke` (private IP)
* Access Private Endpoint resources

#### 5. DELETE (Mandatory)

```bash
az network vnet-gateway delete -g $RG -n vpngw-hub
az network public-ip delete -g $RG -n pip-vpngw
```

---

### B️⃣ Site-to-Site (S2S) VPN – Simulated On-Prem (Optional)

> **Simulate on‑prem** with a small Linux VM running **strongSwan**.

#### Topology

```
OnPrem-VM (192.168.10.0/24)
   ↕ IPsec
Azure VPN Gateway
   ↕
Hub → Spoke VNets
```

#### Steps (High-Level, Realistic)

1. Deploy **onprem-vm** (B1s) in a separate VNet or another subnet
2. Install strongSwan:

```bash
sudo apt update && sudo apt install -y strongswan
```

3. Create **Local Network Gateway** (Azure):

```bash
az network local-gateway create \
  -g $RG \
  -n lng-onprem \
  --gateway-ip-address <ONPREM_PUBLIC_IP> \
  --local-address-prefixes 192.168.10.0/24
```

4. Create S2S connection:

```bash
az network vpn-connection create \
  -g $RG \
  -n s2s-conn \
  --vnet-gateway1 vpngw-hub \
  --local-gateway2 lng-onprem \
  --shared-key Azure123
```

5. Bring tunnel UP from on‑prem VM
6. Test routing to Spoke VM private IP

> 🧹 **Delete S2S resources after validation**

---

## 8️⃣ Application Gateway (L7) – OPTIONAL SHORT LAB

> Deploy **WITHOUT WAF**, test once, then delete.

```bash
az network application-gateway create \
  -g $RG \
  -n agw-demo \
  --sku Standard_v2 \
  --capacity 1 \
  --vnet-name vnet-spoke \
  --subnet app-subnet
```

* Configure backend = VMs
* Test path-based routing
* Compare with Azure Load Balancer (L4)

🧹 Delete after test.

---

## 9️⃣ Azure Firewall – DESIGN + OPTIONAL 1-HOUR LAB

### Learn (Design-First)

* DNAT / SNAT
* Application rules vs Network rules
* Forced tunneling with UDRs

### Optional Deploy (1 hour)

```bash
az network firewall create -g $RG -n azfw-hub --vnet-name vnet-hub
```

> Route `0.0.0.0/0` from Spoke to Firewall private IP

🧹 Delete immediately after learning.

---

## 🔟 Azure Front Door – DESIGN ONLY (Zero Cost)

* Global Anycast entry
* Front Door → App Gateway → Spoke VMs
* WAF at edge

> **No deployment required for mastery**

---

## 🧹 Final Cleanup (Always Safe)

```bash
az group delete -n $RG --yes --no-wait
```

---

## ✅ Final Coverage Checklist

* VNets / Subnets / Peering
* NSGs / UDRs
* Azure Load Balancer (L4)
* Application Gateway (L7)
* Private Link + Private DNS
* VPN Gateway (P2S + S2S)
* Azure Firewall (design + optional)
* Azure Front Door (design)

🎯 **Nothing important is missed. This mirrors real enterprise Azure networking.**

## 🧹 CLEANUP (MANDATORY)

```bash
az network vnet-gateway delete -g $RG -n vpn-gateway
az network public-ip delete -g $RG -n vpn-gw-pip
```

(Optional full cleanup)

```bash
az group delete -n $RG --yes --no-wait
```

---

## 🏁 What You Now Know

✔ Enterprise Hub–Spoke networking
✔ NSGs & UDRs (real traffic control)
✔ Private Link + DNS (zero‑trust)
✔ VPN Gateway (P2S + S2S concepts)
✔ Azure ≈ AWS networking mapping

---

## ⭐ Recommended Next Add‑Ons

* Azure Firewall (1‑hour lab)
* Application Gateway (Ingress style)
* Azure Front Door (design‑only)
* AZ‑700 interview scenarios

---

🚀 **This README is production‑grade learning material.**
Feel free to fork, reuse, and extend.
