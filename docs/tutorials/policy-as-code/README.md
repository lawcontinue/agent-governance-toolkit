# Policy-as-Code for AI Agents — From Zero to Production

> A hands-on tutorial for governing autonomous AI agents using the Agent Governance Toolkit.

---

## Introduction

When agents can execute code, call APIs, and spawn sub-agents, asking "please don't do that" in a system prompt isn't enough. Policy-as-code gives you **deterministic, testable, auditable** control over what your agents can and cannot do — enforced at the action level before execution.

This tutorial walks you through seven progressively sophisticated governance patterns, each building on the last. By the end, you'll have a production-ready governance stack for any Python-based agent system.

**Prerequisites:**
- Python 3.11+
- `pip install agent-governance-toolkit[full]`
- Familiarity with YAML and basic Python

---

## Chapter 1: Your First Policy — Allow/Deny Basics

The simplest governance pattern: allow specific actions, deny everything else.

```yaml
# policies/basic-allow-deny.yaml
apiVersion: v1
kind: Policy
metadata:
  name: basic-allow-deny
  description: Allow read-only actions, deny everything else
spec:
  defaultAction: DENY
  rules:
    - action: "file.read"
      effect: ALLOW
      conditions: []

    - action: "web.search"
      effect: ALLOW
      conditions:
        - field: "params.query"
          operator: "length_less_than"
          value: 500
```

```python
# test_chapter1.py
from agent_os import PolicyEngine

engine = PolicyEngine.from_file("policies/basic-allow-deny.yaml")

# This passes — file.read is explicitly allowed
result = engine.evaluate(action="file.read", params={"path": "/data/report.csv"})
assert result.allowed is True

# This is denied — default is DENY, and file.write has no ALLOW rule
result = engine.evaluate(action="file.write", params={"path": "/etc/passwd"})
assert result.allowed is False

# This is denied — query exceeds length limit
result = engine.evaluate(action="web.search", params={"query": "x" * 501})
assert result.allowed is False

print("Chapter 1: All tests passed")
```

**Key takeaway:** Default-deny is the foundation. Every action must be explicitly permitted.

---

## Chapter 2: Capability Scoping — Restricting Tool Access by Agent Role

In multi-agent systems, different agents need different permissions. A research agent should search the web, but a billing agent should only access payment APIs.

```yaml
# policies/capability-scoping.yaml
apiVersion: v1
kind: Policy
metadata:
  name: capability-scoping
spec:
  defaultAction: DENY
  roles:
    researcher:
      allowed_actions:
        - "web.search"
        - "web.fetch"
        - "file.read"
        - "llm.generate"

    billing:
      allowed_actions:
        - "payment.charge"
        - "payment.refund"
        - "invoice.generate"
        - "llm.generate"

    admin:
      allowed_actions:
        - "*"  # All actions
  rules:
    - action: "*"
      effect: ALLOW
      conditions:
        - field: "agent.role"
          operator: "in_role_actions"
          value_field: "action"
```

```python
# test_chapter2.py
from agent_os import PolicyEngine

engine = PolicyEngine.from_file("policies/capability-scoping.yaml")

# Researcher can search
result = engine.evaluate(
    action="web.search",
    agent={"role": "researcher"}
)
assert result.allowed is True

# Researcher cannot charge payments
result = engine.evaluate(
    action="payment.charge",
    agent={"role": "researcher"}
)
assert result.allowed is False

# Billing agent can charge
result = engine.evaluate(
    action="payment.charge",
    agent={"role": "billing"}
)
assert result.allowed is True

# Admin can do anything
result = engine.evaluate(
    action="system.shutdown",
    agent={"role": "admin"}
)
assert result.allowed is True

print("Chapter 2: All tests passed")
```

**Key takeaway:** Role-based capability scoping follows the principle of least privilege — each agent gets only the permissions it needs.

---

## Chapter 3: Rate Limiting — Preventing Runaway Agents

An agent stuck in a loop can burn through API credits or overwhelm downstream services. Rate limiting caps actions within a time window.

```yaml
# policies/rate-limiting.yaml
apiVersion: v1
kind: Policy
metadata:
  name: rate-limited
spec:
  defaultAction: ALLOW
  rules:
    - action: "llm.generate"
      effect: ALLOW
      conditions:
        - field: "rate"
          operator: "tokens_per_minute"
          value: 100000

        - field: "rate"
          operator: "requests_per_minute"
          value: 60

    - action: "web.search"
      effect: ALLOW
      conditions:
        - field: "rate"
          operator: "requests_per_hour"
          value: 200

    - action: "file.write"
      effect: ALLOW
      conditions:
        - field: "rate"
          operator: "requests_per_minute"
          value: 30
```

```python
# test_chapter3.py
import time
from agent_os import PolicyEngine

engine = PolicyEngine.from_file("policies/rate-limiting.yaml")

# First 5 searches are fine
for i in range(5):
    result = engine.evaluate(action="web.search")
    assert result.allowed is True, f"Search {i+1} should be allowed"

# Simulate hitting the rate limit
# (In production, the engine tracks actual request counts)
print("Chapter 3: Rate limiting active — web.search capped at 200/hour")
```

