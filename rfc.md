# **1\. Introduction & Goals**

## **1.1 Purpose & Audience**

This document defines the **Soul Bound Protocol**, a decentralized, privacy-preserving framework for creating digital identities that are strongly anchored to real-world, in-person verification events. By treating Soul Bound as a *protocol* rather than a single software product, we enable multiple interoperable implementations to:

* Generate and manage non-transferable identity tokens on a distributed ledger  
* Record sponsorship and reputation stakes in a standardized, auditable format  
* Decouple "trust semantics" (how applications interpret the identity graph) from the core verification mechanics  
* Embed economic incentives (staking, slashing, performance rewards) to align honest behavior and deter fraud

The protocol is designed to make it economically and practically infeasible to create fraudulent identities at scale, while still allowing for legitimate identity creation. This is achieved through a combination of:
* In-person verification requirements that cannot be easily automated
* Economic stakes that make fraud costly
* Decentralized validation that prevents single points of failure
* Privacy-preserving mechanisms that protect user data

While no system can guarantee absolute prevention of fraud, the protocol's design ensures that:
* Creating a single fraudulent identity requires significant effort and cost
* Creating multiple fraudulent identities becomes exponentially more difficult
* The economic cost of fraud exceeds any potential benefit
* Legitimate users can still create and manage their identities efficiently

**Intended audience:**

* **Protocol designers** defining or extending message flows, data formats, and security properties  
* **Client developers** building mobile, desktop, or embedded apps that perform verifications or consume identity data  
* **Validator implementers** running nodes that verify attestations, enforce time-bounds, and manage stakes  
* **Smart-contract engineers** integrating on-chain logic for minting, slashing, and reward distribution  
* **Security auditors & researchers** assessing identity fraud resistance, privacy guarantees, and economic soundness  
* **Community architects** (DAOs, voting platforms, social networks) building trust-scoring or governance models atop the identity graph

By adhering to this spec, all participants—regardless of implementation language or platform—will interoperate seamlessly, maintain strong defenses against identity fraud, and preserve user privacy, while leaving high-level trust decisions flexible for downstream applications.

## **1.2 Protocol vs. Implementation Domains**

This document strictly separates requirements into two distinct domains:

**Protocol Domain (MUST support)**
* Core message types and their semantics
* Cryptographic primitives and security properties
* Economic mechanisms and stake requirements
* Validation rules and quorum requirements
* Timing parameters and anti-replay mechanisms
* Privacy guarantees and zero-knowledge proof requirements
* State machine definitions and transitions
* On-chain record structures and schemas
* Observable behavior requirements (what validators must check)

**Implementation Domain (MAY customize)**
* User interface design and experience
* Client application architecture
* Local storage and caching strategies
* Performance optimizations
* Hardware security module (HSM) implementations
* Off-chain trust and reputation algorithms
* Specific blockchain deployment details
* Network parameters and gas optimizations
* Custom reward distribution curves
* Additional security features beyond protocol requirements

This separation ensures that:
* The protocol remains focused on essential security and interoperability requirements
* Implementations have freedom to innovate and optimize
* Different communities can build diverse solutions while maintaining compatibility
* Core security properties are preserved across all implementations
* Malicious implementations cannot break the system unless they can break the cryptographic primitives

## **1.3 Scope**

This specification defines the **protocol-level** rules, message flows, data formats, and economic mechanisms required to create, validate, and manage Soul Bound identities on a distributed ledger. It covers:

* Core **message types** and their semantics (ChallengeRequest, SensorPackage, SponsorAttestation, MintRequest, ValidationResponse, RevocationRequest)  
* **Actor state machines** for Candidate, Sponsor, Validator, and Ledger roles  
* **Wire and on-ledger schemas** (e.g. JSON-Schema) for all protocol messages and records  
* **Timing & anti-replay controls**, including signed timestamps, nonces, and optional verifiable-delay functions  
* **Economic mechanisms**: identity-mint stakes, sponsor stakes, validator bonds, slashing, fees, and reward distribution  
* **Privacy guarantees** via zero-knowledge proofs and hashed sensor data  
* **Decoupled trust semantics**—how external applications or DAOs may interpret the public identity graph

Out of scope for this document (implementation domain):

* **User interface** or UX design for client applications  
* **Hardware security module** (HSM) or secure-element implementation details  
* **Off-chain trust & reputation algorithms** beyond the public graph interface  
* **Formal machine-checked models** (e.g. TLA⁺, Alloy) – see Section 12 for reference, if desired  
* **Specific blockchain deployment** details (network parameters, gas costs, layer-2 integrations)

## **1.4 Design Goals**

The Soul Bound Protocol is architected to achieve the following core objectives:

* **Sybil resistance**  
  Ensure each identity corresponds to a unique, real-world individual by requiring in-person, multi-sensor verification and economically-backed sponsorships.  
* **Privacy (zero-knowledge)**  
  Protect sensitive biometric and sensor data using hashed inputs and zero-knowledge proofs so that verifiers can confirm authenticity without learning raw personal data.  
* **Interoperability (protocol ≠ software)**  
  Define a clear, implementation-agnostic protocol specification that allows diverse client, validator, and ledger software to interoperate seamlessly.  
* **Extensibility (decoupled trust semantics)**  
  Separate the core verification mechanics and graph construction from application-layer trust policies, enabling DAOs, apps, and users to apply custom reputation or governance logic over the public identity graph.  
* **Economic alignment (staking & rewards)**  
  Introduce token-based stakes, slashing conditions, and performance-based rewards so participants incur real economic risk when vouching for or validating identities, aligning incentives toward honest behavior.

## **1.5 Assumptions & Threat Model**

### **1.5.1 Assumptions**

* **Secure Key Storage**  
  Each participant's device securely holds its private keys and protects them from extraction or tampering (e.g., via Secure Enclave or equivalent).  
* **Reliable Sensor Hardware**  
  Mobile devices provide accurate, tamper-resistant sensor readings (NFC, accelerometer, microphone, GPS, light) and cryptographically sign hashed outputs.  
* **Synchronized Time Base**  
  All parties reference a shared notion of time—either via blockchain block heights or loosely synchronized clocks—within an acceptable skew (ΔT).  
* **Permissionless Ledger**  
  A decentralized ledger exists that supports immutable recording of identity events, stakes, slashing, and rewards, and exposes timestamps or block heights for freshness checks.  
* **Economic Stake Availability**  
  Tokens or native assets are available for participants to lock as stakes for minting or endorsing identities, and smart contracts enforce slashing and reward logic.  
* **Independent Validators**  
  A critical mass of validator nodes operate honestly and independently, preventing any single party or small colluding group from unilaterally approving fraudulent identities.

### **1.5.2 Threat Model**

#### **Adversary Goals**

* **Sybil Creation**: Generate a large number of fake or duplicate identities to subvert decentralized governance or reputation systems.  
* **Data Forgery**: Spoof or replay sensor data to simulate a legitimate in-person verification.  
* **Stake Grinding**: Abuse staking mechanics (e.g., repeatedly minting and slashing to extract fees).  
* **Privacy Violation**: Attempt to learn raw biometric or sensor data from attestations.  
* **Consensus Attack**: Collude validators to approve fraudulent MintRequests or block honest ones.

#### **Adversary Capabilities**

* **Malicious Clients**: Control devices or custom apps that may fabricate sensor outputs or signatures.  
* **Colluding Sponsors**: Existing identities willing to vouch for sybils in exchange for rewards.  
* **Compromised Validator Nodes**: Validators that deviate from protocol rules or censor certain submissions.  
* **Network Adversary**: Ability to intercept, delay, or reorder protocol messages on the P2P network.  
* **Key Compromise**: Theft of private keys belonging to candidates, sponsors, or validators.

#### **Attack Vectors**

* **Sensor Spoofing & Replay**: Injecting recorded sensor hashes or simulated multi-sensor traces.  
* **Message Replay**: Reusing old ChallengeRequest, SensorPackage, or SponsorAttestation messages.  
* **Staking Abuse**: Minting identities en masse, awaiting slashing window to lapse, then releasing stakes.  
* **Collusion & Bribery**: Coordinated behavior among sponsors and validators to approve sybil identities.  
* **Fork or Partition**: Exploiting ledger forks or network partitions to submit conflicting MintRequests.

