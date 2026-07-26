
# Azure Compute Master Guide
## From AWS Professional to Azure Solutions Architect Expert

> **Author:** Rajesh Singamsetti
>
> **Target Audience**
>
> - AWS Cloud Engineers transitioning to Azure
> - Azure Administrators
> - Azure Engineers
> - Azure Infrastructure Engineers
> - Azure DevOps Engineers
> - Site Reliability Engineers (SRE)
> - Azure Solutions Architects
> - Microsoft Certification Candidates
>
> **Skill Level**
>
> ⭐ Intermediate → Expert → Enterprise Architect

---

# Purpose

This repository is designed to transform an experienced AWS Cloud Engineer into an Azure Compute Subject Matter Expert.

Unlike beginner Azure courses, this guide assumes you already understand cloud computing concepts and focuses on Azure architecture, enterprise implementation, production operations, migration strategies, and Microsoft interview expectations.

Every topic includes:

- Enterprise Architecture
- AWS Comparison
- Azure Portal
- Azure CLI
- Azure PowerShell
- Terraform
- Bicep
- ARM Templates
- REST API
- SDK Examples
- Production Best Practices
- Networking
- Storage
- Security
- Monitoring
- High Availability
- Disaster Recovery
- Cost Optimization
- Troubleshooting
- Microsoft Interview Questions
- Hands-on Labs
- Enterprise Case Studies
- Certification Preparation

---

# Learning Roadmap

## Foundation

- Azure Global Infrastructure
- Regions
- Region Pairs
- Availability Zones
- Availability Sets
- Resource Groups

---

## Azure Compute Fundamentals

- Azure Compute Overview
- Azure Virtual Machines
- VM Sizes
- VM Images
- Azure Compute Gallery
- Managed Disks
- Ephemeral OS Disks
- Ultra Disks
- Shared Disks
- Disk Encryption
- Snapshots

---

## Business Continuity

- Azure Backup
- Azure Site Recovery
- Boot Diagnostics
- Serial Console
- VM Extensions
- Run Command
- Custom Script Extension

---

## Identity Integration

- Managed Identity
- Microsoft Entra ID
- RBAC
- PIM
- Conditional Access

---

## Enterprise Compute

- Dedicated Host
- Capacity Reservation
- Spot Virtual Machines
- Proximity Placement Groups

---

## High Availability

- Availability Sets (Deep Dive)
- Availability Zones (Deep Dive)
- Virtual Machine Scale Sets
- Autoscaling

---

## Load Balancing

- Azure Load Balancer
- Application Gateway
- Azure Front Door

---

## Monitoring

- Azure Monitor
- Log Analytics
- Azure Policy
- Azure RBAC

---

## Hybrid Cloud

- Azure Arc
- Azure Image Builder
- Azure Automanage
- Azure Update Manager
- Azure Hybrid Benefit
- Azure VMware Solution

---

## Modern Compute

- Azure Virtual Desktop
- Azure Batch
- Azure Functions
- Azure Container Instances
- Azure Container Apps
- Azure Kubernetes Service

---

## Platform Services

- Azure Spring Apps
- Azure App Service
- Azure Dev Box
- Azure Deployment Environments

---

## Enterprise Compute Extensions

- Azure Elastic SAN
- Azure Chaos Studio

---

## Final Enterprise Architecture

- Complete Azure Enterprise Compute Design
- Landing Zone Integration
- Hub and Spoke Architecture
- Multi-Region Design
- Disaster Recovery
- Production Operations

---

# Teaching Methodology

Every topic follows the same enterprise learning model.

## Section 1

Overview

- What
- Why
- History
- Enterprise Adoption
- Design Goals
- Benefits
- Limitations

---

## Section 2

AWS Comparison

- Equivalent AWS Service
- Feature Mapping
- Differences
- Migration Considerations
- Pricing
- Performance
- Security
- Operations

---

## Section 3

Architecture

Includes

- Internal Components
- Request Flow
- Backend Services
- Storage
- Networking
- Identity
- Monitoring
- Diagnostics

ASCII architecture diagrams are included.

---

## Section 4

Enterprise Implementation

Includes

Development

QA

UAT

Production

Landing Zones

Hub-Spoke

Governance

Naming Standards

Tagging

Subscriptions

Management Groups

Policies

Cost Management

---

## Section 5

Azure Portal

Step-by-step

Every option explained

Recommended enterprise settings

Hidden settings

Common mistakes

---

## Section 6

Azure CLI

Production-ready scripts

Verification

Cleanup

Automation

---

## Section 7

Azure PowerShell

Enterprise PowerShell

Automation examples

---

## Section 8

Terraform

Production Module Design

Variables

Outputs

Dependencies

Remote State

Enterprise Folder Structure

---

## Section 9

Bicep

Enterprise Bicep

Modules

Reusable Templates

---

## Section 10

ARM Templates

Enterprise ARM

ARM vs Terraform vs Bicep

---

## Section 11

