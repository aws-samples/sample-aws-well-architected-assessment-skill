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

## Check: S3_006 — Bucket Policy Overly Permissive
- **API**: `aws s3api get-bucket-policy`
- **Pass**: No `Principal: "*"` without conditions
- **Fail**: Policy allows public access via wildcard principal
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Restrict bucket policy to specific principals and conditions.

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
- **API**: `aws s3api get-bucket-intelligent-tiering-configuration`
- **Pass**: Intelligent tiering configured for variable access patterns
- **Fail**: No intelligent tiering on large buckets
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Cost Optimization
- **Recommendation**: Enable S3 Intelligent-Tiering for automatic cost savings.