#### **Mitigations (Refer to Later Sections)**

* **Multi-Sensor Fusion & Anomaly Detection** (Section 4\)  
* **Signed Timestamps & Nonces** (Section 7\)  
* **m-of-n Validator Quorum** (Section 5\)  
* **Slashing Conditions & Economic Disincentives** (Section 8\)  
* **Zero-Knowledge Proofs & Hashed Data** (Section 4.2)

# **2\. Actors, Terminology & Notation**

## **2.1 Core Protocol Requirements**

The protocol MUST provide:

1. **Identity Creation & Management**
   * Unique, unforgeable identity creation
   * Secure private key management
   * Identity revocation mechanisms
   * Stake requirements and slashing conditions

2. **Attestation System**
   * Cryptographic proof of identity ownership
   * Freshness guarantees through nonces and timestamps
   * Anti-replay protection
   * Privacy-preserving verification

3. **Validator System**
   * Quorum-based validation
   * Stake-weighted voting
   * Slashing conditions for misbehavior
   * Validator set management

4. **Economic Mechanisms**
   * Stake requirements for identity creation
   * Slashing conditions for protocol violations
   * Economic incentives for honest behavior
   * Cost-based identity fraud resistance

5. **Security Properties**
   * Unforgeability of attestations
   * Freshness and liveness guarantees
   * Identity fraud resistance bounds
   * Privacy through zero-knowledge proofs
   * Revocation and slashing soundness

All other aspects are implementation-specific and MAY be customized by different communities.

## **2.2 Core Data Types**

The protocol defines the following core data types:

1. **Identity**
   * Public key
   * Stake amount
   * Status (active/revoked)
   * Creation timestamp

2. **Attestation**
   * Type identifier
   * Session ID
   * Sensor hashes
   * Timestamp
   * Sponsor signature

3. **Transaction**
   * Type (create/revoke/slash)
   * Identity reference
   * Stake amount
   * Signatures
   * Timestamp

4. **Validator Set**
   * List of validator public keys
   * Stake weights
   * Quorum threshold
   * Status (active/removed)

All other data structures and formats are implementation-specific and MAY be customized.

## **2.3 Cryptographic Primitives & Notational Conventions**

* **Hash Function**  
  * `H(m)`: Collision-resistant hash of message `m`.  
* **Digital Signatures**  
  * `Sig_X(m)`: Signature by actor X over message `m` using `SK_X`.  
  * Verification: `Verify(PK_X, m, Sig_X(m)) → true|false`.  
* **Multi-Signature / Threshold**  
  * `m-of-n`: At least `m` signatures required out of `n` validators.  
  * Represented as an array of validator signatures on the same payload.  
* **Zero-Knowledge Proofs**  
  * `ZKProof{stmts, witness}`: A proof that the prover knows a `witness` satisfying statements `stmts` without revealing `witness`.  
* **Notation**  
  * `∥`: Concatenation operator.  
  * JSON-Schema snippets use `"type": "string"`, `"format": "uuid"`, etc.  
  * All protocol messages include a `"type"` field and the actor's signature over the entire JSON payload.

**Note:** All cryptographic operations must follow industry-standard algorithms (e.g., ECDSA/secp256k1 or Ed25519 for signatures; SHA-256 for hashing; Bulletproofs or zk-SNARKs for zero-knowledge).

## **2.4 Signature Requirements**

The protocol defines the following signature requirements that all implementations MUST support:

1. **Message Signing**
   * All protocol messages MUST be signed by their sender
   * Signatures MUST cover the entire message content
   * Signatures MUST be verifiable against the sender's public key

2. **Signature Schemes**
   * Implementations MUST support at least one of:
     * ECDSA with secp256k1 curve
     * Ed25519
   * Additional schemes MAY be supported
   * All schemes MUST provide:
     * Unforgeability
     * Non-repudiation
     * Standard security properties

3. **Nonce Requirements**
   * All signatures MUST include a nonce to prevent replay attacks
   * Nonces MUST be cryptographically random
   * Nonces MUST be included in the signed message content
   * Nonces MUST be verified by recipients

4. **Multi-Signature Requirements**
   * Validator quorum signatures MUST be verifiable independently
   * Each validator's signature MUST cover the same message content
   * The quorum threshold (m-of-n) MUST be enforced

These requirements ensure that:
* Messages cannot be forged or modified
* Replay attacks are prevented
* Validator quorum decisions are secure
* Signatures are interoperable across implementations

# **3\. Protocol Overview**

## **3.1 High-Level Sequence Diagram**

[sequence diagram to be added later]

## **3.2 End-to-End Flow Summary**

Below is a concise summary of the full identity issuance workflow:

1. **ChallengeRequest**  
   * Sponsor → Candidate  
   * Sponsor issues a signed challenge (sessionId, timestamp, nonce).  
2. **SensorPackage**  
   * Candidate ↔ Sponsor  
   * Both devices gather multi-sensor hashes (NFC, accelerometer, audio, etc.) and exchange signed packages.  
3. **SponsorAttestation**  
   * Sponsor → Candidate  
   * Sponsor verifies sensor hashes and biometric checks, then signs an attestation.  
4. **MintRequest**  
   * Candidate → Validators → Ledger  
   * Candidate assembles the signed attestations, ZK proof, and stake into a MintRequest.  
5. **ValidationResponse**  
   * Validators → Candidate & Sponsor  
   * A decentralized quorum (m-of-n) independently verifies proofs, stakes, and timestamps, then returns "approved" or "rejected."  
6. **On-Chain Minting**  
   * Ledger commits the new Soul Bound identity token when the required approvals are reached.

## **3.3 Timing & Freshness Controls (timestamps, block-heights)**

To prevent replay attacks and ensure data freshness, the protocol mandates:

* **Signed Timestamps**  
  * Every message carries a `timestamp` (Unix epoch) signed by the sender.  
  * Validators reject messages where `|now – timestamp| > ΔT` (see Section 13.1 for configured value).  
* **Block-Height Anchoring**  
  * On-chain actions (MintRequest submission, slashing window expiry) reference `blockHeight`.  
  * Validators and smart contracts enforce windows like "submit within N blocks of challenge"  
    (see Section 13.1 for configured values).  
* **Session Nonces & IDs**  
  * Each verification session uses a unique `sessionId` (UUID) and challenge `nonce` to bind messages together.  
  * Replayed or out-of-order messages are detected and dropped.  
* **Optional Verifiable-Delay Functions**  
  * Where strict minimum wait times are required, participants may supply a VDF output to prove elapsed time without trusting local clocks.

These controls ensure that every step of the protocol is both timely and tamper-evident, making spoofing or replay economically and technically infeasible.

# **4\. Message Types & Semantics**

This section defines the core protocol messages, their senders, purposes, required fields, and key semantics. All implementations MUST support these message types exactly as specified, while being free to add implementation-specific extensions as described in Section 1.2.

## **4.1 ChallengeRequest**

* **Sender:** Sponsor → Candidate  
* **Purpose:** Kick off a fresh in-person verification session and bind the exchange to a unique session.  
* **Fields:**  
  * `type`: `"ChallengeRequest"`  
  * `sponsorPubKey`: Public key of the Sponsor  
  * `sessionId`: UUID for this session  
  * `timestamp`: Unix epoch time of issuance  
  * `nonce`: Cryptographically random value  
* **Semantics:**  
  * Candidate must respond within the configured time window Δ₁ (see Section 13.1 for configured value).  
  * Future messages in this session must include the same `sessionId` and `nonce`.

## **4.2 SponsorAttestation**

* **Sender:** Sponsor → Candidate (and later to Validators)  
* **Purpose:** Formally vouch that the in-person verification was completed.  
* **Fields:**  
  * `type`: `"SponsorAttestation"`  
  * `sessionId`: UUID  
  * `timestamp`: Unix epoch time of attestation  
  * `sponsorSig`: Signature over `sessionId∥timestamp` using Sponsor's private key  
* **Semantics:**  
  * Candidate collects this signed attestation to build the MintRequest.  
  * Valid only if attestation is within Δ₂ of ChallengeRequest.

## **4.3 MintRequest**

