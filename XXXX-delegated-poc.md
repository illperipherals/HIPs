# HIP-XXXX: Delegated Proof-of-Coverage for Meshed Gateways

- **Author(s):** Travis L. Teague
- **Start Date:** 2025-08-04
- **Category:** Technical
- **Status:** Draft
- **Discussions-To:** [https://discord.com/channels/404106811252408320/1075540098197757962]
- **Created:** 2025-08-04
- **License:** Apache 2.0

## Summary

**PISSD:** PoC Integration for Solar‑Scaled Deployments

This HIP proposes **Delegated Proof-of-Coverage (PoC)** to enable meshed gateways in ChirpStack/RAKWireless deployments to earn PoC rewards on-chain.  
In this model, a **Border Gateway** cryptographically submits PoC witness/challenge receipts on behalf of **upstream meshed gateways**.  
Each meshed node has its own on-chain identity and receives rewards for its RF work.

## Motivation
In current Helium PoC:
- Only gateways directly connected to the blockchain are rewarded.
- In meshed networks, all RF and data traffic flows through a **Border Gateway**.
- Meshed nodes that do RF work are invisible to PoC and earn no rewards.

As a result:
- Dense meshed deployments are economically unattractive.
- PoC mapping under-represents actual RF coverage.

**Delegated PoC** addresses this by:
- Allowing meshed nodes to participate in PoC without blockchain connectivity.
- Maintaining all rewards and attribution **fully on-chain**.
- Preserving PoC’s existing cryptographic trust model.

## Stakeholders
- **Gateway Owners**: Gain incentive to deploy meshed nodes.
- **Helium Network**: Gains more accurate coverage maps.
- **Manufacturers/Operators**: Can deploy mesh without losing reward participation.

## Goals
- Attribute PoC rewards to the actual RF-originating meshed gateway.
- Keep reward distribution entirely on-chain.
- Preserve security guarantees of current PoC.

## Non-Goals
- Off-chain reward distribution.
- Eliminating incentives for Border Gateways.

## Specification

### 1. Virtual Gateway Identities
- Each meshed node generates a **Helium gateway keypair**.
- Registered on-chain like a normal hotspot, with location assertion and owner wallet.
- Keys are used to sign PoC metadata only — no blockchain connection needed.

### 2. RF Origin Proof Signing
When a meshed node **receives** a PoC packet:
- Signs:
  - Gateway public key
  - RSSI/SNR
  - Frequency/sub-band
  - Timestamp
  - Random nonce
  - Optional hashed location

When a meshed node **transmits** a PoC packet:
- Logs and signs challenge metadata for verification.

### 3. Border Gateway Delegation
- Receives wrapped mesh packet with RF origin proof.
- Verifies meshed node’s signature.
- Submits a **Delegated PoC Receipt**:
  ```
  witness: <meshed node address>
  delegate_of: <border gateway address>
  signature: <meshed node proof signature>
  ```
- Sent to the PoC verifier as if the meshed node received the packet directly.

### 4. PoC Verifier Processing
- Validates:
  - `witness` is a registered hotspot.
  - Signature matches registered public key.
  - `delegate_of` is a registered hotspot.
- Credits rewards directly to meshed node’s wallet.
- Optionally gives **relay incentive** to the Border Gateway.

## Reward Model
Default recommendation:
- 90% to RF-originating meshed node.
- 10% to Border Gateway as relay incentive.

Other splits can be set by governance.

## Rationale
- Works within current PoC model — only adds new receipt type.
- Keeps all attribution and rewards on-chain.
- Encourages network densification.
- Improves RF coverage mapping accuracy.

## Backwards Compatibility
- No changes for non-meshed gateways.
- Works alongside existing PoC without disruption.

## Security Considerations
- Private keys must be securely stored on meshed nodes.
- Nonce/timestamp in signatures prevent replay attacks.
- Border Gateway cannot fabricate RF proofs without meshed node’s private key.
- Sybil resistance maintained via standard onboarding/location assertion process.

## Reference Implementation
- **Meshed Node**: ChirpStack/RAK firmware update to:
  - Generate/handle gateway keypairs.
  - Sign RF metadata.
  - Wrap PoC packets for mesh forwarding.
- **Border Gateway** (`gateway-rs` or sidecar):
  - Accept delegated receipts.
  - Verify signatures.
  - Submit to PoC verifier.
- **PoC Verifier**:
  - Accept and process `delegate_of` receipts.
  - Apply reward distribution rules.

## Unresolved Questions
- Final reward split defaults — governance decision.
- Handling multi-hop mesh reward distribution.
- Interaction with denylist classifiers.

## Discussion Link
[https://discord.com/channels/404106811252408320/1075540098197757962]
