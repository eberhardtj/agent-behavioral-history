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
