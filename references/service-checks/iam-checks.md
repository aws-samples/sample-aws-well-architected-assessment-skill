# IAM Service Checks

Reference: service-screener-v2 `services/iam/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

## Check: IAM_001 — Root Account MFA
- **API**: `aws iam get-account-summary` → `.SummaryMap.AccountMFAEnabled`
- **Pass**: `AccountMFAEnabled == 1`
- **Fail**: `AccountMFAEnabled == 0`
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: IAM_002 — Root Account Access Keys
- **API**: `aws iam get-account-summary` → `.SummaryMap.AccountAccessKeysPresent`
- **Pass**: `AccountAccessKeysPresent == 0`
- **Fail**: `AccountAccessKeysPresent > 0`
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: IAM_003 — Password Policy Strength
- **API**: `aws iam get-account-password-policy`
- **Pass**: For this sample's baseline, `MinimumPasswordLength >= 14`, all four character requirements are true, `ExpirePasswords == true`, and `MaxPasswordAge` is between 1 and 90 days.
- **Fail**: An applicable account password policy does not meet the sample baseline, including absent or disabled expiry, or there is no custom policy.
- **Not assessed**: The policy cannot be read, or applicability to the organization’s identity arrangements is unknown. Confirm the organization's requirements; this sample baseline is not a universal AWS mandate.
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: IAM_004 — User MFA Enabled
- **API**: `aws iam list-users` + `list-mfa-devices`
- **Pass**: All console users have MFA enabled
- **Fail**: Console users without MFA
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: IAM_005 — Access Key Rotation
- **API**: `aws iam list-access-keys` → `.CreateDate`
- **Pass**: Access keys rotated within 90 days
- **Fail**: Access keys older than 90 days
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: IAM_006 — Unused Credentials
- **API**: `aws iam get-credential-report`; decode the existing report's CSV content and inspect its generation time and credential last-used fields.
- **Pass**: Applicable active credentials were used within 90 days, based on a sufficiently recent report.
- **Fail**: An applicable active credential has not been used for more than 90 days, with the report providing sufficient evidence.
- **Not assessed**: The report is missing, stale for the assessment, inaccessible, or its values do not establish the credential's usage. Do not call `generate-credential-report`: generating a report changes account state.
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: IAM_007 — Overly Permissive Policies
- **API**: `aws iam list-policies` + `get-policy-version`
- **Pass**: Do not report an account-wide pass based on this limited pattern check.
- **Fail**: An inspected policy contains an unconditional full-administrator allow statement; review attachment and effective permissions.
- **Not assessed**: Effective permissions when inline policies, permission boundaries, SCPs, conditions, or other relevant policy types have not been assessed.
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: IAM_008 — IAM Users vs Roles
- **API**: `aws iam list-users`
- **Pass**: Minimal IAM users; workloads use IAM roles
- **Fail**: Many IAM users with programmatic access
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: IAM_009 — Support Role
- **API**: `aws iam list-entities-for-policy` (AWSSupportAccess)
- **Pass**: Support access role/user configured
- **Fail**: No entity with AWSSupportAccess
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Operational Excellence

## Check: IAM_010 — Access Analyzer
- **API**: `aws accessanalyzer list-analyzers`
- **Pass**: IAM Access Analyzer enabled
- **Fail**: No analyzer configured
- **Severity**: 🟡 MEDIUM | **Pillar**: Security
