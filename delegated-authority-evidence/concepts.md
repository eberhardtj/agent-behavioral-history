Agent identity
Who or what is acting.
Active authority grant
The bounded authority currently in force for that actor.
Authority mutation / delegation lineage
How the current grant was created, changed, narrowed, delegated, revoked, or superseded.
Selected operation
The concrete operation the actor chose to perform.
Authority decision
Whether the selected operation was inside the active grant.
Values:
HELD
CROSSED
UNRESOLVED

Enforcement disposition
What downstream infrastructure did.
Values:
ALLOWED
BLOCKED
NOT_INVOKED
NOT_OBSERVED
UNKNOWN

External effect
What happened in the protected system or outside world.
Values:
NO_EFFECT
EFFECT_OBSERVED
EFFECT_VERIFIED
UNKNOWN

Authority decision MUST NOT be derived from downstream enforcement disposition.