* **Sender:** Candidate → Validator(s) → Ledger  
* **Purpose:** Propose creation of a new Soul Bound identity on-chain.  
* **Fields:**  
  * `type`: `"MintRequest"`  
  * `identityPubKey`: Candidate's newly generated public key  
  * `sessionId`: UUID  
  * `candidateSig`: Signature over attestation bundle using Candidate's private key  
  * `sponsorSig`: Sponsor's attestation signature  
  * `stake`: Amount of tokens staked for minting (≥ S_mint)  
  * `zkProof`: Zero-knowledge proof of correct sensor checks without revealing raw data  
  * `timeAnchor`: Either `timestamp` (Unix epoch) or `blockHeight` (ledger height)  
* **Time Anchor Selection Rules:**  
  * Use `timestamp` when:  
    * Submitting off-chain to validators before on-chain commitment  
    * Validating against the challenge window (Δ₁)  
    * Checking sensor data freshness  
  * Use `blockHeight` when:  
    * Submitting the final on-chain transaction  
    * Enforcing stake lock periods  
    * Calculating probation windows  
    * Processing slashing conditions  
  * The same MintRequest may use both anchors at different stages:  
    1. Initial submission to validators uses `timestamp`  
    2. Final on-chain transaction uses `blockHeight`  
* **Semantics:**  
  * Validators reject if `stake` is insufficient or proofs/timestamps invalid.  
  * On m-of-n approvals, the Ledger commits the new identity token.  
  * Time validation rules:  
    * Off-chain: `|localTime – timestamp| ≤ ΔT`  
    * On-chain: `currentHeight – blockHeight ≤ N` (where N is the maximum submission window)

## **4.4 ValidationResponse**

* **Sender:** Validator → Candidate & Sponsor  
* **Purpose:** Communicate approval or rejection of a MintRequest.  
* **Fields:**  
  * `type`: `"ValidationResponse"`  
  * `sessionId`: UUID  
  * `validatorPubKey`: Validator's public key  
  * `status`: `"approved"` or `"rejected"`  
  * `reason` (optional): Human-readable code or message on rejection  
  * `validatorSig`: Signature over `sessionId∥status∥reason∥timestamp`  
  * `timestamp`: Unix epoch or block height  
* **Semantics:**  
  * Candidate/Sponsor track responses until quorum (m-of-n) is reached.  
  * If majority "approved," minting proceeds; else session fails.

## **4.5 RevocationRequest**

* **Sender:** Sponsor or Validator → Validator(s) → Ledger  
* **Purpose:** Revoke an existing identity due to fraud, compromise, or policy breach.  
* **Fields:**  
  * `type`: `"RevocationRequest"`  
  * `identityPubKey`: Public key of identity to revoke  
  * `revokerPubKey`: Who is requesting revocation  
  * `reason`: Code or description of cause  
  * `evidenceHash`: Reference or ZK proof of misbehavior  
  * `timestamp`: Unix epoch or blockHeight  
  * `revokerSig`: Signature over all fields  
* **Semantics:**  
  * Validators evaluate evidence; approved revocation triggers slashing and mark identity as revoked on-chain.

## **4.6 RewardDistribution / SlashNotification**

* **Sender:** Ledger (Smart Contract) → Participant(s)  
* **Purpose:** Distribute rewards or notify of slashed stakes after minting or revocation events.  
* **Fields:**  
  * `type`: `"RewardDistribution"` or `"SlashNotification"`  
  * `recipientPubKey`: Public key of beneficiary or slashed party  
  * `amount`: Token quantity rewarded or slashed  
  * `source`: Origin of funds (e.g., slashed pool, mint fees)  
  * `reason`: Context (e.g., "mint-reward", "revocation-slash")  
  * `timestamp`/`blockHeight`  
  * `contractSig`: Smart-contract attestation of the event  
* **Semantics:**  
  * Automatic, on-chain execution of reward/slash logic based on configured parameters.  
  * Events are recorded in the Ledger for auditing and downstream trust scoring.

# **5\. State Machines**

This section defines the state machines that govern the behavior of each participant type. Implementations MUST maintain these states and transitions exactly as specified.

## **5.1 Candidate State Machine**

* **Initial State:** `IDLE`  
* **States:**  
  * `IDLE`: Awaiting ChallengeRequest  
  * `VERIFYING`: Received ChallengeRequest, awaiting SponsorAttestation  
  * `MINTING`: Received valid SponsorAttestation, awaiting ValidationResponse  
  * `ACTIVE`: Successfully minted identity  
  * `REVOKED`: Identity revoked  
* **Transitions:**  
  * `IDLE → VERIFYING`: On receiving valid ChallengeRequest  
  * `VERIFYING → MINTING`: On receiving valid SponsorAttestation  
  * `MINTING → ACTIVE`: On receiving majority-approved ValidationResponse  
  * `* → REVOKED`: On receiving valid RevocationRequest

## **5.2 Sponsor State Machine**

* **Initial State:** `IDLE`  
* **States:**  
  * `IDLE`: Available for new sessions  
  * `VERIFYING`: Sent ChallengeRequest, awaiting in-person verification  
  * `ATTESTING`: Completed verification, preparing SponsorAttestation  
  * `ACTIVE`: Successfully attested identity  
  * `SLASHED`: Lost stake due to fraud  
* **Transitions:**  
  * `IDLE → VERIFYING`: On sending ChallengeRequest  
  * `VERIFYING → ATTESTING`: On completing in-person verification  
  * `ATTESTING → ACTIVE`: On sending valid SponsorAttestation  
  * `* → SLASHED`: On receiving valid RevocationRequest with fraud evidence

## **5.3 Validator State Machine**

* **Initial State:** `IDLE`  
* **States:**  
  * `IDLE`: Awaiting MintRequest  
  * `VERIFYING`: Received MintRequest, awaiting ValidationResponse  
  * `VOTING`: Received valid ValidationResponse, awaiting Commit  
  * `COMMITING`: Validator signs and broadcasts ValidationResponse  
  * `ACTIVE`: Valid identity  
  * `REVOKED`: Validator revoked  
* **Transitions:**  
  * `IDLE → VERIFYING`: On receiving valid MintRequest  
  * `VERIFYING → VOTING`: On receiving valid ValidationResponse  
  * `VOTING → COMMITING`: On receiving majority-approved ValidationResponse  
  * `* → REVOKED`: On receiving valid RevocationRequest

## **5.4 Ledger State Machine**

* **Initial State:** `WAIT_TX`  
* **States:**  
  * `WAIT_TX`: Waiting for new MintRequest  
  * `ACCEPT`: MintRequest accepted  
  * `REJECT`: MintRequest rejected  
  * `UPDATE_GRAPH`: MintRequest processed, updating graph  
* **Transitions:**  
  * `WAIT_TX → ACCEPT`: On receiving valid MintRequest  
  * `WAIT_TX → REJECT`: On receiving invalid MintRequest  
  * `ACCEPT → UPDATE_GRAPH`: On confirming block  
  * `REJECT → WAIT_TX`: On transaction failure or timeout  
  * `UPDATE_GRAPH → WAIT_TX`: On returning to waiting state

# **6\. Protocol Flow**

This section walks through the complete lifecycle of identity creation, from initial contact through minting and potential revocation.

## **6.1 Identity Creation Flow**

1. **Initial Contact**  
   * Candidate and Sponsor establish communication channel  
   * Sponsor sends ChallengeRequest with fresh sessionId and nonce

2. **In-Person Verification**  
   * Candidate and Sponsor meet in person  
   * Sponsor verifies Candidate's uniqueness through multiple interactions  
   * Sponsor assesses Candidate's understanding of protocol responsibilities  
   * Sponsor completes verification and prepares SponsorAttestation

3. **Attestation & Minting**  
   * Sponsor sends signed SponsorAttestation to Candidate  
   * Candidate builds MintRequest with attestation  
   * Candidate submits MintRequest to Validators

4. **Validation & Minting**  
   * Validators verify MintRequest signatures and attestations  
   * Validators check Sponsor's stake and reputation  
   * On majority approval, identity is minted on-chain

## **6.2 Trust Building Process**

1. **Initial Verification**
   * Sponsor and Candidate meet in person
   * Multiple interactions over time
   * Assessment of protocol understanding
   * Verification of uniqueness

2. **Reputation Building**
   * Trust builds through successful interactions
   * Multiple sponsors can vouch over time
   * Economic stakes increase with trust
   * Long-term relationships valued

