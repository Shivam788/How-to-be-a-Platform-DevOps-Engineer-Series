# Terraform and Automation

This document explains how the platform is automated using Terraform and Azure DevOps.

The goal is to make the platform repeatable, auditable, and maintainable across environments and teams.

## Terraform design approach

Use Terraform modules to keep responsibilities separated.

Recommended module boundaries:
- hub network,
- spoke network,
- policy,
- security,
- OpenShift,
- monitoring.

Each module should be small, predictable, and reusable.

## Why modules matter

Modules help you:
- avoid duplication,
- standardize implementation,
- reduce drift,
- and make changes easier to review.

## Remote state management

Use remote state for:
- collaboration,
- state locking,
- recoverability,
- and separation of concerns.

Separate state files should be used for different layers of the platform to reduce blast radius.

## Environment structure

Use clear environment separation:
- dev,
- test,
- prod.

Environment-specific values should be stored separately from reusable module logic.

## Azure DevOps integration

Use Azure DevOps pipelines for:
- code validation,
- linting,
- Terraform plan,
- approvals,
- and deployment.

The pipeline should follow a controlled promotion path.

## Secure automation practices

Use automation in a secure way:
- least privilege identities,
- protected service connections,
- secret storage outside the codebase,
- review gates for production,
- and policy checks before deployment.

## Validation steps

Before applying changes, validate:
- formatting,
- syntax,
- variable values,
- state access,
- and plan output.

Before promoting to production, confirm:
- approvals,
- policy compliance,
- and rollback readiness.

## Why this matters

Automation is most effective when it is consistent and safe.

In enterprise environments, fast delivery without control usually creates more work later.
