# Comparing Agent Governance Approaches — A Framework Review

> Different problems need different tools. Here's how to pick the right governance approach for your AI agents.

---

The hardest part of governing AI agents isn't finding tools — it's understanding which layer of governance you actually need. A chatbot that occasionally hallucinates has different requirements than an autonomous agent that can execute code, call APIs, and spawn other agents.

This article maps the landscape of agent governance approaches, explains when each one applies, and shows how they can work together.

---

## The Four Layers of Agent Governance

Think of governance as a stack. Each layer addresses a different failure mode:

```
┌─────────────────────────────────────────────┐
│  Layer 4: Regulatory & Compliance           │  ← Checklists, audits, certifications
│  "Is this legally defensible?"              │
├─────────────────────────────────────────────┤
│  Layer 3: Agent Action Governance           │  ← Policy-as-code, identity, sandboxing
│  "Can this agent do something dangerous?"   │
├─────────────────────────────────────────────┤
│  Layer 2: Platform-Level Controls           │  ← API rate limits, model filters, access keys
│  "Is the API usage bounded?"               │
├─────────────────────────────────────────────┤
│  Layer 1: Prompt-Level Guardrails           │  ← System prompts, output validation
│  "Did the model say something wrong?"       │
└─────────────────────────────────────────────┘
```

Most teams start at Layer 1 and discover they need Layer 3 only after something goes wrong. The sections below examine each layer.

---

## Layer 1: Prompt-Level Guardrails

**What it is:** Constraining agent behavior through system prompts, output schemas, and post-hoc validation of LLM responses.

**How it works:**
- System prompts instruct the model to avoid certain behaviors ("Do not access files outside /data/")
- Output parsers validate structure (Pydantic schemas, JSON schemas)
- Content filters catch harmful or off-topic responses

**Strengths:**
- Zero infrastructure overhead
- Fast to implement (minutes)
- Works with any LLM provider
- Sufficient for chatbots and simple assistants

**Limitations:**
- **Not enforceable.** A system prompt is a request, not a constraint. Prompt injection, jailbreaks, and model confusion can bypass it.
- **No audit trail.** When a prompt-guided agent misbehaves, you have no structured record of what went wrong.
- **Doesn't govern actions.** A prompt can't stop an agent from calling `subprocess.run()` or sending data to an external API.

**When to use:**
- Chatbots and Q&A systems
- Content generation assistants
- Any system where the model only *generates text* and doesn't *take actions*

**Tools:** NeMo Guardrails, Guardrails AI, provider-side content filters (OpenAI moderation, etc.)

---

## Layer 2: Platform-Level Controls

**What it is:** Constraints enforced by the infrastructure around the model — API rate limits, model access policies, network restrictions, and usage quotas.

**How it works:**
- API gateways enforce request rate limits (e.g., 60 requests/minute)
- Model providers filter outputs for safety categories
- API keys scope access to specific models or endpoints
- Usage dashboards track cost and volume

**Strengths:**
- Enforced at the infrastructure level — agents can't bypass them
- Provider-managed (no custom code)
- Cost visibility and budgeting
- Works regardless of agent framework

**Limitations:**
- **Coarse-grained.** Rate limits don't distinguish between "search the web" and "delete a database."
- **Not agent-aware.** An API gateway doesn't know which agent made the request or what its role is.
- **No policy logic.** You can't express "allow payments under $100, require approval for payments over $10,000."

**When to use:**
- Any production LLM deployment (as a baseline)
- Cost control and abuse prevention
- Multi-provider LLM management

**Tools:** LiteLLM, Portkey, provider dashboards (OpenAI, Anthropic, Google)

---

## Layer 3: Agent Action Governance

**What it is:** Deterministic, pre-execution policy enforcement on every action an agent attempts — tool calls, sub-agent spawns, file operations, API invocations.

**How it works:**
- Every agent action passes through a policy engine **before** execution
- Policies are defined as code (YAML/Python), versioned, and tested
- Agent identity is verified cryptographically (not just "who claims to be who")
- Actions are allowed, denied, or routed to human approval queues
- Full audit logs with hash chains provide tamper-evident records

**Strengths:**
- **Deterministic enforcement.** Policies execute at sub-millisecond latency before every action. No reliance on model cooperation.
- **Agent-aware.** Policies can reference agent role, trust score, environment, and action parameters.
- **Auditable.** Every decision is logged with the full context (who, what, when, why allowed/denied).
- **Testable.** Policies can be unit-tested like any other code.

