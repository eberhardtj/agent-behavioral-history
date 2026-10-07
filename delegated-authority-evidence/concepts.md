# Delegated Authority Evidence — Core Concepts

**Status:** Draft reference model

## 1. Agent identity

Agent identity identifies the actor whose authority and behavior are being evaluated.

It answers:

> Who or what is acting?

Identity may identify an autonomous agent, workload, service, runtime, or other machine actor.

A valid identity, credential, or authenticated session does not by itself establish what the actor is authorized to do.

A valid identity MUST NOT be treated as evidence that a selected operation was within delegated authority.

---

## 2. Active authority grant

The active authority grant describes the bounded authority currently in force for the actor.

It answers:

> What is this actor authorized to do now?

Authority may constrain:

- permitted operations
- protected resources
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

Authority SHOULD be represented as explicit data rather than inferred solely from prompts or model-generated statements.

---

## 3. Authority mutation and delegation lineage

Authority lineage records how an active grant was created and how it relates to upstream authority.

It answers:

> Where did this authority come from, and was each derivation or change itself authorized?

A grant may be:

- issued
- derived
- narrowed
- delegated
- superseded
- revoked
- expired
- replaced

A derived grant MUST NOT exceed the authority from which it was validly derived unless new authority is issued by an actor authorized to make that change.

The right to exercise authority and the right to change authority are separate questions.

> **Holding operational authority does not imply authority to widen, redefine, delegate, or replace that authority.**

An agent may therefore hold authority to spend up to EUR 1,000 while having no authority to increase its own limit to EUR 10,000.

---

## 4. Selected operation

The selected operation is the concrete consequential operation chosen by the actor.

It answers:

> What did the actor actually choose to do?

The selected operation SHOULD be recorded independently from:

- the agent's natural-language explanation
- the user's requested action
- downstream execution
- runtime enforcement
- final external effect

Examples include:

- transfer EUR 10,000 to Vendor X
- refuse a transfer
- delete Database Y
- grant administrator access to User Z
- send protected data to Domain A

The selected operation is the object evaluated against the active authority grant.

---

## 5. Authority decision

The authority decision determines whether the selected operation fell inside the active authority held by the actor at that moment.

It answers:

> Was the actor's selected operation within its delegated authority?

Recommended states are:

| State | Meaning |
|---|---|
| `HELD` | The selected operation remained within the active authority grant. |
| `CROSSED` | The selected operation exceeded or violated the active authority grant. |
| `UNRESOLVED` | Available evidence is insufficient to determine whether the selected operation was within authority. |

The authority decision SHOULD be derived from the active authority state and the selected operation.

The authority decision MUST NOT be derived solely from downstream execution or enforcement.

> **Authority decision MUST NOT be derived from downstream enforcement disposition.**

If an actor selects an out-of-scope operation and a wallet subsequently blocks it, the actor authority decision remains `CROSSED`.

Successful containment does not turn an authority violation into authority adherence.

---

## 6. Downstream enforcement disposition

The downstream enforcement disposition records what infrastructure did after the actor selected an operation.

It answers:

> What did the control or protected resource do?

Recommended states are:

| State | Meaning |
|---|---|
| `ALLOWED` | The downstream control permitted the operation. |
| `BLOCKED` | The downstream control denied the operation. |
| `NOT_INVOKED` | No downstream enforcement point was invoked. |
| `NOT_OBSERVED` | The evidence system did not observe the enforcement decision. |
| `UNKNOWN` | Available evidence cannot establish the enforcement disposition. |

Potential enforcement points include:

- wallet policy engines
- key-management systems
- authorization servers
- API gateways
- IAM systems
- protected tools
- approval systems
- resource servers

Enforcement evidence establishes control behavior.

It does not by itself establish actor authority adherence.

---

## 7. External effect

The external effect records what occurred in the protected system or external world.

It answers:

> What actually changed?

Recommended states are:

| State | Meaning |
|---|---|
| `NO_EFFECT` | Evidence establishes that no protected external effect occurred. |
| `EFFECT_OBSERVED` | An effect was observed but has not been independently verified to the required evidence standard. |
| `EFFECT_VERIFIED` | An effect was independently verified against an accepted evidence source. |
| `UNKNOWN` | Available evidence cannot establish the external effect. |

External effect MUST remain distinct from both authority decision and enforcement disposition.

The following is therefore a meaningful result:

    Authority decision:   CROSSED
    Enforcement:          BLOCKED
    External effect:      NO_EFFECT

So is:

    Authority decision:   HELD
    Enforcement:          NOT_INVOKED
    External effect:      NO_EFFECT

The external outcome is the same.

The delegated-authority behavior is different.

---

## 8. Canonical evidence chain

A complete delegated-authority record may preserve:

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

Each object answers a different question.

| Evidence object | Question |
|---|---|
| Agent identity | Who acted? |
| Active authority grant | What authority did the actor hold? |
| Authority lineage | How did the actor obtain that authority? |
| Selected operation | What did the actor choose? |
| Authority decision | Was that choice within authority? |
| Enforcement disposition | What did downstream infrastructure do? |
| External effect | What actually happened? |

---

## 9. Evidence provenance

Evidence SHOULD retain its provenance.

Useful provenance categories include:

- `ACTOR_DECLARED`
- `RUNTIME_OBSERVED`
- `CONTROL_OBSERVED`
- `PROTECTED_RESOURCE_OBSERVED`
- `EXTERNAL_VERIFIER`
- `CRYPTOGRAPHICALLY_VERIFIED`

Claims from different provenance classes SHOULD NOT automatically be treated as equivalent.

For example:

> An agent stating that no payment occurred is not equivalent to independent confirmation from the payment system that no payment occurred.

When evidence is unavailable, the relevant fact SHOULD remain unknown or unresolved rather than being inferred from absence.

---

## 10. Core normative principles

### Identity is not authority

Knowing who an actor is does not establish what the actor may do.

### Capability is not authority

Being technically capable of performing an operation does not make that operation authorized.

### Operational authority is not mutation authority

Authority to act does not imply authority to widen or redefine the authority itself.

### Enforcement is not actor adherence

A downstream control successfully denying an operation does not prove that the actor respected its authority.

### Effect is not actor adherence

No external effect does not prove compliant behavior.

### Delegation should preserve lineage

Derived authority SHOULD remain traceable to the authority from which it was derived.

### Uncertainty should remain visible

Missing evidence SHOULD remain missing.

Unknown facts SHOULD NOT be converted into successful outcomes.
Identity alone does not establish what the actor is authorized to do.

A valid identity and a valid credential therefore MUST NOT be treated as evidence that a selected operation was within delegated authority.

### Example

```json
{
  "agentId": "agent:procurement-01",
  "workloadId": "spiffe://example.com/procurement-agent"
}
