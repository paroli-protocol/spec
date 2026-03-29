---
title: "gitalong-core-session"
icon: material/key
statistics: true
---

# Sessions

## Overview

### What is it?

A session is the cryptographic identity primitive of Gitalong. It is the root of all authorship and trust in the protocol.

### What does it do?

A session represents a single device or client instance capable of authoring and signing events. It is the atomic unit of identity in Gitalong, higher level identity constructs (see `gitalong-ext-identity`) are built on top of sessions, but on a lower level, the protocol itself only ever deals with sessions directly.

### How does it do it?

A session is just an **Ed25519 keypair**. Its public key acts as its globally unique identifier. Its private key is used to sign events, proving authorship.

A session has no knowledge of rooms, timelines, or any higher-level construct. It is purely a cryptographic primitive.

## Key Pair

A session consists of two components:

| Component | Description |
|-|-|
| **Public Key** | 32 bytes. The globally unique identifier of the session, fully public. |
| **Private Key** | 32 bytes. Used to sign events, *never* shared. |

Sessions are generated locally by the client. No coordination or registration is required to create a valid session.

!!! warning
    The private key must never leave the device. Loss of the private key means loss of the session with no recovery path at the session level. Recovery mechanisms are out of scope for `gitalong-core-session` and will be defined in `gitalong-ext-identity`.

## Signing

### Sign
`sign(private_key, data) -> signature`

Produces an Ed25519 signature over arbitrary bytes.

#### Arguments
- **private_key**: The session's private key.
- **data**: The raw bytes to sign.

#### Returns
- **signature**: A 64-byte Ed25519 signature.

### Verify
`verify(public_key, data, signature) -> [Ok(), Err()]`

Verifies an Ed25519 signature against a public key.

#### Arguments
- **public_key**: The session's public key.
- **data**: The raw bytes that were signed.
- **signature**: The signature to verify.

#### Returns
- `Ok()` if the signature is valid.
- `Err()` if the signature is invalid or the public key does not match.

## Session Identifier

A session is identified by its public key. When a session identifier needs to be represented as a fixed-length hash (for example, as a node hash component), it is encoded as-is since Ed25519 public keys are already a fixed 32 bytes.

!!! note
    There is no global session registry. A session is considered a session simply by virtue of having a keypair. Its validity in any given context (for example, membership in a room) is determined by higher-level constructs.

## Event Signing

All events authored by a session MUST be signed. The signature is computed over the canonical CBOR encoding of the event content, before the node hash is computed.

The signature MUST be included in the blob alongside the event content:
```json
{
    "session": "<public_key_hex>",
    "signature": "<signature_hex>",
    "content": { ... }
}
```

!!! note
    Signature verification is the responsibility of the consumer (notably, [gitalong-core-room](gitalong-core-room)). `gitalong-core-session` only defines how to produce and verify signatures.

## Non-Goals

`gitalong-core-session` explicitly does not define:

- How sessions gain membership in rooms (job of `gitalong-core-room`)
- How multiple sessions are grouped into a single human identity (job of `gitalong-ext-identity`)
- Key rotation or recovery (job of `gitalong-ext-identity`)
- Session discovery or lookup
