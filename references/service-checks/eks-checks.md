# EKS Service Checks

Reference: service-screener-v2 `services/eks/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Check: EKS_001 — Cluster Version Currency
- **API**: `aws eks describe-cluster` → `.version`
- **Pass**: Running a Kubernetes version supported under the cluster's applicable EKS support policy, verified against current AWS documentation.
- **Fail**: Running an end-of-support version
- **Severity**: 🔴 HIGH
- **Pillar**: Security, Operational Excellence
- **Recommendation**: Upgrade to a supported Kubernetes version.

## Check: EKS_002 — Control Plane Logging
- **API**: `aws eks describe-cluster` → `.logging.clusterLogging`
- **Pass**: All log types enabled (api, audit, authenticator, controllerManager, scheduler)
- **Fail**: Some or all log types disabled
- **Severity**: 🟡 MEDIUM
- **Pillar**: Operational Excellence, Security
- **Recommendation**: Enable all control plane log types for audit and troubleshooting.

## Check: EKS_003 — Private Endpoint Access
- **API**: `aws eks describe-cluster` → `.resourcesVpcConfig.endpointPrivateAccess`
- **Pass**: `endpointPrivateAccess == true`
- **Fail**: `endpointPrivateAccess == false`
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security
- **Recommendation**: Enable private endpoint access for cluster API server.

## Check: EKS_004 — Public Endpoint Restriction
- **API**: `aws eks describe-cluster` → `.resourcesVpcConfig.endpointPublicAccess`, `.publicAccessCidrs`
- **Pass**: Public access disabled OR restricted to specific CIDRs
- **Fail**: Public access enabled with `0.0.0.0/0`
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Restrict public endpoint to specific CIDRs or disable it.

## Check: EKS_005 — Secrets Encryption
- **API**: `aws eks describe-cluster` → `.version`, `.encryptionConfig`
- **Pass**: For EKS clusters running Kubernetes 1.28 or later, account for EKS default envelope encryption; verify any additional customer-managed-key requirement separately. For older versions, verify the applicable secrets encryption configuration.
- **Fail**: Sufficient evidence shows the cluster does not meet its applicable encryption requirement.
- **Not assessed**: Version, configuration, or the required key-management policy cannot be established.
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Review [EKS envelope encryption](https://docs.aws.amazon.com/eks/latest/userguide/enable-kms.html) and the workload's key-management requirements.

## Check: EKS_006 — Node Group Multi-AZ
- **API**: `aws eks describe-nodegroup` → `.subnets`
- **Pass**: Node group spans multiple AZs
- **Fail**: Node group in single AZ
- **Severity**: 🟡 MEDIUM
- **Pillar**: Reliability
- **Recommendation**: Distribute node groups across multiple AZs.

## Check: EKS_007 — Managed Node Groups
- **API**: `aws eks list-nodegroups`
- **Pass**: Using managed node groups
- **Fail**: Using self-managed nodes only
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Operational Excellence
- **Recommendation**: Use managed node groups for automated patching and updates.

## Check: EKS_008 — Add-on Version Currency
- **API**: `aws eks list-addons` + `describe-addon`
- **Pass**: Add-ons are up to date
- **Fail**: Add-ons running outdated versions
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security, Operational Excellence
- **Recommendation**: Update EKS add-ons to latest compatible versions.