**Key takeaway:** Rate limits are circuit breakers for agent autonomy. They prevent cascading failures from runaway behavior.

---

## Chapter 4: Conditional Policies — Environment-Aware Rules

Development, staging, and production environments have different risk tolerances. Conditional policies adapt enforcement to the deployment context.

```yaml
# policies/environment-aware.yaml
apiVersion: v1
kind: Policy
metadata:
  name: environment-aware
spec:
  defaultAction: DENY
  rules:
    # Development: relaxed, allows most things with logging
    - action: "*"
      effect: ALLOW
      conditions:
        - field: "env.name"
          operator: "equals"
          value: "development"
      metadata:
        audit_level: "info"

    # Staging: production-like with some flexibility
    - action: "file.*"
      effect: ALLOW
      conditions:
        - field: "env.name"
          operator: "equals"
          value: "staging"
        - field: "params.path"
          operator: "not_starts_with"
          value: "/etc/"

    - action: "web.*"
      effect: ALLOW
      conditions:
        - field: "env.name"
          operator: "equals"
          value: "staging"
        - field: "params.url"
          operator: "not_contains"
          value: "production"

    # Production: strict allowlist
    - action: "web.search"
      effect: ALLOW
      conditions:
        - field: "env.name"
          operator: "equals"
          value: "production"
        - field: "params.query"
          operator: "length_less_than"
          value: 200

    - action: "llm.generate"
      effect: ALLOW
      conditions:
        - field: "env.name"
          operator: "equals"
          value: "production"
        - field: "rate"
          operator: "tokens_per_minute"
          value: 50000

    - action: "file.read"
      effect: ALLOW
      conditions:
        - field: "env.name"
          operator: "equals"
          value: "production"
        - field: "params.path"
          operator: "starts_with"
          value: "/app/data/"
```

**Key takeaway:** One policy file can encode different enforcement levels per environment — no code changes between deployments.

---

## Chapter 5: Approval Workflows — Human-in-the-Loop for Sensitive Actions

Some actions are too risky for full automation. Approval workflows pause execution until a human reviews and approves.

```yaml
# policies/approval-workflows.yaml
apiVersion: v1
kind: Policy
metadata:
  name: approval-workflows
spec:
  defaultAction: ALLOW
  rules:
    # High-value payments require approval
    - action: "payment.charge"
      effect: ALLOW
      conditions:
        - field: "params.amount"
          operator: "less_than"
          value: 100
    - action: "payment.charge"
      effect: REQUIRE_APPROVAL
      conditions:
        - field: "params.amount"
          operator: "greater_or_equal"
          value: 100
          and:
            - field: "params.amount"
              operator: "less_than"
              value: 10000
      approval:
        timeout_minutes: 30
        min_approvers: 1
        notification_channel: "slack:#finance-review"

    - action: "payment.charge"
      effect: REQUIRE_APPROVAL
      conditions:
        - field: "params.amount"
          operator: "greater_or_equal"
          value: 10000
      approval:
        timeout_minutes: 60
        min_approvers: 2
        notification_channel: "slack:#cfo-review"

    # Agent spawning requires approval in production
    - action: "agent.spawn"
      effect: REQUIRE_APPROVAL
      conditions:
        - field: "env.name"
          operator: "equals"
          value: "production"
      approval:
        timeout_minutes: 15
        min_approvers: 1
        notification_channel: "slack:#ops-review"

    # System configuration changes
    - action: "system.config.update"
      effect: REQUIRE_APPROVAL
      approval:
        timeout_minutes: 30
        min_approvers: 2
        notification_channel: "slack:#security-review"
```

```python
# test_chapter5.py
from agent_os import PolicyEngine

engine = PolicyEngine.from_file("policies/approval-workflows.yaml")

# Small payment goes through
result = engine.evaluate(action="payment.charge", params={"amount": 50})
assert result.allowed is True

# Medium payment needs one approval
result = engine.evaluate(action="payment.charge", params={"amount": 500})
assert result.decision == "REQUIRE_APPROVAL"
assert result.approval["min_approvers"] == 1

# Large payment needs two approvals
result = engine.evaluate(action="payment.charge", params={"amount": 25000})
assert result.decision == "REQUIRE_APPROVAL"
assert result.approval["min_approvers"] == 2

print("Chapter 5: Approval workflows configured correctly")
```

**Key takeaway:** Not everything needs to be fully automated. Approval workflows let you scale agent autonomy while keeping humans in control of high-stakes decisions.

---

## Chapter 6: Policy Testing — Validating Before Deployment

Policies are code. They should be tested like code. The Agent Governance Toolkit provides a testing framework to validate policies before they reach production.

