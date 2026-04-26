<!-- Copyright (c) Microsoft Corporation. Licensed under the MIT License. -->

# Fundamental Rights Impact Assessment (FRIA) Template

**For AI Agent Systems — EU AI Act Article 27 Compliance**

> **Regulation Reference**: Article 27, Regulation (EU) 2024/1689 (EU AI Act)
> **Deadline**: 2 August 2026 — high-risk AI deployer obligations take effect
> **When Required**: Before deploying any high-risk AI agent system (Annex III categories)
> **Prepared**: 2026-04-26

---

## Purpose

Deployers of high-risk AI systems must conduct a Fundamental Rights Impact Assessment *before* putting the system into use. This template helps you document that assessment for AI agent deployments governed by the Agent Governance Toolkit (AGT).

**This is not legal advice.** Consult qualified legal counsel for jurisdiction-specific compliance.

> ⚠️ **Important**: Completing this template alone does not constitute full Article 27 compliance. It is a structured documentation tool to support — not replace — a proper legal assessment. Deployers should engage qualified legal counsel to validate their FRIA before deployment.

---

## When Is a FRIA Required?

| Condition | FRIA Required? |
|-----------|---------------|
| Agent classified as high-risk under Annex III | **Yes** |
| Agent used by public body or on behalf of public body | **Yes** |
| Agent affects access to essential services (health, education, employment) | **Yes** |
| Agent is limited-risk (transparency only) | No (but recommended) |
| Agent is minimal-risk | No (voluntary) |

---

## 1. System Overview

| Field | Description |
|-------|-------------|
| **System name** | [Name of the AI agent system] |
| **Deployer** | [Organization deploying the system] |
| **Provider** | [Organization that developed the system] |
| **Deployment date** | [Planned or actual deployment date] |
| **High-risk category** | [Which Annex III category applies] |
| **Agent governance version** | [AGT version used, e.g., v0.6.0] |

### 1.1 System Description

*Describe what the agent does, its inputs/outputs, and its decision-making scope. Be specific: "This agent processes loan applications for amounts under €50,000 and outputs approve/reject/escalate recommendations."*

<!-- Answer here -->

### 1.2 Deployment Context

*Who uses the agent? Who is affected by its decisions? What's the human oversight mechanism?*

<!-- Answer here -->

---

## 2. Fundamental Rights Assessment

EU AI Act Article 27(1) requires assessing impacts on fundamental rights protected by the EU Charter of Fundamental Rights. Evaluate each right below.

### Scoring

| Score | Meaning |
|-------|---------|
| ⬜ **No impact** | Right not affected by this deployment |
| 🟡 **Low impact** | Minor, temporary, or indirect effect; easily reversible |
| 🟠 **Medium impact** | Significant effect requiring mitigation measures |
| 🔴 **High impact** | Severe or irreversible effect on the right |

### Assessment Table

| # | Fundamental Right | Impact | Affected Group | Justification |
|---|------------------|--------|---------------|---------------|
| 1 | **Human dignity** (Art. 1) | ⬜ 🟡 🟠 🔴 | | |
| 2 | **Right to life** (Art. 2) | ⬜ 🟡 🟠 🔴 | | |
| 3 | **Right to integrity** (Art. 3) | ⬜ 🟡 🟠 🔴 | | |
| 4 | **Freedom of thought/conscience** (Art. 10) | ⬜ 🟡 🟠 🔴 | | |
| 5 | **Freedom of expression/information** (Art. 11) | ⬜ 🟡 🟠 🔴 | | |
| 6 | **Respect for private/family life** (Art. 7) | ⬜ 🟡 🟠 🔴 | | |
| 7 | **Protection of personal data** (Art. 8) | ⬜ 🟡 🟠 🔴 | | |
| 8 | **Non-discrimination** (Art. 21) | ⬜ 🟡 🟠 🔴 | | |
| 9 | **Right to an effective remedy** (Art. 47) | ⬜ 🟡 🟠 🔴 | | |
| 10 | **Right to a fair trial** (Art. 48) | ⬜ 🟡 🟠 🔴 | | |
| 11 | **Workers' rights** (Art. 30-34) | ⬜ 🟡 🟠 🔴 | | |
| 12 | **Right to education** (Art. 14) | ⬜ 🟡 🟠 🔴 | | |
| 13 | **Freedom of assembly/association** (Art. 12) | ⬜ 🟡 🟠 🔴 | | |
| 14 | **Right to property** (Art. 17) | ⬜ 🟡 🟠 🔴 | | |
| 15 | **Consumer protection** (Art. 38) | ⬜ 🟡 🟠 🔴 | | |
| 16 | **Equality before the law** (Art. 20) | ⬜ 🟡 🟠 🔴 | | |
| 17 | **Right to good administration** (Art. 41) | ⬜ 🟡 🟠 🔴 | | |

*Add additional rights as applicable to your specific deployment context.*

---

## 3. Risk Analysis

### 3.1 Identified Risks

| Risk ID | Description | Affected Right(s) | Likelihood | Severity | Risk Level | Root Cause |
|---------|-------------|-------------------|------------|----------|------------|------------|
| R-001 | *e.g., Agent disproportionately rejects applications from [demographic]* | Art. 21 | Low/Med/High | Low/Med/High | 🟡🟠🔴 | |
| R-002 | | | | | | |
| R-003 | | | | | | |

