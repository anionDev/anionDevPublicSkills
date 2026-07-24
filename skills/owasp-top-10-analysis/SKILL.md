---
name: owasp-top-10-analysis
description: Analyze a repository for security weaknesses based on the OWASP Top 10 and provide actionable remediation advice.
purpose: "Analysis for repositories."
version: 1.0.0
---
Perform a repository security analysis focused on the OWASP Top 10.
Before starting the analysis, research the latest official OWASP Top 10 list/version and use that as the baseline for all findings.
Use this structured workflow:
1. Scope and attack surface discovery:
- Identify components with real external attack surface first (for example: public HTTP endpoints, API gateways, auth flows, admin interfaces, file upload paths, webhook handlers, message consumers, third-party integration points, CI/CD and deployment entry points).
- Mark each component as internet-exposed, internal-only, or local-only.
- Focus deep analysis on internet-exposed and high-impact internal components.
2. Asset and trust-boundary mapping:
- Document sensitive assets (credentials, tokens, PII, financial data, signing keys, internal services).
- Identify trust boundaries and data flows across components.
- Highlight where untrusted input crosses boundaries.
3. OWASP Top 10 assessment:
- Assess the discovered attack-surface components against the latest OWASP Top 10 categories.
- For each category, check relevant code, configuration, dependency manifests, build settings, and runtime hardening.
- Distinguish between confirmed findings, likely findings, and assumptions when evidence is incomplete.
4. Risk rating and prioritization:
- Prioritize by exploitability, potential impact, exposure, and ease of abuse.
- Use a clear criticality level per finding: Critical, High, Medium, Low.
- Include at least one practical mitigation proposal per finding.
5. Validation and coverage check:
- Mention missing security tests for high-risk paths.
- Mention useful secure defaults and hardening opportunities.
- If a category has no findings, state that explicitly.
Output requirements:
- Structure the result by OWASP category.
- For each finding provide: title, affected component, attack surface, risk, evidence, impact, and recommendation.
- End with a short prioritized remediation plan.
- End with a summary table of all findings including at least: finding ID, OWASP category, affected component, criticality, confidence, and status (open/mitigated/needs-validation).
Do not actually change any file unless the user explicitly asks for fixes.
