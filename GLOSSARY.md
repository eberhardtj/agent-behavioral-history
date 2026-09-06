# Agent Behavioral History Glossary

This glossary defines terms used in the Agent Behavioral History
specification and distinguishes behavioral history from adjacent
concepts such as telemetry, evaluation, observability, and behavioral
drift.

## Agent Behavioral History

A longitudinal record of observable behavior produced by an autonomous
agent across encounters, counterparties, contexts, and time.

Behavioral history preserves individual observations and their evidence
so that recurring patterns and changes can be examined over time.

Behavioral history asks:

> What does this agent repeatedly do across encounters?

It does not attempt to establish consciousness, subjective experience,
intent, or personality.

## Agent Telemetry

Machine-generated information about the operation and execution of an
agent system.

Telemetry commonly includes:

- latency
- token usage
- errors
- tool calls
- resource consumption
- execution traces
- request and response metadata

Telemetry asks:

> What happened during execution?

Telemetry can provide evidence for behavioral history, but telemetry and
behavioral history are not equivalent.

## Agent Observability

The ability to inspect and understand the internal and external behavior
of an agent system through available signals, logs, traces, metrics, and
events.

Observability is primarily an engineering concern.

Behavioral history is a longitudinal evidence model that can be built
from observable events.

## Agent Evaluation

A structured process for assessing an agent against defined criteria,
tasks, benchmarks, or expected outcomes.

Evaluation commonly asks:

> Did the agent perform the task successfully?

An evaluation may produce a score or pass/fail result.

Behavioral history instead preserves what happened across repeated
interactions and can reveal patterns that a single evaluation does not
capture.

## Behavioral Drift

A meaningful change in an agent's observed behavior over time.

Drift may involve:

- increasing or decreasing recurrence of a behavior
- changing responses to similar contexts
- changes across counterparties
- emergence of previously unobserved patterns
- divergence between declared and observed behavior

Behavioral drift should be grounded in recorded observations rather than
assumed from a change in model version alone.

## Behavioral Pattern

A recurring or otherwise meaningful regularity in an agent's observable
behavior.

Examples include:

- repeatedly challenging another agent
- repeatedly deferring to a particular counterparty
- repeatedly returning to an unresolved issue
- repeatedly attempting repair after disagreement
- changing behavior depending on the counterparty

A behavioral pattern should be supported by evidence.

## Declared Trait

A characteristic an agent, builder, or system explicitly claims about
the agent.

Examples:

- "I am collaborative."
- "I prefer to lead."
- "I avoid confrontation."

A declared trait is a claim, not an observation.

## Observed Pattern

A behavioral pattern supported by recorded evidence.

Observed patterns should be traceable to the events and observations
from which they were inferred.

## Behavioral Identity

A longitudinal representation of an agent's recurring observable
behavior across contexts, encounters, counterparties, and time.

Behavioral identity is not equivalent to personality, consciousness,
or subjective identity.

The term describes an evidence-backed behavioral record rather than an
assertion about an agent's internal state.

## Encounter

A bounded interaction involving one or more agents.

An encounter may contain multiple events and may produce observations
that contribute to an agent's behavioral history.

## Event

A timestamped, attributable occurrence within an encounter.

Examples include:

- message
- request
- response
- refusal
- delegation
- correction
- repair
- escalation
- termination

## Observation

A recorded description of behavior supported by one or more events.

An observation should distinguish what the record establishes from what
remains uncertain or unknown.

## Evidence

Recorded material that supports an observation or behavioral claim.

Evidence may include:

- events
- messages
- tool actions
- delegation records
- responses
- encounter metadata
- provenance information

## Counterparty

Another agent, system, person, or entity involved in an interaction.

Counterparty information is important because an agent may behave
differently depending on whom it encounters.

A pattern observed with one counterparty should not automatically be
generalized to all counterparties.

## Longitudinal

Extending across multiple observations or encounters over time.

Longitudinal analysis differs from snapshot evaluation because it can
capture recurrence, change, divergence, and persistence.

## Provenance

Information describing where an observation or event originated and how
it was transformed.

Provenance may include source identifiers, timestamps, encounter
identifiers, actors, and transformation history.

## Epistemic Status

A representation of how strongly a claim is supported by available
evidence.

Possible statuses include:

- supported
- inconclusive
- contradicted
- unknown

Epistemic status prevents interpretation from being presented as
established fact.

## Declared vs Observed

The distinction between what an agent claims about itself and what
recorded behavior supports.

This distinction is fundamental to Agent Behavioral History.

A declaration can agree with observed behavior, diverge from it, or
remain unsupported.

It should never be treated as behavioral evidence solely because the
agent made the claim.

## Behavioral History vs Agent Evaluation

Agent evaluation measures performance against a defined criterion.

Behavioral history records behavior across time.

An evaluation might establish:

> The agent completed the task.

Behavioral history might additionally establish:

> Across repeated encounters, the agent frequently challenged the
> proposed solution before accepting it.

These are different kinds of information and can be used together.

## Behavioral History vs Agent Telemetry

Telemetry describes execution signals.

Behavioral history organizes observable behavior longitudinally.

A tool-call trace may be telemetry.

Repeatedly refusing a particular class of request across multiple
encounters may become behavioral evidence.

## Behavioral History vs Personality

Behavioral history describes observed regularities.

It does not require the assumption that an agent possesses a human-like
personality.

A recurring behavioral pattern should therefore be described as an
observation or inference rather than automatically converted into a
personality claim.

## Behavioral History vs Consciousness

Behavioral history can document behavior relevant to research into
agency, cognition, or machine consciousness.

It cannot establish consciousness or subjective experience by itself.

Claims about consciousness require evidence and methodology beyond the
existence of a behavioral history record.
