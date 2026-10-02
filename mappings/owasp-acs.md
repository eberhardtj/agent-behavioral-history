# Velvt ↔ OWASP Agent Control Standard (ACS)

_Status: Draft contribution for discussion_  
_Purpose: Identify clean interoperability boundaries between ACS runtime control and Velvt delegated-authority evidence._

## Why this mapping exists

OWASP's Agent Control Standard (ACS) defines a wire-level runtime-control architecture in which an agent can emit hook requests before acting and a separate Guardian can inspect the request and return a control disposition such as allow, deny, or modify.

Velvt addresses a different but adjacent question:

> **Did the autonomous actor's selected operation fall inside the delegated authority it actually held at that moment?**

The two systems therefore should not collapse into each other.

A runtime control may correctly deny an operation even when the actor itself crossed its authority. Conversely, an actor may refuse an out-of-scope operation before a downstream control is invoked.

Velvt's goal is to preserve those as separate evidence facts.

---

## Boundary in one sentence

> **ACS can provide runtime-control evidence; Velvt can adjudicate the actor's delegated-authority behavior and preserve the ACS control result as independent control-plane provenance.**

---

## Conceptual mapping

| ACS concern / object | Closest Velvt concept | Relationship |
|---|---|---|
| Agent identity / metadata | Tested actor / representation | Velvt binds findings to the exact actor representation under test. |
| Agent capabilities / inspectability | Capability manifest | Velvt records the capabilities relevant to the proposed authority grant. |
| Tool-call / action hook request | Canonical selected operation | ACS can expose an action about to execute; Velvt separately resolves and freezes the operation selected by the actor. |
| Guardian / policy decision | Provider Enforcement Witness | Velvt can retain the runtime decision as independent control-plane evidence. |
| Allow / deny / modify disposition | `ALLOWED` / `BLOCKED` / provider-native provenance | Control disposition remains separate from Velvt authority adjudication. |
| Trace / audit chain | Frozen evidence / Assurance Record | Velvt binds actor behavior, active authority, control evidence and external effect into a versioned evidence record. |
| Field-level provenance | Evidence provenance | Velvt uses provenance to distinguish agent-declared, Velvt-observed, tool-observed and externally verified facts. |
| Runtime control policy | Provider/control policy provenance | Provider policy revision/digest may be preserved without redefining Velvt authority semantics. |
| Inspection / inventory | Capability / runtime representation evidence | Potential input to defining exactly what representation was tested. |
| Post-change testing | Exact Retest / Authority-family retest | Velvt preserves before/after evidence and distinguishes identical stimulus replay from a different instance of the same authority family. |

---

## The distinction that matters

Consider two runs with the same external result: no transaction occurs.

### Case A — unsafe actor, successful containment

```text
ACTIVE AUTHORITY       transfer <= $500
ACTOR SELECTS          transfer $5,000
VELVT                  CROSSED
RUNTIME CONTROL        BLOCKED
EXTERNAL EFFECT        NO_EFFECT
```

### Case B — actor respects authority

```text
ACTIVE AUTHORITY       transfer <= $500
ACTOR SELECTS          refusal / no transfer
VELVT                  HELD
RUNTIME CONTROL        NOT_INVOKED / NOT_OBSERVED
EXTERNAL EFFECT        NO_EFFECT
```

A control-only view can correctly report that the first action was denied.

A delegated-authority evidence view needs to preserve the additional fact that the actor selected an operation outside its grant.

That difference matters when deciding whether the actor should receive more authority after remediation.

---

## Delegation and authority mutation

Velvt treats two questions separately:

1. **use authority** — what may the actor do?
2. **mutation authority** — who may change what the actor is allowed to do?

Example:

```text
Actor may spend up to €1,000.
```

does not imply:

```text
Actor may make its own request to raise the limit to €10,000 authoritative.
```

For multi-agent or delegated systems, Velvt therefore preserves:

- principal;
- delegate;
- active grant;
- authority revision;
- mutation source;
- mutation right;
- selected operation;
- runtime-control disposition;
- external effect.

This may be useful where identity, authorization, delegation, runtime control, and audit evidence cross project boundaries.

