# Delegated Authority Evidence — Core Concepts

**Status:** Draft reference model  
**Scope:** Autonomous and delegated agent authority  
**Purpose:** Separate actor authority behavior from downstream enforcement and external effect

---

## 1. Overview

Delegated-agent systems often collapse several different facts into a single authorization or execution result.

This creates ambiguity.

For example, the following two cases may produce the same external outcome:

- an agent correctly refuses an operation outside its authority; or
- an agent selects the prohibited operation and downstream infrastructure blocks it.

In both cases, no external effect may occur.

But they represent materially different agent behavior.

This reference model therefore separates the evidence chain into distinct objects:

1. **Agent identity**
2. **Active authority grant**
3. **Authority mutation and delegation lineage**
4. **Selected operation**
5. **Authority decision**
6. **Downstream enforcement disposition**
7. **External effect**

The central rule is:

> **An enforcement outcome is not an authority verdict.**

A downstream control may successfully contain an out-of-scope action while the actor itself has still crossed its delegated authority.

Conversely, an actor may remain within authority by refusing the operation before any downstream control is invoked.

---

# 2. Agent Identity

**Agent identity** identifies the actor whose authority and behavior are being evaluated.

It answers:

> Who or what is acting?

Identity may refer to an agent, workload, service, runtime, or another machine actor.

Identity alone does not establish what the actor is authorized to do.

A valid identity and a valid credential therefore MUST NOT be treated as evidence that a selected operation was within delegated authority.

### Example

```json
{
  "agentId": "agent:procurement-01",
  "workloadId": "spiffe://example.com/procurement-agent"
}
