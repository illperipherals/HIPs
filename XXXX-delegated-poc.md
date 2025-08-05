# HIP-XXXX: Delegated Proof-of-Coverage for Meshed Gateways

- **Author(s):** tteague
- **Start Date:** 2025-08-04
- **Category:** Technical
- **Status:** Draft
- **Discussions-To:** https://discord.com/channels/404106811252408320/1075540098197757962
- **Created:** 2025-08-04

## Summary

This HIP proposes **Delegated Proof-of-Coverage (PoC)**, allowing [meshed ChirpStack/RAKWireless gateways](https://www.chirpstack.io/docs/chirpstack-gateway-mesh/) to earn **fully on‑chain PoC rewards** even when their traffic is routed through a Border Gateway.  
Currently, only the Border Gateway is recognized on‑chain and receives all PoC rewards, even when meshed gateways perform the actual RF work. This creates a fairness and incentive problem — valuable coverage is added to the network without corresponding rewards.  

Delegated PoC solves this by giving each meshed node its own on‑chain identity and cryptographic signing key. These nodes sign RF origin proofs, which the Border Gateway verifies and submits as **delegated PoC receipts** to the blockchain. Rewards are credited directly to the originating meshed node, with optional relay incentives for the Border Gateway.

---

## Motivation

Helium’s Proof-of-Coverage system is designed to reward **actual RF coverage**.  
In meshed deployments:
- Meshed nodes provide RF coverage that is potentially far from the Border Gateway.
- All traffic is tunneled back to the Border Gateway over mesh links.
- Only the Border Gateway is rewarded for PoC, even when it never heard the RF signal directly.

This is particularly problematic in:
- **Rural deployments** — where solar‑powered meshed gateways extend network reach.
- **Community deployments** — where individuals install meshed nodes but rewards flow to someone else.
- **Scalable urban mesh backhaul** — where many micro‑gateways backhaul over a single Border Gateway.

Without Delegated PoC, participants have **no economic reason** to deploy meshed gateways unless they control the Border Gateway.

**Example:**  
A farmer deploys three solar‑powered meshed gateways across remote fields to track livestock and sensors.  
All RF coverage is routed through a town‑center Border Gateway.  
Today, that farmer earns **zero PoC rewards** — the Border Gateway operator receives them all.  
With Delegated PoC, each solar‑powered meshed gateway would receive its fair share of rewards, based on actual RF work.

---

## Stakeholders

- **Gateway Owners** — Gain direct incentive to deploy meshed nodes.
- **Helium Network** — Gains more accurate and dense coverage mapping.
- **Manufacturers/Operators** — Can offer economically viable mesh‑based deployments.

---

## Goals

- Attribute PoC rewards to the gateway that actually performed the RF work.
- Maintain all reward attribution and distribution fully on‑chain.
- Preserve PoC’s cryptographic trust guarantees.

---

## Non-Goals

- Off-chain reward distribution or trust‑based splitting.
- Removing incentives for Border Gateways to participate.

---

## Specification

1. **Virtual Gateway Identities**  
   - Each meshed node generates a **Helium gateway keypair** (Ed25519 or ECC).  
   - Registered on‑chain like a standard hotspot:
     - Location asserted.
     - Owner wallet set.  
   - Keys are used only to sign PoC metadata — no direct blockchain connection needed.

2. **RF Origin Proof Signing**  
   When a meshed node **receives** a PoC packet:
   - Signs:
     - Gateway public key
     - RSSI/SNR
     - Frequency/sub‑band
     - Timestamp
     - Random nonce
     - Optional hashed location

   When a meshed node **transmits** a PoC packet:
   - Logs and signs challenge metadata for verification.

3. **Delegated Receipt Format**  

| Field         | Description                  |
|---------------|------------------------------|
| `witness`     | Meshed node address          |
| `delegate_of` | Border gateway address       |
| `rssi`        | Measured RSSI                |
| `snr`         | Measured SNR                 |
| `frequency`   | Frequency / sub-band         |
| `timestamp`   | UTC timestamp                |
| `nonce`       | Unique per-packet nonce      |
| `signature`   | Meshed node proof signature  |

4. **Border Gateway Role**  
   - Verifies the meshed node’s signature.  
   - Submits the delegated receipt to the PoC verifier.  
   - Acts as the sole uplink to the Helium network for meshed nodes.

5. **PoC Verifier Processing**  
   - Confirms:
     - `witness` is a registered hotspot.
     - Signature matches registered public key.
     - `delegate_of` is a registered hotspot.  
   - Credits rewards to the meshed node’s wallet.  
   - Optionally credits a relay incentive to the Border Gateway.

---

## Reward Model

Default recommendation:
- **90%** to the RF-originating meshed node.
- **10%** to the Border Gateway for transport/relay.
- Governance may adjust split ratios, and apply different splits for **multi-hop** mesh scenarios.
- Governance may allow performance‑based relay incentives for high‑quality Border Gateways.

---

## Rationale

- Works within the existing PoC trust model — only adds a new delegated receipt type.
- Keeps all reward attribution and distribution **fully on‑chain**.
- Encourages network densification.
- Improves RF coverage mapping accuracy.

---

## Backwards Compatibility

- No changes for non‑meshed gateways.
- Fully compatible with current PoC system.
- Delegated PoC receipts are additive — no disruption to existing witness/challenge flows.

---

## Security Considerations

### Sybil & Gaming Resistance
Delegated PoC introduces a potential vector for fraudulent witnesses if meshed node proofs can be faked. The design addresses this by:
- Binding each meshed node to an on‑chain identity with location assertion.
- Using cryptographic signatures for all delegated PoC receipts.
- Requiring Border Gateways to verify proofs before submission.
- Applying the same PoC plausibility checks (RSSI/SNR/geographic) to meshed nodes as to physical hotspots.

### Non‑ECC Participation & Anti‑Gaming Measures
While ECC‑enabled gateways provide the strongest hardware‑bound proof guarantees, non‑ECC meshed gateways can still participate to preserve inclusivity. To mitigate gaming risk:

1. **Onboarding Cost & Location Assertion**  
   - Standard HIP‑19 onboarding fee for each meshed gateway ID, even if virtual.  
   - Require location assertion or GPS‑verified coordinates at registration.

2. **Border Gateway Reputation Tracking**  
   - All delegated PoC receipts are tied to the submitting Border Gateway ID.  
   - Suspicious activity can lead to loss of relay incentives or delegated PoC rights.

3. **Radio‑Realistic Validation**  
   - RSSI/SNR plausibility checks for each meshed gateway’s asserted location.  
   - Flags identical or unrealistic signal patterns.

4. **Replay‑Resistant Software Keys**  
   - Non‑ECC gateways must use software keypairs (e.g., Ed25519) to sign each PoC receipt.  
   - Each signature must include a nonce, timestamp, and packet hash.

5. **Reward Ramp for New Identities**  
   - Limit PoC rewards for the first 30 days of a new meshed gateway.  
   - Increase rewards over time after passing plausibility checks.

6. **Witness Diversity Analysis**  
   - Detect repetitive or correlated witness patterns that indicate gaming.

---

## Benefits to the Network

- **Fairness** — Rewards follow the RF work, not the backhaul topology.
- **Incentive for Expansion** — Encourages solar‑powered and rural deployments.
- **Better Mapping** — More accurate PoC coverage maps.
- **Flexibility** — Works for ECC and non‑ECC gateways.
- **Security** — Maintains PoC trust model with added anti‑gaming layers.
- **Industry Alignment** - Accomodates the observed movement toward meshed deployments.

---

## Reference Implementation

- **Meshed Node** — ChirpStack/RAK firmware:
  - Generate/handle gateway keypairs.
  - Sign RF metadata.
  - Wrap PoC packets for mesh forwarding.

- **Border Gateway** — `gateway-rs` or sidecar:
  - Accept delegated receipts.
  - Verify signatures.
  - Submit to PoC verifier.

- **PoC Verifier** — Consensus component:
  - Accept `delegate_of` receipts.
  - Apply reward distribution logic.

---

## Unresolved Questions

- Optimal default reward split.
- Governance rules for multi‑hop mesh rewards.
- Interaction with denylist classifiers.

---

## Discussion Link

https://discord.com/channels/404106811252408320/1075540098197757962