REST APIs

Authentication

Bearer Tokens

Microsoft Graph (where applicable)

ARM REST APIs

Examples

---

## Section 12

SDK Examples

Python

C#

Java

Go

---

## Section 13

Networking

Virtual Network

Subnet

NSG

Route Table

Private Endpoint

Service Endpoint

Firewall

Load Balancer

Application Gateway

Front Door

ExpressRoute

VPN Gateway

Bastion

Private DNS

DDoS Protection

---

## Section 14

Storage

Managed Disks

Snapshots

Images

Compute Gallery

Storage Accounts

Performance

Encryption

Caching

---

## Section 15

Identity

Managed Identity

Entra ID

RBAC

PIM

Conditional Access

Authentication

Authorization

---

## Section 16

Monitoring

Azure Monitor

Log Analytics

Activity Logs

Diagnostic Settings

Metrics

Alerts

Action Groups

Application Insights

Workbooks

---

## Section 17

Security

Microsoft Defender for Cloud

Key Vault

JIT

Disk Encryption

Encryption at Host

CMK

Compliance

Security Best Practices

---

## Section 18

High Availability

Availability Sets

Availability Zones

VM Scale Sets

Load Balancer

Geo-Redundancy

Failover

Disaster Recovery

---

## Section 19

Performance

Sizing

CPU

Memory

IOPS

Disk

Caching

Burst

Optimization

---

## Section 20

Cost Optimization

Pricing

Reserved Instances

Savings Plans

Spot VMs

Automation

Cost Analysis

Right Sizing

---

## Section 21

Production Troubleshooting

Enterprise Incidents

Examples include

- SSH Failure
- RDP Failure
- VM Unreachable
- Boot Diagnostics
- Extension Failure
- Disk Latency
- High CPU
- Managed Identity Failure
- DNS Issues
- Storage Problems
- Backup Failures

---

## Section 22

Interview Questions

Microsoft Interview Style

Basic

Intermediate

Advanced

Architecture

Scenario-Based

---

## Section 23

Hands-on Labs

Portal

CLI

PowerShell

Terraform

Verification

Cleanup

Expected Results

Estimated Cost

---

## Section 24

Enterprise Case Studies

Healthcare

Finance

Retail

Government

Manufacturing

Media

AI

DevOps

---

## Section 25

Microsoft Well-Architected Framework

Reliability

Security

Performance

Operational Excellence

Cost Optimization

---

## Section 26

Cheat Sheet

Architecture Summary

Decision Matrix

Comparison Tables

Best Practices

---

## Section 27

Certification Mapping

AZ-104

AZ-305

AZ-400

SC-100

Important Exam Objectives

---

# Repository Structure

```
Azure-Compute-Master-Guide
│
├── README.md
│
├── 01-Azure-Global-Infrastructure.md
├── 02-Regions.md
├── 03-Region-Pairs.md
├── 04-Availability-Zones.md
├── 05-Availability-Sets.md
├── 06-Resource-Groups.md
├── 07-Compute-Overview.md
├── 08-Azure-Virtual-Machines.md
├── ...
├── 60-End-to-End-Enterprise-Architecture.md
│
├── diagrams
├── terraform
├── bicep
├── arm
├── cli
├── powershell
├── sdk
├── labs
├── interview
├── troubleshooting
└── images
```

---

# Expected Outcome

After completing this guide you will be able to:

- Design enterprise Azure compute platforms.
- Migrate AWS workloads to Azure.
- Build production-ready landing zones.
- Deploy Azure infrastructure using Terraform and Bicep.
- Operate Azure environments at enterprise scale.
- Troubleshoot complex Azure compute issues.
- Pass Microsoft Azure certifications.
- Succeed in senior Azure Engineer and Azure Solutions Architect interviews.
- Design highly available, secure, scalable, and cost-optimized Azure compute architectures for Fortune 500 organizations.

---

# Prerequisites

- Experience with AWS (EC2, Auto Scaling, ELB, IAM, VPC, EBS, CloudWatch, CloudFormation)
- Basic Linux administration
- Basic Windows Server administration
- Terraform fundamentals
- Networking fundamentals
- Identity and access management concepts

---

# Recommended Lab Environment

- Azure Pay-As-You-Go or Azure for Students subscription
- Visual Studio Code
- Azure CLI
- Azure PowerShell
- Terraform
- Bicep CLI
- Git
- Docker Desktop (optional)
- Windows Terminal

---

# Study Strategy

For each chapter:

1. Read the theory.
2. Compare Azure with AWS.
3. Draw the architecture.
4. Deploy via the Azure Portal.
5. Deploy via Azure CLI.
6. Deploy via PowerShell.
7. Deploy via Terraform.
8. Deploy via Bicep.
9. Verify resources.
10. Monitor and troubleshoot.
11. Optimize cost and security.
12. Complete the hands-on lab.
13. Answer interview questions.
14. Explain the topic back in your own words before moving to the next chapter.
