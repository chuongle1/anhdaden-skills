---
name: cloud-kubernetes-security
description: Assess cloud infrastructure and Kubernetes security across identity, network, workloads, control planes, and delivery pipelines.
---

# Cloud and Kubernetes Security

## Scope
Identify provider, accounts/projects/subscriptions, regions, clusters, namespaces, workloads, identity boundaries, environments, and permitted read/write actions. Prefer read-only posture review. Confirm approval before changing live policies or resources.

## Cloud control plane
- IAM: least privilege, federation, role chaining, service identities, key age/rotation, wildcard permissions.
- Network: public exposure, segmentation, security groups/firewalls, private endpoints, egress controls.
- Storage/data: public buckets, encryption, access logging, backup, retention, KMS policy.
- Compute: image provenance, patching, metadata service protections, hardened images.
- Audit: control-plane logs, retention, immutable storage, alert coverage.
- Secrets: managed secret stores, workload identity, rotation, no secrets in user data or images.

## Kubernetes
- API server: authentication, authorization/RBAC, network reachability, audit policy.
- Workloads: non-root, read-only root FS, seccomp, capabilities drop, resource limits, probes.
- Pod Security: enforce restricted baseline where feasible; document exceptions.
- NetworkPolicy: default-deny and explicit service flows; verify CNI enforcement.
- Secrets: encryption at rest, external secret integration, rotation, access scope.
- Supply chain: signed images, digest pinning, trusted registries, admission policies, SBOM.
- Runtime: detect exec/shell, unexpected process, host mounts, privileged pods, host namespaces.
- Nodes: patching, metadata access, kubelet exposure, container runtime, node isolation.
- Multi-tenancy: namespace boundaries, quotas, RBAC, network and secret isolation.

## IaC review
Inspect Terraform/CloudFormation/Helm/Kustomize manifests for public exposure, wildcard IAM, unencrypted data, unsafe defaults, secret literals, missing policies, drift, and destructive plans. Never apply a plan without explicit approval.

## Validation
Use provider-native read-only tools and policy scanners. Record account/cluster context and tool version. Avoid dumping secret values; inspect metadata and access paths instead.

## Output
Asset map, identity/network findings, workload posture, policy/IaC findings, attack paths, prioritized remediation, and validation commands.

## Shared standards
Follow the repository-level `CLAUDE.md` for authorization, evidence discipline, privacy, and reporting. If it is unavailable, apply these minimum rules: verify scope before active testing; prefer non-destructive validation; distinguish facts from hypotheses; never invent tool output; redact secrets; report limitations.
