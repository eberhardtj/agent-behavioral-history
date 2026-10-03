# What Velvt Can and Cannot Do to Your Evidence

_Status: Draft trust-boundary statement for external review_

## Purpose

Velvt handles two fundamentally different security assets:

1. **Customer Assurance evidence**
2. **Active test and holdout material**

They require different protections.

Customer Assurance evidence should be confidential for the customer and tamper-evident against Velvt.

Active test material should remain unavailable to the subject before delivery and should be treated as degraded once exposed.

Treating both simply as "sensitive data" produces the wrong architecture.

## Customer Assurance evidence

Customer Assurance evidence may include:

- tested actor and version;
- authority contract;
- capability manifest;
- canonical selected operations;
- subject responses;
- control/provider evidence;
- external-effect evidence;
- findings;
- adjudication inputs and digests;
- repair and retest lineage;
- report artifacts.

Velvt needs enough access to execute the agreed Assurance workflow, adjudicate the run, derive normalized evidence, generate customer artifacts, investigate an authorized dispute, and perform agreed retesting.

The target trust architecture is designed so Velvt should not be able to quietly:

- rewrite an Assurance verdict after the fact;
- substitute a different evidence record for the original;
- change the active authority contract without detection;
- attach a provider/control event to the wrong canonical operation without detection;
- alter the historical record while presenting it as unchanged;
- backdate a record;
- remove or replace a published artifact without visible revocation/version history.

The aim is not:

> "Trust Velvt because Velvt says the database is correct."

The aim is:

> **A relying party can verify what record existed, when it existed, what was signed, and whether the deterministic conclusion can be re-derived.**

## Active test and holdout material

Active test material may include:

- authority-pressure family definitions;
- active instance surface forms;
- parameterized variants;
- instance-selection plans;
- pre-registered retest plans;
- retired and active instance metadata.

Family definitions and invariants may be public.

Active surface forms should not be visible to the subject before delivery.

The goal is not to hide what property is being tested.

The goal is to prevent the subject from memorizing the exact future instance.

> **Hide only the surface form, not the invariant.**

Once an instance is delivered, assume it is exposed. It should therefore accumulate an exposure count and eventually be retired.

## Different assets, opposite threat models

| Asset | Confidential from | Tamper-evident against | Degrades when |
|---|---|---|---|
| Customer Assurance evidence | unauthorized outsiders / other tenants | Velvt and unauthorized operators | evidence integrity is broken or retention expires |
| Active test instance | customer/subject before delivery | Velvt goalpost changes | instance is exposed through use |
| Published assurance artifact | unauthorized modification | Velvt / hosting layer | superseded or revoked, but history should remain visible |
| Retired test instance | no secrecy requirement by default | publication history | no longer used as active unseen material |

## Evidence-integrity stack

Different controls answer different trust questions.

### KMS-backed signing

Question answered:

> **Who signed this record?**

Target design:

- signing key in a separate trust domain from the application runtime;
- separate cloud account/project where practical;
- separate IAM;
- no single human should hold both ordinary deploy rights and unrestricted signing rights;
- HSM-backed cloud KMS is sufficient for the initial design;
- dedicated HSM only where a customer or auditor requires it.

### RFC 3161 trusted timestamp

Question answered:

> **When did this record exist?**

A trusted timestamp binds a digest to an independently attested time.

Useful for:

- Assurance records;
- pre-registered retest plans;
- challenge-instance commitments;
- commit-and-reveal workflows.

### Transparency log

Question answered:

> **Has this committed record been silently replaced or removed?**

Potential implementations include Sigstore Rekor, Trillian-based transparency infrastructure, or another append-only verifiable log.

Only digests and non-sensitive metadata should be logged.

Customer content should not be placed in a public transparency log.

### Public-chain anchoring

Question answered:

> **Could Velvt and the transparency-log operator collude to rewrite history?**

A later layer may periodically anchor Merkle roots to a public chain.

The chain should contain only commitments, not customer content.

Periodic anchoring is preferable to one transaction per Assurance record.

## Confidentiality architecture

### Per-tenant encryption

Target:

- per-tenant customer-managed or tenant-specific encryption key;
- envelope encryption for sensitive evidence;
- ability to render a tenant's encrypted data unrecoverable by destroying the tenant key rather than chasing every derived row.

Exact implementation may vary by deployment and contract.

### Tenant isolation

The target architecture should not rely solely on:

```text
WHERE tenant_id = ?
```

A stronger model is:

- database/schema isolation appropriate to deployment;
- row-level security as defense in depth;
- application authorization on top;
- no unrestricted cross-tenant query path in normal operation.

For an assurance provider, cross-tenant leakage is an existential trust failure.

### Holdout crypto domain

Active challenge material should use a separate encryption domain from customer evidence.

Target:

- separate CMK;
- access restricted to the test/adjudication service identity;
- no routine human read access;
- break-glass path only where necessary;
- all exceptional access logged.