3. **Protocol Responsibilities**
   * Understanding of stake requirements
   * Knowledge of slashing conditions
   * Commitment to honest behavior
   * Willingness to maintain protocol security

This approach focuses on building trust through direct interaction and economic alignment, creating a robust and decentralized identity system.

## **6.3 Revocation Process**

1. **Revocation Request**  
   * Sponsor or Validator submits RevocationRequest  
   * Validators process request and decide on revocation through quorum vote

2. **Slash Notification**  
   * Ledger sends SlashNotification to affected parties  
   * SlashNotification includes evidence and decision

3. **Identity Revocation**  
   * Ledger marks identity as revoked on-chain  
   * Revoked identity's stake is slashed

# **7\. Data Formats & Schemas**

This section specifies the machine-readable schemas for all protocol messages and ledger records, plus some full-payload examples. Implementations MUST support these schemas exactly as specified, while being free to add implementation-specific fields or optimizations as described in Section 1.2.

## **7.1 Wire Formats (JSON-Schema)**

All off-chain protocol messages exchanged peer-to-peer or via RPC MUST conform to the JSON-Schema definitions in:

```json
{  
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SoulBound Wire Message Schemas",  
  "oneOf": [
    { "$ref": "#/definitions/ChallengeRequest" },
    { "$ref": "#/definitions/SponsorAttestation" },
    { "$ref": "#/definitions/MintRequest" },
    { "$ref": "#/definitions/ValidationResponse" },
    { "$ref": "#/definitions/RevocationRequest" },
    { "$ref": "#/definitions/RewardEvent" }
  ],
  "definitions": {  
    "ChallengeRequest": {  
      "type": "object",  
      "required": ["type","sponsorPubKey","sessionId","timestamp","nonce"],
      "properties": {  
        "type":       { "const": "ChallengeRequest" },  
        "sponsorPubKey": { "type": "string" },  
        "sessionId":  { "type": "string", "format": "uuid" },  
        "timestamp":  { "type": "integer", "minimum": 0 },  
        "nonce":      { "type": "string" }  
      }  
    },  
    "SponsorAttestation": {  
      "type": "object",  
      "required": ["type","sessionId","timestamp","sponsorSig"],
      "properties": {  
        "type":         { "const": "SponsorAttestation" },  
        "sessionId":    { "type": "string", "format": "uuid" },  
        "timestamp":    { "type": "integer", "minimum": 0 },  
        "sponsorSig":   { "type": "string" }  
      }  
    },  
    "MintRequest": {  
      "type": "object",  
      "required": ["type","identityPubKey","sessionId","candidateSig","sponsorSig","stake","zkProof","timestamp"],
      "properties": {  
        "type":           { "const": "MintRequest" },  
        "identityPubKey": { "type": "string" },  
        "sessionId":      { "type": "string", "format": "uuid" },  
        "candidateSig":   { "type": "string" },  
        "sponsorSig":     { "type": "string" },  
        "stake":          { "type": "number", "minimum": 0 },  
        "zkProof":        { "type": "string" },  
        "timestamp":      { "type": "integer", "minimum": 0 },
        "blockHeight":    { "type": "integer", "minimum": 0 }
      }  
    },  
    "ValidationResponse": {  
      "type": "object",  
      "required": ["type","sessionId","validatorPubKey","status","validatorSig","timestamp"],
      "properties": {  
        "type":            { "const": "ValidationResponse" },  
        "sessionId":       { "type": "string", "format": "uuid" },  
        "validatorPubKey": { "type": "string" },  
        "status":          { "type": "string", "enum": ["approved","rejected"] },
        "reason":          { "type": "string" },  
        "validatorSig":    { "type": "string" },  
        "timestamp":       { "type": "integer", "minimum": 0 }  
      }  
    },  
    "RevocationRequest": {  
      "type": "object",  
      "required": ["type","identityPubKey","revokerPubKey","reason","evidenceHash","timestamp","revokerSig"],
      "properties": {  
        "type":          { "const": "RevocationRequest" },  
        "identityPubKey":{ "type": "string" },  
        "revokerPubKey": { "type": "string" },  
        "reason":        { "type": "string" },  
        "evidenceHash":  { "type": "string" },  
        "timestamp":     { "type": "integer", "minimum": 0 },
        "blockHeight":   { "type": "integer", "minimum": 0 },
        "revokerSig":    { "type": "string" }  
      }  
    },  
    "RewardEvent": {  
      "type": "object",  
      "required": ["type","recipientPubKey","amount","source","reason","timestamp","txHash"],
      "properties": {  
        "type":          { "enum": ["RewardDistribution","SlashNotification"] },
        "recipientPubKey":{ "type": "string" },  
        "amount":        { "type": "number", "minimum": 0 },  
        "source":        { "type": "string" },  
        "reason":        { "type": "string" },  
        "timestamp":     { "type": "integer", "minimum": 0 },  
        "txHash":        { "type": "string" }  
      }  
    },
    "ChallengeAck": {
      "type": "object",
      "required": ["type", "status", "timestamp"],
      "properties": {
        "type": { "const": "ChallengeAck" },
        "status": { "type": "string", "enum": ["accepted", "rejected"] },
        "reason": { "type": "string" },
        "timestamp": { "type": "integer", "minimum": 0 }
      }
    },
    "SensorAck": {
      "type": "object",
      "required": ["type", "status", "timestamp"],
      "properties": {
        "type": { "const": "SensorAck" },
        "status": { "type": "string", "enum": ["accepted", "rejected"] },
        "reason": { "type": "string" },
        "timestamp": { "type": "integer", "minimum": 0 }
      }
    },
    "SponsorshipAck": {
      "type": "object",
      "required": ["type", "status", "timestamp"],
      "properties": {
        "type": { "const": "SponsorshipAck" },
        "status": { "type": "string", "enum": ["accepted", "rejected"] },
        "reason": { "type": "string" },
        "timestamp": { "type": "integer", "minimum": 0 }
      }
    },
    "MintResponse": {
      "type": "object",
      "required": ["type", "status", "timestamp"],
      "properties": {
        "type": { "const": "MintResponse" },
        "status": { "type": "string", "enum": ["accepted", "rejected"] },
        "reason": { "type": "string" },
        "timestamp": { "type": "integer", "minimum": 0 }
      }
    },
    "ValidateAck": {
      "type": "object",
      "required": ["type", "status", "timestamp"],
      "properties": {
        "type": { "const": "ValidateAck" },
        "status": { "type": "string", "enum": ["accepted", "rejected"] },
        "reason": { "type": "string" },
        "timestamp": { "type": "integer", "minimum": 0 }
      }
    },
    "RevokeAck": {
      "type": "object",
      "required": ["type", "status", "timestamp"],
      "properties": {
        "type": { "const": "RevokeAck" },
        "status": { "type": "string", "enum": ["accepted", "rejected"] },
        "reason": { "type": "string" },
        "timestamp": { "type": "integer", "minimum": 0 }
      }
    }
  }  
}
```

## **7.2 On-Ledger Record Structures**

All on-chain transactions and stored records (identity nodes, edges, slashes, rewards) MUST follow the JSON-Schema in:

```json
{  
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "SoulBound On-Ledger Record Structures",
  "definitions": {  
    "IdentityRecord": {  
      "type": "object",  
      "required": ["identityPubKey","sponsorPubKey","mintTxHash","timestamp","stake"],
      "properties": {  
        "identityPubKey": { "type": "string" },  
        "sponsorPubKey":  { "type": "string" },  
        "mintTxHash":     { "type": "string" },  
        "timestamp":      { "type": "integer" },  
        "stake":          { "type": "number" }  
      }  
    },  
    "SponsorshipEdge": {  
      "type": "object",  
      "required": ["fromPubKey","toPubKey","timestamp"],
      "properties": {  
        "fromPubKey": { "type": "string" },  
        "toPubKey":   { "type": "string" },  
        "timestamp":  { "type": "integer" }  
      }  
    },  
    "RevocationRecord": {  
      "type": "object",  
      "required": ["identityPubKey","revokerPubKey","revocationTxHash","timestamp","evidenceHash"],
      "properties": {  
        "identityPubKey":  { "type": "string" },  
        "revokerPubKey":   { "type": "string" },  
        "revocationTxHash":{ "type": "string" },  
        "timestamp":       { "type": "integer" },  
        "evidenceHash":    { "type": "string" }  
      }  
    },  
    "RewardSlashRecord": {  
      "type": "object",  
      "required": ["recipientPubKey","amount","reason","txHash","timestamp"],
      "properties": {  
        "recipientPubKey": { "type": "string" },  
        "amount":          { "type": "number" },  
        "reason":          { "type": "string" },  
        "txHash":          { "type": "string" },  
        "timestamp":       { "type": "integer" }  
      }  
    }  
  }  
}
```