### 3.2 Risk Level Matrix

| | **Severity: Low** | **Severity: Medium** | **Severity: High** |
|---|---|---|---|
| **Likelihood: Low** | 🟡 Low | 🟡 Low | 🟠 Medium |
| **Likelihood: Medium** | 🟡 Low | 🟠 Medium | 🔴 High |
| **Likelihood: High** | 🟠 Medium | 🔴 High | 🔴 High |

### 3.3 Aggregate Risk Summary

| Risk Level | Count | Action Required |
|-----------|-------|----------------|
| 🔴 High | 0 | Must mitigate before deployment |
| 🟠 Medium | 0 | Mitigation plan required |
| 🟡 Low | 0 | Monitor and document |
| ⬜ None | 17 | No action needed |

> *Note: Counts above are template defaults. Update after completing your assessment.*

---

## 4. Mitigation Measures

### 4.1 Mitigation Tracker

| Risk ID | Mitigation Measure | Owner | Deadline | Status | Residual Risk |
|---------|-------------------|-------|----------|--------|--------------|
| R-001 | *e.g., Implement bias audit with ≥1,000 test cases quarterly* | | | ⬜ Planned 🔄 In progress ✅ Done | 🟡🟠🔴 |
| R-002 | | | | | |
| R-003 | | | | | |

### 4.2 AGT Integration Points

The following AGT features support FRIA mitigation:

| AGT Feature | FRIA Use Case | Reference |
|-------------|--------------|-----------|
| Policy rules engine | Block prohibited practices (Art. 5), enforce constraints | `docs/GOVERNANCE.md` |
| Compliance checker | Classify agent risk tier, trigger FRIA requirement | `packages/agent-mesh/examples/06-eu-ai-act-compliance/` |
| Audit logging | Record agent decisions for accountability (Art. 12) | EU AI Act Checklist |
| Human oversight | Require human approval for high-impact decisions (Art. 14) | EU AI Act Checklist |
| Transparency controls | Disclose AI interaction to affected persons (Art. 50) | EU AI Act Checklist |

---

## 5. Governance and Review

### 5.1 Approval

| Role | Name | Decision | Date |
|------|------|----------|------|
| **Data Protection Officer** | | ⬜ Approved ⬜ Rejected ⬜ Conditional | |
| **Legal/Compliance** | | ⬜ Approved ⬜ Rejected ⬜ Conditional | |
| **Technical Lead** | | ⬜ Approved ⬜ Rejected ⬜ Conditional | |
| **Business Owner** | | ⬜ Approved ⬜ Rejected ⬜ Conditional | |
| **AI Ethics Lead** *(optional)* | | ⬜ Approved ⬜ Rejected ⬜ Conditional | |

### 5.2 Review Schedule

| Review Type | Frequency | Next Review Date |
|-------------|-----------|-----------------|
| Risk reassessment | Every 6 months | |
| Mitigation audit | Quarterly | |
| Full FRIA update | Annually or upon material change | |

### 5.3 Stakeholder Consultation

*Document any consultations with affected communities, civil society organizations, or external experts required under Art. 27(1)(d).*

<!-- Answer here -->

---

## 6. Conclusion

| Question | Answer |
|----------|--------|
| Can the deployment proceed? | ⬜ Yes ⬜ Yes with conditions ⬜ No |
| Remaining high risks? | |
| Conditions for deployment | |
| Next review date | |

---

## References

- **EU AI Act**: Regulation (EU) 2024/1689, Article 27 (Deployer obligations — Fundamental Rights Impact Assessment)
- **EU Charter of Fundamental Rights**: [2012/C 326/02](https://eur-lex.europa.eu/eli/treaty/char_2012/oj)
- **AGT EU AI Act Checklist**: `docs/compliance/eu-ai-act-checklist.md`
- **AGT Governance Guide**: `docs/GOVERNANCE.md`
- **AGT Threat Model**: `docs/THREAT_MODEL.md`

---

## Appendix A: Completing This Template — Quick Guide

1. **System Overview** (Section 1): Fill in basic info. Be specific about what the agent does and who it affects. 15 minutes.

2. **Rights Assessment** (Section 2): Go through each right. Most will be "No impact" — that's fine. Focus on the ones that aren't. 30-45 minutes with your legal team. Note: 17 rights listed; add more as needed for your deployment context.

3. **Risk Analysis** (Section 3): For every right with 🟠 or 🔴 impact, create a risk entry. Use the matrix to determine overall risk level. 30 minutes.

4. **Mitigation** (Section 4): For every 🟠 or 🔴 risk, define a concrete mitigation. Include owner, deadline, and how you'll verify it works. 30-60 minutes.

5. **Governance** (Section 5): Get sign-offs. Set review dates. Document stakeholder consultations. 15 minutes.

**Total time for first FRIA**: 2-3 hours. Subsequent assessments for similar agents: ~1 hour using this template as a baseline.

---

*This template is part of the [Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit). Contributions welcome — see [CONTRIBUTING.md](../CONTRIBUTING.md) for guidelines.*
