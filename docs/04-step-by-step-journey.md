# Step-by-Step Journey

This document explains the build in sequence.

The goal is to show how to move from planning to a working platform in a controlled and production-ready way.

## Step 1: Understand the target outcome

Start by defining:
- business goals,
- security expectations,
- compliance needs,
- availability targets,
- and workload types.

This helps you avoid building technology for its own sake.

## Step 2: Design the landing zone

Create the platform foundation first:
- management groups,
- subscriptions,
- policies,
- access control,
- and logging.

This becomes the control plane for everything else.

## Step 3: Build the hub network

Deploy the hub to host shared services:
- Azure Firewall,
- private DNS,
- monitoring,
- routing,
- and secure admin access.

The hub is the central trust boundary for the platform.

## Step 4: Create spoke networks

Build isolated spokes for workloads and teams.

Each spoke should have a clear purpose and a defined boundary.  
This keeps the environment scalable and easier to govern.

## Step 5: Add routing and security controls

Configure:
- UDRs,
- NSGs,
- firewall inspection,
- and private connectivity.

This ensures traffic is controlled and observable.

## Step 6: Deploy OpenShift

Deploy Azure Red Hat OpenShift in a private and production-ready pattern.

Focus on:
- private endpoints,
- secure ingress and egress,
- identity integration,
- and region planning.

## Step 7: Automate with Terraform

Break the platform into reusable modules.

Recommended modules:
- hub network,
- spoke network,
- policy,
- OpenShift,
- monitoring,
- security.

## Step 8: Integrate Azure DevOps pipelines

Use pipelines for:
- validation,
- formatting,
- planning,
- approval,
- and deployment.

The pipeline should protect the platform, not just push changes faster.

## Step 9: Apply governance

Assign policies and RBAC after the core design is in place.

This prevents teams from creating non-compliant or unsafe resources.

## Step 10: Validate production readiness

Before considering the platform complete, test:
- failover,
- DNS resolution,
- private connectivity,
- logging,
- deployment paths,
- and recovery procedures.

## Why this sequence matters

Building enterprise infrastructure in the correct order reduces risk.

If you start with applications before the foundation is ready, you usually end up with security gaps, technical debt and inconsistent operations.

# Azure Landing Zone Playbook for Multi-Region OpenShift

This document is a practical playbook for designing and building an Azure landing zone for multi-region, multi-cluster OpenShift environments.

It is written for engineers who want to understand:
- what to do,
- why it matters,
- and how to implement it in a production-ready way.

The guidance below follows an enterprise solution architect mindset and is intended to help newcomers build the foundation correctly from scratch.

## Table of contents

