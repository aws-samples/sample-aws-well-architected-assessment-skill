# ElastiCache Service Checks

Reference: service-screener-v2 `services/elasticache/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Check: ELASTICACHE_001 — Encryption at Rest
- **API**: `aws elasticache describe-cache-clusters` / `describe-replication-groups` → `.AtRestEncryptionEnabled`
- **Pass**: `AtRestEncryptionEnabled == true`
- **Fail**: `AtRestEncryptionEnabled == false`
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: ELASTICACHE_002 — Encryption in Transit
- **API**: `aws elasticache describe-replication-groups` → `.TransitEncryptionEnabled`
- **Pass**: `TransitEncryptionEnabled == true`
- **Fail**: `TransitEncryptionEnabled == false`
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: ELASTICACHE_003 — Multi-AZ / Automatic Failover
- **API**: `aws elasticache describe-replication-groups` → `.AutomaticFailover`
- **Pass**: `AutomaticFailover == "enabled"`
- **Fail**: `AutomaticFailover == "disabled"`
- **Severity**: 🟡 MEDIUM | **Pillar**: Reliability

## Check: ELASTICACHE_004 — Engine Version Currency
- **API**: `aws elasticache describe-cache-clusters` → `.EngineVersion`
- **Pass**: Running supported, current engine version
- **Fail**: Running outdated or deprecated version
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: ELASTICACHE_005 — Backup Enabled
- **API**: `aws elasticache describe-replication-groups` → `.SnapshotRetentionLimit`
- **Pass**: `SnapshotRetentionLimit > 0`
- **Fail**: `SnapshotRetentionLimit == 0`
- **Severity**: 🟡 MEDIUM | **Pillar**: Reliability

## Check: ELASTICACHE_006 — Node Type Currency
- **API**: `aws elasticache describe-cache-clusters` → `.CacheNodeType`
- **Pass**: The node type is confirmed current generation in current ElastiCache documentation.
- **Fail**: The node type is confirmed previous generation; this is an optimization opportunity, not proof of unsupported status.
- **Not assessed**: Generation status or workload requirements cannot be established. Do not infer status from the family name alone.
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Performance Efficiency, Cost Optimization

## Check: ELASTICACHE_007 — Auth Token (Redis AUTH)
- **API**: `aws elasticache describe-replication-groups` → `.AuthTokenEnabled`
- **Pass**: `AuthTokenEnabled == true`
- **Fail**: `AuthTokenEnabled == false`
- **Severity**: 🟡 MEDIUM | **Pillar**: Security
