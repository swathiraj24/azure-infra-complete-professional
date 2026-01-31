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

## 7️⃣ VPN Gateway – OPTIONAL SHORT LAB (DELETE SAME DAY)

### Public IP

```bash
az network public-ip create \
  -g $RG \
  -n vpn-gw-pip \
  --sku Standard
```

### VPN Gateway

```bash
az network vnet-gateway create \
  -g $RG \
  -n vpn-gateway \
  --public-ip-address vpn-gw-pip \
  --vnet vnet-hub \
  --gateway-type Vpn \
  --vpn-type RouteBased \
  --sku VpnGw1
```

> Configure **Point‑to‑Site** in Portal (Azure Cert or Entra ID)

---

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

