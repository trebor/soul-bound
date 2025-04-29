# Soul Bound Protocol - Stake System Design

## Introduction

The Soul Bound Protocol implements a stake-based security model to ensure the integrity and reliability of identity verification and validation processes. This document outlines the economic and security mechanisms that govern participation in the network.

### Key Principles

1. **Economic Security**
   - Stake requirements create financial incentives for honest behavior
   - Slashing mechanisms deter malicious actions
   - Rewards align participant interests with network health

2. **Scalable Trust**
   - Validator set size scales with network growth
   - Quorum requirements maintain security at all scales
   - Stake amounts balance security with accessibility

3. **Progressive Participation**
   - Three-tiered stake system (Identity, Sponsor, Validator)
   - Clear progression path for network participants
   - Graduated responsibilities and rewards

4. **Dynamic Governance**
   - Stake-based voting for protocol changes
   - Transparent slashing conditions
   - Adjustable parameters based on network conditions

### Document Structure

This document is organized into the following sections:
- Stake Types and Requirements: Details of each stake type and their conditions
- Stake Operations: Rules for locking and withdrawing stakes
- Slashing Conditions: Consequences for protocol violations
- Validator Scaling: Security model for network growth
- Economic Analysis: Incentive structures and security parameters
- Actor Incentives: Detailed breakdown of participant roles and rewards

## Stake Types and Requirements

| Stake Type | Actor | Purpose | Amount | Lock Period | Slashing Conditions | Notes |
|------------|-------|---------|--------|-------------|---------------------|-------|
| Identity Minting Stake (S_mint) | Candidate | Required to create a new identity | 100 units | Identity lifetime | - Fraudulent identity creation<br>- Protocol violations<br>- Multiple identity attempts | Cannot be withdrawn while identity is active |
| Sponsor Stake (S_sponsor) | Sponsor | Required to vouch for a new identity | 10 units | Duration of sponsorship | - Sponsoring fraudulent identities<br>- Protocol violations during verification<br>- False attestations | Must be maintained for active sponsorships |
| Validator Bond (V_bond) | Validator | Required to participate in validation | 1000 units | While active as validator | - Invalid validations<br>- Protocol violations<br>- Censorship<br>- Non-participation | Must exceed S_mint × networkSecurityFactor (10) |

## Stake Operations

| Operation | Actor | Stake Type | Conditions | Notes |
|-----------|-------|------------|------------|-------|
| Locking | Candidate | S_mint | - Must meet minimum amount (100 units)<br>- Must be available | Required before minting |
| Locking | Sponsor | S_sponsor | - Must meet minimum amount (10 units)<br>- Must be available | Required before sponsorship |
| Locking | Validator | V_bond | - Must meet minimum amount (1000 units)<br>- Must be available | Required to join validator set |
| Withdrawal | Identity Holder | S_mint | - Identity must be revoked<br>- No active sponsorships | Cannot withdraw while active |
| Withdrawal | Sponsor | S_sponsor | - Sponsorship period ended<br>- No active violations | Must wait for cooling period |
| Withdrawal | Validator | V_bond | - Not in active validator set<br>- No pending violations | Must wait for rotation period |

## Slashing Conditions

| Violation | Affected Stake | Slashing Amount | Cascading Effect |
|-----------|---------------|-----------------|------------------|
| Fraudulent Identity Creation | S_mint | 50% (slashFraction) | None |
| False Sponsorship | S_sponsor | 50% (slashFraction) | Direct sponsor loses 50% (sponsorSlashFraction) |
| Invalid Validation | V_bond | 50% (slashFraction) | None |
| Protocol Violation | All stakes | 50% (slashFraction) | Direct sponsor loses 50% (sponsorSlashFraction) |
| Multiple Identity Attempt | S_mint | 50% (slashFraction) | None |
| Censorship | V_bond | 50% (slashFraction) | None |
| Non-participation | V_bond | Proportional to missed validations | None |

## Validator Scaling and Security

The validator set size follows a square root scaling function:
- Formula: n = min(100, max(2, ceil(√x)))
- Where x is the number of network participants
- This ensures:
  - x=1 → 2 validators (bootstrap)
  - x=4 → 2 validators
  - x=9 → 3 validators
  - x=16 → 4 validators
  - x=25 → 5 validators
  - x=36 → 6 validators
  - x=49 → 7 validators
  - x=100 → 10 validators
  - x=10000 → 100 validators
  - x=1000000 → 100 validators (capped)

Quorum requirements:
- Minimum validators: 7
- Maximum validators: 100
- Fault tolerance: 20%
- Quorum size: m = ceil(n * (1 - fault_tolerance))