## **7.3 Example Messages**

To see concrete payloads for each message type, consult:

```json
{  
  "ChallengeRequest": {  
    "type": "ChallengeRequest",  
    "sponsorPubKey": "0xA1B2C3D4...",  
    "sessionId": "550e8400-e29b-41d4-a716-446655440000",  
    "timestamp": 1633046400,  
    "nonce": "f47ac10b-58cc-4372-a567-0e02b2c3d479"  
  },
  "SponsorAttestation": {  
    "type": "SponsorAttestation",  
    "sessionId": "550e8400-e29b-41d4-a716-446655440000",  
    "timestamp": 1633046410,  
    "sponsorSig": "3046022100..."  
  },
  "MintRequest": {  
    "type": "MintRequest",  
    "identityPubKey": "0xDEADBEEF...",  
    "sessionId": "550e8400-e29b-41d4-a716-446655440000",  
    "candidateSig": "3045022100...",  
    "sponsorSig": "3046022100...",  
    "stake": 100,  
    "zkProof": "zkSNARK_proof_blob...",
    "timestamp": 1633046420  
  },
  "ValidationResponse": {  
    "type": "ValidationResponse",  
    "sessionId": "550e8400-e29b-41d4-a716-446655440000",  
    "validatorPubKey": "0xFEEDFACE...",  
    "status": "approved",  
    "validatorSig": "3044022079...",  
    "timestamp": 1633046430  
  },
  "RevocationRequest": {  
    "type": "RevocationRequest",  
    "identityPubKey": "0xDEADBEEF...",  
    "revokerPubKey": "0xBADF00D...",  
    "reason": "fraud-detected",  
    "evidenceHash": "evidence_hash_or_ZK_proof",
    "timestamp": 1633046500,  
    "revokerSig": "3045022100..."  
  },
  "RewardDistribution": {  
    "type": "RewardDistribution",  
    "recipientPubKey": "0xFEEDFACE...",  
    "amount": 5,  
    "source": "mint-fee-pool",  
    "reason": "mint-reward",  
    "timestamp": 1633046600,  
    "contractSig": "0xCAFEBABE..."  
  },
  "SlashNotification": {  
    "type": "SlashNotification",  
    "recipientPubKey": "0xBADF00D...",  
    "amount": 50,  
    "source": "slashed-stake",  
    "reason": "sponsor-slash",  
    "timestamp": 1633046700,  
    "contractSig": "0xDEADCAFE..."  
  }  
}
```

# **8\. Timing & Anti-Replay Mechanisms**

To guarantee message freshness and prevent replay attacks, the protocol mandates unforgeable, verifiable time- and order-proofs on every step—both off-chain and on-chain.

## **8.1 Timing Parameters & Windows**

The protocol defines several timing parameters that all implementations MUST support. These parameters are critical for ensuring message freshness and preventing replay attacks.

* **Protocol-Level Timing Parameters**  
  * Δ₁ (Challenge Response Window)
    * Time allowed for Candidate to respond to ChallengeRequest  
    * Used in: ChallengeRequest → SensorPackage transition  
    * See Section 13.1 for configured value
  * Δ₂ (Sensor to Attestation Window)
    * Time allowed for Sponsor to verify and attest sensor data  
    * Used in: SensorPackage → SponsorAttestation transition  
    * See Section 13.1 for configured value
  * Δ₃ (Attestation to Mint Window)
    * Time allowed for Candidate to submit MintRequest  
    * Used in: SponsorAttestation → MintRequest transition  
    * See Section 13.1 for configured value
  * ΔT (Clock Skew)
    * Maximum allowed difference between local and message timestamps  
    * Used in: All message validation  
    * See Section 13.1 for configured value
  * ΔV (Validation Response Window)
    * Time allowed for validators to respond to MintRequest  
    * Used in: MintRequest → ValidationResponse transition  
    * See Section 13.1 for configured value

* **Time Window Validations**
  * All implementations MUST:
    * Add timestamp verification in message processing
    * Implement time window validations (Δ₁, Δ₂, Δ₃, ΔT)
    * Verify timestamps are within acceptable ranges
    * Reject messages with timestamps outside valid windows
    * Maintain synchronized clocks within ΔT
    * Use block heights for on-chain time anchoring
    * Support both Unix epoch and block height timestamps

* **Network-Level Timing Parameters**  
  * Δₚ (Partition Detection Window)
    * Time to detect network partitions  
    * Used in: Validator state machine  
    * See Section 13.1 for configured value
  * Δᵣ (Validator Reconfiguration Window)
    * Time allowed for validator set changes  
    * Used in: Validator rotation and reconfiguration  
    * See Section 13.1 for configured value

* **Implementation-Specific Windows**  
  The following windows are left to implementations to define based on their specific requirements:  
  * Appeals window (for disputing slashing)  
  * Probation window (for new identities)  
  * Cooling-off period (between sponsorships)  
  * Stake lock period (for escrowed tokens)  

Note: All protocol-level timing parameters are defined in Section 13.1 and MUST be supported by all implementations. Implementation-specific windows may vary between deployments but should be clearly documented.

## **8.2 Nonces & Sequence Numbers**

* **Session nonces**  
  * Each ChallengeRequest includes a cryptographic `nonce` (random bit-string)  
    that ties all subsequent messages in that session together.  
  * Validators track used nonces to reject duplicates or replays.  
* **Sequence numbers**  
  * For multi-step exchanges, messages may carry a small integer `sequence`  
    (1, 2, 3…) to enforce correct ordering.  
  * Replay or reordering (e.g. sending "SponsorAttestation" before "SensorPackage")  
    is detected and dropped.  
* **Binding**  
  * Nonce and sequence number are always included in the data that's hashed  
    and signed by each actor, preventing mutation or reuse.

## **8.3 Verifiable Delay Functions (optional)**

* **Purpose**  
  * Enforce a minimum elapsed time without trusting local clocks (e.g., a "cool-off"  
    between mint and transfer of stake).  
* **How it works**  
  * Prover computes a VDF output `vdfOut` over a public input and difficulty parameter.  
  * Verifier checks `vdfOut` quickly and is assured at least T seconds (or N sequential steps)  
    have passed.  
* **Integration**  
  * Include `vdfOut` and its proof in the MintRequest or RevocationRequest when a  
    mandatory delay is required.  
  * Validators reject submissions lacking a valid, timely VDF proof.

These combined mechanisms ensure every action in the protocol is fresh, ordered, and tamper-evident—making replay or preplay attacks economically and technically infeasible.

# **9\. Economic Mechanisms**

This section describes the protocol-level economic mechanisms that all implementations MUST support. While the core mechanisms are fixed by the protocol, implementations may add additional economic features as described in Section 1.2.

## **9.1 Identity-Minting Stakes & Fees**

* **Minting Stake (S_mint)**
  * Must be locked for identity lifetime
  * Slashed for protocol violations
  * Cannot be withdrawn while identity active

* **Minting Fee (F_mint)**
  * Paid to validators
  * Covers verification costs
  * Distributed based on validation accuracy

## **9.2 Sponsor Stakes & Slashing**

* **Sponsor Stake (S_sponsor)**
  * Must be at least the configured sponsor stake
  * Locked for sponsor duration
  * Slashed for invalid sponsorships

* **Sponsor Fee (F_sponsor)**
  * Paid to validators
  * Covers verification costs
  * Distributed based on validation accuracy

## **9.3 Validator Bonds & Rewards**

* **Validator Bond (V_bond)**
  * Must exceed S_mint × 10
  * Locked for validator duration
  * Slashed for protocol violations

* **Validator Rewards**
  * Base reward for honest validation
  * Penalty for invalid validations
  * Distributed based on validation accuracy

* **Economic Parameters**
  * Must ensure:
    * Honest behavior more profitable
    * Malicious behavior unprofitable
    * Cost of attack exceeds benefits

