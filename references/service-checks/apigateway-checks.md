# API Gateway Service Checks

Reference: service-screener-v2 `services/apigateway/`

Modification notice: This guide adapts and extends upstream assessment guidance
for an agent-led workflow. See [third-party notices](../../THIRD-PARTY-LICENSES)
for the source record and Apache 2.0 terms.

Unless stated otherwise, these checks use REST API (`apigateway`) operations.
Do not report HTTP or WebSocket APIs as assessed by REST-only checks.

## Check: APIGW_001 — Access Logging
- **API**: `aws apigateway get-stages` → `.accessLogSettings`
- **Pass**: Access log settings configured with destination ARN
- **Fail**: No access logging configured
- **Severity**: 🟡 MEDIUM | **Pillar**: Operational Excellence, Security

## Check: APIGW_002 — Authorization
- **API**: REST: `aws apigateway get-resources` + `get-method` → `.authorizationType`. HTTP/WebSocket: `aws apigatewayv2 get-routes` and `get-authorizers` for the selected API.
- **Pass**: Each in-scope non-public method or route has an appropriate IAM, Cognito/JWT, or Lambda authorizer and the relevant configuration has been reviewed.
- **Fail**: An in-scope non-public method or route has no authorization.
- **Not assessed**: Public/private intent, route inventory, or authorizer effectiveness cannot be established. An API key is not an authorization mechanism.
- **Severity**: 🔴 HIGH | **Pillar**: Security

## Check: APIGW_003 — Throttling Configuration
- **API**: `aws apigateway get-stages` → `.methodSettings.*.throttlingRateLimit`
- **Pass**: Throttling limits configured
- **Fail**: Default throttling only (no custom limits)
- **Severity**: 🟡 MEDIUM | **Pillar**: Reliability, Security

## Check: APIGW_004 — WAF Integration
- **API**: `aws wafv2 get-web-acl-for-resource`
- **Pass**: WAF WebACL associated with API stage
- **Fail**: No WAF protection
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: APIGW_005 — TLS Version
- **API**: `aws apigateway get-domain-names` → `.securityPolicy`
- **Pass**: `securityPolicy == "TLS_1_2"`
- **Fail**: `securityPolicy == "TLS_1_0"`
- **Severity**: 🟡 MEDIUM | **Pillar**: Security

## Check: APIGW_006 — CloudWatch Metrics
- **API**: `aws apigateway get-stages` → `.methodSettings.*.metricsEnabled`
- **Pass**: CloudWatch metrics enabled
- **Fail**: Metrics disabled
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Operational Excellence

## Check: APIGW_007 — Caching
- **API**: `aws apigateway get-stages` → `.cacheClusterEnabled`
- **Pass**: Caching enabled for appropriate stages
- **Fail**: No caching configured
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Performance Efficiency

## Check: APIGW_008 — X-Ray Tracing
- **API**: `aws apigateway get-stages` → `.tracingEnabled`
- **Pass**: `tracingEnabled == true`
- **Fail**: `tracingEnabled == false`
- **Severity**: 🟢 IMPROVEMENT | **Pillar**: Operational Excellence
