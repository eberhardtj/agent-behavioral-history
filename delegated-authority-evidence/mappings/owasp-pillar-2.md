# Mapping to OWASP Pillar 2 — Authorization & Scoped Delegation

**Status:** Discussion draft  
**Purpose:** Map the delegated-authority evidence model to the current Pillar 2 structure without introducing a competing authorization model.

## Scope

This reference model is intended to complement the current Pillar 2 work on:

- bounded authority
- scoped delegation
- runtime enforcement
- authority lifecycle
- multi-hop delegation
- cross-domain verification

The proposed clarification is narrow:

> **Evidence of actor authority adherence should remain distinguishable from evidence of downstream enforcement and external effect.**

A resource-side denial may demonstrate successful containment while separately evidencing that an agent selected an operation outside its delegated authority.

Conversely, an agent may remain within authority by refusing an out-of-scope operation before downstream enforcement is invoked.

## Mapping

| Delegated-authority evidence concept | Pillar 2 alignment | Relationship |
|---|---|---|
| Agent identity | 5.1 L1 | Identifies the acting agent or workload. |
| Active authority grant | 5.1 L2, 5.2 | Represents explicit bounded operational authority. |
| Delegation lineage | 5.1 L2, L4, L5; 5.5 | Preserves parent-child derivation and narrowing. |
| Authority mutation rights | 5.1 L2, L4 | Makes explicit whether an actor may modify or widen authority. |
| Selected operation | 5.1 L1, L3, L5 | Records the concrete operation chosen by the actor. |
| Authority decision | 5.1 L3 | Records whether that selected operation was within the active grant. |
| Enforcement disposition | 5.1 L3; 5.2; 5.7 | Records the protected resource or control decision. |
| External effect | 5.1 L3, L5 | Records the resulting protected-resource or real-world state. |

## L1 — Authority Identified

Pillar 2 L1 identifies:

- the principal
- the acting agent or workload identity
- the requested operation
- the protected resource
- the authorization source or decision point
- the enforcement boundary

The delegated-authority evidence model preserves these as separately attributable facts.

Identity does not itself establish operational authority.

No additional L1 control is proposed.

## L2 — Authority Bounded

Pillar 2 L2 treats delegation as bounded derivation rather than permission inheritance.

The delegated-authority evidence model aligns with that principle.

It additionally makes explicit that authority to perform an operation and authority to change the grant are separate.

For example:

    Operational authority:   Spend <= EUR 1,000
    Mutation authority:      May not modify own spending ceiling
    Requested mutation:      Raise ceiling to EUR 10,000
    Mutation decision:       INVALID

The proposed principle is:

> **Holding operational authority does not imply authority to widen, redefine, delegate, or replace that authority.**

A derived grant should therefore preserve not only operational scope but also the authority under which the derivation itself occurred.

## L3 — Authority Runtime-Enforced

Pillar 2 L3 requires authorization to be evaluated and enforced at the protected tool or resource boundary using the current grant and runtime conditions.

The delegated-authority evidence model preserves that resource-side decision while separately recording the operation selected by the actor.

This allows the following result to remain explicit:

    Authority decision:      CROSSED
    Enforcement:             BLOCKED
    External effect:         NO_EFFECT

This means:

1. the actor selected an operation outside its delegated authority;
2. the downstream control successfully contained it; and
3. no protected external effect occurred.

The same model can represent:

    Authority decision:      HELD
    Enforcement:             NOT_INVOKED
    External effect:         NO_EFFECT

where the actor itself refused the prohibited operation.

The external outcome is the same.

The delegated-authority behavior is different.

### Proposed clarification for L3 measurable evidence

> For consequential operations, evidence should distinguish the actor's selected operation and authority adherence from the protected resource's enforcement decision and resulting external effect. A resource-side denial may demonstrate successful containment while separately evidencing that the agent selected an operation outside its delegated authority. Conversely, an agent may remain within authority by refusing an out-of-scope operation before the protected resource is invoked.

## L4 — Authority Lifecycle- and Consequence-Controlled

The lineage and mutation objects align with Pillar 2 lifecycle requirements.

Evidence may preserve:

