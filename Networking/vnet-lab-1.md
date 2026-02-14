Here is your **clean, professional `README.md` format** — properly structured, minimal, and ready to paste directly into GitHub.

---

# 🎯 Azure VNet Master Lab (Cost-Optimized)

> Real-world Azure networking using smallest SKUs and minimal cost.

---

## 🛑 Cost Safety Rules

* ✅ Use `Standard_B1s` VM size only
* ✅ Avoid Firewall Premium
* ✅ Avoid ExpressRoute
* ✅ Avoid WAF_v2
* ✅ Stop VMs when not in use
* ✅ Delete Resource Group after lab

Check usage anytime:

```bash
az consumption usage list --top 5
```

---

## 🧠 Architecture Overview

![Hub-Spoke Architecture](https://learn.microsoft.com/en-us/azure/architecture/networking/guide/images/private-link-hub-spoke-network-basic-hub-spoke-diagram.svg)

![Private Endpoint](https://learn.microsoft.com/en-us/azure/private-link/media/private-endpoint-overview/private-endpoint-diagram.png)

![UDR Example](https://learn.microsoft.com/en-us/azure/virtual-network/media/virtual-networks-udr-overview/routing-example.png)

---

# 🟢 LAB 1 – Create Resource Group

```bash
az group create \
  --name rg-vnet-lab \
  --location centralindia
```

---

# 🟢 LAB 2 – Create VNet & Subnets

Create VNet + Public Subnet:

```bash
az network vnet create \
  --resource-group rg-vnet-lab \
  --name vnet-lab \
  --address-prefix 10.10.0.0/16 \
  --subnet-name public-subnet \
  --subnet-prefix 10.10.1.0/24
```

Create Private Subnet:

```bash
az network vnet subnet create \
  --resource-group rg-vnet-lab \
  --vnet-name vnet-lab \
  --name private-subnet \
  --address-prefix 10.10.2.0/24
```

---

# 🟢 LAB 3 – Deploy Public VM

```bash
az vm create \
  --resource-group rg-vnet-lab \
  --name vm-public \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --vnet-name vnet-lab \
  --subnet public-subnet \
  --admin-username azureuser \
  --generate-ssh-keys
```

Check public IP:

```bash
az vm list-ip-addresses -g rg-vnet-lab -n vm-public
```

---

# 🟢 LAB 4 – Deploy Private VM

```bash
az vm create \
  --resource-group rg-vnet-lab \
  --name vm-private \
  --image Ubuntu2204 \
  --size Standard_B1s \
  --vnet-name vnet-lab \
  --subnet private-subnet \
  --public-ip-address "" \
  --admin-username azureuser \
  --generate-ssh-keys
```

---

# 🟢 LAB 5 – Network Security Group (NSG)

Create NSG:

```bash
az network nsg create \
  --resource-group rg-vnet-lab \
  --name nsg-public
```

Allow SSH:

```bash
az network nsg rule create \
  --resource-group rg-vnet-lab \
  --nsg-name nsg-public \
  --name allow-ssh \
  --priority 100 \
  --direction Inbound \
  --access Allow \
  --protocol Tcp \
  --destination-port-range 22
```

Attach NSG to subnet:

```bash
az network vnet subnet update \
  --resource-group rg-vnet-lab \
  --vnet-name vnet-lab \
  --name public-subnet \
  --network-security-group nsg-public
```

---

# 🟢 LAB 6 – Route Table (UDR)

Create Route Table:

```bash
az network route-table create \
  --resource-group rg-vnet-lab \
  --name rt-private
```

Block Internet Access:

```bash
az network route-table route create \
  --resource-group rg-vnet-lab \
  --route-table-name rt-private \
  --name block-internet \
  --address-prefix 0.0.0.0/0 \
  --next-hop-type None
```

Attach to private subnet:

```bash
az network vnet subnet update \
  --resource-group rg-vnet-lab \
  --vnet-name vnet-lab \
  --name private-subnet \
  --route-table rt-private
```

---

# 🟢 LAB 7 – Service Endpoint (Storage)

Enable service endpoint:

```bash
az network vnet subnet update \
  --resource-group rg-vnet-lab \
  --vnet-name vnet-lab \
  --name private-subnet \
  --service-endpoints Microsoft.Storage
```

Create storage account:

```bash
az storage account create \
  --name shivalabstorage123 \
  --resource-group rg-vnet-lab \
  --location centralindia \
  --sku Standard_LRS
```

Restrict network access from portal to selected networks only.

---

# 🟢 LAB 8 – Private Endpoint

```bash
az network private-endpoint create \
  --resource-group rg-vnet-lab \
  --name pe-storage \
  --vnet-name vnet-lab \
  --subnet private-subnet \
  --private-connection-resource-id $(az storage account show --name shivalabstorage123 --query id -o tsv) \
  --group-id blob \
  --connection-name storage-connection
```

---

# 🟢 LAB 9 – VNet Peering

Create second VNet:

```bash
az network vnet create \
  --resource-group rg-vnet-lab \
  --name vnet-second \
  --address-prefix 10.20.0.0/16 \
  --subnet-name subnet1 \
  --subnet-prefix 10.20.1.0/24
```

Create peering on both VNets.

---

# 🟢 LAB 10 – NAT Gateway (Optional)

Create Public IP:

```bash
az network public-ip create \
  --resource-group rg-vnet-lab \
  --name nat-pip \
  --sku Standard
```

Create NAT Gateway:

```bash
az network nat gateway create \
  --resource-group rg-vnet-lab \
  --name nat-gw \
  --public-ip-addresses nat-pip \
  --idle-timeout 4
```

Attach NAT Gateway to private subnet.

---

# 🟢 LAB 11 – Network Watcher

Enable Network Watcher:

```bash
az network watcher configure \
  --locations centralindia \
  --enabled true
```

Test connectivity:

```bash
az network watcher test-connectivity \
  --source-resource vm-public \
  --dest-address 8.8.8.8 \
  --dest-port 80 \
  --resource-group rg-vnet-lab
```

---

# 🛑 CLEANUP (VERY IMPORTANT)

```bash
az group delete --name rg-vnet-lab --yes --no-wait
```

---

# 🎯 Concepts Covered

* Public vs Private Subnets
* NSG vs Route Table
* Service Endpoints
* Private Endpoints
* VNet Peering
* NAT Gateway
* Internet Blocking
* Network Diagnostics

---

🚀 You just built a cost-optimized, production-style Azure networking lab.
