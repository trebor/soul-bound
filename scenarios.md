# Soul Bound Protocol Scenarios

This document outlines various scenarios and how they would play out according to the Soul Bound Protocol specification. Each scenario demonstrates different aspects of the protocol's security, economic, and operational mechanisms.

## 1. Basic Identity Creation

### Scenario: Alice creates her first Soul Bound identity

**Participants:**
- Alice (Candidate)
- Bob (Sponsor)
- Validator Set (5 validators, m=3 quorum)

**Timeline:**
1. **Initial Meeting (t=0)**
   - Alice and Bob meet in person
   - Bob verifies Alice's uniqueness through direct interaction
   - Bob explains protocol requirements and risks
   - Both parties confirm understanding

2. **Challenge Request (t=0)**
   - Bob generates fresh sessionId and nonce
   - Bob locks S_sponsor (10 tokens) in escrow
   - Bob sends signed ChallengeRequest to Alice
   - Alice receives and acknowledges ChallengeRequest

3. **Sponsor Attestation (t=60s)**
   - Bob completes in-person verification
   - Bob signs SponsorAttestation
   - Alice receives and verifies attestation

4. **Mint Request (t=300s)**
   - Alice generates new identity key pair
   - Alice locks S_mint (100 tokens) in escrow
   - Alice builds MintRequest with:
     - Identity public key
     - Signed attestation from Bob
     - Locked S_mint stake
     - Locked S_sponsor stake
     - ZK proof of protocol compliance
   - Alice submits MintRequest to validators

5. **Validation (t=600s)**
   - Validators verify:
     - All signatures and proofs
     - Both stakes properly locked
     - Timestamps within windows
   - 4 out of 5 validators approve
   - Quorum (m=3) reached

6. **Identity Creation (t=601s)**
   - Identity is minted on-chain
   - S_mint remains locked for identity lifetime
   - S_sponsor remains locked for Δₚ
   - Validator rewards distributed

**Outcome:**
- Alice has a valid Soul Bound identity
- Bob's sponsor stake is locked for Δₚ
- Validators receive rewards
- All protocol requirements satisfied

## 2. Failed Identity Creation

### Scenario: Charlie attempts to create an identity with insufficient stake

**Participants:**
- Charlie (Candidate)
- Dave (Sponsor)
- Validator Set

**Timeline:**
1. **Initial Meeting (t=0)**
   - Charlie and Dave meet in person
   - Dave verifies Charlie's uniqueness
   - Dave explains protocol requirements
   - Both parties confirm understanding

2. **Challenge Request (t=0)**
   - Dave generates sessionId and nonce
   - Dave locks S_sponsor (10 tokens)
   - Dave sends ChallengeRequest
   - Charlie receives and acknowledges

3. **Sponsor Attestation (t=60s)**
   - Dave completes verification
   - Dave signs SponsorAttestation
   - Charlie receives attestation

4. **Mint Request (t=300s)**
   - Charlie generates key pair
   - Charlie attempts to lock only 50 tokens (insufficient)
   - Charlie builds MintRequest
   - Charlie submits to validators

5. **Validation (t=600s)**
   - Validators verify:
     - Signatures valid
     - Timestamps valid
     - Stake insufficient (50 < S_mint=100)
   - All validators reject
   - No quorum reached

6. **Session Abort (t=601s)**
   - Session marked as failed
   - Dave's sponsor stake unlocked
   - Charlie's partial stake returned
   - No rewards distributed

**Outcome:**
- Identity creation failed
- All stakes returned
- No economic penalties
- Protocol requirements enforced

## 3. Identity Revocation

### Scenario: Eve's identity is revoked due to fraud

**Participants:**
- Eve (Revoked Identity)
- Frank (Sponsor)
- Grace (Validator who detected fraud)
- Validator Set

**Timeline:**
1. **Initial Creation (t=0)**
   - Eve creates identity with Frank's sponsorship
   - Identity successfully minted
   - Both stakes locked

2. **Fraud Detection (t=86400s)**
   - Grace's validator node receives MintRequest from Eve
   - Grace's node checks on-chain records and finds:
     - Existing identity record with Eve's public key
     - Active sponsorship from Frank
     - Locked S_mint stake
   - Grace's node verifies:
     - Current MintRequest's public key matches existing identity
     - Timestamps show concurrent identity creation attempt
     - ZK proof reveals attempt to bypass uniqueness check
   - Grace gathers cryptographic evidence:
     - Hash of conflicting MintRequests
     - Signed timestamps showing concurrent attempts
     - ZK proof verification results
   - Grace submits RevocationRequest with:
     - Evidence hash
     - Reason: "duplicate-identity-attempt"
     - Timestamp and signature

3. **Revocation Validation (t=86460s)**
   - Validators verify:
     - Evidence validity
     - Grace's signature
     - Timestamps
   - 4 out of 5 validators approve revocation
   - Quorum reached

4. **Slashing (t=86461s)**
   - Identity marked as revoked
   - Eve's S_mint (100 tokens) slashed at 50%
   - Frank's S_sponsor (10 tokens) slashed at 50%
   - Slashed tokens distributed to validators
   - Grace receives additional reward for detection

**Outcome:**
- Eve's identity revoked
- Both stakes slashed
- Validators receive rewards
- Protocol security maintained

## 4. Validator Misbehavior

### Scenario: Validator attempts to approve invalid identity

