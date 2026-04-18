# Four Ways to Govern AI Agents — And When Each One Works

Governance is the thing nobody wants to think about until something goes wrong. You ship an agent, it works fine in testing, and then at 2 AM it decides the best way to fulfill "optimize our cloud costs" is to delete half your production databases. Not because the LLM hallucinated — because the action was technically correct and nobody told it not to.

The question isn't whether you need governance. It's which kind, at which layer, and when to layer them. Here are four approaches we've evaluated and used in production, with honest trade-offs for each.

---

## Approach 1: Prompt Engineering Guardrails

The simplest approach: tell the agent what not to do in its system prompt.

```
You are a helpful assistant. Never delete files. Never send emails 
without confirmation. Always ask before making irreversible changes.
```

**How it works:** You rely on the model's instruction-following ability to constrain behavior. The guardrails live in natural language and are evaluated by the same model that's taking actions.

**Where it works:**
- Prototypes and demos
- Low-risk internal tools
- Quick iteration when you're still figuring out what the agent should do

**Where it breaks down:**
- Prompt injection can override instruction-based constraints. Not "might" — *can*. This is well-documented and remains an unsolved problem at the prompt layer.
- Constraints are probabilistic. The model follows them most of the time, which is fine until the one time it doesn't.
- No auditability. You can't verify after the fact whether the constraints were evaluated or what the reasoning was.
- Scale poorly. As you add more constraints, prompt length grows, comprehension degrades, and edge cases multiply.

**Honest assessment:** Every production agent system needs something beyond this. Prompt guardrails are a useful first layer — the seatbelt warning light, not the seatbelt.

---

## Approach 2: Platform-Level Restrictions

API rate limits, model content filters, token ceilings, IP allowlists. The infrastructure layer enforces constraints regardless of what the agent asks for.

**How it works:** The platform (API provider, gateway proxy, or orchestration layer) enforces hard limits on what agents can do. These are deterministic and apply uniformly.

**Where it works:**
- Cost control (token budgets, rate limits)
- Content safety (model-level refusal of harmful outputs)
- Preventing obvious abuse (too many requests, too fast)
- Multi-tenant environments where you need uniform policies

**Where it breaks down:**
- Platform restrictions operate at the *request* level, not the *action* level. A rate limit of 100 requests/minute doesn't distinguish between 100 safe reads and 100 destructive writes. The agent that's rate-limited is also the agent you might need most in an emergency.
- No semantic understanding. API filters can catch "generate a bomb recipe" but can't catch "delete all user records older than 30 days" when the agent has legitimate database access.
- Vendor lock-in. Your governance model is tied to your platform's feature set.

**Honest assessment:** Essential infrastructure, but insufficient for agent governance. Rate limits don't understand agent intent. Content filters don't understand action consequences.

---

## Approach 3: Policy-as-Code Governance

Policies expressed as code (YAML, Rego, Cedar, or native policy DSLs), version-controlled, tested, and enforced deterministically before actions execute.

```yaml
version: "1.0"
name: production-agent-policy
rules:
  - name: block-destructive-actions
    condition:
      field: tool_name
      operator: in
      value: [delete_database, drop_table, rm_rf]
    action: deny
    priority: 100

  - name: require-approval-for-external-writes
    condition:
      field: tool_name
      operator: in
      value: [send_email, publish_message, deploy_production]
    action: require_approval
    priority: 50
```

**How it works:** Each agent action passes through a policy evaluator before execution. The evaluator checks the action against the policy rules and returns allow, deny, or escalate. This happens in deterministic code, not in the LLM.