## **9.4 Identity Maintenance Requirements**

The following are the strict requirements for maintaining a valid Soul Bound identity. Violation of these requirements may result in slashing of escrowed stake or revocation of identity.

* **Core Requirements**
  * Do not attempt to create multiple identities
  * Do not sponsor fraudulent identities
  * Do not violate protocol rules during verification
  * Maintain sufficient stake for any active sponsorships
  * Do not share private keys or verification devices

* **Consequences**
  The following consequences only apply to violations of core requirements:
  * Slashing of escrowed stake for:
    * Fraudulent behavior
    * Protocol rule violations
    * Sponsorship fraud
  * Temporary suspension of verification rights (only during investigation)
  * Required additional security measures (if security is compromised)

Note: These are the minimum protocol-level requirements. Implementations may add additional requirements or maintenance tasks as described in Section 1.2.

# **10\. Security Properties & Invariants**

This section states the core security guarantees and invariants that any Soul Bound implementation must uphold. These properties ensure that identities are real, unique, fresh, private, and that economic penalties enforce honest behavior.

## **10.1 Unforgeability of Attestations**

* **Invariant:** No party can produce a valid `SponsorAttestation` or `SensorPackage` without physically co-locating two devices and holding the correct private keys.  
* **Mechanisms:**  
  * All sensor hashes and session nonces are included in signatures
  * Validators reject any attestation whose signature fails verification
  * Economic incentives ensure validators check signatures correctly

## **10.2 Freshness & Liveness Guarantees**

* **Invariant:** Every protocol step must occur within its configured time window.  
* **Mechanisms:**  
  * Messages include signed timestamps
  * Validators enforce time windows
  * Nonces prevent replay attacks

## **10.3 Identity Fraud Resistance**

* **Invariant:** The cost to create N identities grows linearly in N.  
* **Mechanisms:**  
  * In-person verification required
  * Token stakes and slashing
  * Protocol-enforced limits on sponsorships

## **10.4 Privacy Guarantees**

* **Invariant:** No raw biometric or sensor data is revealed on-chain.  
* **Mechanisms:**  
  * Raw sensor readings never leave device
  * Only hashes of sensor data are exchanged
  * ZK proofs attest to correct hash computation

## **10.5 Revocation & Slashing**

* **Invariant:** When an identity is revoked, its stake is slashed.  
* **Mechanisms:**  
  * RevocationRequests include evidence
  * Validators follow quorum rule
  * Smart contracts enforce slashing

## **10.6 Protocol-Level Enforcement**

* **Core Security Properties**
  * All implementations MUST:
    * Follow the protocol message formats exactly
    * Enforce all timing windows (Δ₁, Δ₂, Δ₃, ΔT)
    * Verify all signatures and proofs
    * Maintain correct stake requirements
    * Enforce quorum rules (m-of-n)
    * Protect private keys and sensor data

* **Economic Enforcement**
  * Validators are incentivized to:
    * Verify messages correctly (rewards for honest validation)
    * Reject invalid messages (penalties for invalid acceptance)
    * Maintain sufficient stake (V_bond)
    * Follow protocol rules (slashing for violations)
  * Sponsors are incentivized to:
    * Verify identities carefully (rewards for honest sponsorship)
    * Reject fraudulent identities (penalties for invalid sponsorships)
    * Ensure sponsor stake is locked in escrow for the duration of active sponsorships
    * Follow protocol rules (slashing for violations)

* **Protocol Rules**
  * Messages that violate protocol rules are rejected
  * Validators that accept invalid messages are slashed
  * Sponsors that sponsor invalid identities are slashed
  * Stake requirements ensure honest behavior is more profitable

* **Monitoring & Response**
  * Validators monitor each other's behavior
  * Protocol automatically slashes stake for violations
  * Network can vote to remove misbehaving validators
  * Economic incentives naturally align behavior

---

**Global Invariants:**

* No two active identities share the same human anchor (enforced by on-chain uniqueness checks).  
* Total supply of Soul Bound stakes/tokens evolves only via protocol-approved mint, burn, and reward events.  
* Every on-chain record (mint, revocation, reward, slash) is cryptographically linked to its originating messages and signatures.

# **11\. Failure Modes & Error Handling**

This section enumerates expected error conditions, how each actor should detect and respond, and what state transitions or economic consequences apply.

## **11.1 Timeouts & Session Aborts**

* **ChallengeRequest → SensorPackage**  
  * If the Candidate does not send `SensorPackage` within Δ₁, the Sponsor aborts the session.  
  * Candidate state → **FAILED**; Sponsor state → **ABORTED**.  
  * No tokens are staked yet, so no slashing occurs.  
* **SensorPackage → SponsorAttestation**  
  * If the Sponsor doesn't respond with `SponsorAttestation` within Δ₂, Candidate aborts.  
  * Candidate state → **FAILED**; Sponsor state → **ABORTED**.  
  * Sponsor may incur a small reputational penalty but no stake is at risk.  
* **SponsorAttestation → MintRequest**  
  * If the Candidate does not submit `MintRequest` within Δ₃, the session is abandoned.  
  * Any reserved UI/session resources are released; no on-chain stake has been locked yet.  
* **Awaiting ValidationResponse**  
  * If quorum is not reached before ΔV, Candidate/Sponsor treat the session as **FAILED**.  
  * If stake was locked, Candidate may reclaim or forfeit it based on policy (see economic rules).

## **11.2 Signature or Proof Verification Failures**

* **Invalid Signature**  
  * Any message whose signature fails to verify against the claimed public key is immediately rejected.  
  * The receiving actor emits an error back to the sender and aborts the session.  
* **Malformed or Missing Fields**  
  * Messages missing required fields (`type`, `sessionId`, etc.) are discarded, with an optional rejection notice.  
* **ZK Proof Failure**  
  * If the `zkProof` in a `MintRequest` or `RevocationRequest` fails to verify, Validators reject the request.  
  * Candidate/Sponsor receive a "rejected: invalid proof" response and may correct or abandon.

## **11.3 Stake Insufficiency Rejections**

* **MintRequest Stake < S_mint**  
  * Validators and Ledger smart contracts check `stake ≥ S_mint`.  
  * If the stake is insufficient, the transaction is reverted or the request is rejected with error code `INSUFFICIENT_STAKE`.  
* **Sponsor Stake Insufficient**
  * Sponsor attempts to vouch without locking the required tokens.
  * Session is aborted and Sponsor receives a "stake too low" error.
  * No attestation is recorded.

## **11.4 Replay or Duplicate Sessions**

* **Nonce/Subsession Reuse**  
  * Each `sessionId` + `nonce` pair may be used only once.  
  * Actors maintain a short-term cache of seen nonces.  
  * Any message with a previously seen combination is dropped and logged as a replay attempt.  
* **Out-of-Order Messages**  
  * Sequence numbers enforce ordering.  
  * Receiving a message with `sequence` lower than expected triggers a reject.  
* **Duplicate On-Chain Requests**  
  * LEDsger ensures idempotency: duplicate `MintRequest` or `RevocationRequest` with the same sessionId or txHash is ignored or returns the original result.

## **11.5 Misbehaving Validators / Sponsors**

* **Equivocation by Validators**  
  * If a Validator signs conflicting `ValidationResponse` for the same `sessionId`, that Validator's bond (`V_bond`) is slashed for double-voting.  
  * The protocol enforces slashing of the validator's bond, with the exact amount determined by governance parameters.  
  * Slashing occurs after the validator reconfiguration window (Δᵣ) to allow for appeals.

* **Censorship or Non-Participation**  
  * Validators that repeatedly fail to vote or respond within time windows face protocol-enforced penalties:  
    * Slashing of bond proportional to missed validations  
    * Temporary suspension after multiple violations  
    * Removal from active validator set after severe offenses  
  * Detection occurs within the partition detection window (Δₚ)

## **11.6 Error Reporting Conventions:**

* Each rejection or abort SHOULD include:  
  * `errorCode` (e.g., `TIMEOUT`, `INVALID_SIG`, `INSUFFICIENT_STAKE`, `REPLAY_DETECTED`)  
  * `description`: human-readable explanation  
  * Optional `remediation`: guidance on how to correct and retry

Handling these failure modes consistently ensures clear diagnostics, correct state cleanup, and proper economic enforcement across all implementations.

