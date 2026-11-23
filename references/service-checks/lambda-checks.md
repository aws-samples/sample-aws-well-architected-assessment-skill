# Lambda Service Checks

Reference: service-screener-v2 `services/lambda_/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Safe output handling

Lambda configuration responses can include environment variable values. Use an
approved local AWS CLI with debug output disabled and output field allowlists:

```bash
aws lambda list-functions --query 'Functions[].{Name:FunctionName,Runtime:Runtime}' --output json
aws lambda get-function-configuration --function-name <function-name> --query '{DeadLetterConfig:DeadLetterConfig,KMSKeyArn:KMSKeyArn,Timeout:Timeout,MemorySize:MemorySize,VpcId:VpcConfig.VpcId,TracingMode:TracingConfig.Mode}' --output json
```

Do not display `Environment` or raw responses. CLI filtering is local, not
server-side; the CLI still receives the full response. If the tool or its logs
could expose the raw response to the agent or model, do not call it and mark
the corresponding checks not assessed. Apply the same rule to other tools.

## Check: LAMBDA_001 — Runtime Currency
- **API**: `aws lambda list-functions` → `.Runtime`
- **Pass**: Supported, non-deprecated runtime
- **Fail**: Deprecated or end-of-life runtime (python3.8, nodejs14.x, etc.)
- **Severity**: 🔴 HIGH
- **Pillar**: Security, Operational Excellence
- **Recommendation**: Upgrade to a supported runtime version.

## Check: LAMBDA_002 — Dead Letter Queue Configuration
- **API**: `aws lambda get-function-configuration` → `.DeadLetterConfig`
- **Pass**: DLQ (SQS or SNS) configured
- **Fail**: No DLQ configured
- **Severity**: 🟡 MEDIUM
- **Pillar**: Reliability
- **Recommendation**: Configure a DLQ to capture failed async invocations.

## Check: LAMBDA_003 — Execution Role Scope
- **API**: `aws iam get-role-policy` / `list-attached-role-policies`
- **Pass**: Role follows least privilege (no `*` resource with broad actions)
- **Fail**: Role has overly permissive policies (e.g., `Action: *`)
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Scope IAM role to minimum required permissions.

## Check: LAMBDA_004 — Environment Variable Encryption
- **API**: `aws lambda get-function-configuration` → `.KMSKeyArn`
- **Pass**: KMS key configured for environment variable encryption
- **Fail**: Using default encryption (no customer-managed KMS key)
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security
- **Recommendation**: Use a customer-managed KMS key for sensitive env vars.

## Check: LAMBDA_005 — Timeout Configuration
- **API**: `aws lambda get-function-configuration` → `.Timeout`
- **Pass**: Timeout set appropriately (not default 3s for complex functions)
- **Fail**: Default timeout (3s) on functions with external dependencies
- **Severity**: 🟡 MEDIUM
- **Pillar**: Reliability
- **Recommendation**: Set timeout based on expected execution duration.

## Check: LAMBDA_006 — Memory Optimization
- **API**: `aws lambda get-function-configuration` → `.MemorySize`
- **Pass**: Memory sized appropriately (use Lambda Power Tuning)
- **Fail**: Default 128MB or significantly over-provisioned
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Cost Optimization, Performance Efficiency
- **Recommendation**: Use AWS Lambda Power Tuning to find optimal memory.

## Check: LAMBDA_007 — VPC Configuration
- **API**: `aws lambda get-function-configuration` → `.VpcConfig`
- **Pass**: Functions accessing VPC resources are VPC-attached
- **Fail**: Functions accessing private resources without VPC config
- **Severity**: 🟡 MEDIUM
- **Pillar**: Security
- **Recommendation**: Attach Lambda to VPC when accessing private resources.

## Check: LAMBDA_008 — Tracing Enabled
- **API**: `aws lambda get-function-configuration` → `.TracingConfig.Mode`
- **Pass**: `Mode == "Active"`
- **Fail**: `Mode == "PassThrough"`
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Operational Excellence
- **Recommendation**: Enable X-Ray active tracing for observability.

## Check: LAMBDA_009 — Reserved Concurrency
- **API**: `aws lambda get-function-concurrency`
- **Pass**: Reserved concurrency set for critical functions
- **Fail**: No reserved concurrency on critical functions
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Reliability
- **Recommendation**: Set reserved concurrency to guarantee capacity.

## Check: LAMBDA_010 — Code Signing
- **API**: `aws lambda get-function-code-signing-config` → `.CodeSigningConfigArn`
- **Pass**: Code signing configured
- **Fail**: The service confirms that an existing function has no code signing configuration.
- **Not assessed**: The function's existence or configuration cannot be confirmed, including ambiguous resource-not-found errors or access denied.
- **Severity**: 🟢 IMPROVEMENT
- **Pillar**: Security
- **Recommendation**: Enable code signing to ensure deployment integrity.
