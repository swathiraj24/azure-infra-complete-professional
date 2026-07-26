

## 📚 Table of Contents

- [Overview](#overview)
- [Virtualization Background](#virtualization-background)
- [Azure VM Architecture](#azure-vm-architecture)
- [Choosing the Right VM](#choosing-the-right-vm)
- [Windows VM Deployment Lab](#windows-vm-deployment-lab)
- [Explain Every Deployment Option](#explain-every-deployment-option)
- [Availability and Resilience](#availability-and-resilience)
- [Networking in Azure VM](#networking-in-azure-vm)
- [Storage and Disks](#storage-and-disks)
- [Security Best Practices](#security-best-practices)
- [Monitoring and Operations](#monitoring-and-operations)
- [Cost Optimization for Student Subscription](#cost-optimization-for-student-subscription)
- [Interview Questions](#interview-questions)
- [Troubleshooting](#troubleshooting)
- [Hands-On Summary](#hands-on-summary)
- [Cleanup Section](#cleanup-section)

---

## Overview

Azure Virtual Machines (VMs) are on-demand, scalable computing resources provided by Microsoft Azure. They are the foundation of Infrastructure as a Service (IaaS) and represent the closest cloud equivalent to traditional physical or virtual servers in on-premises data centers.

### What are Azure Virtual Machines?

Azure VMs provide the ability to run operating systems and applications in the cloud without the need to purchase, maintain, or manage physical hardware. Each VM includes:

- A virtualized operating system (Windows or Linux)
- Allocated CPU cores, RAM, and storage
- Network connectivity
- Security configurations

### Why are VMs Used in Cloud Computing?

1. **Flexibility**: Run any application, legacy or modern, without rewriting code
2. **Scalability**: Scale up (more resources) or scale out (more instances) based on demand
3. **Cost Efficiency**: Pay-as-you-go model eliminates capital expenditure
4. **Geographic Reach**: Deploy globally across Azure's worldwide data center network
5. **Disaster Recovery**: Built-in backup and replication capabilities
6. **Development & Testing**: Create temporary environments without hardware constraints

### How Azure VM Differs from Physical Servers

| Aspect | Physical Servers | Azure Virtual Machines |
|--------|------------------|----------------------|
| **Hardware Management** | Requires physical maintenance | Fully managed by Microsoft |
| **Scalability** | Limited by physical capacity | Elastic, can resize in minutes |
| **Cost Structure** | High upfront CapEx | Pay-as-you-go OpEx |
| **Provisioning Time** | Weeks to months | Minutes |
| **Physical Space** | Requires data center/rack space | No physical footprint |
| **Power/Cooling** | Significant infrastructure costs | Included in pricing |
| **Hardware Failures** | Manual intervention required | Automated recovery |
| **Lifecycle** | 3-5 years depreciation | Continuous updates |

### Relationship Between Virtualization, Hypervisors, and Cloud Compute

The cloud computing model builds upon virtualization technology:

```
Physical Hardware → Hypervisor → Virtual Machines → Cloud Services
         ↓               ↓              ↓              ↓
    CPU, RAM, Disk   Abstraction   Isolated OS   IaaS, PaaS, SaaS
```

### Where Azure VM Fits in IaaS

Azure Virtual Machines is the core IaaS offering. In the cloud computing stack:

- **IaaS**: Azure VMs provide fundamental compute, storage, and networking
- **PaaS**: Azure App Services, SQL Database (built on VMs)
- **SaaS**: Office 365, Dynamics 365 (built on VMs)

### Common Real-World Use Cases

1. **Web Hosting**: Deploy websites, web applications, and APIs
2. **Development/Test Environments**: Spin up isolated environments for coding
3. **Application Migration**: Lift-and-shift on-premises applications
4. **Big Data Analytics**: Process large datasets with compute-optimized VMs
5. **Database Servers**: Run SQL Server, MySQL, PostgreSQL
6. **Active Directory**: Extend or replace on-premises directory services
7. **DevOps Pipelines**: Build agents and CI/CD infrastructure
8. **VDI (Virtual Desktop Infrastructure)**: Host virtual desktops
9. **HPC (High Performance Computing)**: Scientific and engineering workloads
10. **Containers Hosting**: Run Kubernetes clusters and containerized apps

---

## Virtualization Background

### What is Virtualization?

Virtualization is a technology that creates virtual versions of computing resources—servers, storage, networks, or operating systems—by abstracting physical hardware. It allows multiple virtual machines to run simultaneously on a single physical computer, each with its own operating system and applications.

**Key Benefits:**
- **Resource Utilization**: Maximize hardware usage by consolidating workloads
- **Isolation**: VMs are separated from each other for security and stability
- **Agility**: Create, delete, or migrate VMs quickly
- **Hardware Independence**: Abstract applications from underlying hardware
- **Cost Reduction**: Fewer physical servers, reduced power and cooling

### Hypervisor Concept

A hypervisor, also called a virtual machine monitor (VMM), is software that creates and runs virtual machines. It sits between the physical hardware and the virtual machines, managing the allocation of physical resources.

**Core Responsibilities:**
- CPU scheduling across VMs
- Memory allocation and paging
- I/O management (storage, networking)
- VM lifecycle management (create, start, stop, migrate)
- Resource isolation and security

### Type 1 vs Type 2 Hypervisors

| Aspect | Type 1 (Bare-Metal) | Type 2 (Hosted) |
|--------|-------------------|-----------------|
| **Installation** | Runs directly on hardware | Runs on host OS |
| **Performance** | Near-native performance | Performance overhead |
| **Resource Management** | Direct hardware access | Through host OS |
| **Security** | More secure | Less secure |
| **Examples** | Hyper-V, ESXi, Xen | VirtualBox, VMware Workstation |
| **Use Case** | Enterprise, Cloud | Development, Testing |

### How Azure Uses Virtualization

Azure implements a sophisticated virtualization infrastructure:

1. **Hyper-V Hypervisor**: Azure uses Microsoft Hyper-V at its core, highly customized for cloud-scale operations
2. **Host Hardware**: Azure data centers contain hundreds of thousands of physical servers (hosts)
3. **Fabric Controller**: Manages the entire virtualized environment, including VM placement and recovery
4. **Azure Resource Manager (ARM)**: The management layer that handles resource provisioning, API requests, and RBAC
5. **Guest OS Optimization**: Azure provides optimized Windows and Linux images with cloud-optimized drivers

**Azure's Virtualization Stack:**

```
┌─────────────────────────────────────────────┐
│           Azure Portal / CLI / APIs         │
├─────────────────────────────────────────────┤
│      Azure Resource Manager (ARM)           │
├─────────────────────────────────────────────┤
│         Fabric Controller                   │
│     (Orchestrates VM Placement)             │
├─────────────────────────────────────────────┤
│         Host Server                         │
│  ┌───────────┐  ┌───────────┐              │
│  │ VM-01     │  │ VM-02     │              │
│  │ Guest OS  │  │ Guest OS  │              │
│  ├───────────┤  ├───────────┤              │
│  │ Hyper-V   │  │ Hyper-V   │              │
│  │ Hypervisor │  │ Hypervisor│              │
│  ├───────────┤  ├───────────┤              │
│  │ Physical  │  │ Physical  │              │
│  │ Hardware  │  │ Hardware  │              │
│  └───────────┘  └───────────┘              │
└─────────────────────────────────────────────┘
```

### Why Virtualization is Important in Cloud Environments

1. **Multi-tenancy**: Run thousands of customer workloads on shared hardware
2. **Resource Optimization**: Dynamically allocate resources based on demand
3. **Elasticity**: Quickly provision and de-provision resources
4. **Geographic Distribution**: Deploy VMs across global data centers
5. **Disaster Recovery**: Migrate VMs between physical hosts without downtime
6. **Cost Model**: Pay only for resources consumed
7. **Automation**: Programmatically manage infrastructure at scale

### Benefits and Tradeoffs of Virtual Machines

**Benefits:**
- ✅ Cost-effective vs. physical servers
- ✅ Rapid provisioning and scaling
- ✅ Isolated environments for security
- ✅ Hardware abstraction and compatibility
- ✅ Disaster recovery and backup capabilities
- ✅ Flexible resource allocation

**Tradeoffs:**
- ⚠️ Virtualization overhead (10-15% performance impact)
- ⚠️ "Noisy neighbor" issues in multi-tenant environments
- ⚠️ Complexity in networking and security configuration
- ⚠️ License management for guest operating systems
- ⚠️ Resource contention during peak loads
- ⚠️ Management overhead for large deployments

---

## Azure VM Architecture

![Azure VM Components](images/azure-vm-components.png)

Understanding the complete Azure VM architecture is essential for designing and deploying robust solutions. Let's explore each component in detail.

### 1. Subscription

**What it is:** Azure Subscription is a logical container for billing and resource management. Every Azure resource belongs to exactly one subscription.

**Why it's used:** Subscriptions provide:
- Billing separation and tracking
- Resource limits and quotas
- Access control boundaries (RBAC)
- Billing budgets and cost management

**When to choose:** For a student account, you get a single subscription with free credits. In enterprise scenarios, multiple subscriptions may be used for different departments, environments, or purposes.

**Cost Considerations:** Subscriptions themselves are free. Resources within a subscription accrue costs.

**Interview Relevance:** 
- *"How do you organize Azure resources across subscriptions?"*
- *"What is the difference between management groups and subscriptions?"*

### 2. Resource Group

**What it is:** A Resource Group (RG) is a logical container for related Azure resources. All resources in a resource group share a common lifecycle.

**Why it's used:**
- Organize related resources (e.g., all resources for a web application)
- Apply consistent policies (tags, locks, RBAC)
- Manage lifecycle together (deploy, update, delete)
- Simplify cost tracking and monitoring

**When to choose:** Create a new resource group for each application, environment, or logical grouping. A good practice is to group resources that will be deployed, updated, and deleted together.

**Cost Considerations:** Resource groups themselves are free.

**Interview Relevance:**
- *"Can a resource be moved between resource groups?"* (Yes, with limitations)
- *"What happens to resources if you delete the resource group?"* (All resources are deleted)

### 3. Region

**What it is:** Azure Regions are geographic areas where Azure data centers are located. Each region contains multiple availability zones.

**Why it's used:**
- Reduce latency by placing resources near users
- Comply with data residency requirements (GDPR, etc.)
- Access region-specific services (some are only available in certain regions)
- Leverage availability zones for high availability

**When to choose:** Select the region closest to your users or where compliance requires data to reside. Students should often choose regions with the lowest costs (e.g., Central India, East US) or those with free tier services.

**Cost Considerations:** Pricing varies by region (due to local costs, electricity, etc.). South-Central US and East US are often more economical.

**Interview Relevance:**
- *"How do you choose the best region for your workload?"*
- *"What is the difference between a region and an availability zone?"*

### 4. Availability Zone

**What it is:** Availability Zones are physically separate data centers within an Azure region. Each zone has independent power, cooling, and networking.

**Why it's used:**
- Provide higher availability (99.99% SLA when using zones)
- Isolate workloads from zone failures
- Enable zone-redundant services

**When to choose:** For production workloads requiring high availability. For student labs, you might skip zones to save costs, but understanding them is crucial for interviews.

**Cost Considerations:** Some services incur additional costs for zone-redundant configurations. Standard VM deployment (without zone targeting) is cheaper.

**Interview Relevance:**
- *"What is the difference between availability zones and availability sets?"*
- *"Can you deploy in one zone and failover to another?"*

### 5. Availability Set

**What it is:** An Availability Set distributes VMs across multiple fault domains and update domains within a single data center.

**Why it's used:**
- Protect against hardware failures (fault domains)
- Protect during planned maintenance (update domains)
- Achieve 99.95% SLA for VMs in the same availability set

**When to choose:** Use availability sets when availability zones aren't needed or available, for legacy applications, or when you need to ensure VMs are updated sequentially.

**Cost Considerations:** Availability sets themselves are free. You pay for the VMs that are part of them.

**Interview Relevance:**
- *"What is the maximum number of fault domains in Azure?"* (3)
- *"How many update domains are there?"* (Up to 20, default 5)

### 6. Virtual Network

**What it is:** A Virtual Network (VNet) is the fundamental networking component in Azure. It's a logically isolated network in the Azure cloud.

**Why it's used:**
- Enable communication between Azure resources
- Connect to on-premises networks via VPN or ExpressRoute
- Segment and isolate workloads
- Control IP addressing and routing

**When to choose:** Every VM must be deployed in a VNet. Create separate VNets for different environments (production, test, dev) or applications that shouldn't communicate.

**Cost Considerations:** VNets themselves are free. Some costs apply for VPN Gateway or other networking services.

**Interview Relevance:**
- *"What is the address space limit for a VNet?"* (/16 or smaller)
- *"Can VNets communicate with each other?"* (Yes, via peering)

### 7. Subnet

**What it is:** A Subnet is a range of IP addresses within a VNet. It provides a way to segment and organize your VNet.

**Why it's used:**
- Organize resources by function (web tier, app tier, database tier)
- Apply different security rules to each subnet
- Control traffic flow between subnets
- Determine the range of IP addresses available for resources

**When to choose:** Create separate subnets for different tiers of your application or for different resource types. For a simple lab, one subnet is sufficient.

**Cost Considerations:** Subnets are free.

**Interview Relevance:**
- *"How many subnets can a VNet have?"* (Up to 3000)
- *"What IP addresses are reserved in each subnet?"* (First three and last one)

### 8. Network Interface Card (NIC)

**What it is:** A Network Interface Card (NIC) is a virtual network interface attached to a VM. It connects the VM to a VNet.

**Why it's used:**
- Provide network connectivity for the VM
- Assign private and public IP addresses
- Enable communication between VMs and the internet
- Multiple NICs allow different subnet connections

**When to choose:** One NIC is usually sufficient for most scenarios. Multiple NICs are used for network appliances or when VMs need multiple networks.

**Cost Considerations:** NICs themselves are free. Public IP addresses attached to NICs may incur costs.

**Interview Relevance:**
- *"Can you attach multiple NICs to an Azure VM?"* (Yes, depending on VM size)
- *"Can you add or remove a NIC after creation?"* (Yes, but primary NIC cannot be removed)

### 9. Public IP

**What it is:** A Public IP is a public IPv4 address assigned to a resource, enabling internet connectivity.

**Why it's used:**
- Access the VM from the internet (RDP/SSH)
- Expose services to the internet (web servers)
- Enable hybrid connectivity scenarios

**When to choose:** You need a Public IP to connect to your VM from outside the VNet. For internal workloads, skip the Public IP and use Azure Bastion or VPN.

**Cost Considerations:** Public IP addresses incur costs. You pay for:
- Dynamic IP: free (unless you keep it idle)
- Static IP: $0.004/hour (approximately $2.88/month)
- Use only when necessary in student labs

**Interview Relevance:**
- *"What is the difference between static and dynamic public IP addresses?"*
- *"Can a public IP be moved between VMs?"* (Yes, for static IPs)

### 10. Private IP

**What it is:** A Private IP is an IP address within the VNet address space. It's used for internal communication.

**Why it's used:**
- Communication between Azure resources
- Internal application access
- Management and monitoring

**When to choose:** Every VM must have a private IP. Dynamic allocation is fine for most cases, but static is recommended for critical services like domain controllers.

**Cost Considerations:** Private IPs are free.

**Interview Relevance:**
- *"Can a VM have multiple private IPs?"* (Yes, up to per NIC limits)
- *"Is a private IP accessible from the internet?"* (No, not without NAT)

### 11. Network Security Group (NSG)

**What it is:** A Network Security Group (NSG) is a firewall that controls inbound and outbound traffic for VMs and subnets.

**Why it's used:**
- Control network traffic based on IP addresses, ports, and protocols
- Implement security boundaries
- Allow/deny traffic at the subnet or NIC level

**When to choose:** Always use NSGs to secure your VMs. Create separate NSGs for different tiers (web, app, database) with appropriate rules.

**Cost Considerations:** NSGs are free.

**Interview Relevance:**
- *"What is the difference between an NSG and a firewall?"*
- *"How does NSG rule evaluation work?"* (Priority order, from highest to lowest)

### 12. OS Disk

**What it is:** The OS Disk is the storage containing the operating system (Windows or Linux).

**Why it's used:**
- Boot the VM
- Store OS files and system configuration
- Maintain OS state across reboots

**When to choose:** The OS Disk is required for every VM. For student VMs, Standard SSD is a good balance between cost and performance.

**Cost Considerations:** OS disks are charged based on size and tier. Windows OS disks require additional licensing costs.

**Interview Relevance:**
- *"How large is the default OS disk?"* (Usually 127 GB for Windows)
- *"Can you change OS disk size after VM creation?"* (Yes, with limitations)

### 13. Data Disk

**What it is:** Data Disks are additional storage volumes for applications and data.

**Why it's used:**
- Separate data from the OS
- Increase storage capacity
- Enable backup of data separately from the VM
- Improve performance by using faster disks

**When to choose:** Always separate application data and OS for better manageability, backup, and performance.

**Cost Considerations:** Data disks incur storage costs. Use smaller disks if you don't need much storage. Unmanaged growth increases costs.

**Interview Relevance:**
- *"What is the maximum number of data disks you can attach to a VM?"* (Depends on VM size)
- *"What is the difference between OS disk and data disk?"*

### 14. Image

**What it is:** An Image is a template containing the operating system and initial configuration.

**Why it's used:**
- Standardize VM deployments
- Reduce deployment time
- Include pre-configured applications
- Maintain consistency across environments

**When to choose:** Use platform images (Microsoft-provided) for standard deployments. Create custom images when you have specific configurations, software, or compliance requirements.

**Cost Considerations:** Platform images are free. Custom images stored in Azure Compute Gallery incur storage costs.

**Interview Relevance:**
- *"What is the difference between a marketplace image and a custom image?"*
- *"How can you create a custom image?"* (Capture from existing VM or use Azure Compute Gallery)

### 15. VM Size

**What it is:** VM Size determines the hardware resources (CPU cores, RAM, storage throughput) allocated to the VM.

**Why it's used:**
- Match workloads with appropriate resources
- Scale up or down based on requirements
- Balance cost and performance

**When to choose:** Select size based on workload requirements. For students: B1s or B1ms for light workloads, or B2s for more demanding applications.

**Cost Considerations:** VMs are billed by size and uptime. Smaller sizes cost less per hour. Stopping/deallocating the VM stops billing.

**Interview Relevance:**
- *"What is the difference between D-series and B-series VMs?"* (B-series is burstable, D-series is standard)
- *"Can you change VM size after creation?"* (Yes, restart required)

### 16. Extensions

**What it is:** Extensions are small applications that run on the VM to perform various tasks during or after deployment.

**Why it's used:**
- Install software and configurations
- Run scripts
- Enable monitoring and management
- Apply security updates
- Join domains or Active Directory

**When to choose:** Use extensions when you need to customize the VM after creation. Common extensions include Custom Script Extension, Azure Monitor Agent, and Antimalware.

**Cost Considerations:** Extensions themselves are free. Software installed by extensions may have licensing costs.

**Interview Relevance:**
- *"What is the Custom Script Extension used for?"*
- *"How do you configure Azure Bastion using extensions?"*

### 17. Managed Identity

**What it is:** Managed Identity is an Azure Active Directory identity that can be assigned to a VM for authenticating to Azure services.

**Why it's used:**
- Securely access Azure resources without storing credentials
- Simplify authentication to services like Key Vault, Storage, SQL
- Improve security by eliminating hardcoded credentials

**When to choose:** Always use managed identities when your VM needs to access other Azure resources securely.

**Cost Considerations:** Managed identities are free.

**Interview Relevance:**
- *"What is the difference between system-assigned and user-assigned managed identities?"*
- *"How does managed identity work with Key Vault?"*

### 18. Boot Diagnostics

**What it is:** Boot Diagnostics is a feature that captures serial console logs and screenshots of the VM boot process.

**Why it's used:**
- Troubleshoot boot issues
- Debug kernel panics and system crashes
- Monitor VM boot health

**When to choose:** Always enable boot diagnostics for production VMs. For student labs, it's free and useful for learning.

**Cost Considerations:** Boot diagnostics is free up to 5 GB of storage.

**Interview Relevance:**
- *"How do you troubleshoot a VM that won't boot?"*
- *"What information does boot diagnostics provide?"*

### 19. Azure Monitor

**What it is:** Azure Monitor is a comprehensive service for collecting, analyzing, and responding to monitoring data from Azure resources.

**Why it's used:**
- Monitor VM performance metrics (CPU, memory, disk, network)
- Collect and analyze logs
- Set up alerts for critical conditions
- Create dashboards and visualizations

**When to choose:** Use Azure Monitor for all production VMs. For student labs, it's useful for learning and detecting issues.

**Cost Considerations:** Azure Monitor has free tiers for basic metrics and logs. Advanced features incur costs.

**Interview Relevance:**
- *"What is the difference between Azure Monitor and Application Insights?"*
- *"How do you set up alerts for high CPU usage?"*

### 20. Backup

**What it is:** Azure Backup provides simple, secure, and cost-effective data protection for VMs.

**Why it's used:**
- Protect against data loss or corruption
- Enable point-in-time recovery
- Comply with data retention requirements

**When to choose:** Always implement backup for production VMs. For student labs, you may skip backup due to cost, but understand the concept.

**Cost Considerations:** Backup is charged based on:
- Data protection (per VM size)
- Storage (backup data stored in Azure)
- Student accounts may have limited backup capabilities

**Interview Relevance:**
- *"What is the difference between Azure Backup and Azure Site Recovery?"*
- *"How do you configure backup policy?"*

### 21. Azure Bastion

**What it is:** Azure Bastion is a fully-managed PaaS service that provides secure and seamless RDP/SSH access to VMs directly from the Azure portal.

**Why it's used:**
- Securely access VMs without public IP addresses
- Eliminate exposure of RDP/SSH ports to the internet
- Provide single-click access from the Azure portal

**When to choose:** Use Bastion when you need secure remote access. It's a best practice for production and also useful for students who want to avoid Public IP costs.

**Cost Considerations:** Bastion is billed per hour ($0.19/hour for standard tier). For students, using a jumpbox VM might be cheaper.

**Interview Relevance:**
- *"How does Azure Bastion improve security?"*
- *"What is the difference between Bastion and a jumpbox?"*

### 22. Load Balancer

**What it is:** Azure Load Balancer distributes incoming traffic across multiple VMs to ensure high availability and scalability.

**Why it's used:**
- Distribute traffic across multiple VMs
- Achieve high availability
- Scale applications horizontally
- Provide health probing

**When to choose:** Use load balancers when you have multiple VMs serving the same application. For a single VM lab, you can skip this.

**Cost Considerations:** Load Balancer is free for basic tier. Standard tier incurs costs per rule and data processed.

**Interview Relevance:**
- *"What is the difference between Layer 4 and Layer 7 load balancers?"*
- *"What is Azure Application Gateway vs. Load Balancer?"*

### 23. Dedicated Host

**What it is:** Dedicated Host is a physical server allocated to a single customer. All VMs on the host run only for that customer.

**Why it's used:**
- License compliance (e.g., Windows Server with Software Assurance)
- Physical isolation requirements
- Regulatory compliance
- Resource assurance and performance predictability

**When to choose:** Only when you have specific compliance, licensing, or isolation requirements. Typically enterprise workloads, not student labs.

**Cost Considerations:** Dedicated Hosts are expensive ($3,000+/month). Not recommended for student accounts.

**Interview Relevance:**
- *"When would you use a Dedicated Host over shared VMs?"*
- *"What is the benefit of Dedicated Host for licensing?"*

---

## Choosing the Right VM

### When to Use Windows VM

Choose Windows VM when you need:
- Windows Server operating system
- .NET framework applications
- SQL Server or other Microsoft databases
- Windows desktop applications
- Active Directory integration
- Microsoft-specific technologies
- Legacy Windows applications

### When to Use Linux VM

Choose Linux VM when you need:
- Open-source operating systems
- LAMP/LEMP stacks
- Containerization (Docker, Kubernetes)
- Programming languages (Python, Java, Ruby, Go)
- Developer tools and workflows
- Cost optimization (Linux is generally cheaper)

### VM Size Categories

| Category | Description | Use Cases | Example Sizes |
|----------|-------------|-----------|---------------|
| **General Purpose** | Balanced CPU-to-memory ratio | Web servers, development, testing | B1s, B1ms, B2s, D2s_v3, D4s_v3 |
| **Compute Optimized** | Higher CPU-to-memory ratio | Batch processing, application servers | F2s_v2, F4s_v2, F8s_v2 |
| **Memory Optimized** | Higher memory-to-CPU ratio | Databases, in-memory caching | E2s_v3, E4s_v3, E8s_v3 |
| **Storage Optimized** | High disk throughput and IOPS | Big data, SQL Server, NoSQL | L4s, L8s, L16s |
| **GPU** | NVIDIA GPU accelerators | AI, ML, graphics rendering | NC4as_T4_v3, NV4as_v4 |
| **Burstable (B-series)** | Base performance with bursting | Development, web servers with variable load | B1s, B1ms, B2s |
| **HPC** | High-performance computing | Scientific simulations, financial modeling | HB120rs_v2, HC44rs |

### How to Select VM Size

Consider these factors:

1. **Workload Requirements**: What is the application's CPU, memory, and disk usage pattern?
2. **Performance Needs**: What are the IOPS (disk operations) and network bandwidth requirements?
3. **Cost Budget**: How much can you spend per month?
4. **Growth Plans**: Will the workload grow over time?
5. **Availability Needs**: Does the application require high availability?

**For Students:**

| Workload Type | Recommended VM Size | Hourly Cost (approx) |
|---------------|-------------------|---------------------|
| **Basic learning & testing** | B1s (1 vCPU, 1 GB RAM) | $0.012 |
| **Windows desktop apps** | B1ms (1 vCPU, 2 GB RAM) | $0.016 |
| **Web server (small)** | B2s (2 vCPU, 4 GB RAM) | $0.048 |
| **Development environment** | D2s_v3 (2 vCPU, 8 GB RAM) | $0.096 |
| **Database server (small)** | D4s_v3 (4 vCPU, 16 GB RAM) | $0.192 |

### How to Avoid Expensive Sizes

1. **Use B-series** instead of D-series for variable workloads
2. **Use spot instances** for non-production workloads (up to 90% discount)
3. **Right-size**: Monitor actual utilization and reduce size if over-provisioned
4. **Stop/deallocate** when not in use (billing stops)
5. **Use reserved instances** for production workloads (up to 72% discount for 3-year commitments)
6. **Leverage free tier** (Linux VMs for 12 months for Azure students)
7. **Choose budget-friendly regions** (South Central US, East US, Central India)

### Cost-Saving Comparison

| Scenario | Strategy | Monthly Cost (approx) |
|----------|----------|----------------------|
| **Always on** | D2s_v3 24/7 | $70 |
| **Auto-shutdown nights** | D2s_v3 (12 hours/day) | $35 |
| **B-series** | B2s (16 hours/day) | $24 |
| **Student free tier** | B1s Linux (12 months) | $0 |

---

## Windows VM Deployment Lab

### Prerequisites

Before starting this lab, ensure you have:

1. **Azure Student Account**: Sign up at [azure.microsoft.com/free/students](https://azure.microsoft.com/free/students/)
2. **Azure Subscription**: Active student subscription with credits
3. **Azure Portal Access**: [portal.azure.com](https://portal.azure.com)
4. **Basic Knowledge**: Understanding of cloud concepts
5. **RDP Client**: Windows Remote Desktop or an alternative client
6. **$100-$200 Credit**: Azure student credit should be available

### Step 1: Login to Azure Portal

1. Go to [portal.azure.com](https://portal.azure.com)
2. Sign in with your student credentials (Microsoft account)
3. If you don't have an account, create one using your student email

![Azure Portal Login](images/azure-portal-login.png)

### Step 2: Create Resource Group

1. In the Azure portal, search for "Resource groups" in the search bar
2. Click **+ Create**
3. Fill in the details:
   - **Subscription**: Select your student subscription
   - **Resource group**: `RG-WindowsVM-Student`
   - **Region**: Select `East US` or your preferred region
4. Click **Review + create**
5. Click **Create**

### Step 3: Create Virtual Network and Subnet

1. In the Azure portal, search for "Virtual networks"
2. Click **+ Create**
3. Fill in the details:
   - **Subscription**: Your student subscription
   - **Resource group**: Select `RG-WindowsVM-Student`
   - **Name**: `VNet-WindowsVM`
   - **Region**: Same as resource group
4. Click **Next: IP Addresses**
   - **IPv4 address space**: `10.0.0.0/16`
   - **Subnet name**: `default`
   - **Subnet address range**: `10.0.0.0/24`
5. Click **Review + create**
6. Click **Create**

### Step 4: Create Windows VM

1. In the Azure portal, search for "Virtual machines"
2. Click **+ Create** → **Azure virtual machine**

#### Step 4.1: Basics Tab

![VM Creation Basics](images/vm-creation-basics.png)

| Setting | Value | Explanation |
|---------|-------|-------------|
| **Subscription** | Your student subscription | Billing account |
| **Resource group** | RG-WindowsVM-Student | Logical grouping |
| **Virtual machine name** | `winvm-student` | Resource name |
| **Region** | East US (or your choice) | Geographic location |
| **Availability options** | No infrastructure redundancy required | Single VM lab |
| **Security type** | Standard | Basic security |
| **Image** | Windows Server 2022 Datacenter - x64 Gen2 | Windows OS |
| **VM architecture** | x64 | Standard architecture |
| **Size** | B1s (1 vCPU, 1 GiB memory) | Smallest, cheapest |
| **Username** | `azureuser` | Admin account |
| **Password** | Choose a strong password | 12+ chars, complex |
| **Public inbound ports** | RDP (3389) | For remote access |

> **Warning:** Setting public inbound ports to "Allow RDP (3389)" exposes the VM to the internet. We will add an NSG later to secure this.

#### Step 4.2: Disks Tab

| Setting | Value | Explanation |
|---------|-------|-------------|
| **OS disk type** | Standard SSD | Good balance of cost and performance |
| **Delete with VM** | Enabled | Disk deletes when VM deletes |
| **Data disks** | Add data disk (optional) | For data separation |

Add a data disk (recommended for learning):
1. Click **Create and attach a new disk**
2. **Disk name**: `datadisk01`
3. **Storage type**: Standard SSD
4. **Size**: 32 GiB (minimum)
5. Click **OK**

#### Step 4.3: Networking Tab

| Setting | Value | Explanation |
|---------|-------|-------------|
| **Virtual network** | VNet-WindowsVM | Your VNet |
| **Subnet** | default | Your subnet |
| **Public IP** | Create new | For RDP access |
| **NIC network security group** | Basic | Default security |
| **Public inbound ports** | Allow selected ports | Set RDP port |
| **Select inbound ports** | RDP (3389) | For remote access |

#### Step 4.4: Management Tab

| Setting | Value | Explanation |
|---------|-------|-------------|
| **Boot diagnostics** | Enable with managed storage account | Troubleshoot issues |
| **Auto-shutdown** | Enable | Save costs |
| **Auto-shutdown time** | 18:00 (6 PM) | Stop at end of lab |
| **Time zone** | Your local time | |
| **Send notification** | Enabled | Email alert |

#### Step 4.5: Monitoring Tab

| Setting | Value | Explanation |
|---------|-------|-------------|
| **Enable VM insights** | Optional (leave disabled) | Advanced monitoring |

#### Step 4.6: Advanced Tab

| Setting | Value | Explanation |
|---------|-------|-------------|
| **Extensions** | None | Add later if needed |

#### Step 4.7: Tags Tab

| Setting | Value | Explanation |
|---------|-------|-------------|
| **environment** | `lab` | Environment tagging |
| **purpose** | `learning` | Purpose tagging |
| **owner** | `yourname` | Owner tagging |

#### Step 4.8: Review + Create

1. Review all settings
2. Click **Review + create**
3. Click **Create** after validation passes

![Review + Create](images/vm-review-create.png)

### Step 5: Connect Using RDP

#### Method 1: Using Azure Portal (Easy)

1. Go to the VM in the Azure portal
2. Click **Connect** → **RDP**
3. Download the RDP file
4. Open the RDP file
5. Enter your credentials (username: `azureuser`, password: your chosen password)
6. Accept certificate warnings
7. You should see the Windows Server desktop

![Connect via RDP](images/vm-rdp-connect.png)

#### Method 2: Using Windows Remote Desktop

1. In Windows, search for "Remote Desktop Connection"
2. Enter the **Public IP** address of your VM
3. Click **Connect**
4. Enter credentials
5. Click **Yes** on the certificate warning

#### Method 3: Using Azure CLI (Advanced)

```bash
# Get the public IP address
az vm show -d -g RG-WindowsVM-Student -n winvm-student --query publicIps -o tsv

# Use the IP address in your RDP client
```

### Step 6: Post-Deployment Validation

1. **Check VM Status**: In the Azure portal, verify VM is "Running"

2. **Check IP Configuration**: In the VM, open Command Prompt and run:
   ```cmd
   ipconfig
   ```
   Verify you have an IP address in the 10.0.0.0/24 range

3. **Check Internet Connectivity**:
   ```cmd
   ping 8.8.8.8
   ```
   You should receive replies (Azure VMs have outbound internet by default)

4. **Check Disk Configuration**:
   - Open **File Explorer** → **This PC**
   - You should see:
     - C: Drive (OS disk)
     - D: or E: Drive (Temporary disk)
     - F: or G: Drive (Data disk, if added)

5. **Install Basic Tools** (Optional):
   ```cmd
   # Install Chocolatey package manager
   @"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
   
   # Install common utilities using Chocolatey
   choco install notepadplusplus googlechrome 7zip sysinternals -y
   ```

6. **Create a Test File**:
   ```cmd
   echo "Azure VM is working!" > C:\test.txt
   ```

---

## Explain Every Deployment Option

### 1. Image

**What it is:** The template that defines the operating system and base configuration.

**Options:**
- **Windows Server 2022 Datacenter**: Full Windows Server with all roles
- **Windows Server 2019 Datacenter**: Slightly older, still supported
- **Windows 10/11 Pro/Enterprise**: Desktop OS (not recommended for server workloads)
- **Custom Image**: Your own customized image

**When to choose:**
- Windows Server 2022 for new workloads
- Windows Server 2019 for compatibility with older applications
- Custom images for pre-installed applications and configurations

**Cost Considerations:** Windows Server images include licensing costs. Windows Desktop images require separate licensing.

### 2. Size

**What it is:** The hardware resources (vCPUs, RAM, temporary storage) allocated to the VM.

**Options:**
- **B-series (Burstable)**: Base performance with bursting capabilities
- **D-series (General Purpose)**: Balanced CPU and memory
- **E-series (Memory Optimized)**: More RAM per vCPU
- **F-series (Compute Optimized)**: More CPU per vCPU

**When to choose:** Match workload requirements:
- B-series for development/testing
- D-series for web servers
- E-series for databases
- F-series for compute-intensive workloads

**Cost Considerations:** Larger sizes cost more per hour.

### 3. Availability Options

**What it is:** How the VM is placed within the Azure infrastructure for resilience.

**Options:**

| Option | Description | SLA | Cost Impact |
|--------|-------------|-----|-------------|
| **No infrastructure redundancy** | Single VM in one rack | 99.9% | Base cost |
| **Availability set** | Distributed across fault/update domains | 99.95% | No extra cost |
| **Availability zone** | Distributed across physically separate data centers | 99.99% | Zone billing |

**When to choose:**
- No redundancy: Learning, dev/test, non-critical
- Availability Set: Production without zone support
- Availability Zone: High-availability production

### 4. Zone

**What it is:** The specific availability zone to deploy the VM.

**Options:**
- Zone 1, 2, 3 (where available)
- No zone (default)

**When to choose:** For production workloads requiring high availability across zones.

**Cost Considerations:** Zone-redundant services may cost more.

### 5. Availability Set

**What it is:** A logical grouping that spreads VMs across fault and update domains.

**Options:**
- Create new or use existing availability set

**When to choose:** When deploying multiple VMs in the same region that need high availability.

**Cost Considerations:** Availability sets themselves are free.

### 6. No Infrastructure Redundancy

**What it is:** VM deployed on a single physical host without distribution.

**When to choose:** Development, testing, non-production workloads.

**Cost Considerations:** The cheapest option.

### 7. Security Type

**What it is:** The security features enabled for the VM.

**Options:**

| Type | Features | When to Use |
|------|----------|-------------|
| **Standard** | Basic security | General workloads |
| **Trusted Launch** | Secure Boot, vTPM, VM encryption | Production workloads |
| **Confidential VMs** | Full encryption, AMD SEV-SNP | Highly sensitive data |

**When to choose:** 
- Standard: Learning, low-risk workloads
- Trusted Launch: Most production workloads
- Confidential: Regulatory compliance, sensitive data

### 8. Authentication Type

**What it is:** How you authenticate to the VM.

**Options:**
- **Password**: Traditional username/password
- **SSH public key** (Linux only): Secure key-based authentication
- **Azure Active Directory**: Passwordless authentication

**When to choose:**
- Password: Simple lab environments
- SSH Key: Secure Linux deployments
- Azure AD: Production Windows/Linux environments

### 9. Disk Type

**What it is:** The storage performance tier for disks.

| Type | IOPS | Throughput | Use Case | Cost |
|------|------|------------|----------|------|
| **Standard HDD** | Up to 500 | Up to 60 MB/s | Backup, infrequently accessed data | Lowest |
| **Standard SSD** | Up to 6,000 | Up to 750 MB/s | Web servers, dev/test | Medium |
| **Premium SSD** | Up to 20,000 | Up to 900 MB/s | Production databases | High |
| **Ultra Disk** | Up to 160,000 | Up to 4,000 MB/s | High-performance databases | Very high |

**When to choose:**
- Standard HDD: Archival, backup
- Standard SSD: Most workloads (best value)
- Premium SSD: Performance-critical
- Ultra Disk: Extreme performance workloads

### 10. OS Disk

**What it is:** The disk where the operating system resides.

**Options:**
- OS disk type (Standard HDD, Standard SSD, Premium SSD)
- OS disk size (default 127 GB for Windows)
- Delete with VM (Yes/No)

**When to choose:** 
- Keep OS disk separate from data disks
- Choose delete with VM for temporary environments
- Uncheck delete for persistent environments

### 11. Delete with VM Settings

**What it is:** Whether the disk is deleted when the VM is deleted.

**Options:**
- Enabled (delete disk with VM)
- Disabled (retain disk after VM deletion)

**When to choose:**
- Enabled: Temporary or lab environments
- Disabled: Production data disk retention

**Cost Considerations:** Retained disks continue to incur storage costs.

### 12. Public IP Settings

**What it is:** Configuration for public IP address assignment.

**Options:**
- **Standard SKU**: Regional, secure by default
- **Basic SKU**: Simpler, less secure
- **Dynamic**: IP changes on stop/start
- **Static**: IP remains constant

**When to choose:**
- Standard: Production workloads
- Basic: Learning environments
- Dynamic: Temporary access
- Static: Consistent public endpoint

**Cost Considerations:** Static IPs cost more than dynamic.

### 13. NIC Settings

**What it is:** Configuration of the network interface card.

**Options:**
- NIC name
- Network Security Group
- Accelerated Networking (on/off)
- Private IP assignment (dynamic/static)

**When to choose:**
- Multiple NICs for network appliances
- Accelerated Networking for high-performance workloads

**Cost Considerations:** Accelerated Networking is free but requires compatible VM sizes.

### 14. Accelerated Networking

**What it is:** High-performance network feature using SR-IOV for lower latency and higher throughput.

**When to choose:** 
- High-performance workloads (e.g., web servers, databases)
- Applications sensitive to network latency

**Cost Considerations:** Free on supported VM sizes.

### 15. Boot Diagnostics

**What it is:** Logging of boot processes for troubleshooting.

**Options:**
- Disabled
- Enabled with managed storage
- Enabled with custom storage

**When to choose:** Enable for all VMs (cost is minimal).

### 16. Extensions

**What it is:** Post-deployment configuration scripts and agents.

**Common Extensions:**

| Extension | Purpose |
|-----------|---------|
| **Custom Script Extension** | Run PowerShell/bash scripts on VM |
| **Azure Monitor Agent** | Enable monitoring and metrics |
| **Azure Backup** | Enable VM backup |
| **Antimalware** | Install Microsoft Antimalware |
| **Domain Join** | Join VM to Active Directory |

### 17. Auto-Shutdown

**What it is:** Automatically shut down the VM at a scheduled time.

**When to choose:** Enable for all non-production VMs to save costs.

**Cost Considerations:** Shutdown stops billing for VM compute (not for disk storage).

### 18. Tags

**What it is:** Key-value pairs for resource organization and management.

**Common Tags:**
- `environment`: production, test, dev
- `purpose`: web-server, database, app
- `owner`: team or individual
- `cost-center`: department code
- `expiry`: when resource should be deleted

**When to choose:** Use tags for all resources to enable organization, cost tracking, and automation.

---

## Availability and Resilience

### Availability Set vs. Availability Zones

| Aspect | Availability Set | Availability Zone |
|--------|-----------------|-------------------|
| **Physical Location** | Single data center | Separate data centers |
| **Fault Domains** | 3 (distinct racks) | 3 (independent power/cooling) |
| **Update Domains** | Up to 20 (default 5) | Not applicable |
| **SLA** | 99.95% | 99.99% |
| **Cost** | No additional cost | Zone-to-zone traffic charges |
| **Use Case** | Region-level availability | Region-level disaster recovery |
| **Number of VMs** | Up to 200 in a set | Unlimited across zones |

**When to Use Each:**

- **Availability Set**: Applications that can tolerate rack-level failures but need protection from host OS updates. Good for legacy applications.
- **Availability Zone**: Applications requiring high availability for critical workloads. Protects against entire data center failures.
- **Both**: Combine both for maximum resilience (multiple availability sets across zones).

### Fault Domains and Update Domains

**Fault Domains (FDs):**
- Physical racks in a data center
- Each rack has independent power, cooling, and networking
- Azure distributes VMs across FDs automatically
- Maximum of 3 FDs per availability set

**Update Domains (UDs):**
- Groups of VMs that are updated together
- When Azure performs maintenance, VMs in one UD are updated at a time
- Default: 5 UDs per availability set
- Maximum: 20 UDs per availability set

### Best Practices for Learners

1. **Start Simple**: Deploy VMs without redundancy while learning
2. **Understand Concepts**: Even if you don't implement, understand availability sets and zones
3. **Use Availability Sets for Production**: Multiple VMs need at least an availability set
4. **Leverage Zones for Critical Workloads**: If your region supports them, use availability zones
5. **Regular Testing**: Test failover scenarios to understand behavior

### Dedicated Host Explained

**What it is:** A physical server allocated solely to your Azure subscription.

**Benefits:**
- License compliance (per-core licensing)
- Hardware isolation
- Control over server configuration
- Predictable performance

**Use Cases:**
- Government and defense workloads
- Financial services compliance
- Healthcare applications (HIPAA)
- Legacy applications needing specific hardware
- Software with strict licensing requirements

**When NOT to Use:**
- ✅ Most workloads (standard VMs are sufficient)
- ✅ Learning environments
- ✅ Cost-sensitive projects
- ✅ Workloads without special licensing needs

**Cost Considerations:** Very expensive ($3,000+/month). Not recommended for student accounts.

---

## Networking in Azure VM

### Virtual Network Design

**Address Space:** Choose appropriate IP ranges:

| Application Size | Recommended Address Space |
|------------------|---------------------------|
| Small lab | 10.0.0.0/16 |
| Medium application | 10.0.0.0/16, 10.1.0.0/16 |
| Large enterprise | 10.0.0.0/8 (many subnets) |

**Best Practices:**
- Use non-overlapping IP ranges
- Plan for growth
- Reserve address space for VPN/ExpressRoute
- Use standardized naming conventions

### Subnets

**Subnet Design Considerations:**

| Tier | Address Range | Services | NSG Rules |
|------|---------------|----------|-----------|
| **Web** | 10.0.1.0/24 | IIS, Nginx | Allow HTTPS (443) from internet |
| **Application** | 10.0.2.0/24 | App servers | Allow from Web subnet |
| **Database** | 10.0.3.0/24 | SQL Server, PostgreSQL | Allow from App subnet |
| **Management** | 10.0.4.0/24 | Bastion, Jumpbox | Allow from admin IPs |

### Private IP vs Public IP

| Aspect | Private IP | Public IP |
|--------|------------|-----------|
| **Accessibility** | Only within VNet | Accessible from internet |
| **Cost** | Free | $0.004/hour (Static) |
| **Scope** | Azure VNet | Global |
| **Use Case** | Internal communication | RDP/SSH, web hosting |
| **Change on restart** | Dynamic by default | Dynamic if not Static |
| **NAT** | Not applicable | Required for internet | 

### NSG Inbound and Outbound Rules

**Inbound Rules** (Traffic to VM):

| Priority | Name | Source | Protocol | Port | Action |
|----------|------|--------|----------|------|--------|
| 100 | AllowRDP | Your_IP | TCP | 3389 | Allow |
| 200 | AllowHTTP | * | TCP | 80 | Allow |
| 300 | AllowHTTPS | * | TCP | 443 | Allow |
| 65000 | DenyAll | * | * | * | Deny |

**Outbound Rules** (Traffic from VM):

| Priority | Name | Destination | Protocol | Port | Action |
|----------|------|-------------|----------|------|--------|
| 100 | AllowInternet | * | TCP | * | Allow |
| 65000 | DenyAll | * | * | * | Deny |

**Important:** The default deny rule is implicit and doesn't appear in the list.

### RDP Access

**Best Practices for RDP:**
1. Change default port (3389) to a custom port (33890-33899)
2. Use jumpbox/bastion instead of direct internet access
3. Enable Multi-Factor Authentication (MFA)
4. Use network-level authentication (NLA)
5. Restrict source IPs to your IP range
6. Use Azure Bastion when possible

**RDP Configuration Example:**

```powershell
# Change RDP port using PowerShell (on the VM)
Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber" -Value 33890

# Restart the service
Restart-Service TermService -Force

# Update NSG rule to allow port 33890
```

### Internet Connectivity

**Default Behavior:**
- Azure VMs have outbound internet access by default via NAT
- Inbound access requires explicit NSG rules
- Public IP required for inbound internet access

**Outbound Traffic:**
- Azure automatically NATs outbound traffic
- Multiple VMs share public IPs (by default)
- Explicit outbound policies available

### NAT Gateway

**What it is:** A managed service that provides outbound-only internet connectivity for VMs in a subnet.

**Benefits:**
- Dedicated public IPs for outbound traffic
- Fixed source IP for outbound connections
- Higher throughput for outbound traffic
- Reduces IP address exhaustion

**Use Cases:**
- Services requiring outbound internet access with specific IPs
- Reducing outbound connection failures
- Improving security (known source IPs)

### Azure Bastion

**What it is:** A PaaS service providing secure RDP/SSH access to VMs without public IPs.

**Architecture:**

```
Internet → Azure Portal → Azure Bastion Host → VNet → VM (Private IP)
```

**Benefits:**
- No public IP required for VMs
- RDP/SSH over HTTPS
- Single sign-on with Azure AD
- No exposure of RDP ports to internet
- Session recording (with additional configuration)

**When to Use:**
- ✅ Production environments
- ✅ Security-sensitive workloads
- ✅ Replacing jumpbox VMs
- ✅ When you want to reduce management overhead

**When NOT to Use:**
- ❌ Cost-sensitive labs (Bastion is $0.19/hour)
- ❌ Simple learning environments (cost vs. benefit)
- ❌ Low-bandwidth scenarios (Bastion uses compression)

**Alternative for Students:**
- Use Public IP + NSG restriction to your IP
- Create a small jumpbox VM (B1s) instead of Bastion
- Use Azure VPN or ExpressRoute (if available)

### Routing Basics

**Default Routes:**
- Traffic within VNet: Direct communication
- Internet traffic: Via Azure's NAT
- Peer VNet traffic: Via peering
- On-premises traffic: Via VPN/ExpressRoute

**Custom Routing (UDR - User-Defined Routes):**
- Override default routing behavior
- Route traffic through Network Virtual Appliance (NVA)
- Enforce egress via specific paths

### DNS Basics

**Azure DNS:**
- Default DNS resolution for VNet
- Default server: 168.63.129.16
- Internal DNS for Azure resources

**Custom DNS:**
- Can override default DNS
- Active Directory Domain Services (AD DS) integration
- Custom DNS servers for private zones

**DNS Resolution Flow:**
```
VM → DNS Server (168.63.129.16) → DNS Forwarder → External DNS
```

### Service Endpoints vs. Private Endpoints

| Aspect | Service Endpoint | Private Endpoint |
|--------|------------------|------------------|
| **Resource Access** | Azure service | Any service (Azure/Third-party) |
| **Network Path** | Direct from VNet to service | via VNet | 
| **IP Address** | Public IP (service endpoint) | Private IP in your VNet |
| **Security** | Firewall-based filtering | VNet isolation |
| **Cost** | Free | Charged per hour |
| **Complexity** | Simple | More complex |

**When to Use:**
- Service Endpoint: Quick, no-cost secure access to Azure services
- Private Endpoint: Strict network isolation, on-premises access

---

## Storage and Disks

### OS Disk

**Purpose:** Contains the operating system files and boot loader.

**Characteristics:**
- Default size: 127 GB (Windows), 30 GB (Linux)
- Billed separately from VM
- Persistent across VM restarts
- Usually created from a platform image

**Best Practices:**
1. Don't store application data on OS disk
2. Keep OS disk separate from data disks
3. Create backups of OS disk separately
4. Consider enabling disk encryption

### Temporary Disk

**What it is:** A local, ephemeral disk on the physical host.

**Characteristics:**
- D: drive on Windows, /dev/sdb1 on Linux
- Not persistent (data lost on VM restart)
- High performance (often SSD or NVMe)
- Included with VM (no additional cost)

**Use Cases:**
- Scratch space for temporary data
- Pagefile/swap file
- Session data for stateless applications
- Cache for SQL Server tempdb

**When NOT to Use:**
- Data that must persist across reboots
- Critical application data
- Data requiring backup

### Data Disk

**Purpose:** Persistent storage for application data.

**Characteristics:**
- Up to 64 TB per disk
- Multiple disks per VM (depends on size)
- Attach/detach dynamically
- Configured for specific performance tiers
- Can be backed up independently

**Best Practices:**
1. Use multiple smaller disks for better performance
2. Configure RAID for redundancy (if needed)
3. Encrypt data disks
4. Backup data disks separately from OS disk
5. Choose appropriate performance tier

### Managed Disks vs. Unmanaged Disks

**Managed Disks (Recommended):**

| Feature | Description |
|---------|-------------|
| **Management** | Azure handles storage accounts and scaling |
| **Scalability** | Up to 50,000 disks per subscription per region |
| **Availability** | Automatic replication within region |
| **Backup** | Integrated backup and snapshot |
| **Performance** | Optimized based on disk type |

**Unmanaged Disks (Legacy):**

| Feature | Description |
|---------|-------------|
| **Management** | Customer manages storage accounts |
| **Scalability** | Limited by storage account limits |
| **Availability** | Manual configuration required |
| **Backup** | Manual snapshots and copying |
| **Performance** | Depends on storage account configuration |

**Why Managed Disks are Preferred:**
1. Automatic optimization
2. Simplified scaling
3. Integrated security
4. Better performance predictability
5. No storage account limits

### Disk SKU Types

#### Standard HDD

| Attribute | Value |
|-----------|-------|
| **Use Case** | Backup, archival, infrequent access |
| **Max IOPS** | 500 |
| **Max Throughput** | 60 MB/s |
| **Cost** | Lowest ($$) |
| **Best For** | Cost-sensitive workloads, small VMs |

#### Standard SSD

| Attribute | Value |
|-----------|-------|
| **Use Case** | Web servers, dev/test, lightly used databases |
| **Max IOPS** | 6,000 |
| **Max Throughput** | 750 MB/s |
| **Cost** | Medium ($$$) |
| **Best For** | Most Azure workloads, student VMs |

#### Premium SSD

| Attribute | Value |
|-----------|-------|
| **Use Case** | Production databases, I/O intensive applications |
| **Max IOPS** | 20,000 |
| **Max Throughput** | 900 MB/s |
| **Cost** | High ($$$$) |
| **Best For** | SQL Server, enterprise applications |

#### Ultra Disk

| Attribute | Value |
|-----------|-------|
| **Use Case** | Extreme performance, transactional workloads |
| **Max IOPS** | 160,000 |
| **Max Throughput** | 4,000 MB/s |
| **Cost** | Very High ($$$$$) |
| **Best For** | SAP, high-performance databases |

### Disk Attachment and Formatting on Windows

**Step 1: Initialize Disk**
```powershell
# Install disk management tools
Install-WindowsFeature -Name File-Services

# Initialize disk
Initialize-Disk -Number 2 -PartitionStyle GPT
```

**Step 2: Create Partition**
```powershell
# Create a partition
New-Partition -DiskNumber 2 -Size 32GB -DriveLetter F -UseMaximumSize
```

**Step 3: Format Disk**
```powershell
# Format the partition
Format-Volume -DriveLetter F -FileSystem NTFS -NewFileSystemLabel "DataDisk" -Confirm:$false
```

**Step 4: Verify**
```powershell
# List all volumes
Get-Volume

# Check disk details
Get-Disk
```

### Drive Letters and Storage Planning

**Common Drive Assignments:**

| Drive Letter | Purpose | Persistence |
|--------------|---------|-------------|
| C: | OS Disk | Persistent |
| D: | Temporary Disk | Ephemeral |
| E: | CD-ROM | Virtual drive |
| F: - Z: | Data Disks | Persistent |

**Best Practices:**
1. Use meaningful labels (e.g., "DataDisk", "Logs")
2. Use consistent drive letters across VMs
3. Don't use system drive letters (A:, B:)
4. Plan for future growth

---

## Security Best Practices

### Strong Password Policy

**Complexity Requirements:**
- Minimum length: 12 characters
- Uppercase letters (A-Z)
- Lowercase letters (a-z)
- Numbers (0-9)
- Special characters (!@#$%^&*)
- No dictionary words or common patterns

**Examples:**
- ✅ Good: `AzureLab@2024!Secure`
- ❌ Bad: `Password123` or `AzureVM!`

### Avoiding Open RDP to the Internet

**Risk:**
- Brute force attacks on port 3389
- Vulnerabilities in RDP protocol
- Unauthorized access to the VM

**Best Practices:**
1. **Source IP Restriction** (Critical):
   ```
   Source: Your IP address
   Destination: VM IP
   Port: 3389
   Action: Allow
   ```

2. **Change Default Port**:
   ```powershell
   # On the VM
   Set-ItemProperty -Path "HKLM:\System\CurrentControlSet\Control\Terminal Server\WinStations\RDP-Tcp" -Name "PortNumber" -Value 33890
   ```

3. **Use Just-In-Time Access** (if available):
   - Opens RDP only when needed
   - Automatically closes after set time
   - Uses Azure Defender for Cloud

4. **Use Azure Bastion**:
   - No public IP needed
   - RDP over HTTPS
   - Single sign-on

### Using Bastion Where Possible

**Advantages over Public IP + NSG:**
1. No exposure of RDP port to internet
2. Reduces attack surface dramatically
3. No need to maintain public IP addresses
4. Integrated with Azure AD authentication
5. Session recording capability

**Cost Consideration:**
- Bastion: ~$0.19/hour when active
- Public IP + NSG: Free (with dynamic IP)
- For students: Public IP + NSG is often more cost-effective

### NSG Hardening

**Minimum Required Rules:**

| Priority | Name | Source | Protocol | Port | Action | Reason |
|----------|------|--------|----------|------|--------|--------|
| 100 | AllowRDP | Your_IP | TCP | 3389 | Allow | Remote access |
| 200 | AllowHTTPS | * | TCP | 443 | Allow | Web traffic |
| 300 | AllowHTTP | * | TCP | 80 | Allow | Web traffic |
| 400 | DenyAll | * | * | * | Deny | Catch all |

**Additional Hardening:**
1. Close all unnecessary ports
2. Use Application Security Groups for logical grouping
3. Implement service tags for Azure services
4. Log NSG flow logs for monitoring

### Just-in-Time Access Concept

**What it is:** A feature of Azure Defender for Cloud that provides on-demand access to VMs.

**How it Works:**
1. User requests access to a VM
2. Azure Defender validates permissions
3. NSG rule is temporarily opened
4. User connects within the time window
5. NSG rule is automatically closed

**Benefits:**
- Minimizes open ports
- Reduces attack surface
- Provides audit trail
- Granular access control

**Cost:** Included with Azure Defender for Cloud.

### Defender for Cloud Overview

**What it is:** A cloud-native application protection platform (CNAPP) for Azure resources.

**Key Features:**
1. Security assessments
2. Vulnerability scanning
3. Threat detection
4. Just-in-time access
5. Compliance monitoring
6. Security recommendations

**Pricing:**
- Free tier: Basic security assessment
- Paid tier: Full threat protection

### Updates and Patching

**Manual Patching:**
```powershell
# Check for updates
Get-WindowsUpdate -List

# Install all updates
Install-WindowsUpdate -AcceptAll -AutoReboot
```

**Automated Patching Options:**
1. **Azure Update Management**
   - Centralized patching
   - Scheduled deployments
   - Compliance tracking

2. **Azure Automation**
   - Custom patching schedules
   - Integration with existing tools

3. **Configuration Manager**
   - Deep integration with on-premises
   - Detailed reporting

### Least Privilege Principle

**Definition:** Users and processes should have only the permissions they need.

**Implementation:**
1. Use Azure AD RBAC for management
2. Use separate admin and user accounts
3. Avoid running applications as administrator
4. Use managed identities for service access
5. Review permissions regularly

### Managed Identity

**What it is:** An Azure AD identity assigned to a resource.

**Types:**
- **System-assigned**: Tied to the VM lifecycle
- **User-assigned**: Independent of VM, can be shared

**Benefits:**
- No credentials to manage
- Secure access to Azure services
- Automatic rotation
- Integration with Azure AD

**Example Usage:**
```powershell
# VM uses managed identity to access Key Vault
$secret = (Get-AzKeyVaultSecret -VaultName "myvault" -Name "connectionstring").SecretValueText
```

### Secure Admin Access

**Best Practices:**
1. **Use Azure AD Authentication**
   ```powershell
   # Join VM to Azure AD
   Add-Computer -AzureAD -TenantId "your-tenant-id"
   ```

2. **Enable Azure AD Login**
   ```powershell
   # Install Azure AD extension
   Set-AzVMExtension -ResourceGroupName "RG-WindowsVM" -VMName "winvm" -Name "AADLoginForWindows" -Publisher "Microsoft.Azure.ActiveDirectory" -Type "AADLoginForWindows" -TypeHandlerVersion "1.0"
   ```

3. **Disable Local Admin** (if using Azure AD)
4. **Enable MFA for all admins**
5. **Implement Privileged Identity Management (PIM)**

---

## Monitoring and Operations

### Azure Monitor

**What it is:** A comprehensive monitoring service for Azure resources.

**Key Components:**
1. **Metrics**: Performance data (CPU, memory, disk)
2. **Logs**: System and application logs
3. **Alerts**: Notifications for critical events
4. **Dashboards**: Custom monitoring views

**Metrics Collected:**
| Metric | Description | Threshold |
|--------|-------------|-----------|
| Percentage CPU | CPU utilization | > 80% for production |
| Available Memory | Free memory | < 10% |
| Disk IOPS | Disk operations/sec | > 80% of limit |
| Network In/Out | Network bandwidth | > 80% of capacity |

### VM Insights

**What it is:** A feature of Azure Monitor for comprehensive VM monitoring.

**Capabilities:**
- Performance maps (visual)
- Dependency mapping
- Application diagnostics
- Health tracking

**Access:**
```
Azure Portal → Monitor → Insights → Virtual Machines
```

### Logs and Metrics

**Activity Log:**
- Audit trail of management operations
- Who did what, when, and where
- Compliance and troubleshooting

**Metrics:**
- Performance counters
- System resource usage
- Service health

**Diagnostic Settings:**
```powershell
# Send metrics to Log Analytics workspace
Set-AzDiagnosticSetting -ResourceId $VM.Id -WorkspaceId $LogAnalyticsId -Enabled $true
```

### Boot Diagnostics

**What it is:** Captures serial console output and screenshots during boot.

**When to Use:**
- VM won't boot
- Boot loop issues
- Kernel panics
- Driver problems

**Viewing:**
```
Azure Portal → VM → Support + troubleshooting → Boot diagnostics
```

### Activity Log

**What it is:** A log of all management operations on the VM.

**Information Captured:**
- VM creation/deletion
- Size changes
- Disk operations
- Network changes

**Query Example:**
```powershell
# Find all operations on a VM
Get-AzActivityLog -ResourceId $VM.Id -StartTime (Get-Date).AddDays(-7)
```

### Alerts

**Types:**
1. **Metric Alerts**: Based on performance metrics
2. **Log Alerts**: Based on log searches
3. **Activity Log Alerts**: Based on management events

**Common Alert Rules:**

| Alert | Condition | Action |
|-------|-----------|--------|
| High CPU | CPU > 90% for 5 mins | Email, Scale set |
| Low Memory | Available memory < 10% | Email, Restart |
| VM Down | Heartbeat missing | Email, Phone |
| Disk Full | Free space < 5% | Email, Cleanup |

**Creating an Alert:**
```powershell
# Create a metric alert for high CPU
New-AzMetricAlertRuleV2 -Name "HighCPUAlert" -ResourceGroupName "RG-WindowsVM" -Scope $VM.Id -TargetResourceType "Microsoft.Compute/virtualMachines" -TargetResourceId $VM.Id -WindowSize 00:05:00 -Frequency 00:01:00 -Severity 2 -Criteria (New-AzMetricAlertRuleV2Criteria -MetricName "Percentage CPU" -Operator GreaterThan -Threshold 90 -TimeAggregation Average)
```

### Auto-Shutdown

**What it is:** Automatically deallocates the VM at a scheduled time.

**Benefits:**
- Cost savings
- Resource management
- Automated governance

**Configuration:**
```
Azure Portal → VM → Auto-shutdown → Configure
```

**Settings:**
- Time: 18:00 (6 PM)
- Time zone: Your local time
- Notification: Send email

### Backup

**What it is:** Azure Backup provides data protection for VMs.

**Key Concepts:**
- **Recovery Services Vault**: Storage for backups
- **Backup Policy**: Schedule and retention
- **Instant Recovery**: Restore quickly

**Backup Policy Example:**

| Setting | Value |
|---------|-------|
| Frequency | Daily |
| Time | 23:00 |
| Retention | 7 days |
| Weekly backup | Sunday |
| Monthly backup | Last Sunday |
| Yearly backup | Dec 31 |

**Cost Elements:**
1. Protected instance fee (based on VM size)
2. Backup storage cost
3. Restore data transfer (if cross-region)

### Restore Concept

**Restore Methods:**
1. **Full VM Restore**: Restore entire VM to original location
2. **Disk Restore**: Restore individual disks
3. **File Recovery**: Restore specific files/folders

**Restore Process:**
```
1. Choose restore point
2. Select restore method
3. Specify target (new or existing VM)
4. Confirm and restore
5. Verify VM operation
```

### Troubleshooting Basics

**Common Issues:**
1. High CPU → Check processes, scale up or optimize
2. High memory → Check memory leak, scale up
3. Slow performance → Check disk IOPS, network latency
4. VM down → Check health status, boot diagnostics

**Troubleshooting Steps:**
1. Check Azure Monitor metrics
2. Review Activity Log for events
3. Check Boot Diagnostics
4. Review NSG rules
5. Check disk storage
6. Verify network connectivity

---

## Cost Optimization for Student Subscription

### How to Stay Free or Low-Cost

**Azure Student Benefits:**
- $100-$200 in free credits
- 12 months of free services
- Free tier services available

**Free Tier Services:**
- Linux VMs (B1s, 750 hours/month)
- Windows VMs (B1s, 750 hours/month)
- Azure SQL Database
- Azure Storage (5 GB)
- Azure App Service

### Pick the Smallest Practical VM Size

**Tiered Approach:**

1. **Start with B1s** (1 vCPU, 1 GB RAM) → $0.012/hour
2. **Monitor Performance** → Scale only if needed
3. **Try B2s** (2 vCPU, 4 GB RAM) → $0.048/hour
4. **Consider D-series only for production**

**Performance Testing:**
```powershell
# Check CPU and memory usage
Get-WmiObject -Class Win32_PerfFormattedData_PerfOS_Processor
Get-WmiObject -Class Win32_PerfFormattedData_PerfOS_Memory
```

### Choose Low-Cost Regions

**Least Expensive Regions:**
1. Central India
2. East US
3. South Central US
4. North Central US
5. East Asia

**Most Expensive Regions:**
1. West US
2. West Europe
3. Australia Southeast
4. Brazil South

**Region Selection Strategy:**
1. Choose physically closest to users
2. Check regional pricing differences
3. Consider data residency requirements
4. Evaluate available services

### Public IP Usage

**When to Use Public IP:**
- RDP/SSH access from internet
- Web server hosting
- VPN gateway

**When NOT to Use Public IP:**
- Internal-only workloads
- VMs accessible via Bastion
- VMs requiring only private IP

**Cost Impact:**
| IP Type | Hourly Cost | Monthly Cost |
|---------|-------------|--------------|
| Dynamic IP | $0.004 | $2.88 |
| Static IP | $0.004 | $2.88 |
| No Public IP | $0 | $0 |

### Delete Resources After Lab

**Importance:**
- Stop billing for compute
- Prevent unexpected charges
- Keep subscription clean

**Resource Lifecycle:**
```
1. Create VM → 2. Use for lab → 3. Stop/Deallocate → 4. Delete when done
```

### Unmanaged Growth in Disks and IPs

**Hidden Costs:**
- OS Disk: $0.08/GB/month
- Data Disks: $0.08/GB/month
- Managed Disk Snapshots: $0.05/GB/month
- Public IP: $0.004/hour each

**Prevention:**
1. Delete unnecessary disks
2. Remove unused public IPs
3. Clean up snapshots and images
4. Review billing regularly

### Use of Auto-Shutdown

**Implement Auto-Shutdown:**
```powershell
# Set auto-shutdown via CLI
az vm auto-shutdown -g RG-WindowsVM -n winvm-student --time 1800
```

**Benefits:**
- Stops VM at end of workday
- Prevents weekend usage charges
- Save up to 70% on costs
- Automates management

### Clean-up Checklist

**For Each VM:**
- [ ] Stop/Deallocate when not in use
- [ ] Delete VM if no longer needed
- [ ] Delete OS Disk (unless needed)
- [ ] Delete Data Disks (unless needed)
- [ ] Delete Public IP (unless needed)
- [ ] Delete NIC (if VM deleted)

**For Resource Group:**
- [ ] Delete whole resource group (if done)
- [ ] Verify all resources are deleted
- [ ] Check billing dashboard

### Common Billing Mistakes to Avoid

| Mistake | Impact | Solution |
|---------|--------|----------|
| Leaving VM running overnight | $0.10-$1.00/day | Auto-shutdown |
| Forgetting to delete VM | $10-$50/month | Delete when done |
| Leaving public IP attached | $2.88/month/IP | Remove when not needed |
| Unused data disks | $0.08/GB/month | Delete unused disks |
| Not using free tier | $0.012-$0.096/hour | Leverage free tier |
| Premium disks for dev | $10-$50/month | Use Standard SSD |
| Not deleting snapshots | $0.05/GB/month | Clean up snapshots |
| Cross-region data transfer | $0.02/GB | Minimize transfers |

---

## Interview Questions

### Beginner Questions

**1. What is Azure Virtual Machine?**
> A: Azure Virtual Machine is an on-demand, scalable computing resource provided by Azure. It's like a computer in the cloud with its own operating system, CPU, memory, and storage. You can create and manage VMs using the Azure Portal, CLI, or APIs.

**2. What is the difference between an availability set and an availability zone?**
> A: An availability set distributes VMs across different racks (fault domains) within a single data center and handles planned maintenance (update domains). Availability zones distribute VMs across physically separate data centers within a region. Availability zones offer higher SLA (99.99% vs 99.95%) but can cost more due to cross-zone traffic.

**3. How do you connect to a Windows VM?**
> A: You connect to a Windows VM using Remote Desktop Protocol (RDP) on port 3389. For security, you should restrict the source IP to your IP address using Network Security Groups, or better, use Azure Bastion which provides secure RDP access through the Azure Portal without exposing the VM to the internet.

**4. What are the different types of disk storage in Azure?**
> A: Azure offers four types: Standard HDD (lowest cost, backup), Standard SSD (good balance for most workloads), Premium SSD (high performance, production), and Ultra Disk (extreme performance, specialized workloads). Students should use Standard SSD for a good balance of cost and performance.

**5. What is a resource group?**
> A: A resource group is a logical container for Azure resources. All resources in a group share the same lifecycle, making it easy to deploy, manage, and delete related resources together. Resources can be in different regions but must be in the same subscription.

### Intermediate Questions

**6. Explain NSG (Network Security Group) in detail.**
> A: NSG is a firewall for Azure VMs that controls inbound and outbound traffic. It contains rules with priority, source/destination (IP, service tag, or application security group), protocol, port, and action (allow/deny). NSGs can be applied at the subnet or individual NIC level. Rules are evaluated in priority order.

**7. How do you choose the right VM size for a workload?**
> A: Consider CPU requirements, memory needs, disk IOPS, network bandwidth, and cost. Use B-series for variable workloads, D-series for balanced workloads, E-series for memory-intensive applications, and F-series for compute-intensive tasks. Start with smaller sizes and scale up based on monitoring data.

**8. What is Azure Managed Identity and why is it useful?**
> A: Managed Identity provides an Azure AD identity for the VM without managing credentials. It allows the VM to authenticate to Azure services (Key Vault, Storage, SQL) securely. Use system-assigned when the identity is tied to the VM's lifecycle, or user-assigned for sharing across resources.

**9. How does Azure Backup work for VMs?**
> A: Azure Backup uses the Recovery Services Vault to store backups. It supports application-consistent backups using VSS on Windows. Backup policies define schedule and retention. Restore options include full VM restore, disk restore, or file-level recovery.

**10. What is Azure Bastion?**
> A: Azure Bastion is a PaaS service that provides secure RDP/SSH access to VMs without public IP addresses. It works through the Azure Portal over HTTPS, eliminating the need to open RDP ports to the internet. It integrates with Azure AD for single sign-on.

### Advanced Interview Questions

**11. Describe the difference between Azure Load Balancer and Application Gateway.**
> A: Load Balancer operates at Layer 4 (TCP/UDP) and distributes traffic based on IP and port. Application Gateway operates at Layer 7 (HTTP/HTTPS) and provides URL-based routing, SSL termination, and Web Application Firewall (WAF). Use Load Balancer for simple distribution and Application Gateway for web applications requiring advanced routing.

**12. Explain the concept of availability sets and how they handle maintenance.**
> A: Availability sets distribute VMs across three fault domains (physical racks) and up to 20 update domains. During planned maintenance, Azure updates one update domain at a time with a 30-minute delay between domains. This ensures at least one copy of the application is available during maintenance windows.

**13. How do you implement disaster recovery for Azure VMs?**
> A: Use Azure Site Recovery (ASR) to replicate VMs to a different Azure region. ASR provides continuous replication with RPO as low as 15 minutes. Recovery plans define sequenced failover steps. Test failover can be performed without impacting production.

**14. What is Azure Dedicated Host and when would you use it?**
> A: Dedicated Host provides a physical server solely for your workloads. Use it for license compliance (Windows Server, SQL Server with SA), physical isolation requirements (government, financial), and hardware control. It's expensive ($3,000+/month) and not for student labs.

**15. How does accelerated networking improve VM performance?**
> A: Accelerated networking uses Single Root I/O Virtualization (SR-IOV) to bypass the host's network stack, delivering lower latency and higher throughput. It's available on specific VM sizes and can improve performance by up to 30% for network-intensive workloads.

### Real-World Scenario Questions

**16. Scenario: You need to deploy a web application with high availability and automatic scaling. How would you design it?**
> A: Use an Azure Load Balancer or Application Gateway in front of an Azure Virtual Machine Scale Set (VMSS). Configure the scale set to auto-scale based on CPU or custom metrics. Deploy VMs across availability zones for high availability. Use Azure Monitor for health monitoring and alerts.

**17. Scenario: Your VM is running slowly. How would you troubleshoot?**
> A: Check Azure Monitor metrics for CPU, memory, and disk usage. Review the Activity Log for recent changes. Use Boot Diagnostics if it's a boot issue. Check NSG rules for network throttling. Consider upgrading VM size if resource utilization is consistently high.

**18. Scenario: You suspect a security breach on your Windows VM. What steps would you take?**
> A: Immediately isolate the VM using NSG rules. Check the Activity Log for suspicious activity. Review VM logs (Event Viewer, Security logs). Use Azure Defender for Cloud for threat detection. Create a forensic snapshot before remediation. Rebuild from a clean image and restore data from backup.

**19. Scenario: How would you migrate 100 on-premises VMs to Azure?**
> A: Use Azure Migrate for assessment and migration. Start with discovery and dependency mapping. Use the Azure Migrate migration tool for agentless migration or Azure Site Recovery for agent-based migration. Plan for networking (VPN/ExpressRoute), storage (disk sizing), and security (NSG/Azure AD). Test with a pilot before full migration.

**20. Scenario: You need to cost-optimize an Azure VM deployment with varying workloads. What strategies would you use?**
> A: Use B-series burstable VMs for variable workloads. Implement auto-shutdown for non-production VMs. Use Azure Reserved Instances for production workloads. Right-size VMs based on actual utilization data. Use spot instances for batch jobs. Leverage Azure Hybrid Benefit for Windows licensing.

---

## Troubleshooting

### RDP Connection Failures

| Symptom | Possible Cause | Solution |
|---------|---------------|----------|
| "Connection refused" | NSG blocking port 3389 | Check NSG rules, add inbound allow rule |
| "Network error" | Public IP missing/deleted | Create a public IP and associate |
| "Credential error" | Wrong username/password | Reset password via Azure Portal |
| "Timeout" | VM not running | Start the VM in Azure Portal |
| "Certificate error" | Security warning | Accept the certificate or trust it |
| "Authentication failed" | NLA enabled, remote creds | Disable NLA temporarily |

**Quick Fixes:**
```bash
# Reset password via Azure CLI
az vm user update -g RG-WindowsVM -n winvm-student -u azureuser --password "NewPassword123!"

# Restart VM
az vm restart -g RG-WindowsVM -n winvm-student

# Check NSG rules
az network nsg list -g RG-WindowsVM --output table
```

### NSG Blocking Traffic

**Symptoms:**
- "Connection timed out"
- "No route to host"
- Unable to access web applications

**Solutions:**
1. Verify NSG rules in Azure Portal
2. Ensure inbound rules allow traffic
3. Check priority order (smaller number = higher priority)
4. Verify source/destination IP addresses
5. Check for default deny rule

### Wrong Username/Password

**Reset Password via Portal:**
1. Go to VM in Azure Portal
2. Click "Reset password" under Support + troubleshooting
3. Enter new username and password
4. Click Update

**Reset via CLI:**
```bash
az vm user update -g RG-WindowsVM -n winvm-student -u azureuser --password "NewPassword123!"
```

### VM Stopped or Deallocated

**States:**
- **Stopped (Allocated)**: VM stopped, but resources allocated (still billing for compute)
- **Stopped (Deallocated)**: VM stopped and deallocated (no compute billing)

**Start VM:**
```bash
az vm start -g RG-WindowsVM -n winvm-student
```

**Stop and Deallocate:**
```bash
az vm deallocate -g RG-WindowsVM -n winvm-student
```

### Disk Not Visible in Windows

**Symptoms:**
- Data disk attached but not visible in File Explorer
- Disk not initialized

**Solution:**
1. Open Disk Management (diskmgmt.msc)
2. Initialize the disk
3. Create a partition
4. Format the partition
5. Assign a drive letter

```powershell
# Initialize disk
Initialize-Disk -Number 2 -PartitionStyle GPT

# Create partition and format
New-Partition -DiskNumber 2 -Size 32GB -DriveLetter F | Format-Volume -FileSystem NTFS -Confirm:$false
```

### Public IP Issues

**Symptoms:**
- Public IP not attached
- Public IP changed (dynamic)

**Solution:**
1. Check Public IP in Azure Portal
2. For dynamic IP, note the current IP
3. Consider using static IP for consistent access

```bash
# Get current public IP
az vm show -d -g RG-WindowsVM -n winvm-student --query publicIps -o tsv

# Associate a static public IP
az network public-ip create -g RG-WindowsVM -n myStaticIP --allocation-method Static
az network nic ip-config update -g RG-WindowsVM --nic-name winvm-student-nic --ip-config-name ipconfig1 --public-ip-address myStaticIP
```

### DNS Issues

**Symptoms:**
- Unable to resolve domain names
- "DNS name doesn't exist"

**Solutions:**
1. Check DNS settings in VNet
2. Verify custom DNS server configuration
3. Use Azure DNS for private zones

### Boot Problems

**Symptoms:**
- VM fails to boot
- Boot loop
- Blue screen

**Solutions:**
1. Review Boot Diagnostics in Azure Portal
2. Check boot order in VM settings
3. Repair OS disk
4. Restore from backup

### Low Disk Space

**Symptoms:**
- "Disk is full" errors
- Performance degradation

**Solutions:**
1. Extend the disk
   ```powershell
   # Increase disk size via portal, then extend partition
   Resize-Partition -DriveLetter C -Size 50GB
   ```
2. Clean temporary files
   ```powershell
   # Disk Cleanup
   cleanmgr /sagerun:1
   ```
3. Move files to data disk
4. Archive/delete old files

### Slow VM Performance

**Troubleshooting Steps:**

1. **Check CPU utilization**:
   ```powershell
   Get-WmiObject -Class Win32_Processor | Select-Object LoadPercentage
   ```

2. **Check memory usage**:
   ```powershell
   Get-WmiObject -Class Win32_OperatingSystem | Select-Object FreePhysicalMemory, TotalVisibleMemorySize
   ```

3. **Check disk performance**:
   ```powershell
   Get-WmiObject -Class Win32_PerfFormattedData_PerfDisk_PhysicalDisk
   ```

4. **Consider upgrading VM size**
5. **Check network bandwidth**
6. **Look for resource contention**

---

## Hands-On Summary

### What Was Created

During this lab, you created a complete Azure VM environment including:

1. **Resource Group** (`RG-WindowsVM-Student`)
   - Logical grouping of all resources

2. **Virtual Network** (`VNet-WindowsVM`)
   - Private network with address space 10.0.0.0/16
   - Subnet (`default`) with 10.0.0.0/24

3. **Windows VM** (`winvm-student`)
   - Windows Server 2022 Datacenter
   - B1s size (1 vCPU, 1 GB RAM)
   - Standard SSD OS disk

4. **Public IP** (Dynamic)
   - For RDP access from internet

5. **Network Security Group** (Auto-created)
   - Inbound RDP rule (port 3389)
   - Other default rules

6. **Optional Data Disk** (32 GB)
   - For application data separation

7. **Monitoring & Management**
   - Boot Diagnostics enabled
   - Auto-shutdown configured (6 PM)

### What Each Resource Does

| Resource | Purpose | Key Points |
|----------|---------|------------|
| **VM** | Compute resource | Runs your application |
| **OS Disk** | Operating system | Contains Windows Server |
| **Data Disk** | Application data | Separate from OS |
| **VNet** | Private network | Isolated network |
| **Subnet** | IP addressing | Defines IP range for VMs |
| **NIC** | Network connectivity | Connects VM to VNet |
| **Public IP** | Internet access | Enables RDP from internet |
| **NSG** | Network security | Firewall rules |
| **Resource Group** | Resource management | Organizational container |

### What the Learner Should Remember

**Core Concepts:**
1. VMs are the foundation of IaaS in Azure
2. Resource organization is critical (Resource Groups, Tags)
3. Security is everyone's responsibility (NSG, passwords, encryption)
4. Availability and resilience require planning (Sets, Zones)
5. Cost management is essential (right-sizing, shutdown, cleanup)

**Practical Skills:**
- Deploy a Windows VM in 5 minutes
- Connect via RDP securely
- Attach and format data disks
- Create and manage NSG rules
- Monitor VM performance
- Implement backup and restore
- Clean up resources to avoid charges

### What Concepts are Important for Interviews

**Must-Know:**
- IaaS vs PaaS vs SaaS
- VM architecture (VNet, Subnet, NIC, NSG)
- Availability (Sets, Zones, Fault/Update Domains)
- Storage (Disk types, managed vs unmanaged)
- Security (NSG, Public vs Private IP)

**Nice-to-Know:**
- Azure Bastion
- Azure Backup and Site Recovery
- Managed Identity
- Azure Monitor
- Auto-scaling (VMSS)

**Differentiators:**
- Cost optimization strategies
- Advanced networking
- Automation with CLI/PowerShell
- Architecture best practices
- Troubleshooting methodology

---

## Cleanup Section

### Delete Resources to Avoid Charges

#### Option 1: Delete Entire Resource Group (Recommended)

**Using Azure Portal:**
1. Go to Resource Groups
2. Select `RG-WindowsVM-Student`
3. Click **Delete resource group**
4. Type the name to confirm
5. Click **Delete**

**Using Azure CLI:**
```bash
az group delete -n RG-WindowsVM-Student --yes --no-wait
```

**Using PowerShell:**
```powershell
Remove-AzResourceGroup -Name RG-WindowsVM-Student -Force
```

#### Option 2: Delete Individual Resources

**Step 1: Delete the VM**
```
Azure Portal → Virtual Machines → winvm-student → Delete
```

**Step 2: Delete the OS Disk**
```
Azure Portal → Disks → winvm-student_OsDisk → Delete
```

**Step 3: Delete Data Disk (if created)**
```
Azure Portal → Disks → datadisk01 → Delete
```

**Step 4: Delete Public IP**
```
Azure Portal → Public IP addresses → winvm-student-ip → Delete
```

**Step 5: Delete Network Interface**
```
Azure Portal → Network interfaces → winvm-student-nic → Delete
```

**Step 6: Delete Network Security Group**
```
Azure Portal → Network security groups → winvm-student-nsg → Delete
```

**Step 7: Delete Virtual Network**
```
Azure Portal → Virtual networks → VNet-WindowsVM → Delete
```

**Step 8: Delete Resource Group (if empty)**
```
Azure Portal → Resource groups → RG-WindowsVM-Student → Delete
```

### Cleanup Checklist

**Verify all resources are deleted:**
- [ ] VM
- [ ] OS Disk
- [ ] Data Disks (if created)
- [ ] Public IP
- [ ] NIC
- [ ] NSG
- [ ] VNet
- [ ] Resource Group

**Check for hidden costs:**
- [ ] Disks in Azure (managed/unmanaged)
- [ ] Snapshots
- [ ] Public IP addresses
- [ ] Network interfaces
- [ ] Azure Monitor logs
- [ ] Backup instances

### Cost Verification

**Check Current Billing:**
```
Azure Portal → Cost Management + Billing → Cost analysis
```

**Verify Resources:**
```bash
# List all resources in subscription
az resource list --output table

# List all virtual machines
az vm list --output table

# List all disks
az disk list --output table

# List all public IPs
az network public-ip list --output table
```

### Prevention for Future Labs

**1. Use Auto-Shutdown:**
```bash
az vm auto-shutdown -g RG-WindowsVM -n winvm-student --time 1800
```

**2. Use Scheduled Events:**
```bash
# Query scheduled events on VM
az vm get-scheduled-events -g RG-WindowsVM -n winvm-student
```

**3. Set Budget Alerts:**
```
Azure Portal → Cost Management + Billing → Budgets → Create
```

**4. Tag Resources for Tracking:**
```bash
# Apply tags for easy identification
az resource tag -g RG-WindowsVM -n winvm-student --resource-type Microsoft.Compute/virtualMachines --tags environment=lab purpose=learning owner=student
```

**5. Regular Cleanup:**
- Remove VMs not used for 48 hours
- Delete unused disk and snapshots
- Review billing weekly

---

## Additional Resources

### Official Documentation

- [Azure Virtual Machines Documentation](https://docs.microsoft.com/en-us/azure/virtual-machines/)
- [Azure Virtual Network Documentation](https://docs.microsoft.com/en-us/azure/virtual-network/)
- [Azure Storage Documentation](https://docs.microsoft.com/en-us/azure/storage/)
- [Azure Monitor Documentation](https://docs.microsoft.com/en-us/azure/azure-monitor/)

### Learning Paths

- [Azure Fundamentals (AZ-900)](https://docs.microsoft.com/en-us/learn/certifications/azure-fundamentals/)
- [Azure Administrator (AZ-104)](https://docs.microsoft.com/en-us/learn/certifications/azure-administrator/)
- [Azure Developer (AZ-204)](https://docs.microsoft.com/en-us/learn/certifications/azure-developer/)

### Tools

- [Azure Portal](https://portal.azure.com/)
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/)
- [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/)
- [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/)
- [Azure Resource Explorer](https://resources.azure.com/)

### Community

- [Azure on Reddit](https://www.reddit.com/r/Azure/)
- [Azure on Stack Overflow](https://stackoverflow.com/questions/tagged/azure)
- [Microsoft Azure Community](https://azure.microsoft.com/en-us/community/)
- [Azure Tech Community](https://techcommunity.microsoft.com/t5/azure/ct-p/Azure)

### Student Resources

- [Azure for Students](https://azure.microsoft.com/en-us/free/students/)
- [Microsoft Learn Student Hub](https://docs.microsoft.com/en-us/learn/student-hub/)
- [Azure Free Account](https://azure.microsoft.com/en-us/free/)
- [Azure Education Hub](https://azure.microsoft.com/en-us/education/)

---

## Conclusion

You've now completed a comprehensive journey through Azure Virtual Machines, from understanding virtualization fundamentals to deploying a production-grade Windows VM with best practices for security, networking, storage, availability, and cost optimization.

### Key Takeaways

1. **Virtualization is the Foundation**: Understand hypervisors, type 1 vs type 2, and how Azure leverages virtualization at scale.

2. **Architecture Matters**: Each component (VNet, NSG, Disk, etc.) has a specific purpose and cost implications.

3. **Security is Paramount**: Always use strong passwords, restrict RDP access, and implement monitoring.

4. **Cost Management is Critical**: Use the smallest size, auto-shutdown, and always clean up after labs.

5. **Availability Requires Planning**: Know when to use availability sets vs zones.

6. **Practice Makes Perfect**: The best way to learn Azure is to do hands-on labs.

### What's Next?

1. **Deploy a Linux VM** to understand differences
2. **Add another VM** and configure a load balancer
3. **Set up Azure Backup** for your VM
4. **Implement Azure Site Recovery** for disaster recovery
5. **Use Azure DevOps** to deploy infrastructure as code

### Final Tips

- 💰 **Always monitor costs** - Azure credits are valuable
- 🔒 **Security first** - Never expose RDP to internet without restriction
- 📚 **Document your learning** - Create your own reference notes
- 🤝 **Share knowledge** - Help other students in the community
- 🏗️ **Build projects** - Apply your knowledge to real-world problems

---

## Mermaid Diagram: Azure VM Architecture Flow

```mermaid
graph TB
    User[User/Developer] --> Portal[Azure Portal/CLI]
    Portal --> RG[Resource Group<br/>RG-WindowsVM-Student]
    
    RG --> VNet[Virtual Network<br/>VNet-WindowsVM]
    VNet --> Subnet[Subnet<br/>default]
    Subnet --> NIC[Network Interface<br/>winvm-student-nic]
    
    RG --> VM[Virtual Machine<br/>winvm-student]
    NIC --> VM
    VM --> OS_OSDisk[OS Disk<br/>128 GB]
    VM --> DataDisk[Data Disk<br/>32 GB]
    
    NIC --> PIP[Public IP]
    PIP --> Internet[Internet]
    
    NIC --> NSG[Network Security Group<br/>Inbound: RDP (3389)]
    
    VM --> AzureMonitor[Azure Monitor<br/>Metrics + Logs]
    VM --> BootDiag[Boot Diagnostics]
    VM --> AutoShutdown[Auto-Shutdown<br/>18:00]
    
    RG --> BG[Backup<br/>Optional]
    RG --> Tags[Tags:<br/>environment=lab<br/>purpose=learning]
    
    classDef azure fill:#0072C6,stroke:#005A9E,color:white
    classDef compute fill:#F0B800,stroke:#D4A000,color:black
    classDef storage fill:#B8D432,stroke:#9BBF2A,color:black
    classDef network fill:#5EADAD,stroke:#4A8F8F,color:white
    
    class VM,VMSS compute
    class PIP,NSG,NIC,VNet,Subnet network
    class OS_OSDisk,DataDisk storage
    class RG,Portal,User azure
```

---

## Images Placeholders

### Architecture Diagram
```
![Azure VM Architecture](images/azure-vm-architecture.png)
```
*Place your Azure VM architecture diagram here showing all components.*

### Deployment Flow
```
![Windows VM Deployment Flow](images/windows-vm-flow.png)
```
*Place your step-by-step deployment flow diagram here.*

### Component Map
```
![Azure VM Components](images/azure-vm-components.png)
```
*Place your component interaction map here.*

### Security Diagram
```
![Azure VM Security](images/azure-vm-security.png)
```
*Place your security architecture diagram here.*

### Cost Optimization
```
![Cost Optimization](images/cost-optimization.png)
```
*Place your cost optimization visualization here.*

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## Contributing

Contributions are welcome! Please read the [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct and the process for submitting pull requests.

---

## Support

If you found this guide helpful, please consider:
- ⭐ Starring the repository
- 🍴 Forking the repository
- 📖 Sharing with other learners
- 💬 Providing feedback and suggestions

---

**Happy Learning!** 🚀

*"The best way to learn Azure is to build things, break things, and fix things."*

---

*© 2024 Azure Virtual Machines Master Guide | Made with ❤️ for Azure Students*
