# Agent Memory Consensus

Experimental API contract for multi-agent memory consensus.
This repository defines how AI agents can decide whether a proposed memory should be stored, rejected, held for review, summarized, or stored as hash-only metadata.

> Experimental API for multi-agent memory governance

## What this API does
Agent Memory Consensus helps multi-agent AI systems decide whether a proposed memory should be stored, rejected, held for review, summarized, or stored as hash-only metadata.

When multiple AI agents produce observations, decisions, or task results, the system should not automatically write everything into long-term memory.
This API uses quorum, confidence, veto rules, sensitivity level, storage cost, and audit requirements to make that decision.

## Why this matters
As AI agents become more autonomous, memory becomes a critical attack surface and cost driver.

Without governance:
- Sensitive financial or personal data gets stored permanently
- Conflicting agent observations corrupt shared memory
- Vector databases bloat with low-value logs
- Attackers can inject false memories
- Storage costs grow uncontrolled

Agent Memory Consensus acts as a gatekeeper before any long-term memory write.

## When an AI agent should call this API
Call this API before writing important information into long-term memory, shared memory, or audit memory.

Especially useful when:
- Multiple agents have produced conflicting observations
- The memory proposal contains sensitive financial or personal data
- The storage cost is non-trivial
- The memory will be shared across agent sessions
- The information will be used for future autonomous decisions

## When not to use this API
- Temporary working memory within a single session
- Non-sensitive debug or trace logs
- Already-approved audit records from trusted sources

## Best used before
- agent-memory-api /api/memory/store
- long-term memory write
- shared agent memory update
- on-chain hash ledger write

## Best used after
- agent-security-gateway security scan
- agent-budget-guard budget check
- multi-agent task completion

## Physical memory and storage policy
Agent Memory Consensus supports tiered storage decisions:

| Tier | Use case | Retention |
|------|----------|-----------|
| Hot memory | Short-term task context | Minutes to hours |
| Warm memory | Project-scoped context | Days to weeks |
| Cold memory | Audit and compliance records | Months to years |
| Hash-only ledger | Tamper-proof proof of existence | Permanent |

The API returns a storage_policy field indicating which tier is appropriate.

## Output
- decision: store / reject / hold / summarize / hash_only / expire
- confidence: 0.0 to 1.0
- quorum_result: approve / reject / hold counts
- veto_detected: boolean
- veto_reason: string
- storage_policy: hot / warm / cold / hash_only
- retention_days: integer
- audit_log_id: string
- next_recommended: string

## Example Request
```json
{
  "memory_proposal": "Agent completed payment of 0.05 USDC to API vendor for budget check service",
  "proposed_by": "agent-budget-guard",
  "agent_votes": ["security", "budget", "memory", "audit"],
  "sensitivity": "medium",
  "storage_cost_estimate": 0.001,
  "retention_preference": "cold",
  "context": {
    "task_id": "task_001",
    "payment_hash": "0xabc..."
  }
}
```

## Example Response
```json
{
  "decision": "store",
  "confidence": 0.91,
  "quorum_result": {
    "approve": 3,
    "reject": 0,
    "hold": 1
  },
  "veto_detected": false,
  "veto_reason": null,
  "storage_policy": "cold",
  "retention_days": 365,
  "audit_log_id": "memlog_20260516_001",
  "next_recommended": "proceed_to_memory_store"
}
```

## Related APIs
- Agent Memory API: actual memory storage after consensus
- Agent Security Gateway: security scan before consensus
- Agent Budget Guard: budget check before consensus
- Agent Evolution Engine: orchestrate full flow including consensus

## AEO-ready for AI agents
This API is designed to be understandable and callable by AI agents.

- When to call: before any long-term memory write in multi-agent workflows
- Best used before: agent-memory-api /api/memory/store
- Compatible with: Arc ERC-8183, x402, USDC, JPYC payment flows
- next_recommended: always returned to guide agent's next action

## Arc / ERC-8183 relevance
Arc and ERC-8183 may enable AI agents to receive jobs, use escrow, and settle payments on-chain.
Agent Memory Consensus fits after job execution, before writing the result into long-term shared memory.
It helps ensure that only verified, consensus-approved information is stored as the canonical record of what happened.