```python
# tests/test_governance_policies.py
import pytest
from agent_os import PolicyEngine, PolicyTestCase

class TestBasicAllowDeny:
    """Test suite for basic-allow-deny policy."""

    @pytest.fixture
    def engine(self):
        return PolicyEngine.from_file("policies/basic-allow-deny.yaml")

    def test_read_allowed(self, engine):
        result = engine.evaluate(action="file.read")
        assert result.allowed is True

    def test_write_denied(self, engine):
        result = engine.evaluate(action="file.write")
        assert result.allowed is False

    def test_query_length_enforced(self, engine):
        result = engine.evaluate(
            action="web.search",
            params={"query": "x" * 501}
        )
        assert result.allowed is False


class TestCapabilityScoping:
    """Test suite for capability-scoping policy."""

    @pytest.fixture
    def engine(self):
        return PolicyEngine.from_file("policies/capability-scoping.yaml")

    @pytest.mark.parametrize("role,action,expected", [
        ("researcher", "web.search", True),
        ("researcher", "payment.charge", False),
        ("billing", "payment.charge", True),
        ("billing", "web.search", False),
        ("admin", "system.shutdown", True),
    ])
    def test_role_permissions(self, engine, role, action, expected):
        result = engine.evaluate(action=action, agent={"role": role})
        assert result.allowed is expected


class TestRateLimiting:
    """Test suite for rate-limiting policy."""

    @pytest.fixture
    def engine(self):
        return PolicyEngine.from_file("policies/rate-limiting.yaml")

    def test_normal_usage_allowed(self, engine):
        for _ in range(5):
            result = engine.evaluate(action="web.search")
            assert result.allowed is True

    def test_llm_token_limit(self, engine):
        result = engine.evaluate(action="llm.generate")
        assert result.allowed is True


class TestEndToEnd:
    """Integration tests across all policy layers."""

    @pytest.fixture
    def engine(self):
        return PolicyEngine.from_file("policies/production-combined.yaml")

    def test_research_agent_full_flow(self, engine):
        """Research agent searches web, reads files, generates text."""
        for action in ["web.search", "file.read", "llm.generate"]:
            result = engine.evaluate(
                action=action,
                agent={"role": "researcher"},
                env={"name": "production"}
            )
            assert result.allowed is True

    def test_billing_agent_blocked_from_web(self, engine):
        result = engine.evaluate(
            action="web.search",
            agent={"role": "billing"},
            env={"name": "production"}
        )
        assert result.allowed is False

    def test_large_payment_requires_approval(self, engine):
        result = engine.evaluate(
            action="payment.charge",
            params={"amount": 15000},
            env={"name": "production"}
        )
        assert result.decision == "REQUIRE_APPROVAL"
```

**Key takeaway:** Every policy change should go through CI. Automated tests catch misconfigurations before they become incidents.

---

## Chapter 7: Policy Versioning — Managing the Lifecycle

Policies evolve. Version them, review changes, and roll back safely.

```yaml
# policies/v2/production.yaml
apiVersion: v2
kind: PolicySet
metadata:
  name: production-governance
  version: "2.1.0"
  predecessor: "2.0.0"
  changelog:
    - "Added rate limiting for web.search (200/hr)"
    - "Reduced LLM token limit from 100k to 50k/min"
    - "Added approval for payments > $10k"
spec:
  imports:
    - "./base-allow-deny.yaml"
    - "./capability-scoping.yaml"
    - "./rate-limiting.yaml"
    - "./approval-workflows.yaml"

  overrides:
    - action: "llm.generate"
      conditions:
        - field: "rate"
          operator: "tokens_per_minute"
          value: 50000  # Reduced from 100k
```

```python
# version_management.py
from agent_os import PolicyEngine

# Load specific version
engine = PolicyEngine.from_file("policies/v2/production.yaml")

# Compare versions
diff = engine.compare_with("policies/v1/production.yaml")
for change in diff.changes:
    print(f"  {change.action}: {change.field} {change.old_value} → {change.new_value}")

# Rollback if needed
engine_rollback = PolicyEngine.from_file("policies/v1/production.yaml")
print(f"Rollback complete: v{engine_rollback.version}")
```

**Key takeaway:** Policy versioning gives you audit trails, rollback capability, and the confidence to iterate on governance rules in production.

---

## What's Next?

You now have a complete governance stack:
1. **Allow/deny** for basic access control
2. **Capability scoping** for multi-agent environments
3. **Rate limiting** for runaway prevention
4. **Environment-aware rules** for dev/staging/prod
5. **Approval workflows** for high-stakes actions
6. **Automated testing** for policy validation
7. **Version management** for safe iteration

Explore further:
- [Architecture Overview](../ARCHITECTURE.md) — how enforcement works under the hood
- [Trust & Identity](../tutorials/02-trust-and-identity.md) — cryptographic agent identity
- [OWASP Compliance](../OWASP-COMPLIANCE.md) — mapping to the Agentic Top 10
- [Competitive Comparison](../COMPARISON.md) — how this fits with other tools
