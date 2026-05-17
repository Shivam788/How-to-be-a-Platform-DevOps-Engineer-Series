# Deployment Prerequisites

This document lists the prerequisites required before building the platform.

The purpose is to make sure the environment is ready before any infrastructure is deployed. This avoids delays, rework, and avoidable errors during implementation.

## What should already be in place

Before starting, confirm the following:

- Azure subscription access with sufficient permissions.
- Management group structure defined.
- Naming standards and tagging strategy agreed.
- IP address ranges and subnet plan prepared.
- Azure DevOps project created.
- Terraform installed and validated.
- Remote state storage account available.
- Key Vault strategy defined for secrets and certificates.
- Logging and monitoring targets identified.
- Azure Red Hat OpenShift capacity and region planning completed.
- Security and compliance requirements documented.

## Identity and access prerequisites

You should know:
- who owns the platform,
- who can deploy infrastructure,
- who can approve production changes,
- and who can access the workloads.

Set up:
- least privilege RBAC,
- service connections for automation,
- and access boundaries between platform and application teams.

## Network prerequisites

Prepare the network design before building anything.

You need:
- hub address space,
- spoke address spaces,
- subnet layout,
- routing model,
- private DNS plan,
- and failover-aware region planning.

## Terraform prerequisites

Make sure the Terraform workflow is ready:
- Terraform version agreed.
- Remote state backend created.
- State locking enabled.
- Environment-specific variable files prepared.
- Modules are structured clearly.
- Code review process defined.

## Azure DevOps prerequisites

Your pipeline environment should include:
- a project,
- a repository,
- service connections,
- variable groups or key vault integration,
- approval gates,
- and a clear branching strategy.

## Validation before deployment

Before continuing, verify the following:
- subscriptions exist and are accessible,
- required providers are registered,
- permissions are correct,
- all IP ranges are documented,
- and the architecture has been reviewed.

## Why this matters

In enterprise environments, most deployment issues are caused by missing prerequisites rather than bad code.

If the foundation is not ready, the platform will fail later in ways that are harder to fix.
