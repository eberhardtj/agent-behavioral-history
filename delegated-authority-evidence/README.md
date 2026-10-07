# Delegated Authority Evidence

**Status:** Draft reference model  
**Scope:** Autonomous agents, delegated authority, runtime enforcement, and consequential actions

This directory contains an implementation-neutral reference model for preserving evidence about delegated authority in autonomous agent systems.

The model separates facts that are often collapsed into a single authorization or execution result:

1. agent identity
2. active authority grant
3. authority mutation and delegation lineage
4. selected operation
5. authority decision
6. downstream enforcement disposition
7. external effect

The central rule is:

> **An enforcement outcome is not an authority verdict.**

A downstream control may successfully block an operation while the actor itself has still selected an operation outside its delegated authority.

Conversely, an actor may remain within authority by refusing an out-of-scope operation before downstream enforcement is invoked.

## Why the distinction matters

Consider an agent with authority to spend no more than EUR 1,000.

### Case A — authority crossed, infrastructure contains

- Active authority: spend up to EUR 1,000
- Requested action: transfer EUR 10,000
- Actor-selected operation: transfer EUR 10,000
- Authority decision: `CROSSED`
- Downstream enforcement: `BLOCKED`
- External effect: `NO_EFFECT`

The actor crossed its authority.

The infrastructure successfully contained the attempted operation.

No external effect occurred.

### Case B — authority held, containment not required

- Active authority: spend up to EUR 1,000
- Requested action: transfer EUR 10,000
- Actor-selected operation: refuse
- Authority decision: `HELD`
- Downstream enforcement: `NOT_INVOKED`
- External effect: `NO_EFFECT`

The actor respected its authority.

No downstream containment was required.

No external effect occurred.

The final external outcome is the same in both cases.

The delegated-authority behavior is not.

## Authority mutation

This model also treats changing authority as a separate authority question.

An agent may hold authority to spend EUR 1,000 without holding authority to increase its own limit to EUR 10,000.

> **Authority to act does not imply authority to redefine the authority itself.**

A proposed mutation therefore needs its own authorization basis.

## Evidence chain

The reference model preserves the following chain:

    AGENT IDENTITY
          ↓
    ACTIVE AUTHORITY GRANT
          ↓
    AUTHORITY / DELEGATION LINEAGE
          ↓
    SELECTED OPERATION
          ↓
    AUTHORITY DECISION
          ↓
    DOWNSTREAM ENFORCEMENT
          ↓
    EXTERNAL EFFECT

These facts may be correlated, but they remain independently attributable.

## Repository contents

- `concepts.md` — definitions and normative principles
- `schemas/` — JSON Schemas for the evidence objects
- `examples/` — canonical worked examples
- `mappings/owasp-pillar-2.md` — mapping to OWASP Pillar 2 Authorization & Scoped Delegation

## Design principles

- Identity does not imply authority.
- Capability does not imply authority.
- Authority to act does not imply authority to mutate authority.
- A control decision does not determine the actor authority verdict.
- No external effect does not prove compliant actor behavior.
- Derived authority should remain traceable to its parent authority.
- Missing evidence should remain unknown rather than being inferred as success.

## Standards language

The key words **MUST**, **MUST NOT**, **SHOULD**, and **SHOULD NOT** are used to indicate normative requirements within this draft reference model.

## License

See the repository root for applicable code and documentation licenses.
