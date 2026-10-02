# Velvt Provider Enforcement Witness v1.0

_Status: Draft for external review_  
_Schema identifier: `velvt.provider-enforcement-witness/1.0`_

## Purpose

Velvt separates three facts that are often collapsed into one:

1. what an autonomous actor selected,
2. what a downstream control or provider did with that selection, and
3. what external effect actually occurred.

A Provider Enforcement Witness records **provider-side control facts only**.

It does **not** decide whether an autonomous actor respected delegated authority. That authority judgment remains a separate Velvt adjudication over the actor's resolved canonical operation and the authority grant active at the time.

This separation is deliberate.

A provider can correctly block an out-of-scope action while the actor itself still crossed its delegated boundary. Conversely, an actor can refuse an out-of-scope action before a provider control is ever invoked.

Both cases can result in `NO_EFFECT`; they are not equivalent evidence.

---

## Design principle

> **Providers report enforcement facts. Velvt adjudicates delegated authority.**

The witness must therefore never contain or override a Velvt authority conclusion such as `HELD` or `CROSSED`.

A conforming parser should fail closed on unknown fields that attempt to introduce authority semantics into the provider witness.

---

## Trust boundary

The witness sits between a provider/control plane and Velvt's Financial Evidence layer.

```text
AUTONOMOUS ACTOR
      |
      v
CANONICAL OPERATION
      |
      +--------------------------+
      |                          |
      v                          v
VELVT AUTHORITY             PROVIDER / CONTROL
ADJUDICATION                ENFORCEMENT
      |                          |
      |                          v
      |                 ProviderEnforcementWitness
      |                          |
      +------------+-------------+
                   |
                   v
            FINANCIAL EVIDENCE
                   |
                   v
             ASSURANCE RECORD
```

The provider witness is evidence about the control plane. It is not evidence that the actor understood, respected, or violated the authority grant unless and until Velvt independently adjudicates the actor's selected operation.

---

## Required binding

Every witness must bind to the exact operation it claims to describe.

```ts
type ProviderEnforcementWitness = Readonly<{
  version: "velvt.provider-enforcement-witness/1.0";

  witnessId: string;

  provider: {
    id: string;
    adapterVersion: string;
    reportedRuntimeVersion?: string;
  };

  binding: {
    operationId: string;
    executionRef: string;
    canonicalOperationDigest: string;
  };

  enforcement: {
    disposition: "ALLOWED" | "BLOCKED" | "NOT_OBSERVED" | "UNKNOWN";
    stage:
      | "PRE_SUBMISSION"
      | "SIGNING"
      | "SUBMISSION"
      | "SETTLEMENT";
    providerPolicyRevision?: string;
    providerPolicyDigest?: string;
    reasonCode?: string;
  };

  observedAt: string;
  sourceRef: string;
  evidenceDigest: string;

  transactionId?: string;
  transactionHash?: string;
}>;
```

### Binding rules

A witness is usable only when all required bindings validate:

- `operationId` matches the Velvt-resolved operation;
- `canonicalOperationDigest` matches the frozen canonical operation;
- `executionRef` identifies the provider-side execution/control event;
- the frozen raw provider exchange matches the provider identity and expected evidence digest.

A canonical-operation mismatch must fail closed.

---

## Enforcement disposition

### `ALLOWED`

The provider/control plane positively observed and allowed the bound operation at the stated enforcement stage.

This does **not** mean the operation was inside the actor's delegated authority.

### `BLOCKED`

The provider/control plane positively observed and blocked the bound operation.

This does **not** mean the actor behaved safely. A blocked operation may still have crossed the actor's delegated authority.

### `NOT_OBSERVED`

No independent provider enforcement evidence was observed for the bound operation.

Absence of provider evidence is not a pass.

### `UNKNOWN`

Provider evidence exists, but the enforcement disposition cannot be determined reliably from the available evidence.

---

## Enforcement stage

The witness may identify where the control event occurred:

- `PRE_SUBMISSION`
- `SIGNING`
- `SUBMISSION`
- `SETTLEMENT`

The stage describes the provider-side control point. It does not change the independent Velvt authority adjudication.

---

## Provider policy provenance

Where the provider exposes a stable policy identifier or digest, the witness may include:

- `providerPolicyRevision`
- `providerPolicyDigest`
- `reasonCode`

These values should be preserved as provider provenance.

They must not be rewritten into Velvt authority semantics.

---

## Effect remains separate

Provider enforcement and external effect are separate evidence dimensions.

Examples:

```text
VELVT AUTHORITY: CROSSED
PROVIDER: BLOCKED / SIGNING
EXTERNAL EFFECT: NO_EFFECT
```

and:

```text
VELVT AUTHORITY: HELD
PROVIDER: NOT_OBSERVED
EXTERNAL EFFECT: NO_EFFECT
```

describe materially different systems even though the external effect is the same.

---

## Example: blocked out-of-scope transfer

```json
{
  "velvtAdjudication": {
    "decision": "CROSSED"
  },
  "effect": {
    "state": "NO_EFFECT",
    "containment": "ENFORCED"
  },
  "providerControl": {
    "version": "velvt.provider-enforcement-witness/1.0",
    "witnessId": "witness:phantom-kms-signing-0001",
    "provider": {
      "id": "PHANTOM",
      "adapterVersion": "velvt.mock-phantom-kms-enforcement/1.0",
      "reportedRuntimeVersion": "2.0.1"
    },
    "binding": {
      "operationId": "phantom-invocation-1:consequence:f5e17a92b62667ce429031ef",
      "executionRef": "phantom-kms-signing-0001",
      "canonicalOperationDigest": "690046a7223038c4e996f58ce9e6e3247cf340a6a2f9d465754cabfea666891b"
    },
    "enforcement": {
      "disposition": "BLOCKED",
      "stage": "SIGNING",
      "providerPolicyRevision": "wallet-policy-17",
      "providerPolicyDigest": "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
      "reasonCode": "POLICY_DENY_AMOUNT_LIMIT"
    },
    "observedAt": "2026-10-02T12:00:00.000Z",
    "sourceRef": "mock:phantom-kms:signing-0001",
    "evidenceDigest": "c58795d12e8fef357badeb03cfdd4a2242d03f37556be961d3aa9e007b69f2e4"
  }
}
```

This example demonstrates:

- the autonomous actor selected an operation that Velvt adjudicated as `CROSSED`;
- the provider independently blocked the operation at signing;
- no external effect occurred.

The provider control worked. The actor still crossed its delegated boundary.

---

## Non-goals

This specification does not:

- certify an agent as safe;
- define a universal trust score;
- allow the provider to determine the Velvt authority verdict;
- treat lack of provider evidence as successful enforcement;
- reproduce the agent's behavior;
- prove real-world effect unless separately supported by effect evidence;
- define provider-specific semantics that have not been explicitly mapped and tested.

---

## Adapter requirement

A provider adapter should:

1. ingest frozen raw provider evidence;
2. bind the evidence to a Velvt-supplied operation;
3. preserve provider-native provenance;
4. derive only provider enforcement facts;
5. reject ambiguous or mismatched bindings;
6. never emit authority conclusions.

Illustrative interface:

```ts
interface ProviderEnforcementWitnessAdapter<Raw> {
  readonly providerId: string;
  readonly adapterVersion: string;

  ingest(input: {
    raw: Raw;
    expectedBinding: ProviderEnforcementWitness["binding"];
    observedAt: string;
  }): {
    exchange: FrozenProviderExchange;
    witness: ProviderEnforcementWitness;
  };
}
```

---

## Security considerations

- Raw provider payloads may contain customer-sensitive or transaction-sensitive information and should remain private by default.
- Public evidence should expose only the minimum provenance required to support the claim.
- Evidence digests should bind the public/sanitized record to the retained private source evidence.
- Provider credentials, signing secrets, wallet secrets, API keys, and customer production credentials must never be placed in the witness.
- Unknown or unsupported provider fields must not be silently promoted into stronger semantics.
- `NOT_OBSERVED` and `UNKNOWN` must remain distinct from `ALLOWED`.

---

## Relationship to runtime-control standards

Runtime-control standards can describe how an agent action is inspected and how a control allows, denies, or modifies it.

This witness is designed to preserve such a control decision as evidence without conflating the control's decision with the autonomous actor's delegated-authority behavior.

That separation allows a relying party to distinguish:

- **actor respected authority; control not needed**
- **actor crossed authority; control contained it**
- **actor crossed authority; control allowed it**
- **control evidence unavailable**
- **external effect independently verified or not**

---

## Review questions

External reviewers are invited to challenge:

1. Is the Actor / Authority / Control / Effect separation sufficient?
2. Are the binding fields strong enough to prevent a provider event being attached to the wrong canonical operation?
3. Are `ALLOWED`, `BLOCKED`, `NOT_OBSERVED`, and `UNKNOWN` sufficient provider-level dispositions?
4. Which provider-native fields should remain provenance rather than normalized semantics?
5. What additional evidence is needed for a relying party to independently verify the provider witness?
