# EFS Service Checks

Reference: service-screener-v2 `services/efs/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Check: EFS_001 — Encryption at Rest
- **API**: `aws efs describe-file-systems` → `.Encrypted`
- **Pass**: `Encrypted == true`
- **Fail**: `Encrypted == false`
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Enable encryption at rest using KMS. Note: requires recreating the file system.

## Check: EFS_002 — Encryption in Transit
- **API**: `aws efs describe-file-system-policy` → evaluate the resource policy's effective `aws:SecureTransport` conditions for NFS client access.
- **Pass**: Evidence establishes that the file system policy enforces TLS for all applicable client access, such as an effective explicit deny of insecure access.
- **Fail**: Evidence establishes that an applicable client can access the file system without TLS.
- **Not assessed**: Policy access is denied or the effective access, principals or conditions cannot be determined. A mount target or TCP port 2049 alone does not prove TLS.
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security
- **Recommendation**: Enforce TLS for all clients in the file system policy and configure clients to use the EFS mount helper with TLS.

## Check: EFS_003 — Backup Policy
- **API**: `aws efs describe-backup-policy`
- **Pass**: `Status == "ENABLED"`
- **Fail**: Backup not enabled
- **Severity**: 🟡 MEDIUM
- **Pillar**: Reliability
- **Recommendation**: Enable automatic backups via AWS Backup.

## Check: EFS_004 — Lifecycle Policy
- **API**: `aws efs describe-lifecycle-configuration`
- **Pass**: Lifecycle policies configured (transition to IA)
- **Fail**: No lifecycle policy
- **Severity**: 🟡 MEDIUM
- **Pillar**: Cost Optimization
- **Recommendation**: Configure lifecycle policy to move infrequent data to IA storage class.

## Check: EFS_005 — File System Policy
- **API**: `aws efs describe-file-system-policy`
- **Pass**: The policy's effective permissions meet the documented workload requirements.
- **Fail**: Evidence establishes that effective access violates a documented requirement.
- **Not assessed**: Effective permissions or workload requirements cannot be established. A policy's mere existence does not prove secure access.
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security
- **Recommendation**: Configure file system policy to enforce encryption in transit and restrict access.

## Check: EFS_006 — Access Points
- **API**: `aws efs describe-access-points`
- **Pass**: Access points configured with POSIX user and root directory
- **Fail**: No access points (direct mount target access only)
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Security
- **Recommendation**: Use EFS access points for application-specific entry points with enforced user identity.