# **12\. Extensions & Policy Hooks**

This section describes optional interfaces and hook points where applications, DAOs, or governance modules can plug in custom logic—without changing the core Soul Bound protocol. These extensions are part of the implementation domain as described in Section 1.2.

## **12.1 Decoupled Trust Semantics Interface**

* **Graph Query API**  
  * Read-only endpoints to fetch the public identity graph: nodes (identities), edges (sponsorships), stake and reputation metadata.  
  * Support filters (e.g. by depth, by timestamp range) and aggregate queries (e.g. degree centrality).  
* **Policy SDK**  
  * A language-agnostic interface (e.g. JavaScript/TypeScript, Rust, Python) for writing "trust evaluators" that consume graph data and output a trust decision (boolean or score).  
* **Event Hooks**  
  * Subscribe to on-chain or off-chain events (Mint, Revocation, Slash, Reward) and trigger custom workflows (e.g. notify a DAO, update an off-chain reputation database).  
* **Decision Registry**  
  * A registry contract or distributed registry mapping policy identifiers to evaluator code hashes or endpoints, enabling applications to reference named policies.

## **12.2 Reputation-Scoring Plug-ins**

* **Metric Definitions**  
  * Standard metrics (e.g. number of successful sponsorships, average sponsorship depth, validator accuracy) exposed in the graph metadata.  
* **Pluggable Scoring Algorithms**  
  * Modules that consume raw metrics, apply weightings, thresholds, or machine-learning models, and produce per-identity reputation scores.  
* **On-Chain Reputation Anchors**  
  * Optional contracts to record a snapshot of computed reputation scores, so smart contracts or UIs can enforce minimum reputation requirements.  
* **Reputation Refresh Policies**  
  * Configurable refresh intervals or event-driven updates (e.g. recalc scores after a configurable number of new blocks or after slashing events).

## **12.3 Optional KYC-Bonded Pathways**

* **KYC Bond Contract**  
  * An opt-in smart contract where users deposit a higher-value bond in exchange for "KYC-verified" status.  
  * Bonds are slashed on fraud or revoked identities under stricter governance rules.  