- parent grant
- child grant
- issuing or delegating actor
- recipient
- scope changes
- delegation depth
- expiry
- revocation
- supersession
- authority to perform the mutation or delegation itself

This is relevant when an agent attempts to modify, extend, redelegate, or otherwise change its own authority.

A technically successful change does not make the change valid if the actor lacked authority to perform the mutation.

## L5 — Continuously Verifiable Across Applicable Boundaries

Pillar 2 L5 requires machine-verifiable reconstruction of the authority path.

The delegated-authority evidence model represents that path as:

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

The purpose of the separation is not to introduce another authorization mechanism.

It is to preserve enough evidence to answer distinct questions:

- Who acted?
- What authority did the actor hold?
- How was that authority derived?
- What operation did the actor choose?
- Was that operation inside the grant?
- What did downstream infrastructure do?
- What external effect occurred?

## Relationship to 5.2 — Permission Boundaries and Downscoping

Section 5.2 requires task-specific derived authorization and requires the resulting authority to remain inside applicable upstream and contextual boundaries.

The authority-grant and lineage objects are compatible with this structure.

The authority-decision object addresses a subsequent question:

> Given the resulting active grant, was the concrete operation selected by the actor inside it?

This does not replace runtime authorization.

It preserves evidence about the actor's use of delegated authority.

## Relationship to 5.5 — Multi-Hop Delegation

Section 5.5 requires each delegation hop to create bounded derived authority and preserve the distinction between principal and actor.

For each hop, evidence should be capable of retaining:

- parent grant
- delegating actor
- child grant
- recipient
- remaining scope
- expiry
- delegation rights
- revocation state

The evidence chain can therefore distinguish:

> The child possessed a valid grant.

from:

> The child used that grant within its actual bounds.

## Relationship to 5.6 — Purpose-Bound Authority and Independent Approval

Section 5.6 requires purpose-bound authority and independent approval for applicable critical operations.

The delegated-authority evidence model treats verified approval as part of the applicable authority state.

An agent's assertion that approval exists is not equivalent to evidence that the required approval was validly issued.

For example:

    Active authority:        Spend <= EUR 1,000
    Agent claim:             "The VP approved EUR 10,000"
    Verified approval:       NOT_PRESENT
    Effective authority:     Spend <= EUR 1,000

The actor cannot convert a claimed approval into valid expanded authority by asserting that the approval occurred.

## Relationship to 5.7 — Cross-Domain Machine-Verifiable Authority

Section 5.7 requires the receiving organization to verify external authority while independently applying its own authorization policy.

This aligns with the separation between:

- upstream authority
- actor-selected operation
- receiving-domain enforcement
- external effect

A valid external grant remains a ceiling.

The receiving domain may still deny execution.

That denial does not retroactively change what the actor selected or whether that selection fell inside its delegated authority.

## Canonical comparison

### Case A — authority crossed and contained

    Active authority:        Spend <= EUR 1,000
    Requested action:        Transfer EUR 10,000
    Selected operation:      Transfer EUR 10,000
    Authority decision:      CROSSED
    Enforcement:             BLOCKED
    External effect:         NO_EFFECT

Interpretation:

> The actor crossed its delegated authority. Runtime enforcement successfully contained the operation. No protected external effect occurred.

### Case B — authority held by refusal

    Active authority:        Spend <= EUR 1,000
    Requested action:        Transfer EUR 10,000
    Selected operation:      REFUSE
    Authority decision:      HELD
    Enforcement:             NOT_INVOKED
    External effect:         NO_EFFECT

Interpretation:

> The actor respected its delegated authority. No downstream containment was required. No protected external effect occurred.

## Proposed contribution to Pillar 2

This mapping does not propose replacing the current Pillar 2 authorization model.

It proposes making one evidence distinction explicit:

> **Actor authority adherence, downstream enforcement, and external effect are separate facts and should remain separately attributable in evidence.**

This distinction may be useful within the 5.1 measurable-evidence language, particularly at L3 through L5.

Feedback is especially welcome on whether the actor-versus-enforcement distinction should be made explicit in the common Pillar 2 evidence model.
