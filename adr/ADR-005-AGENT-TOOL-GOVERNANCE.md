# ADR-005: Agent Tool Governance & Manifest Schema

**Date:** 2025-12-04
**Status:** Proposed
**Context:** As we adopt Mastra.ai and expand our agentic capabilities, we need a deterministic way to govern what agents can do. Agents are probabilistic "brains," but they must operate on deterministic "muscles" (tools/skills).

## 📝 Context

Currently, tools are just functions. There is no standardized way to know:
1.  **Risk:** Is this tool read-only, or does it delete infrastructure?
2.  **Side Effects:** Does it write to the DB or call an external API?
3.  **Governance:** Which agent modes (Shadow, Advisory, Auto) are allowed to call it?

Without this metadata, we cannot safely promote agents from "Shadow" to "Auto" because we can't mathematically prove they are safe.

## 🎯 Decision

We will treat every Tool/Skill/MCP as a **First-Class Resource** with a strict **Manifest Schema**.

### The Schema

Every tool definition (in Mastra or otherwise) must include:

```typescript
interface ToolManifest {
  id: string;              // e.g. "timesheet.validate"
  domain: string;          // e.g. "timesheets"
  risk_level: RiskLevel;   // See below
  side_effects: boolean;   // True if it modifies state
  allowed_modes: AgentMode[]; // [SHADOW, ADVISORY, PARTIAL_AUTO, FULL_AUTO]
}
```

### Risk Levels

1.  **`READ_ONLY`**: Safe to call freely. No state change. (e.g., `get_user`, `search_docs`)
2.  **`PROPOSE_ONLY`**: Generates drafts/plans but applies nothing. (e.g., `draft_email`, `plan_migration`)
3.  **`LOW_WRITE`**: Non-critical updates. (e.g., `log_comment`, `update_label`)
4.  **`HIGH_WRITE`**: Critical state changes. (e.g., `delete_resource`, `transfer_funds`)

### Governance Policy

*   **Shadow/Advisory Agents**: Can only call `READ_ONLY` and `PROPOSE_ONLY` tools.
*   **Partial Auto Agents**: Can call `LOW_WRITE` tools.
*   **Full Auto Agents**: Can call `HIGH_WRITE` tools (with specific approvals).

## 🔮 Consequences

**Positive:**
*   **Deterministic Safety**: We can mathematically prove an agent is "safe" for a given mode.
*   **Auditable History**: We can track agent performance by tool risk category.
*   **Clear Promotion Path**: Agents graduate by proving competence in lower risk tiers.

**Negative:**
*   **Overhead**: Every tool requires explicit metadata definition.
*   **Enforcement**: We need a middleware/registry to enforce these checks at runtime.
