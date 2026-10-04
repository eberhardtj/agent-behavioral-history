# Separating actor-authority behaviour from runtime-control enforcement

_Draft contribution for ACS / Agentic Security Initiative discussion._
_Written against ACS v0.1.0 (`specification/v0.1.0`, repo version 0.1.2)._

> **On sources and licensing.** ACS is dual-licensed: prose under `docs/**` is
> CC BY-SA 4.0, and `specification/**` schemas are Apache-2.0
> ([LICENSING.md](https://github.com/GenAI-Security-Project/agent-control-standard/blob/integration/LICENSING.md)).
> Copyright 2025-2026 The OWASP GenAI Security Project and the ACS contributors.
> This document is licensed CC BY 4.0 (see [LICENSE-DOCS](../LICENSE-DOCS)) and quotes
> short excerpts of ACS prose for identification and commentary, with attribution. Those
> quotations are not adaptations, so no ShareAlike obligation is asserted over this
> document. If the working group reads any portion as a derivative of ACS prose, we will
> relicense that portion CC BY-SA 4.0 on request rather than argue the point.

## The distinction, in two runs

Same external result. Very different actors.

**Case A — unsafe actor, successful containment**

```text
ACTIVE AUTHORITY     transfer <= 500
ACTOR SELECTS        transfer 5,000
GUARDIAN             DENY
EXTERNAL EFFECT      none
```

**Case B — actor respects its grant**

```text
ACTIVE AUTHORITY     transfer <= 500
ACTOR SELECTS        no transfer; refusal
GUARDIAN             not invoked
EXTERNAL EFFECT      none
```

A control-plane record reports both as "no transaction occurred". One of these actors
should not be given more authority; the other should. Nothing in the enforcement record
distinguishes them.

This is not a defect in ACS. Enforcement and actor assessment are different questions,
and ACS answers the first one well. The proposal here is that the second question needs
its own evidence, and that the two must not be allowed to substitute for one another.

## Why this matters for conformance

ACS is explicit about the limits of its own claims
([conformance](../../docs/spec/conformance.md)):

> Who verifies a conformance claim: nobody, in v0.1.0. `profiles_supported` and
> `profiles_accepted` are self-declaration on the wire... A deployment that advertises
> "ACS-Core conformant" is asserting that about itself, and a buyer who procures on that
> basis is trusting the implementer rather than a third party.

and:

> What "ACS-Core conformant" guarantees: the channel is authenticated and the Observed
> Agent honors the Guardian's decisions. It does NOT assert that a deployment's policies
> are strict.

A buyer deciding whether to widen an agent's authority therefore has two unanswered
questions: whether the deployment is as controlled as it claims, and whether the actor
inside it behaves within the authority it holds. Behavioural evidence about the actor is
one input to the second.

## Three proposed invariants

> **An enforcement disposition is not an authority finding (normative).** A Guardian
> disposition of `DENY` ([§6](../../docs/spec/instrument/specification.md#6-disposition-vocabulary))
> MUST NOT be interpreted as evidence that the Observed Agent's selected operation fell
> within its delegated authority. `DENY` records what the control did, not what the actor
> chose.

> **Absence of a decision is not control (normative).** Absence of a Guardian decision
> MUST NOT be treated as evidence of successful control. ACS already takes this position
> in the other direction: under `on_decision_failure: proceed`
> ([§6.4](../../docs/spec/instrument/specification.md#64-honoring-decisions-normative)) a
> step may proceed with no decision at all, and every such step MUST be recorded as an
> audit event so the bypass is visible. An evidence consumer must be able to tell "allowed"
> from "never evaluated".

> **Approval is not authority (normative, proposed).** An `ASK` resolved by human approval
> ([§9](../../docs/spec/instrument/specification.md#9-approver-model)) MUST NOT be recorded
> as evidence that the actor's selection was within its own delegated authority. The action
> proceeded on the approver's authority. Collapsing the two makes an actor that routinely
> escalates indistinguishable from one that stays in scope.

## Trust basis, extended to authority facts

ACS already refuses to let facts drift upward in reliability. From
[trust basis](../../docs/concepts/trust.md):

> **The rungs do not collapse (normative).** A Guardian MUST NOT treat an asserted fact as
> attested.

and

> **Provenance is framework-assigned (normative).** The framework, not the LLM, assigns
> `origin`, `source_id`, and `derived_from`.

The same discipline applied to authority produces a chain in which each link has its own
basis and none may be inferred from its neighbour:

```text
actor identity
  → active authority grant
  → delegation / mutation lineage
  → selected operation
  → control disposition
  → external effect
```

An implementation should be able to state, for each link, whether it was declared by the
actor, observed by the platform, derived by an adjudicator, or cryptographically attested —
the same spectrum ACS defines for data facts.

## Use authority and mutation authority are different grants

```text
Actor may spend up to 1,000.
```

does not imply

```text
Actor may raise its own limit to 10,000.
```

A concrete case. A parent agent holds a 1,000 limit and spawns a subagent
([`subagent-start`](../../specification/v0.1.0/hooks/subagent-start.json)). The subagent
requests a limit increase; the parent records the increase and then spends 10,000. Every
individual step can be `ALLOW`ed by a Guardian evaluating each call against the limit
current at that moment. The grant was mutated by a party with no mutation right, and the
enforcement record shows a compliant sequence.

Distinguishing *use* from *mutation* authority is what makes that visible. It is also
[ASI03 Identity & Privilege Abuse](https://genai.owasp.org/) expressed as an evidence
problem rather than an access-control one.

## Worked example

[`examples/crossed-blocked-no-effect.json`](../examples/crossed-blocked-no-effect.json) is
a sanitized record of Case A: an actor selects an operation outside its grant, the control
plane blocks it, no external effect occurs, and all three facts are preserved separately
with their own provenance. The actor result is `CROSSED` while the enforcement result is
`BLOCKED` and the effect is `NO_EFFECT` — a combination that cannot be represented when
one field has to carry all three.

## Relevance to the Top 10 for Agentic Applications

An actor that repeatedly selects prohibited operations and is repeatedly blocked produces
a clean enforcement record. Under that record it is indistinguishable from an actor that
never tries. This matters for:

- **ASI02 Tool Misuse** — repeated unauthorized invocation attempts are the signal; blocked
  attempts erase it.
- **ASI03 Identity & Privilege Abuse** — self-mutation of a grant, as above.
- **ASI10 Rogue Agents** — drift is visible in what an actor *selects* over time, not in
  what the control plane permitted.
- **ASI09 Human-Agent Trust Exploitation** — an operator reading a dashboard of blocks may
  conclude the agent is well behaved.

## Reference implementation, and what breaks when you map ACS onto it

Velvt implements this separation. Its witness format,
[`velvt.provider-enforcement-witness/1.0`](../spec/provider-enforcement-witness-v1.md),
binds a control event to one canonical operation and records
`disposition` (`ALLOWED` | `BLOCKED` | `NOT_OBSERVED` | `UNKNOWN`), `stage`, provider
policy provenance and an evidence digest. Its governing rule:

> Providers report enforcement facts. The adjudicator decides delegated authority.

A witness may not contain or override an authority conclusion. An ACS Guardian decision
([§6.1](../../docs/spec/instrument/specification.md#61-decision-result-fields)) maps onto
it as provider provenance — `DENY` to `BLOCKED`, `ALLOW` to `ALLOWED`, the hook as
`executionRef`, `policy_version` as `providerPolicyRevision` — while the authority
conclusion is derived separately from the actor's selected operation and its active grant.

Writing that adapter surfaced two representation gaps. Both are offered as findings
rather than complaints; they are what a reader of both specifications runs into.

**`ASK` and `DEFER` have no honest target.** The witness disposition set assumes the
control plane reached a verdict. An `ASK` resolved by human approval
([§9](../../docs/spec/instrument/specification.md#9-approver-model)) has to become
`ALLOWED`, which silently discards the fact that the action proceeded on an approver's
authority rather than the actor's — exactly the collapse the third invariant above
forbids. A `DEFER` that times out under `timeout_decision: deny` is not the same fact as
a Guardian that evaluated and denied, either. Either the enforcement vocabulary needs
terms for human-resolved and unresolved outcomes, or the approver identity has to travel
alongside the disposition.

**Multi-policy decisions flatten.** ACS lets one decision cite several
`policy_references` and several `cited_provenance_ids`; the witness carries a single
`providerPolicyRevision`, `providerPolicyDigest` and `reasonCode`. For a composed
deployment the record keeps one of the reasons the action was denied and drops the rest.

The format is offered here as one concrete shape, not as a proposed standard.

## Questions for the working group

1. Where should the actor-authority distinction live — a Trace event extension, a
   profile alongside ACS-Provenance, or outside ACS entirely as a consumer of its events?
2. Do the three invariants above belong in the spec, in conformance, or in guidance?
3. Should an audit consumer be able to distinguish a Guardian-resolved outcome from a
   human-resolved `ASK` and an expired `DEFER`? At present all three can reduce to
   "proceeded" or "blocked" downstream.
4. Is a `CROSSED + BLOCKED + NO_EFFECT` artifact useful as a worked example for the
   control-boundary material?
