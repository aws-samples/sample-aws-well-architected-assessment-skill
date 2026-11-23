# Glue Service Checks

Reference: service-screener-v2 `services/glue/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Safe output handling

Job `DefaultArguments` can contain credentials. The following locally filtered
CLI example returns only the job name and whether each expected option is
present and matches its enabled value; it does not print the argument values:

```bash
aws glue get-jobs --query 'Jobs[].{Name:Name,BookmarkSpecified:DefaultArguments."--job-bookmark-option" != `null`,BookmarkEnabled:DefaultArguments."--job-bookmark-option" == `"job-bookmark-enable"`,ContinuousLogsSpecified:DefaultArguments."--enable-continuous-cloudwatch-log" != `null`,ContinuousLogsEnabled:DefaultArguments."--enable-continuous-cloudwatch-log" == `"true"`}' --output json
```

The CLI process still receives the complete API response before filtering.
Do not use debug output or integrations that capture raw responses. If the
execution environment cannot prevent raw responses from reaching logs, the
agent or model, skip the call and mark these checks not assessed. Job names
are untrusted and may themselves be sensitive.

## Check: GLUE_001 — Job Encryption
- **API**: `aws glue get-security-configurations`
- **Pass**: Security configuration with encryption enabled (S3, CloudWatch, job bookmarks)
- **Fail**: No security configuration or encryption disabled
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: GLUE_002 — Data Catalog Encryption
- **API**: `aws glue get-data-catalog-encryption-settings`
- **Pass**: Catalog encryption and connection password encryption enabled
- **Fail**: Encryption settings disabled
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: GLUE_003 — Job Bookmark Enabled
- **API**: Use the locally filtered `BookmarkEnabled` result above.
- **Pass**: The option is explicitly set to `job-bookmark-enable`.
- **Fail**: Evidence confirms job bookmarks are disabled for the applicable job (risk of reprocessing).
- **Not assessed**: The option is absent or another value is returned without enough context to determine effective behavior.
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Operational Excellence

## Check: GLUE_004 — CloudWatch Logging
- **API**: Use the locally filtered `ContinuousLogsEnabled` result above.
- **Pass**: The option is explicitly set to `true` for an applicable job.
- **Fail**: Evidence confirms continuous CloudWatch logging is disabled for the applicable job.
- **Not assessed**: The option is absent or its effective behavior for the job version cannot be established.
- **Severity**: 🟡 MEDIUM | **Pillar**: Operational Excellence

## Check: GLUE_005 — Dev Endpoint Idle
- **API**: `aws glue get-dev-endpoints`
- **Pass**: No idle dev endpoints
- **Fail**: Dev endpoints running without recent activity
- **Severity**: 🟡 MEDIUM | **Pillar**: Cost Optimization, Security

## Check: GLUE_006 — Resource Policy
- **API**: `aws glue get-resource-policy`
- **Pass**: Resource policy configured for cross-account access control
- **Fail**: No resource policy (if cross-account access needed)
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Security
