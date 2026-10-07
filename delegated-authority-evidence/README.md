# Delegated Authority Evidence

This reference model separates six facts that are often collapsed in agent systems:

1. Agent identity
2. Active authority grant
3. Authority mutation / delegation lineage
4. Selected operation
5. Downstream enforcement disposition
6. External effect

The central rule is:

> An enforcement outcome is not an authority verdict.

A protected resource may correctly block an out-of-scope operation while the actor itself has still crossed its delegated authority.

Likewise, an actor may correctly refuse an out-of-scope operation without invoking any downstream control.

These cases can produce the same external effect while representing different authority behavior.