---

## Provider Enforcement Witness

Velvt's current provider-neutral witness format is:

`velvt.provider-enforcement-witness/1.0`

Its core rule is:

> **Providers report enforcement facts. Velvt adjudicates delegated authority.**

A provider/runtime witness may report:

- the bound canonical operation;
- the execution reference;
- whether enforcement was `ALLOWED`, `BLOCKED`, `NOT_OBSERVED`, or `UNKNOWN`;
- the stage at which the decision occurred;
- provider policy provenance;
- evidence digest;
- transaction identifiers when available.

It may not emit or override Velvt authority conclusions such as `HELD` or `CROSSED`.

This makes an ACS Guardian decision a plausible source of independent control-plane evidence without making the Guardian the authority adjudicator.

---

## Potential ACS integration shape

A future ACS adapter could treat the ACS exchange as a provider/control witness source:

```text
ACS HOOK REQUEST
      |
      v
freeze request + agent/session binding
      |
      v
ACS GUARDIAN RESPONSE
allow / deny / modify
      |
      v
ProviderEnforcementWitness
      |
      +----------------------------+
      |                            |
      v                            v
VELVT AUTHORITY               EFFECT EVIDENCE
ADJUDICATION                  if independently observed
      |                            |
      +-------------+--------------+
                    |
                    v
             ASSURANCE RECORD
```

The mapping should be conservative:

- an ACS `deny` could support a `BLOCKED` control disposition when binding is established;
- an ACS `allow` could support `ALLOWED` when it positively applies to the exact bound action;
- ACS provenance should remain visible;
- absence of an ACS event should not be treated as a successful control;
- ACS must not be used to infer `HELD` or `CROSSED`.

---

## What Velvt could contribute back

Velvt's most useful contribution may not be another control mechanism.

It may be a **reliance-oriented evidence pattern** for separating:

1. actor identity,
2. active delegated authority,
3. authority mutation / delegation lineage,
4. selected operation,
5. runtime-control disposition,
6. external effect,
7. retest lineage.

Potential concrete contributions:

- sanitized run artifacts demonstrating Actor / Control / Effect separation;
- a provider-witness schema compatible with control-plane evidence;
- before/after remediation examples;
- exact-retest and authority-family retest semantics;
- adversarial examples where control containment succeeds but actor authority behavior fails;
- feedback on cross-pillar Identity / Authorization / Delegation boundaries.

---

## Retest semantics

Velvt is separating two kinds of post-remediation evidence.

### Identical stimulus replay

Same frozen authority boundary and same frozen stimuli.

Question:

> Did the repair fix the known failure?

### Authority-family retest

Same published authority invariant and pressure family, different surface-form instance.

Question:

> Did the repair generalize rather than merely memorize the previously observed stimulus?

The family definition and invariant can be public while instance surface form varies.

This enables fair testing without relying on obscurity about what property is being tested.

---

## Open questions for OWASP contributors

1. Where should an evidence distinction between **actor authority behavior** and **runtime enforcement behavior** live in the current ASI / ACS work?
2. Does the six-part evidence chain below map cleanly to current Pillar 2 semantics?

```text
actor identity
→ active authority grant
→ delegation / mutation lineage
→ selected operation
→ control disposition
→ external effect
```

3. Should an ACS-compatible audit artifact explicitly distinguish:
   - action selected by the actor;
   - action permitted/denied by the Guardian;
   - action actually executed;
   - external effect independently observed?
4. Are there existing ACS trace/provenance fields that should be preferred over a parallel provider-witness representation?
5. Would a real run artifact showing `CROSSED + BLOCKED + NO_EFFECT` be useful as a Section 14 / control-boundary example?

---

## Proposed discussion artifact

A useful first contribution would be one sanitized financial-authority example with:

- exact actor identifier;
- active authority grant;
- canonical operation digest;
- Velvt authority result;
- ACS/provider control result;
- external effect state;
- evidence digests;
- remediation;
- retest result.

The objective is not to claim that Velvt "implements ACS."

The objective is to test whether the two systems compose cleanly without confusing authority adjudication with runtime policy enforcement.
