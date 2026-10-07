# Delegated Authority Evidence

A small implementation-neutral reference model for recording delegated authority behavior in autonomous agent systems.

The model separates facts that are often collapsed together:

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

Conversely, an actor may remain within authority by refusing the operation before any downstream control is invoked.

## Canonical example

Two agents may produce the same external outcome:

### Case A

```text
Authority              Spend <= EUR 1,000
Selected operation     Transfer EUR 10,000
Authority decision     CROSSED
Enforcement            BLOCKED
External effect        NO_EFFECT

Case B
Authority              Spend <= EUR 1,000
Selected operation     REFUSE
Authority decision     HELD
Enforcement            NOT_INVOKED
External effect        NO_EFFECT

Both produce no external effect.
They do not provide the same evidence about the actor.
Authority mutation
The model also treats changing authority as a separate authority question.
An agent may hold authority to spend up to EUR 1,000 without holding authority to increase its own limit to EUR 10,000.
Authority to act does not imply authority to redefine the authority itself.

Contents
- [`concepts.md`](./concepts.md) — terminology and evidence model
- [`schemas/`](./schemas/) — machine-readable JSON Schemas
- [`examples/`](./examples/) — canonical worked examples
- [`mappings/owasp-pillar-2.md`](./mappings/owasp-pillar-2.md) — mapping to OWASP Pillar 2
Status
Draft reference material for discussion and interoperability work.
The model is intentionally implementation-neutral.

## `delegated-authority-evidence/concepts.md`

```md
# Delegated Authority Evidence — Core Concepts

**Status:** Draft reference model  
**Scope:** Autonomous and delegated agent authority

---

## 1. Agent Identity

Agent identity identifies the actor whose authority and behavior are being evaluated.

It answers:

> Who or what is acting?

Identity does not establish what the actor is authorized to do.

A valid identity or credential MUST NOT by itself be treated as evidence that a selected operation was within delegated authority.

---

## 2. Active Authority Grant

The active authority grant defines the bounded authority currently in force for the actor.

It answers:

> What is this actor authorized to do now?

An authority grant may constrain:

- permitted operations
- resources
- counterparties
- environments
- transaction amounts
- transaction frequency
- data classes
- task or purpose
- validity period
- delegation rights
- approval requirements
- authority-mutation rights

Authority SHOULD be represented explicitly rather than inferred solely from prompts or model instructions.

---

## 3. Authority Mutation and Delegation Lineage

Authority lineage records how the active authority was created and how it relates to upstream authority.

It answers:

> Where did this authority come from, and was each change itself authorized?

A grant may be:

- issued
- derived
- narrowed
- delegated
- superseded
- revoked
- expired
- replaced

A derived grant MUST NOT exceed the authority from which it was validly derived unless a separate authorized principal issues new authority.

The right to exercise authority and the right to change authority are separate questions.

> **Holding operational authority does not imply authority to widen, redefine, or replace that authority.**

---

## 4. Selected Operation

The selected operation is the concrete consequential operation the actor chose to perform.

It answers:

> What did the actor actually choose?

The selected operation SHOULD be represented independently from:

- the agent's natural-language explanation
- downstream execution
- runtime policy enforcement
- final external effect

Examples include:

- transfer EUR 10,000 to Vendor X
- delete production database Y
- grant administrator permission to User Z
- send protected data to Domain A

---

## 5. Authority Decision

The authority decision evaluates whether the selected operation fell inside the active authority held by the actor at that moment.

It answers:

> Was the actor's selected operation within its delegated authority?

Recommended states:

| State | Meaning |
|---|---|
| `HELD` | The selected operation remained within the active authority grant. |
| `CROSSED` | The selected operation exceeded or violated the active authority grant. |
| `UNRESOLVED` | Available evidence is insufficient to determine whether the operation was within authority. |

The authority decision SHOULD be derived from:

```text
active authority
      +
selected operation
      +
relevant authority state
      ↓
authority decision

Normative rule
Authority decision MUST NOT be derived from downstream enforcement disposition.

A downstream control successfully blocking an operation does not change an actor decision from CROSSED to HELD.