## Stake Management Rules

1. **Locking Requirements**
   - All stakes must be locked before participation
   - Minimum amounts defined by protocol parameters
   - Lock periods enforced by smart contract

2. **Withdrawal Rules**
   - Stakes cannot be withdrawn while active
   - Must wait for rotation period
   - No pending violations

3. **Slashing Rules**
   - Full slashing for protocol violations
   - Partial slashing for sponsors based on violation type
   - Cascading effects defined by protocol

4. **Governance**
   - Stake amounts configurable via governance
   - Minimum stake ratios must be maintained
   - Changes require quorum approval

## Economic Analysis

1. **Economic Security**
   - Stake amounts must be high enough to deter malicious behavior
   - Slashing must exceed potential gains from violations
   - Rewards must exceed opportunity cost

2. **Network Stability**
   - Minimum stake requirements scale with network size
   - Validator bonds exceed attack incentives
   - Quorum requirements prevent centralization
   - Validator set scales with square root of network size

3. **Implementation Considerations**
   - Efficient slashing execution
   - Proper handling of partial slashing
   - Clear documentation of stake requirements
   - Dynamic validator set management

## Actor Incentives

| Actor | Required Stake | Rewards | Risks | Notes |
|-------|---------------|---------|-------|-------|
| **Sponsor** | 10 units (S_sponsor) | - Base reward per successful sponsorship (0.5 units)<br>- Reputation building for future sponsorships<br>- Potential for higher rewards based on sponsorship history | - 50% stake slashing for false sponsorships<br>- 50% stake slashing for sponsored node violations<br>- Reputation damage for failed sponsorships | - Rewards must exceed opportunity cost of locked stake<br>- Must carefully vet candidates to avoid slashing<br>- Long-term value in building good sponsorship history |
| **Validator** | 1000 units (V_bond) | - Base reward per valid validation (1.0 units)<br>- Performance-based rewards<br>- Honest validator bonus<br>- Additional rewards for high accuracy | - 50% bond slashing for invalid validations<br>- Proportional slashing for missed validations<br>- Removal from validator set for poor performance | - Bond size ensures honest validation more profitable than slashing<br>- Must maintain high uptime and accuracy<br>- Rewards must exceed opportunity cost of locked bond |
| **Identity Holder** | 100 units (S_mint) | - Ability to participate in the network<br>- Potential to become a sponsor<br>- Access to network services and applications | - 50% stake slashing for protocol violations<br>- Loss of identity status if revoked | - Value comes from network participation<br>- Must maintain honest behavior<br>- Long-term value in building reputation |

## Reward Structure

1. **Sponsor Rewards**
   - Base reward: 0.5 units (F_sponsor)
   - Performance bonus: sponsorBonus
   - Reputation multiplier: reputationMultiplier
   - Slashing for:
     - False sponsorships (50% stake)
     - Sponsored node violations (50% stake)

2. **Validator Rewards**
   - Base reward: 1.0 units (F_mint)
   - Performance bonus: validatorBonus
   - Honest validator reward
   - Accuracy bonus: accuracyBonus
   - Slashing for:
     - Invalid validations (50% bond)
     - Missed validations (proportional to missed count)

## Economic Security Parameters

1. **Core Parameters**
   - Honest validator reward > dishonest validator penalty
   - Sponsor reward > sponsor stake
   - Bond size > potential gains from cheating
   - Slashing amounts > potential profits from violations

2. **Network Growth**
   - Validator set scales with √network_size
   - Maximum 100 validators
   - Minimum 7 validators
   - 20% fault tolerance

3. **Economic Scaling**
   - Rewards scale with network participation
   - Slashing scales with violation severity
   - Stake requirements maintain security
   - Validator set scales efficiently

## Example Scenarios

1. **Successful Sponsor**
   ```
   Investment: 10 units (S_sponsor)
   Rewards per sponsorship: 0.5 units (F_sponsor)
   Successful sponsorships needed to break even: 20
   Additional rewards from reputation: Variable
   ```

2. **Active Validator**
   ```
   Investment: 1000 units (V_bond)
   Rewards per validation: 1.0 units (F_mint)
   Additional honest validator reward: 1.0 units
   Validations needed to break even: ~500
   Performance bonuses: Variable
   ```

3. **Risk-Reward Balance**
   - Honest behavior must be more profitable than malicious behavior
   - Slashing must exceed potential gains from violations
   - Rewards must justify stake opportunity cost
   - Long-term value in maintaining good reputation 