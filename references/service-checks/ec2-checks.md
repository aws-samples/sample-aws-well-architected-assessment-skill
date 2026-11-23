# EC2 Service Checks

Reference: service-screener-v2 `services/ec2/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Check: EC2_001 — IMDSv2 Enforcement
- **API**: `aws ec2 describe-instances` → `.MetadataOptions.HttpTokens`
- **Pass**: `HttpTokens == "required"`
- **Fail**: `HttpTokens == "optional"`
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Enforce IMDSv2 to prevent SSRF-based credential theft.

## Check: EC2_002 — EBS Volume Encryption
- **API**: `aws ec2 describe-volumes` → `.Encrypted`
- **Pass**: `Encrypted == true`
- **Fail**: `Encrypted == false`
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Enable EBS encryption. Set default encryption in account settings.

## Check: EC2_003 — Public IP Assignment
- **API**: `aws ec2 describe-instances` → `.PublicIpAddress`
- **Pass**: No public IP or justified (web-facing)
- **Fail**: Public IP on non-web-facing instances
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security
- **Recommendation**: Use private subnets with NAT Gateway or ALB for internet access.

## Check: EC2_004 — Security Group Open Ingress
- **API**: `aws ec2 describe-security-groups` → `.IpPermissions`
- **Pass**: No rules with `0.0.0.0/0` on ports 22, 3389, 3306, 5432, 1433
- **Fail**: Open ingress on sensitive ports
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Restrict security group rules to specific CIDR ranges.

## Check: EC2_005 — Detailed Monitoring
- **API**: `aws ec2 describe-instances` → `.Monitoring.State`
- **Pass**: `State == "enabled"`
- **Fail**: `State == "disabled"`
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Operational Excellence
- **Recommendation**: Enable detailed monitoring for 1-minute metric granularity.

## Check: EC2_006 — Instance Generation Currency
- **API**: `aws ec2 describe-instances` → `.InstanceType`; verify generation against `aws ec2 describe-instance-types` → `.CurrentGeneration` and current regional documentation.
- **Pass**: The type is confirmed current generation.
- **Fail**: The type is confirmed previous generation; consider this an optimization opportunity, not proof it is unsupported.
- **Not assessed**: Current generation status or workload requirements cannot be established.
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Performance Efficiency, Cost Optimization
- **Recommendation**: Evaluate supported alternatives for regional availability, architecture compatibility, performance and cost before migration.

## Check: EC2_007 — EBS Volume Type
- **API**: `aws ec2 describe-volumes` → `.VolumeType`
- **Pass**: `gp3`, `io2`, or appropriate type
- **Fail**: `gp2` (predecessor to gp3)
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Performance Efficiency, Cost Optimization
- **Recommendation**: Migrate gp2 volumes to gp3 for better price-performance.

## Check: EC2_008 — Stopped Instances
- **API**: `aws ec2 describe-instances` → `.State.Name`
- **Pass**: Instance is running, or independent evidence establishes that it stopped within the last 30 days.
- **Fail**: Independent evidence establishes that the instance has been stopped for more than 30 days.
- **Not assessed**: A stopped instance whose stop time cannot be established. The current state alone is not a stop timestamp.
- **Severity**: 🟡 MEDIUM
- **Pillar**: Cost Optimization
- **Recommendation**: Review business requirements and storage charges before considering a snapshot or termination; termination is destructive.

## Check: EC2_009 — SSM Managed
- **API**: `aws ssm describe-instance-information`
- **Pass**: Instance appears in SSM inventory
- **Fail**: Instance not managed by SSM
- **Severity**: 🟡 MEDIUM
- **Pillar**: Operational Excellence, Security
- **Recommendation**: Install SSM Agent for patch management and remote access.

## Check: EC2_010 — Auto Scaling Group Membership
- **API**: `aws autoscaling describe-auto-scaling-instances`
- **Pass**: Instance is part of an ASG
- **Fail**: Standalone instance without ASG
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Reliability
- **Recommendation**: Use Auto Scaling Groups for automatic recovery and scaling.