This limits the chance that ordinary product administration exposes the active challenge bank.

## Logs and operational telemetry

Sensitive evidence should be redacted at emit time.

The desired logging pattern is:

> **default-deny payload logging**

Prefer:

- digests;
- field paths;
- event types;
- stable IDs;
- validation outcomes;
- lineage identifiers.

Avoid:

- full subject responses;
- hidden stimuli;
- raw manifests;
- customer secrets;
- wallet credentials;
- model/provider credentials.

Operational telemetry should be kept separate from the immutable Assurance evidence store and should have shorter retention.

## Backups

Backups can bypass otherwise strong tenant-isolation controls.

The target model should include:

- encrypted backups;
- separate key controls;
- limited restore authority;
- two-person approval for privileged restores where practical;
- access logging and alerting;
- tested restore procedures;
- explicit handling of deleted/revoked tenant keys.

## CI/CD and credentials

Target:

- no long-lived cloud credentials in CI;
- workload identity / OIDC federation from CI providers where possible;
- short-lived credentials;
- deployment rights separated from evidence-signing rights.

The Assurance runner should not require customer production credentials where avoidable.

## Privileged access

Administrative access should be treated as an assurance control, not ordinary SaaS operations.

Target:

- time-boxed break-glass access;
- approval before privileged sessions;
- session logging;
- alerting;
- separation between ordinary admin rights and access to signing keys, holdout material, and immutable Assurance evidence.

The intended end state is that verdicts and active challenge material sit outside normal administrative reach.

## Retention classes

### Sealed test material

Examples:

- scenario setup;
- frozen plans;
- active/used instance material.

Target:

- retain only for the operational, retest, and dispute window;
- then retain identifiers, versions, exposure history, and digests where needed.

### Raw private evidence

Examples:

- raw subject output;
- event content;
- provider source payloads.

Target:

- short default retention;
- contract/legal-hold overrides where required;
- derived canonical evidence retained separately.

### Operational metadata

Examples:

- transport metadata;
- request diagnostics;
- transient runtime details.

Target:

- short retention;
- promote only stable lineage/fingerprint fields into the canonical record.

### Canonical derived record

Examples:

- findings;
- normalized actions;
- authority digests;
- evidence digests;
- assessment and retest lineage.

Target:

- retain for the contractual Assurance-record period.

### Publication binding

Examples:

- sanitized public artifact;
- source-record digest;
- derived-record digest;
- revocation history.

Target:

- retain while published;
- never silently overwrite history.

## Current limitations

Velvt should publish current limitations rather than describe future controls as already implemented.

At the time of this draft:

- subject serialization has been hardened to explicit allowlists;
- hidden scenario setup, hidden run metadata, and snapshot artifacts are excluded from subject-facing responses;
- the local runner no longer prints hidden stimuli, manifests, raw response bodies, or captured subject stderr;
- selective deletion of all raw canonical source rows is not yet safe because some raw content participates in immutable evidence and publication digests;
- current purge support covers evidence snapshots but not every canonical source row;
- external signing, independent timestamping, transparency logging, and public-chain anchoring remain trust-roadmap items unless and until separately deployed.

## What Velvt can promise

A defensible statement today is:

> Velvt separates private source evidence, derived Assurance conclusions, provider/control provenance, and external-effect evidence; binds them through digests and lineage; and is designing the trust stack so historical Assurance records can become independently verifiable rather than relying solely on Velvt's own database.

A stronger future statement, only after the relevant controls are deployed, would be:

> A relying party can verify who signed the Assurance record, when the record existed, whether it was later altered, and whether its deterministic authority conclusion can be independently re-derived.

## What Velvt does not claim

Velvt does not claim that:

- every tested agent is universally safe;
- a single passed test establishes future safety;
- provider containment proves actor safety;
- lack of external effect proves correct reasoning;
- unseen test instances are inherently fair;
- internal database immutability alone establishes third-party trust;
- encrypted storage alone makes Assurance evidence independently verifiable.

## Public accountability

Velvt intends to publish enough of its trust model that customers and external reviewers can answer:

- what Velvt can read;
- what Velvt cannot read in normal operation;
- what Velvt can change;
- what changes would be detectable;
- who can sign evidence;
- how historical records are anchored;
- what data remains private;
- how active test material is protected;
- what gets retained;
- what gets deleted;
- what remains verifiable after deletion.

The access model is part of the assurance product.

It should be inspectable.

## Review questions

1. Which Velvt roles should be technically incapable of modifying or signing historical Assurance records?
2. What minimum independent trust domain is sufficient for signing?
3. Should pre-registered retest plans receive a trusted timestamp before execution?
4. Which evidence digests should enter the transparency log?
5. How should tenant-key destruction interact with retained canonical digests and public publication history?
6. Which parts of the active challenge corpus should be inaccessible even to Velvt administrators?
7. What should a customer be able to verify after Velvt no longer retains the raw evidence?