- [Phase 1: Define the Foundation](#phase-1-define-the-foundation)
- [Phase 2: Build the Hub Network](#phase-2-build-the-hub-network)
- [Phase 3: Build Spoke Networks](#phase-3-build-spoke-networks)
- [Phase 4: Deploy Multi-Region OpenShift](#phase-4-deploy-multi-region-openshift)
- [Phase 5: Implement Governance and Security](#phase-5-implement-governance-and-security)
- [Phase 6: Set Up Terraform Remote State](#phase-6-set-up-terraform-remote-state)
- [Phase 7: Build Azure DevOps Pipelines](#phase-7-build-azure-devops-pipelines)
- [Phase 8: Cost Optimization and FinOps](#phase-8-cost-optimization-and-finops)
- [Phase 9: Observability and Monitoring](#phase-9-observability-and-monitoring)
- [Summary](#summary)

---

## Phase 1: Define the Foundation

### Step 1: Understand the business and technical requirements

Before writing any code or creating resources, gather the requirements.

#### Questions to answer
- How many teams will use this platform?
- What applications will run on it?
- What compliance rules apply?
- What availability targets are required?
- Which regions are needed?
- What connectivity model is expected?
- What cost constraints must be respected?

#### Output
Create a one-page requirements document that captures:
- workload types,
- compliance needs,
- availability targets,
- and target Azure regions.

[Reference design guidance](https://us.nttdata.com/en/blog/2021/june/design-principles-for-azure-landing-zones)

---

### Step 2: Choose the network topology

#### Decision: Hub-Spoke vs Virtual WAN

For most enterprise use cases, hub-spoke is a strong fit because:
- it is simpler to operate,
- it provides strong control over routing and security,
- and it scales well for multiple application landing zones.

#### Hub-spoke design
- **Hub VNet**: shared services such as Azure Firewall, Bastion, DNS, and monitoring.
- **Spoke VNets**: workload-specific networks for teams or applications.
- **VNet Peering**: connects hub and spokes with controlled routing.

#### IP address planning
Reserve a large address block and allocate it logically:
- Hub: `10.0.0.0/16`
- Spoke 1: `10.1.0.0/16`
- Spoke 2: `10.2.0.0/16`
- OpenShift Cluster 1: `10.10.0.0/16`
- OpenShift Cluster 2: `10.20.0.0/16`

#### Output
- Network diagram.
- IP address allocation table.

[Azure hub-spoke architecture guidance](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/hub-spoke)

---

### Step 3: Design the subscription and management group structure

#### Management group hierarchy

```text
Root Management Group
├── Platform (shared services)
│   ├── Connectivity (hub networking, firewall)
│   ├── Identity (Active Directory, DNS)
│   └── Management (monitoring, backup)
├── Landing Zones (workloads)
│   ├── Corp (internal apps)
│   ├── Online (internet-facing apps)
│   └── OpenShift (container platform)
└── Sandbox (dev/test, less strict policies)
```

#### Why this matters
- Policies applied at the management group level cascade down.
- Billing and cost tracking are easier per subscription.
- Each subscription can have different RBAC and governance boundaries.

#### Output
- Management group structure diagram.
- Subscription naming convention, for example:
  - `sub-corp-prod-001`
  - `sub-openshift-aro-eastus`

[Reference architecture note](https://azuresecurityarchitect.com/azure-landing-zone/building-an-azure-landing-zone-in-an-existing-hub-and-spoke-architecture/)

---

## Phase 2: Build the Hub Network

### Step 4: Provision the hub VNet and subnets

Create a Terraform module for the hub network.

#### Example

```hcl
# modules/hub-network/main.tf
resource "azurerm_virtual_network" "hub" {
  name                = var.hub_vnet_name
  location            = var.location
  resource_group_name = var.resource_group_name
  address_space       = ["10.0.0.0/16"]
}

resource "azurerm_subnet" "firewall" {
  name                 = "AzureFirewallSubnet"
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.1.0/26"]
}

resource "azurerm_subnet" "gateway" {
  name                 = "GatewaySubnet"
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.2.0/27"]
}

resource "azurerm_subnet" "bastion" {
  name                 = "AzureBastionSubnet"
  resource_group_name  = var.resource_group_name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.3.0/27"]
}
```

#### Why
- Azure Firewall and Gateway subnets require specific names.
- Bastion enables secure access without exposing public IPs on VMs.

[Hub-spoke subnet and networking guidance](https://learn.microsoft.com/en-us/azure/architecture/networking/architecture/hub-spoke)

---

### Step 5: Deploy Azure Firewall for centralized egress control

#### Example

```hcl
# modules/hub-network/firewall.tf
resource "azurerm_firewall" "hub" {
  name                = var.firewall_name
  location            = var.location
  resource_group_name = var.resource_group_name
  sku_name            = "AZFW_VNet"
  sku_tier            = "Standard"

  ip_configuration {
    name                 = "firewall-ipconfig"
    subnet_id            = azurerm_subnet.firewall.id
    public_ip_address_id = azurerm_public_ip.firewall.id
  }
}

resource "azurerm_firewall_policy" "hub" {
  name                = "${var.firewall_name}-policy"
  resource_group_name = var.resource_group_name
  location            = var.location
}
```

#### Why
- All outbound traffic from spokes can be inspected centrally.
- Logging becomes consistent.
- Data exfiltration risk is reduced.

[Network segmentation guidance](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-landing-zone-network-segmentation)

---

### Step 6: Configure User-Defined Routes to force traffic through the firewall

#### Example

```hcl
resource "azurerm_route_table" "spoke" {
  name                = "rt-spoke-to-hub"
  location            = var.location
  resource_group_name = var.resource_group_name

  route {
    name                   = "default-route"
    address_prefix         = "0.0.0.0/0"
    next_hop_type          = "VirtualAppliance"
    next_hop_in_ip_address = azurerm_firewall.hub.ip_configuration.private_ip_address
  }
}
```

#### Why
This forces internet-bound traffic from spoke networks through Azure Firewall for inspection.

[Network segmentation guidance](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-landing-zone-network-segmentation)

---

## Phase 3: Build Spoke Networks

### Step 7: Create spoke VNets with peering to the hub

#### Example

```hcl
# modules/spoke-network/main.tf
resource "azurerm_virtual_network" "spoke" {
  name                = var.spoke_vnet_name
  location            = var.location
  resource_group_name = var.resource_group_name
  address_space       = [var.address_space]
}

resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  name                      = "peer-spoke-to-hub"
  resource_group_name       = var.resource_group_name
  virtual_network_name      = azurerm_virtual_network.spoke.name
  remote_virtual_network_id = var.hub_vnet_id
  allow_forwarded_traffic   = true
  allow_gateway_transit     = false
  use_remote_gateways       = true
}

resource "azurerm_virtual_network_peering" "hub_to_spoke" {
  name                      = "peer-hub-to-spoke"
  resource_group_name       = var.hub_resource_group_name
  virtual_network_name      = var.hub_vnet_name
  remote_virtual_network_id = azurerm_virtual_network.spoke.id
  allow_forwarded_traffic   = true
  allow_gateway_transit     = true
}
```

#### Why
- Spokes are isolated by default.
- Inter-spoke communication can be controlled and inspected.

[Network segmentation guidance](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-landing-zone-network-segmentation)

---

### Step 8: Apply Network Security Groups for subnet-level micro-segmentation

#### Example

```hcl
resource "azurerm_network_security_group" "app_subnet" {
  name                = "nsg-app-subnet"
  location            = var.location
  resource_group_name = var.resource_group_name

  security_rule {
    name                       = "deny-internet-inbound"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Deny"
    protocol                   = "*"
    source_port_range          = "*"
    destination_port_range     = "*"
    source_address_prefix      = "Internet"
    destination_address_prefix = "*"
  }

  security_rule {
    name                       = "allow-https-from-app-gateway"
    priority                   = 110
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "443"
    source_address_prefix      = "10.0.4.0/24"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "app" {
  subnet_id                 = azurerm_subnet.app.id
  network_security_group_id = azurerm_network_security_group.app_subnet.id
}
```

#### Why
NSGs add defense-in-depth and help reduce lateral movement.

[Network segmentation guidance](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-landing-zone-network-segmentation)

---

## Phase 4: Deploy Multi-Region OpenShift

### Step 9: Provision ARO clusters in multiple regions

#### Example

```hcl
# modules/aro-cluster/main.tf
resource "azurerm_redhat_openshift_cluster" "aro" {
  name                = var.cluster_name
  location            = var.location
  resource_group_name = var.resource_group_name

  cluster_profile {
    domain            = var.domain
    version           = "4.14.0"
    resource_group_id = var.cluster_resource_group_id
  }

  network_profile {
    pod_cidr     = "10.128.0.0/14"
    service_cidr = "172.30.0.0/16"
  }

  api_server_profile {
    visibility = "Private"
  }

  ingress_profile {
    visibility = "Private"
  }

  master_profile {
    subnet_id = var.master_subnet_id
    vm_size   = "Standard_D8s_v3"
  }

  worker_profile {
    subnet_id    = var.worker_subnet_id
    vm_size      = "Standard_D4s_v3"
    disk_size_gb = 128
    node_count   = 3
  }

  service_principal {
    client_id     = var.sp_client_id
    client_secret = var.sp_client_secret
  }
}
```

#### Why
- Private API and ingress reduce exposure.
- Multi-region deployment improves resilience and failover readiness.

[Multi-region platform guidance](https://learn.microsoft.com/en-us/azure/aks/reliability-multi-region-deployment-models)

---

### Step 10: Configure private DNS zones

#### Example

```hcl
resource "azurerm_private_dns_zone" "openshift_api" {
  name                = "api.${var.domain}.openshift.com"
  resource_group_name = var.resource_group_name
}

resource "azurerm_private_dns_zone_virtual_network_link" "hub" {
  name                  = "link-to-hub"
  resource_group_name   = var.resource_group_name
  private_dns_zone_name = azurerm_private_dns_zone.openshift_api.name
  virtual_network_id    = var.hub_vnet_id
}
```

#### Why
Private DNS ensures internal services can resolve private OpenShift endpoints correctly.

---

### Step 11: Set up Traffic Manager for regional failover

#### Example

```hcl
resource "azurerm_traffic_manager_profile" "openshift" {
  name                  = "tm-openshift-global"
  resource_group_name   = var.resource_group_name
  traffic_routing_method = "Priority"

  dns_config {
    relative_name = "openshift-global"
    ttl           = 30
  }

  monitor_config {
    protocol = "HTTPS"
    port     = 443
    path     = "/healthz"
  }
}

resource "azurerm_traffic_manager_endpoint" "primary" {
  name                = "aro-eastus"
  resource_group_name = var.resource_group_name
  profile_name        = azurerm_traffic_manager_profile.openshift.name
  type                = "azureEndpoints"
  target_resource_id  = azurerm_public_ip.aro_ingress_eastus.id
  priority            = 1
}

resource "azurerm_traffic_manager_endpoint" "secondary" {
  name                = "aro-westus"
  resource_group_name = var.resource_group_name
  profile_name        = azurerm_traffic_manager_profile.openshift.name
  type                = "azureEndpoints"
  target_resource_id  = azurerm_public_ip.aro_ingress_westus.id
  priority            = 2
}
```

#### Why
Traffic Manager provides DNS-based failover across regions.

[Multi-region disaster recovery reference](https://www.linkedin.com/pulse/designing-multi-region-disaster-recovery-architecture-yasir-imran-ljqpf)

---

## Phase 5: Implement Governance and Security

### Step 12: Deploy custom Azure Policies for compliance

#### Example: deny public storage access

```hcl
# modules/azure-policy/deny-public-storage.tf
resource "azurerm_policy_definition" "deny_public_storage" {
  name         = "deny-public-storage-accounts"
  policy_type  = "Custom"
  mode         = "All"
  display_name = "Deny Storage Accounts with Public Access"

  policy_rule = jsonencode({
    if = {
      allOf = [
        {
          field  = "type"
          equals = "Microsoft.Storage/storageAccounts"
        },
        {
          field  = "Microsoft.Storage/storageAccounts/allowBlobPublicAccess"
          equals = "true"
        }
      ]
    }
    then = {
      effect = "deny"
    }
  })
}

resource "azurerm_policy_assignment" "deny_public_storage" {
  name                 = "assign-deny-public-storage"
  scope                = var.management_group_id
  policy_definition_id = azurerm_policy_definition.deny_public_storage.id
}
```

#### Other useful policies
- Require approved regions.
- Enforce required tags.
- Deny public IPs on sensitive workloads.
- Require encryption in transit.

[Azure Policy reference](https://learn.microsoft.com/en-us/azure/governance/policy/overview)

---

### Step 13: Configure RBAC at subscription and resource group level

#### Example

```hcl
resource "azurerm_role_assignment" "platform_team" {
  scope                = var.subscription_id
  role_definition_name = "Contributor"
  principal_id         = var.platform_team_group_id
}

resource "azurerm_role_assignment" "app_team_reader" {
  scope                = azurerm_resource_group.spoke.id
  role_definition_name = "Reader"
  principal_id         = var.app_team_group_id
}
```

#### Why
- Platform team manages infrastructure.
- App teams get only the access they need.

---

## Phase 6: Set Up Terraform Remote State

### Step 14: Create Azure Storage Account for state

#### Example

```hcl
# bootstrap/backend.tf
resource "azurerm_storage_account" "tfstate" {
  name                     = "sttfstate${var.environment}"
  resource_group_name      = var.resource_group_name
  location                 = var.location
  account_tier             = "Standard"
  account_replication_type = "GRS"

  blob_properties {
    versioning_enabled = true
  }
}

resource "azurerm_storage_container" "tfstate" {
  name                 = "tfstate"
  storage_account_name = azurerm_storage_account.tfstate.name
  container_access_type = "private"
}
```

#### Why
- Remote state supports collaboration.
- State locking prevents conflicts.
- Versioning improves recoverability.

[Remote state discussion](https://medium.com/@aderixxe/managing-terraform-state-remotely-azure-aws-backends-128c4a03ca08)

---

### Step 15: Configure backend in each module

#### Example

```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "sttfstateprod"
    container_name       = "tfstate"
    key                  = "hub-network.tfstate"
  }
}
```

#### Initialize

```bash
terraform init \
  -backend-config="resource_group_name=rg-terraform-state" \
  -backend-config="storage_account_name=sttfstateprod" \
  -backend-config="container_name=tfstate" \
  -backend-config="key=hub-network.tfstate"
```

#### Why
Separate state files reduce blast radius and make the platform easier to operate.

[Terraform remote state guidance](https://www.reddit.com/r/Terraform/comments/koz2ey/best_practice_for_remote_state_file_storage_in/)

---

## Phase 7: Build Azure DevOps Pipelines

### Step 16: Create YAML pipeline for Terraform deployment

#### Example

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main
  paths:
    include:
      - terraform/hub-network/**

pool:
  vmImage: 'ubuntu-latest'

variables:
  - group: terraform-vars

stages:
  - stage: Plan
    jobs:
      - job: TerraformPlan
        steps:
          - task: TerraformInstaller@0
            inputs:
              terraformVersion: '1.9.0'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Init'
            inputs:
              provider: 'azurerm'
              command: 'init'
              workingDirectory: '$(System.DefaultWorkingDirectory)/terraform/hub-network'
              backendServiceArm: 'sp-terraform-backend'
              backendAzureRmResourceGroupName: 'rg-terraform-state'
              backendAzureRmStorageAccountName: 'sttfstateprod'
              backendAzureRmContainerName: 'tfstate'
              backendAzureRmKey: 'hub-network.tfstate'

          - task: TerraformTaskV4@4
            displayName: 'Terraform Plan'
            inputs:
              provider: 'azurerm'
              command: 'plan'
              workingDirectory: '$(System.DefaultWorkingDirectory)/terraform/hub-network'
              environmentServiceNameAzureRM: 'sp-terraform-deploy'
              commandOptions: '-out=tfplan'

          - task: PublishPipelineArtifact@1
            inputs:
              targetPath: '$(System.DefaultWorkingDirectory)/terraform/hub-network/tfplan'
              artifact: 'tfplan'

  - stage: Apply
    dependsOn: Plan
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: TerraformApply
        environment: 'production'
        strategy:
          runOnce:
            deploy:
              steps:
                - download: current
                  artifact: tfplan

                - task: TerraformTaskV4@4
                  displayName: 'Terraform Apply'
                  inputs:
                    provider: 'azurerm'
                    command: 'apply'
                    workingDirectory: '$(Pipeline.Workspace)/tfplan'
                    environmentServiceNameAzureRM: 'sp-terraform-deploy'
                    commandOptions: 'tfplan'
```

#### Why
- Plans run before changes are applied.
- Production deployments require approval and control.
- Backend and deploy identities are separated.

---

### Step 17: Manage service principal expiry and rotation

#### Check expiry

```bash
az ad sp credential list --id <sp-object-id> --query "[].endDateTime" -o table
```

#### Example rotation workflow

```yaml
# rotate-sp-secret.yml
trigger: none

schedules:
  - cron: "0 0 1 * *"
    displayName: Rotate SP Secret
    branches:
      include:
        - main

steps:
  - task: AzureCLI@2
    inputs:
      azureSubscription: 'sp-admin'
      scriptType: 'bash'
      scriptLocation: 'inlineScript'
      inlineScript: |
        NEW_SECRET=$(az ad sp credential reset --id $(SP_CLIENT_ID) --query password -o tsv)
        echo "##vso[task.setvariable variable=NEW_SECRET;issecret=true]$NEW_SECRET"

  - task: AzureCLI@2
    inputs:
      scriptType: 'bash'
      scriptLocation: 'inlineScript'
      inlineScript: |
        az keyvault secret set --vault-name kv-platform-prod \
          --name sp-terraform-secret \
          --value "$(NEW_SECRET)"
```

#### Why
Service principal secrets expire and must be managed carefully to avoid pipeline failures.

---

## Phase 8: Cost Optimization and FinOps

### Step 18: Implement Azure Cost Management budgets

#### Example

```hcl
resource "azurerm_consumption_budget_subscription" "platform" {
  name            = "budget-platform-monthly"
  subscription_id = var.subscription_id

  amount     = 50000
  time_grain = "Monthly"

  time_period {
    start_date = "2026-06-01T00:00:00Z"
  }

  notification {
    enabled        = true
    threshold      = 80
    operator       = "GreaterThan"
    contact_emails = ["platform-team@company.com"]
  }

  notification {
    enabled        = true
    threshold      = 100
    operator       = "GreaterThan"
    contact_emails = ["platform-team@company.com", "finance@company.com"]
  }
}
```

#### Why
Budgets provide early warning before overspend becomes a problem.

---

### Step 19: Right-size workloads and implement auto-shutdown

#### Example

```hcl
resource "azurerm_dev_test_global_vm_shutdown_schedule" "nonprod" {
  virtual_machine_id   = azurerm_linux_virtual_machine.nonprod.id
  location             = var.location
  enabled              = true
  daily_recurrence_time = "1900"
  timezone             = "UTC"

  notification_settings {
    enabled = false
  }
}
```

#### Why
Non-production resources should not run 24/7 unless they need to.

---

## Phase 9: Observability and Monitoring

### Step 20: Deploy Azure Monitor and Log Analytics

#### Example

```hcl
resource "azurerm_log_analytics_workspace" "platform" {
  name                = "law-platform-prod"
  location            = var.location
  resource_group_name = var.resource_group_name
  sku                 = "PerGB2018"
  retention_in_days   = 30
}

resource "azurerm_monitor_diagnostic_setting" "firewall" {
  name                       = "diag-firewall"
  target_resource_id         = azurerm_firewall.hub.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.platform.id

  enabled_log {
    category = "AzureFirewallApplicationRule"
  }

  enabled_log {
    category = "AzureFirewallNetworkRule"
  }

  metric {
    category = "AllMetrics"
  }
}
```

#### Why
Centralised logging improves compliance, troubleshooting, and operational visibility.

---

## Summary

This platform includes:

| Component | Purpose |
|---|---|
| Hub-Spoke VNets | Network isolation and shared services |
| Azure Firewall | Centralized egress control |
| Private Endpoints | Secure private access |
| Multi-region ARO | High availability and failover |
| Custom Azure Policies | Compliance enforcement |
| Modular Terraform | Reusable infrastructure code |
| Remote State | Collaboration and auditability |
| Azure DevOps Pipelines | Automated delivery |
| Cost Monitoring | Budget control and optimization |
| Centralized Logging | Security and operational visibility |

## Key enterprise principles applied

- Security by default.
- Least privilege access.
- Repeatability through automation.
- Resilience through multi-region design.
- Cost governance through visibility and standards.
- Observability through centralised logging.

## Next step

Choose the phase you want to implement first, then build the foundation in the correct order.