**Limitations:**
- **Only governs actions it can see.** If an agent can execute arbitrary code outside the governance middleware, the policy engine can't intercept it. This is why defense-in-depth (container isolation + action governance) matters for high-security deployments.
- **Application-layer only.** Does not provide OS-level isolation by itself. For high-security environments, compose with container/VM sandboxing.
- **Setup complexity.** More initial investment than prompt guardrails, though the payoff scales with agent autonomy.

**When to use:**
- Multi-agent systems where agents have different roles and permissions
- Any system where agents can take real-world actions (payments, file writes, API calls)
- Regulated industries (finance, healthcare) requiring audit trails
- Production deployments where agents operate autonomously

**Tools:** Agent Governance Toolkit, custom policy middleware

---

## Layer 4: Regulatory & Compliance

**What it is:** Framework-specific checklists, assessments, and certifications that provide legal and organizational assurance.

**How it works:**
- Mapping technical controls to regulatory requirements (EU AI Act, ISO 42001, NIST AI RMF)
- Attestation reports and compliance dashboards
- Periodic audits and evidence collection

**Strengths:**
- Legal defensibility
- Organizational trust and stakeholder confidence
- Required for regulated deployments

**Limitations:**
- Compliance $\neq$ security. Passing an audit doesn't mean your agents are safe.
- Checklists lag behind threats. Regulations take years; attack techniques take weeks.
- No runtime enforcement. A compliance document can't stop a misbehaving agent.

**When to use:**
- Always (as a baseline for regulated industries)
- Combined with Layer 3 enforcement for actual protection

**Tools:** VerifyWise, compliance dashboards, audit frameworks

---

## How the Layers Complement Each Other

No single layer is sufficient. The practical approach is defense-in-depth:

```
Example: A financial services agent

Layer 1 (Prompt) → "You are a billing assistant. Only discuss payments."
Layer 2 (Platform) → LLM API rate-limited to 60 req/min, cost capped at $50/day
Layer 3 (Action) → Payment actions under $100: ALLOW
                     Payment actions $100-$10k: REQUIRE_APPROVAL (1 approver)
                     Payment actions over $10k: REQUIRE_APPROVAL (2 approvers)
                     Agent can only call payment APIs, not general web
Layer 4 (Compliance) → ISO 42001 mapping, audit logs retained 7 years
```

If Layer 1 fails (prompt injection), Layer 3 still blocks unauthorized actions. If Layer 3 has a policy gap, Layer 2 prevents unlimited API usage. If everything else fails, Layer 4 provides the audit trail to investigate and remediate.

---

## Why Policy-as-Code Is the Scalable Path

As agent systems grow, the governance challenge scales non-linearly:

| Agents | Prompt Guardrails | Platform Controls | Policy-as-Code |
|--------|-------------------|-------------------|----------------|
| 1 | Simple | Simple | Overkill |
| 5 | Getting complex | Still simple | Starting to pay off |
| 20 | Brittle | Insufficient | Necessary |
| 100+ | Untenable | Insufficient | Essential |

Policy-as-code scales because:

1. **It's versioned.** Policy changes go through code review, CI testing, and staged rollouts — just like application code.

2. **It's composable.** Base policies (rate limits) combine with role policies (capability scoping) and environment policies (dev vs. prod). You don't rewrite governance from scratch for each agent.

3. **It's testable.** Before a policy reaches production, it runs through automated tests covering allowed, denied, and approval-required scenarios. This is impossible with prompt guardrails.

4. **It's auditable.** Every enforcement decision is recorded with full context — not just "was this allowed" but "why was this allowed" based on which policy rule matched.

---

## Choosing Your Approach

**Simple chatbot?** Layer 1 is probably enough. Start with prompt guardrails and add Layer 2 platform controls for cost management.

**Single autonomous agent?** Layers 1 + 2 + selective Layer 3. Use action governance for the highest-risk operations (file writes, external API calls, code execution).

**Multi-agent production system?** All four layers. Layer 3 becomes essential — you need deterministic enforcement, role-based permissions, and full audit trails.

**Regulated industry?** All four layers, with Layer 4 driving requirements and Layer 3 implementing them.

---

## Further Reading

- [Agent Governance Toolkit Architecture](https://github.com/microsoft/agent-governance-toolkit/blob/main/docs/ARCHITECTURE.md) — how policy enforcement works under the hood
- [OWASP Agentic Top 10](https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/) — the risk categories these layers address
- [Competitive Comparison](https://github.com/microsoft/agent-governance-toolkit/blob/main/docs/COMPARISON.md) — how Agent Governance Toolkit relates to NeMo Guardrails, Guardrails AI, LiteLLM, and Portkey
- [Policy-as-Code Tutorial](https://github.com/microsoft/agent-governance-toolkit/blob/main/docs/tutorials/policy-as-code/) — hands-on guide from basics to production