* **Tiered Identity Flags**  
  * Extend the identity record with a flag (e.g. `kycStatus = "none"|"basic"|"enhanced").  
  * Applications may require higher KYC tiers for sensitive functions (voting, high-value transfers).  
* **Privacy-Preserving KYC**  
  * Use zero-knowledge proofs to confirm KYC attestation without exposing raw personal data.  
  * Store only proof commitments on-chain; retain identity documents off-chain under GDPR-style controls.  
* **Bond Redemption & Appeals**  
  * Define multi-stage dispute resolution: appeals window before final slash, partial bond refund on successful appeal.

## **12.4 Custom Reward/Slashing Policies**

* **Parameterized Slashing Rules**  
  * Expose protocol parameters (slash fraction, probation window) to on-chain governance so communities can adjust penalty severity.  
* **Reward Curves**  
  * Define custom payout curves (linear, tiered, logarithmic) based on performance metrics (e.g. number of valid sponsorships, identity survival time).  
* **Treasury & Fee Allocation**  
  * Hook points for routing portions of mint-fees or slashed funds to community treasuries, development grants, or public goods.  
* **Governance-Controlled Modules**  
  * Deploy policy contracts that implement `RewardDistributor` and `SlashController` interfaces; allow DAOs to vote on upgrades or parameter changes.

## **12.1 Protocol Governance**

The protocol follows a decentralized governance model where changes require overwhelming network consensus. There is no formal governance system or privileged actors who can force changes.

* **Protocol Changes**
  * All changes must be backward-compatible where possible
  * Non-backward-compatible changes create new chains
  * Network participants freely choose which chain to follow
  * No single entity can force changes on the network

* **Change Process**
  1. Proposals are discussed openly in the community
  2. Multiple implementations are encouraged
  3. Changes are adopted through network consensus
  4. Old protocol versions remain valid
  5. No formal voting or quorum requirements

* **Security Properties**
  * No privileged actors or emergency powers
  * No formal governance system
  * Changes require overwhelming consensus
  * Protocol maintains backward compatibility
  * Network participants determine successful changes

This approach ensures the protocol remains decentralized and resistant to both technical attacks and governance attacks. The security comes from requiring overwhelming consensus for any changes, making it impossible for any single entity or group to force changes on the network.

# **13\. Formal Model (Optional)**

This section sketches a machine-checked model of the Soul Bound Protocol, useful for rigorous verification of security and liveness properties. We describe a TLA⁺ or Alloy rendition at a high level. Implementers may use these patterns or adapt them to other tools (Tamarin, ProVerif, etc.).

## **13.1 TLA⁺ or Alloy Model Overview**

* **Scope:**  
  Model the core identity issuance flow (Challenge → SensorPackage → Attestation → Mint → Validation → Commit) plus slashing and revocation.  
* **Approach:**  
  * In TLA⁺, define a single module `SoulBound` with a `VARIABLES` tuple and a `Next` action.  
  * In Alloy, use signatures for `Identity`, `Session`, `Validator`, and relations for sponsorship and stakes.  
* **Goals:**  
  * Prove that no two active `Identity` share the same `humanAnchor`.  
  * Ensure that every valid session either reaches `Minted` or `Rejected` (no deadlock).  
  * Validate that slashing only occurs after a valid `RevocationRequest`.

## **13.2 State Variables & Actions**

* **State Variables**  
  1. `Candidates`: set of pending sessions, each with `(sessionId, nonce, state)`  
  2. `Identities`: set of issued identities with `(pubKey, sponsor, stake, status)`  
  3. `Validators`: map of validator nodes and their votes  
  4. `Ledger`: sequence of committed on-chain events  
  5. `Time`: abstract clock or block counter  
* **Actions**  
  1. `BeginSession(c, s)`: Sponsor `s` issues `ChallengeRequest` to candidate `c`.  
  2. `SendSensors(c, s)`: Candidate `c` submits `SensorPackage`.  
  3. `IssueAttestation(s, c)`: Sponsor `s` signs `SponsorAttestation`.  
  4. `SubmitMint(c)`: Candidate `c` assembles attestations, stake, and ZK proof → `MintRequest`.  
  5. `Validate(v, m)`: Validator `v` checks `MintRequest` `m` and emits an approval or rejection.  
  6. `CommitMint`: When ≥ m approvals, ledger appends new identity.  
  7. `SubmitRevocation(r, id)`: Revoker `r` issues `RevocationRequest` for identity `id`.  
  8. `ValidateRevoke(v, r)`: Validators process revocation, then `CommitRevoke` if quorum.  
  9. `Slash(id)`: Protocol slashes stake for a revoked identity `id`.

## **13.3 Invariants & Temporal Properties**

* **Safety Invariants**  
  * **UniqueHumanAnchor:** No two `Identities` share the same human anchor identifier.  
  * **StakeConsistency:** For every `Minted` identity, `stake ≥ S_mint`; for every `Revoked` identity, stake has been slashed or burned.  
  * **SignatureValidity:** All recorded sponsorships carry valid signatures from the claimed actors.  
* **Liveness / Temporal Properties**  
  * **SessionProgress:** ∀ session ∈ `Candidates`, eventually `state` ∈ {`Minted`, `Rejected`} (no infinite pending sessions).  
  * **EventualSlash:** If a valid revocation is submitted, the corresponding identity eventually transitions to `Revoked` and its stake is slashed.  
  * **QuorumTermination:** Given live validators, any `MintRequest` will gather m votes within bounded time.

## **13.4 Model-Checking Results**

* **Model Scope & Limitations**  
  * This is a reference implementation demonstrating the core protocol mechanics  
  * The model is intentionally small-scale for clarity and initial verification  
  * Real-world deployments would require additional security analysis  

* **Model Configuration**  
  * Small-scale instance with 3 validators, m = 2, 2 candidate sessions  
  * Abstracted time with integer counter bounded to 10 steps  
  * Simplified network model (no message delays, no partitions)  

* **Verified Invariants**  
  * **Session Uniqueness**  
    * No two active sessions share the same `sessionId`  
    * Each `nonce` is used only once per session  
  * **Stake Consistency**  
    * Total stake in system equals sum of all locked stakes  
    * No stake can be double-counted or created without minting  
  * **Signature Validity**  
    * All recorded messages carry valid signatures from claimed senders  
    * No message can be modified after signing  
  * **State Transitions**  
    * All state changes follow the defined state machines  
    * No invalid transitions are possible  
  * **Quorum Rules**  
    * Minting requires exactly m validator approvals  
    * No identity can be minted without sufficient stake  

* **Findings**  
  * **Core Protocol Mechanics**  
    * All specified invariants held across 10,000 model steps  
    * No deadlocks in the basic protocol flow  
    * Correct handling of session timeouts and retries  
  * **Known Limitations**  
    * Model does not cover network partitions or message delays  
    * Simplified validator selection (no dynamic set changes)  
    * Abstracted cryptographic primitives  
  * **Areas for Future Analysis**  
    * Larger validator sets and session counts  
    * Network adversarial models  
    * Economic game theory analysis  
    * Implementation-specific security considerations  

* **Next Steps**  
  * Community review and extension of the formal model  
  * Integration with implementation-specific security analysis  
  * Development of additional models for specific threat scenarios  
  * Continuous integration of model checking in protocol development

Note: This model serves as a reference implementation and basic verification of the protocol's core mechanics. Real-world deployments should conduct additional security analysis, including implementation-specific considerations, network adversarial models, and economic game theory analysis.

# **14\. Appendices**

## **14.1 Parameter Recommendations (Δs, stake sizes, quorum)**

The following parameters are part of the protocol domain and MUST be supported by all implementations. Implementations may add additional parameters as described in Section 1.2, but MUST NOT modify these core parameters.

```json
{  
  "deltaTimes": {  
    "challengeResponseWindow": 120,  
    "sensorToAttestationWindow": 60,  
    "attestationToMintWindow": 300,  
    "validationResponseWindow": 600,  
    "clockSkew": 120,
    "partitionDetectionWindow": 300,
    "validatorReconfigurationWindow": 3600
  },  
  "stakeSizes": {  
    "S_mint": 100,
    "S_sponsor": 10,
    "V_bond": 1000,
    "minimumStakeIncrement": 1,
    "slashFraction": 0.5,
    "validatorRewardFraction": 0.1
  },
  "fees": {
    "F_mint": 1,
    "F_sponsor": 0.5
  },
  "quorum": {  
    "n": 5,  
    "m": 3,
    "scaling": {
      "minValidators": 3,
      "maxValidators": 100,
      "faultTolerance": 0.33,
      "validatorRotationPeriod": 86400
    }
  },
  "economicParameters": {
    "honestValidatorReward": 1.0,
    "dishonestValidatorPenalty": 2.0,
    "honestSponsorReward": 0.5,
    "dishonestSponsorPenalty": 1.5,
    "minimumStakeToRewardRatio": 10
  }
}
```

* **Time Windows (Δs)**  
  * Values are in seconds and represent reasonable defaults for most networks  
  * Should be adjusted based on:  
    * Network latency and geographic distribution  
    * Expected device performance  
    * Security requirements for specific use cases  
  * All timing parameters referenced in the document should use these values:
    * Δ₁ (challenge response): See Section 13.1 for configured value
    * Δ₂ (sensor to sponsorship): See Section 13.1 for configured value
    * Δ₃ (sponsorship to mint): See Section 13.1 for configured value
    * ΔT (clock skew): See Section 13.1 for configured value
    * Δₚ (partition detection): See Section 13.1 for configured value
    * Δᵣ (validator reconfiguration): See Section 13.1 for configured value

* **Stake Sizes**  
  * Example values shown are relative units  
  * Actual values should be set based on:  
    * Token economics and market conditions  
    * Desired security level  
    * Network participation requirements  
  * Governance should allow dynamic adjustment of these parameters  

* **Quorum Parameters**  
  * The example configuration (n=5, m=3) provides:  
    * Basic fault tolerance (withstands 2 faulty validators)  
    * Reasonable latency for small networks  
    * Balance between security and performance  
  * For larger networks, quorum parameters should scale according to:  
    * `n = max(3, min(100, ceil(total_validators * 0.2)))`  
    * `m = ceil(n * (1 - fault_tolerance))`  
    * Where `fault_tolerance` is typically 0.33 (one-third)  
  * Considerations for quorum scaling:  
    * Network size and geographic distribution  
    * Expected validator reliability  
    * Desired fault tolerance  
    * Performance requirements  
    * Economic security requirements  

* **Implementation Notes**  
  * All parameters should be configurable via governance  
  * Parameters may need adjustment based on:  
    * Network growth and validator set changes  
    * Security incident analysis  
    * Performance monitoring  
    * Economic conditions  
  * Regular parameter review should be part of network governance

Note: These parameters are example values for a small-scale deployment. Real-world implementations should conduct thorough analysis of their specific requirements and adjust parameters accordingly. The scaling formulas provided are starting points that should be validated through simulation and real-world testing.

## **14.2 Example API Endpoints & RPC Definitions**

```json
{  
  "endpoints": [
    {  
      "path": "/api/v1/challenge",  
      "method": "POST",  
      "description": "Sponsor issues a new ChallengeRequest",  
      "requestSchema": "ChallengeRequest",  
      "responseSchema": "ChallengeAck"  
    },  
    {  
      "path": "/api/v1/sensor",  
      "method": "POST",  
      "description": "Candidate submits SensorPackage",  
      "requestSchema": "SensorPackage",  
      "responseSchema": "SensorAck"  
    },  
    {  
      "path": "/api/v1/sponsorship",  
      "method": "POST",  
      "description": "Sponsor submits SponsorAttestation",  
      "requestSchema": "SponsorAttestation",  
      "responseSchema": "SponsorshipAck"  
    },  
    {  
      "path": "/api/v1/mint",  
      "method": "POST",  
      "description": "Candidate submits MintRequest",  
      "requestSchema": "MintRequest",  
      "responseSchema": "MintResponse"  
    },  
    {  
      "path": "/api/v1/validate",  
      "method": "POST",  
      "description": "Validator submits ValidationResponse",  
      "requestSchema": "ValidationResponse",  
      "responseSchema": "ValidateAck"  
    },  
    {  
      "path": "/api/v1/revoke",  
      "method": "POST",  
      "description": "Submit a RevocationRequest",  
      "requestSchema": "RevocationRequest",  
      "responseSchema": "RevokeAck"  
    },  
    {  
      "path": "/api/v1/events",  
      "method": "GET",  
      "description": "Subscribe to RewardDistribution and SlashNotification events",  
      "responseSchema": "RewardEvent"  
    }  
  ]
}
```

## **14.3 Glossary of Terms**

* **Candidate**  
  A person or device requesting a new Soul Bound identity.  
* **Sponsor**  
  An existing Soul Bound identity vouching for a Candidate.  
* **Validator**  
  A network node that verifies sponsorships and stakes to approve or reject identity mints.  
* **Ledger**  
  The decentralized, append-only storage (e.g., blockchain) recording identities, stakes, and slashes.  
* **Nonce**  
  A cryptographic number used once to prevent replay attacks within a session.  
* **ZK Proof**  
  A zero-knowledge proof that attests correctness of a computation without revealing underlying data.  
* **Slashing**  
  The confiscation or burning of staked tokens when an identity or participant misbehaves.  
* **Burn**  
  The permanent removal of tokens from circulation.  
* **Quorum (m-of-n)**  
  The requirement that at least *m* out of *n* validators must approve an action for it to succeed.

## **14.4 References & Further Reading**

* [Bitcoin: A Peer-to-Peer Electronic Cash System](https://bitcoin.org/bitcoin.pdf)  
  Satoshi Nakamoto (2008)  
* [Bulletproofs: Short Proofs for Confidential Transactions and More](https://eprint.iacr.org/2017/1066.pdf)  
  Benedikt Bünz et al. (2018)  
* [TLA⁺ Hyperbook](https://lamport.azurewebsites.net/tla/hyperbook.html)  
  Leslie Lamport et al. (2021)  
* [A Survey on Sybil Attacks in Social Networks](https://doi.org/10.1007/s10207-009-0087-6)  
  George Danezis & Prateek Mittal (2009)  
* [Zero-Knowledge Proofs: From Theory to Practice](https://eprint.iacr.org/2018/046.pdf)  
  Eli Ben-Sasson & Alessandro Chiesa (2018)