**Participants:**
- Helen (Candidate)
- Ian (Sponsor)
- Validator Set (including malicious Validator V)
- Validator V (malicious)

**Timeline:**
1. **Initial Setup (t=0)**
   - Helen and Ian meet
   - Ian sends ChallengeRequest
   - Helen receives and acknowledges

2. **Mint Request (t=300s)**
   - Helen submits MintRequest
   - Request contains invalid ZK proof
   - Validators begin verification

3. **Validation (t=600s)**
   - Validator V attempts to approve despite invalid proof
   - Other validators perform standard verification:
     - Verify ZK proof against public parameters
     - Check proof validity using protocol-specified verification algorithm
     - Confirm proof fails verification with error code "INVALID_PROOF"
   - Validators compare results:
     - 4 validators report proof verification failure
     - Validator V reports success despite protocol rules
   - Validators detect V's violation through:
     - Cryptographic proof of invalid approval
     - Signed validation results showing contradiction
     - Protocol rule violation evidence
   - Only 1 approval (V) vs 4 rejections
   - No quorum reached

4. **Validator Slashing (t=601s)**
   - Validator V's bond (V_bond=1000) slashed at 50%
   - V temporarily suspended
   - Network votes to remove V from validator set
   - New validator selected to replace V

**Outcome:**
- Identity creation failed
- Malicious validator slashed
- Validator set maintained
- Protocol security preserved

## 5. Network Partition

### Scenario: Network partition during identity creation

**Participants:**
- Jane (Candidate)
- Kevin (Sponsor)
- Validator Set (split by partition)

**Timeline:**
1. **Initial Setup (t=0)**
   - Jane and Kevin meet
   - Kevin sends ChallengeRequest
   - Jane receives and acknowledges

2. **Network Partition (t=300s)**
   - Network splits into two partitions
   - 3 validators in partition A
   - 2 validators in partition B
   - Jane's MintRequest only reaches partition A

3. **Validation Attempt (t=600s)**
   - Partition A validators approve
   - Partition B validators unaware of request
   - Quorum not reached (3/5 < m=3)
   - Request times out after ΔV

4. **Partition Resolution (t=3600s)**
   - Network heals
   - Validators detect partition
   - Jane must restart process
   - No economic penalties

**Outcome:**
- Identity creation failed
- No stakes slashed
- Process must restart
- Protocol maintains consistency

## 6. Time Window Violations

### Scenario: Candidate delays submission beyond time windows

**Participants:**
- Laura (Candidate)
- Mike (Sponsor)
- Validator Set

**Timeline:**
1. **Initial Setup (t=0)**
   - Laura and Mike meet
   - Mike sends ChallengeRequest
   - Laura receives and acknowledges

2. **Delayed Attestation (t=180s)**
   - Laura delays beyond Δ₁ (120s)
   - Mike's SponsorAttestation arrives late
   - Validators detect timestamp violation
   - Session automatically aborted

3. **Stake Return (t=181s)**
   - Mike's sponsor stake unlocked
   - No slashing occurs
   - Session marked as failed
   - No rewards distributed

**Outcome:**
- Identity creation failed
- Stakes returned
- No economic penalties
- Protocol timing enforced

## 7. Multiple Sponsorships

### Scenario: Established identity sponsors multiple candidates

**Participants:**
- Nancy (Established Identity)
- Oliver (Candidate 1)
- Patricia (Candidate 2)
- Validator Set

**Timeline:**
1. **First Sponsorship (t=0)**
   - Nancy meets Oliver
   - Nancy locks S_sponsor (10 tokens)
   - Process completes successfully
   - Oliver's identity created

2. **Second Sponsorship (t=86400s)**
   - Nancy meets Patricia
   - Nancy locks additional S_sponsor (10 tokens)
   - Process completes successfully
   - Patricia's identity created

3. **Sponsorship Limits (t=172800s)**
   - Nancy attempts third sponsorship
   - Protocol smart contract checks:
     - Current active sponsorships count (2)
     - Maximum allowed concurrent sponsorships (2)
     - Locked S_sponsor stakes (20 tokens)
     - Sponsorship time windows
   - Smart contract rejects new sponsorship because:
     - Active sponsorships = maximum allowed
     - No available sponsorship slots
     - Would exceed concurrent sponsorship limit
   - Nancy must wait for existing sponsorships to complete

**Outcome:**
- Two successful sponsorships
- Third attempt prevented
- Protocol limits enforced
- No economic penalties

## 8. Validator Rotation

### Scenario: Validator set rotation during active session

**Participants:**
- Quinn (Candidate)
- Rachel (Sponsor)
- Validator Set (rotating)

**Timeline:**
1. **Initial Setup (t=0)**
   - Quinn and Rachel meet
   - Rachel sends ChallengeRequest
   - Quinn receives and acknowledges

2. **Validator Rotation (t=43200s)**
   - Validator rotation period reached
   - 2 validators rotated out
   - 2 new validators rotated in
   - Quorum requirements maintained

3. **Mint Request (t=43500s)**
   - Quinn submits MintRequest
   - New validator set processes request
   - All validators verify properly
   - Quorum reached with new set

4. **Identity Creation (t=44100s)**
   - Identity successfully minted
   - Stakes properly locked
   - Rewards distributed to current validators
   - Protocol maintains consistency

**Outcome:**
- Identity created successfully
- Validator rotation completed
- Protocol requirements maintained
- No disruption to process 