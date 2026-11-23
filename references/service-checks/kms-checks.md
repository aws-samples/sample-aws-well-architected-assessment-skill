# KMS Service Checks

Reference: service-screener-v2 `services/kms/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Check: KMS_001 — Key Rotation Enabled
- **API**: `aws kms get-key-rotation-status` → `.KeyRotationEnabled`
- **Pass**: `KeyRotationEnabled == true` (for symmetric CMKs)
- **Fail**: `KeyRotationEnabled == false`
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: KMS_002 — Key Policy Restrictive
- **API**: `aws kms get-key-policy` → policy document
- **Pass**: Key policy restricts access to specific principals
- **Fail**: Key policy allows `"Principal": "*"` without conditions
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: KMS_003 — Key State
- **API**: `aws kms describe-key` → `.KeyMetadata.KeyState`
- **Pass**: `KeyState == "Enabled"`
- **Fail**: `KeyState == "PendingDeletion"` (review if intentional)
- **Severity**: 🟡 MEDIUM | **Pillar**: Operational Excellence

## Check: KMS_004 — Key Usage
- **API**: `aws kms describe-key` → `.KeyMetadata.KeyUsage`
- **Not assessed**: `KeyUsage` describes the permitted cryptographic operations, not whether or when a key was used. Review appropriate CloudTrail history and resource associations within the available retention window; absence of events alone does not prove a key is unused.
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Cost Optimization

## Check: KMS_005 — Grant Management
- **API**: `aws kms list-grants`
- **Pass**: Grants are scoped and reviewed
- **Fail**: Excessive or stale grants
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Security

## Check: KMS_006 — Multi-Region Key
- **API**: `aws kms describe-key` → `.KeyMetadata.MultiRegion`
- **Pass**: Multi-region keys used for cross-region workloads
- **Fail**: Single-region keys for multi-region data
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Reliability