The [Agent Governance Toolkit](https://github.com/microsoft/agent-governance-toolkit) implements this model with three gates:

- **TrustGate** — identity verification (who is this agent, what credentials does it hold?)
- **GovernanceGate** — policy evaluation (is this action allowed by the configured rules?)
- **ReliabilityGate** — SRE enforcement (are we within SLOs, error budgets, rate limits?)

**Where it works:**
- Production systems where you need deterministic, auditable enforcement
- Teams that want to review and approve governance rules through code review
- Multi-agent systems where different agents need different capability scopes
- Regulated industries that need proof of governance controls

**Where it breaks down:**
- Policy rules are only as good as the scenarios you anticipate. An unanticipated action that doesn't match any rule is either denied (safe but restrictive) or allowed (permissive but risky). Default-deny is the right posture, but it creates friction for legitimate novel actions.
- Policy rules evaluate actions in isolation. They can catch "this action is dangerous" but not "these five safe actions, taken together, constitute a dangerous trajectory." This is the point-in-time vs. trajectory problem.
- Policy authoring requires upfront investment. You need to think through your threat model, classify your actions, and write rules for each scenario. This is real work, and it's work that pays off disproportionately for production systems.

**Honest assessment:** This is the scalable path for most production agent deployments. It's not the easiest starting point, but it's the one that holds up under real operational pressure. The key insight: policies should be code because code can be tested, reviewed, versioned, and rolled back. Natural language constraints can't.

---

## Approach 4: Regulatory-First Compliance

Start from compliance requirements (SOC 2, ISO 27001, AI Act, sector-specific regulations) and work backwards to governance controls.

**How it works:** Compliance frameworks define what you need to prove. You implement controls that map to those requirements — audit logs, access controls, data handling policies, incident response procedures.

**Where it works:**
- Organizations with regulatory obligations (finance, healthcare, government)
- Systems that need third-party audit attestation
- Establishing baseline governance when you're starting from scratch

**Where it breaks down:**
- Compliance controls are designed for human-operated systems. "Who approved this change?" is straightforward when a human clicked a button. It's less straightforward when an autonomous agent chain executed a multi-step workflow at 3 AM and the human approval was a 5-minute timeout that auto-denied.
- Compliance tends to lag behind technology. Most AI governance frameworks are still catching up to what agents actually do in production.
- Checkbox compliance can create a false sense of security. Passing an audit doesn't mean your agents are well-governed — it means you documented your governance posture.

**Honest assessment:** Necessary for regulated deployments, but not sufficient on its own. Compliance tells you what to document. Policy-as-code tells you how to enforce it. You need both.

---

## When to Use Which

| Scenario | Recommended Approach | Why |
|----------|---------------------|-----|
| Prototype / demo | Prompt guardrails | Speed, no infrastructure |
| Internal tool, low risk | Prompt + platform restrictions | Quick wins, low overhead |
| Production agent, single tenant | Policy-as-code | Deterministic, auditable |
| Multi-agent production system | Policy-as-code + compliance | Handles complexity, proves controls |
| Regulated industry | All four, layered | Defense in depth |

The pattern: **start simple, add layers as risk increases.** But don't stay on prompt guardrails when the risk profile changes. The gap between "our agent works fine" and "our agent deleted production data" is usually one unanticipated edge case and zero enforcement layers.

---

## How They Complement Each Other

None of these approaches are mutually exclusive. A practical setup layers them:

1. **Prompt guardrails** — first line, catches obvious misbehavior at zero cost
2. **Platform restrictions** — hard infrastructure limits, catch volume abuse
3. **Policy-as-code** — deterministic enforcement of specific governance rules
4. **Compliance controls** — prove to auditors that the above are working

The critical boundary is between layers 2 and 3. Layers 1-2 are probabilistic or coarse-grained. Layer 3 is where you get deterministic, action-level control with full auditability. If you're running agents in production and you haven't crossed that boundary, you're relying on the model being well-behaved — which is a bet, not a control.

---

## Why Policy-as-Code Is the Scalable Path

Three reasons:

**Determinism.** Policy evaluation returns the same answer every time for the same input. LLM-based governance doesn't. In production, you need to know that "delete database" is always denied, not denied 99.7% of the time.

**Auditability.** Every policy decision is a log entry. You can trace any agent action back to the policy that allowed or denied it. This matters for debugging, for compliance, and for the 3 AM incident review.

**Evolution.** Policies are code. You can test them in CI, roll them back when they break, and review them in pull requests. Governance evolves through the same engineering practices as the rest of your system.

---

## Practical Next Steps

If you're running agents in production today:

1. **Inventory what your agents can actually do.** Not what you intended — what their tool access and permissions actually allow.
2. **Classify actions by blast radius.** Read-only actions are low risk. External writes are medium. Irreversible actions are high. Anything from a high-fanout agent (output feeds 3+ downstream consumers) gets escalated one tier.
3. **Start with deny rules for the highest-risk actions.** You don't need a complete policy framework on day one. A few well-targeted deny rules catch the scenarios that hurt most.
4. **Add approval workflows for medium-risk actions.** Human-in-the-loop doesn't have to mean human-in-the-way. Timeout-based auto-deny (if nobody approves in 5 minutes, the action is denied) keeps things moving while maintaining oversight.
5. **Iterate.** Your first policies will be wrong. That's fine. The advantage of policy-as-code is that wrong policies are fixable through the same process as wrong code: identify, patch, deploy.

---

*The author has deployed multi-agent governance systems using the Agent Governance Toolkit in production environments, including constitutional constraint layers and blast-radius-aware escalation tiers. Opinions are based on operational experience, not theoretical analysis.*
