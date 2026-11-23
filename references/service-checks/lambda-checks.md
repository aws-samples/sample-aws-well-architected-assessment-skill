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
aws lambda get-function-configuration --function-name <function-name> --query '{Role:Role,DeadLetterConfig:DeadLetterConfig,KMSKeyArn:KMSKeyArn,Timeout:Timeout,MemorySize:MemorySize,VpcId:VpcConfig.VpcId,TracingMode:TracingConfig.Mode}' --output json
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

## Check: LAMBDA_002 — Asynchronous Failure Handling
- **API**: For each relevant function, version or alias, use the safely filtered `aws lambda get-function-configuration` → `.DeadLetterConfig` and `aws lambda get-function-event-invoke-config` → `.DestinationConfig.OnFailure`. `list-function-event-invoke-configs` can help discover configured qualifiers; paginate it and also establish which qualifiers actually receive asynchronous invocations.
- **Pass**: An applicable asynchronous invocation path has a function DLQ or an on-failure destination that meets its documented failure-handling requirements.
- **Fail**: The workload requires failed asynchronous invocations to be retained, and complete evidence for the applicable invocation path confirms neither a function DLQ nor an on-failure destination is configured.
- **Not applicable**: The assessed path is confirmed to be synchronous or is an event source mapping; evaluate its source-specific retry and failure handling separately.
- **Not assessed**: Invocation mode, relevant qualifiers, requirements, configuration, or access cannot be established. An empty list of explicit event-invoke configurations or an ambiguous resource-not-found error alone does not prove the function or qualifier lacks failure handling.
- **Severity**: 🟡 MEDIUM
- **Pillar**: Reliability
- **Recommendation**: For applicable asynchronous paths, consider a DLQ or on-failure destination and verify that it meets the workload's recovery requirements.

## Check: LAMBDA_003 — Execution Role Scope
- **API**: Identify `.Role` using the safely filtered function configuration. Paginate `aws iam list-role-policies` and retrieve each inline policy with `get-role-policy`; paginate `list-attached-role-policies`, then use `get-policy` and `get-policy-version` to retrieve each attached managed policy's default version. Review `get-role` for a permissions boundary and obtain its policy where applicable.
- **Pass**: The relevant policy documents and applicable constraints have been reviewed against established workload requirements, and the assessed permissions are justified. The absence of `Action: "*"` alone does not establish least privilege.
- **Fail**: Evidence establishes that the role has effective permissions beyond the documented workload requirements. Report broad identity-policy grants and any known constraints separately; a broad policy statement alone does not prove effective access if a boundary, explicit deny or other control limits it.
- **Not assessed**: Policy enumeration or retrieval is incomplete, a required boundary or other applicable control cannot be evaluated, or workload permissions and effective scope are unknown. Attached policy names and ARNs alone are not policy contents.
- **Severity**: 🔴 HIGH
- **Pillar**: Security
- **Recommendation**: Review the role's effective permissions against the function's required actions and resources; reduce unjustified grants.

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
