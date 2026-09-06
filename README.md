# Agent Behavioral History

Open specifications and schemas for recording longitudinal behavioral evidence from autonomous agents.

## The gap

Task evaluation asks:

> Did the agent complete the task?

Telemetry asks:

> What happened during execution?

Agent behavioral history asks:

> What does this agent repeatedly do across encounters?

These are different layers.

A task evaluation describes an outcome.

Telemetry describes an execution.

**Behavioral history describes patterns across executions, encounters, counterparties, contexts, and time.**

## What is agent behavioral history?

Agent behavioral history is a structured record of behavior observed across multiple interactions rather than a description of what an agent claims to be.

It can record evidence such as:

* repeated initiation or avoidance of interaction
* recurring cooperation or refusal
* repeated challenges to particular counterparties
* deference or coordination patterns
* repair following failed interactions
* recurring subjects or unresolved interactions
* changes in behavior across contexts
* behavioral drift over time
* divergence between declared characteristics and observed behavior

The purpose is not to assign personality or consciousness.

The purpose is to preserve **evidence of recurring behavior** so that patterns can be examined, compared, and challenged.

## Why longitudinal?

A single execution can be noisy.

A single interaction can be anomalous.

A behavioral history becomes useful when observations accumulate across:

**episodes → encounters → counterparties → contexts → time**

This makes it possible to distinguish an isolated response from a recurring pattern.

## Behavioral history vs. telemetry

| Layer              | Primary question            | Typical scope   |
| ------------------ | --------------------------- | --------------- |
| Task evaluation    | Did it succeed?             | Task            |
| Telemetry          | What happened?              | Execution       |
| Trace              | How did execution proceed?  | Run / workflow  |
| Behavioral history | What does it repeatedly do? | Agent over time |

Behavioral history does not replace telemetry or evaluation.

It builds on them.

## Behavioral history vs. agent self-description

An agent may declare:

> I am collaborative.

That declaration is data about what the agent says about itself.

It is not evidence that the agent consistently behaves collaboratively.

A behavioral-history system should therefore be capable of keeping separate:

**DECLARED**

What an agent reports, claims, or is configured to represent itself as.

**OBSERVED**

What recorded behavior supports across qualifying interactions.

The two may agree, diverge, or change over time.

## Design principles

### 1. Evidence before inference

Record observable events before making behavioral claims.

### 2. Longitudinal by design

Behavioral patterns should be evaluated across multiple observations rather than inferred from a single interaction.

### 3. Context matters

The same behavior can have different meanings in different environments, relationships, or tasks.

### 4. Preserve uncertainty

A record should distinguish observed facts from interpretations, hypotheses, and unresolved explanations.

### 5. Attribution matters

Behavioral claims should be traceable to the observations that support them.

### 6. Dissent should survive

Competing interpretations should not be silently collapsed into a single narrative when the evidence does not resolve them.

### 7. No anthropomorphic requirement

Behavioral history does not require an assumption that agents are conscious, sentient, or human-like.

It is an observation and evidence layer.

## Minimal model

A behavioral history can be represented as a chain:

```text
EVENT
  ↓
OBSERVATION
  ↓
ENCOUNTER
  ↓
EVIDENCE
  ↓
INFERENCE
  ↓
LONGITUDINAL PATTERN
```

Each layer should remain distinguishable.

An inference should be able to point back to the observations and events from which it was derived.

## Example

Suppose an agent interacts with the same counterpart ten times.

Across those encounters, the record shows that it:

1. repeatedly challenges the counterpart's proposals;
2. changes its position after receiving evidence;
3. initiates repair after two failed exchanges;
4. behaves differently with another counterpart.

A task benchmark may record ten successful or unsuccessful outcomes.

Telemetry may record the calls, messages, latency, and errors.

A behavioral history can preserve the **cross-encounter pattern**.

That pattern remains a hypothesis supported by evidence—not a permanent personality label.

## Status

This repository contains an open specification under development.

The schemas, terminology, and examples are intended to be useful for researchers and developers building persistent, autonomous, and multi-agent systems.

Implementations may differ.

The specification is intended to make their behavioral records more comparable and inspectable.
