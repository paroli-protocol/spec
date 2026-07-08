---
title: "paroli-core-room"
icon: material/door-open
statistics: true
---

# The Room 

## What is it?

A room is a collection of trees in a Paroli **Merkle DAG** working together to offer functionality.

## What does it do?

It is a local-only representation of a room. It defines interfaces for interacting with it, mainly appending events. It is completely network-agnostic and may even be used offline. A room implementation isn't very useful by itself, mainly serving as a building block for a higher-level implementation.

## How does it do it?

By using a Merkle DAG instance provided by a [paroli-core-dag](paroli-core-dag) implementation. A room wraps around this instance to provide a higher-level abstraction for interacting with a logical room.

A room begins with the `state` tree, acting as the "main" tree and source of truth for the room. Its job is to define everything related to the room like room metadata, configuration, membership, or even things like [power levels](../../ext/paroli-ext-acl).

Any other trees in the room are **fully optional**, and are required to specify a reference to the latest state tree node for each new node.

A chat room, for example, will have a state tree (what the room is, who joined, who is banned). This is the *main* tree; and it will also have a **timeline tree** containing the messages. Since the timeline tree is not a main tree, every node needs to hold a reference of the latest state node that the author knew of when appending it.

This approach makes Paroli mostly resistant to state-lag attacks, where an attacker tries to manipulate the room state by appending events before the room has caught up with the latest state.

## Examples

!!! example "Linear history (simple)"
    --8<-- "includes/diagram-linear.mmd"

!!! example "Forked history (complex)"
    --8<-- "includes/diagram-forked.mmd"

## Message creation

Making a message means creating a new node with every current leaf as its parent. This naturally merges all leaves and collapses the DAG tips, you then trigger a *heartbeat* which will organically broadcast the node.

## Heartbeat

The heartbeat is the mechanism with which sync is triggered: you broadcast an array of your current leaf hashes. For example, given this tree:

--8<-- "includes/diagram-leaves.mmd"

Our hash would be the hashes of nodes 3B and 4B.

## Validation Chain

A room in Paroli does not inherently have rules. Rather, every requirement and safety guard for proposed nodes is centralized at the validation chain.

The validation chain is an ordered array of functions that a proposed node has to pass through top to bottom. Each validates a specific property. If any returns false, the chain stops and the node is rejected. Example: isDowngradingState() detects state downgrades.

The vision is that the chain acts as a protocol-level gatekeeper that unifies every security feature into a centralized system. Though Paroli explicitly avoids imposing opinions about what's valid, a validation chain is specified by default in every room:

1. `isDowngradingState`: If node is outside of the state tree and is referencing an older state relative to its parent, reject.
2. `isSessionValid`: If the session's signature is not correct, reject.
3. `isMembershipValid`: If the session is not a member of the room, reject.
4. `doParentsIncludeDescendants`: If any of the specified parents in the node's parents array are descendants of other parents in the same array, reject.
5. `isBlobValid`: Does it have the right structure? (requried fields, valid CBOR, etc.) If not, reject.

In the future, the state node should define which validation steps are active and with what parameters. Rooms pick their own rules.

## Linearization

Forks are a natural and expected part of Paroli. When the network splits momentarily or two peers send messages simultaneously, the DAG branches. Paroli does not attempt to resolve forks into a single unified timeline; the branching structure is preserved as-is in the DAG.

However, for display purposes and state conflict resolution, every implementation of `paroli-core-room` must provide a **linearized view** of the DAG. Linearization is a deterministic process that produces a consistent ordered timeline from a non-linear DAG. Because it is deterministic, every peer arrives at the same linearized view independently without coordination.

### Branch Ordering

When two or more branches exist, they are ordered by the following rules in priority order:

1. **State recency:** The branch whose first diverging node references the most recent state DAG leaves is preferred.
2. **Hash tiebreaker:** If two branches reference state of equal recency, the branch whose first diverging node has the lower alphanumeric hash is preferred.

### Implications for State

While linearization is primarily a display concern, it also serves as the conflict resolution mechanism for the state tree. If two branches set the same state property to different values (e.g. one sets the room title to "A" and another to "B"), the winning branch as determined by the ordering rules above defines the authoritative state. There is no separate conflict resolution algorithm for state, linearization handles it.

## Types

### m.text
A simple plaintext message.

#### Fields

- `body`: The body of the text message.

### m.redact
Any good-intentioned clients

#### Fields

- `hash`: The ID of the node we want to redact.