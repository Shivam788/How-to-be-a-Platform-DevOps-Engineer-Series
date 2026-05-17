# Security and Governance

This document explains how security and governance are applied across the platform.

The design assumes a regulated enterprise environment where compliance, auditability, and controlled access are essential.

## Security principles

The platform follows these principles:
- private by default,
- least privilege access,
- policy-driven enforcement,
- no unnecessary public exposure,
- and strong separation of responsibilities.

## Identity and access control

Use RBAC:
- platform administrators,
- security administrators,
- application owners,
- and deployment identities.

Access should be limited to the smallest scope required for the task.

## Network security

The platform should include:
- isolated subnets,
- firewall inspection,
- private endpoints,
- NSGs,
- and restricted inbound and outbound traffic.

This reduces exposure and helps prevent lateral movement.

## Azure Policy

Azure Policy should be used to enforce rules that should not depend on manual review.

Typical policies include:
- deny public endpoints,
- require approved regions,
- enforce tagging,
- require encryption,
- and restrict unsupported SKUs.

## Secrets and certificates

Secrets should never be hardcoded in:
- source code,
- pipelines,
- or plain-text configuration files.

Use a secure secret management strategy, typically based on Key Vault and identity-based access.

## Compliance model

For finance or other regulated environments, governance must support:
- audit readiness,
- traceability,
- controlled deployments,
- and prevention of unsafe changes.

The goal is not only to detect violations, but to stop them before they reach production.

## Logging and monitoring

Security controls must be observable.

Make sure logging is enabled for:
- firewall activity,
- policy events,
- access changes,
- pipeline activity,
- and workload health.

## Why this matters

Security and governance are not separate from platform engineering.  
They are part of the platform design itself.

If they are added too late, the result is usually friction, exceptions, and rework.
