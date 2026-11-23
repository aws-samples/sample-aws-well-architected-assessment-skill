# Threat model

## Scope and trust boundaries

This repository provides instructions, not an enforcement engine. A user runs
an agent with an AWS identity; the agent invokes AWS tools, receives selected
results and may send prompts and results to a model provider. AWS resource
metadata, tool results and user-controlled resource fields are untrusted.
Assessment output can contain sensitive infrastructure information.

| Threat | Intended mitigation | Residual risk |
| --- | --- | --- |
| Prompt injection in names, tags or policies | Treat retrieved content as data, never as instructions; keep per-command approval enabled. | An agent may misinterpret content; user review and platform controls remain necessary. |
| Secrets or customer data exposed to the model provider | Do not request secret values or data-plane content; locally allowlist output fields before delivery to the agent; use approved providers and destinations. | Local CLI processes can receive full API responses before `--query` filtering. Names, policies and filtered fields can still be sensitive; skip calls if raw responses could be logged or forwarded. |
| Over-privileged credentials | Use a dedicated role and short-lived credentials; review and narrow permissions for the assessment. | `SecurityAudit` is a broad starting point, may change, and can expose sensitive metadata; IAM and organizational policies must enforce the actual boundary. |
| Agent executes a changing command | Instructions prohibit mutation, credential-report generation and remediation execution; user approves tool calls; use read-only permissions. | Instructions alone cannot enforce behavior; the agent, tool configuration and IAM permissions must enforce it. |
| False pass or incomplete findings | Distinguish pass, fail, not assessed and not applicable; check pagination, permissions and evidence; review results against current documentation. | Heuristics and model output can still be wrong or stale; a pass is not a security or compliance certification. |

See [SKILL.md](SKILL.md), [README.md](README.md) and [TESTING.md](TESTING.md)
for the intended workflow and manual validation scenarios.
