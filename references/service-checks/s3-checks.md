# S3 Service Checks

Reference: service-screener-v2 `services/s3/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Check: S3_001 — Public Access Block
- **API**: `aws s3control get-public-access-block` (account level) + `aws s3api get-public-access-block` (bucket level)
- **Pass**: All four public access block settings enabled
- **Fail**: Any public access block setting disabled
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Enable all public access block settings at account and bucket level.

## Check: S3_002 — Default Encryption
- **API**: `aws s3api get-bucket-encryption`
- **Pass**: The returned default encryption satisfies the documented workload requirement.
- **Fail**: The configuration does not meet an explicitly established encryption requirement, such as a required customer-managed KMS key.
- **Not assessed**: The call fails or the requirement cannot be determined. S3 encrypts new uploads by default; do not infer that existing objects satisfy a stronger requirement from the bucket default.
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Review the applicable encryption and key-management requirements and existing objects.

## Check: S3_003 — Versioning
- **API**: `aws s3api get-bucket-versioning`
- **Pass**: `Status == "Enabled"`
- **Fail**: Versioning not enabled or suspended
- **Severity**: 🟡 MEDIUM
- **Pillar**: Reliability
- **Recommendation**: Enable versioning for data protection and recovery.

## Check: S3_004 — Lifecycle Policy
- **API**: `aws s3api get-bucket-lifecycle-configuration`
- **Pass**: Lifecycle rules configured
- **Fail**: No lifecycle configuration
- **Severity**: 🟡 MEDIUM
- **Pillar**: Cost Optimization
- **Recommendation**: Configure lifecycle rules to transition/expire objects.

## Check: S3_005 — Access Logging
- **API**: `aws s3api get-bucket-logging`
- **Pass**: Server access logging enabled
- **Fail**: No logging configured
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security, Operational Excellence
- **Recommendation**: Enable server access logging for audit trail.

## Check: S3_006 — Public Bucket Policy
- **API**: `aws s3api get-bucket-policy-status` → `.PolicyStatus.IsPublic`; use `get-bucket-policy` when needed to investigate the policy, subject to the safe-output rules.
- **Pass**: S3 reports the bucket policy as non-public. This applies only to the policy's public classification; it does not establish least privilege or rule out access through ACLs, access points, or other paths.
- **Fail**: S3 reports the bucket policy as public. Report this as a public-policy finding, not proof that the bucket or its objects are reachable: block public access and other controls may restrict effective access.
- **Not applicable**: The service confirms that no bucket policy exists. Do not infer that other access paths are private.
- **Not assessed**: Access is denied, the status is unavailable or unsupported, or policy existence cannot be confirmed. A wildcard principal with a condition must not automatically pass; broad conditions can still be public.
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Review public grants and restrict the policy to the intended principals and conditions. Assess effective exposure separately.

## Check: S3_007 — SSL/TLS Enforcement
- **API**: `aws s3api get-bucket-policy`
- **Pass**: A policy explicitly denies insecure requests with `aws:SecureTransport` set to `false`, covering the relevant bucket and object actions and resources.
- **Fail**: The inspected bucket policy does not enforce the applicable transport requirement.
- **Not assessed**: The policy cannot be retrieved or the relevant request paths and scope are unknown. Merely finding the condition key does not establish an effective deny.
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security
- **Recommendation**: Add policy condition to deny non-HTTPS requests.

## Check: S3_008 — MFA Delete
- **API**: `aws s3api get-bucket-versioning` → `.MFADelete`
- **Pass**: MFA Delete enabled on critical buckets
- **Fail**: MFA Delete not enabled
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Security
- **Recommendation**: Enable MFA Delete for critical data buckets.

## Check: S3_009 — Intelligent Tiering
- **Evidence**: Review `aws s3api get-bucket-lifecycle-configuration` for transitions to the `INTELLIGENT_TIERING` storage class, together with approved, current aggregate storage-class usage evidence and workload access-pattern requirements. Do not retrieve object contents or object inventories as part of this check.
- **Optional API**: `aws s3api list-bucket-intelligent-tiering-configurations` lists optional archive-tier configurations, with pagination. If inspecting one configuration, `get-bucket-intelligent-tiering-configuration` requires its `--id`. These APIs describe optional archive tiers, not whether objects use Intelligent-Tiering.
- **Pass**: Evidence shows that relevant data uses, or an applicable lifecycle rule transitions it to, Intelligent-Tiering in a manner that satisfies the established workload requirements.
- **Fail**: Complete evidence shows that Intelligent-Tiering is not used or configured for the relevant data despite an established requirement to use it.
- **Not applicable**: The workload's documented access and cost requirements make Intelligent-Tiering unsuitable.
- **Not assessed**: Requirements, relevant usage, lifecycle scope, or permissions are insufficient to decide. No archive-tier configuration alone is not evidence that Intelligent-Tiering is unused; a large bucket alone does not establish that it is appropriate.
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Cost Optimization
- **Recommendation**: Evaluate Intelligent-Tiering against object sizes, access patterns, retrieval needs and costs before changing storage class or lifecycle rules.
