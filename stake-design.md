# Soul Bound Protocol - Stake System Design

## Introduction

The Soul Bound Protocol implements a stake-based security model to ensure the integrity and reliability of identity verification and validation processes. This document outlines the economic and security mechanisms that govern participation in the network. [See RFC Section 9](../rfc.md#9-economic-mechanisms)

### Key Principles

1. **Economic Security** [See RFC Section 9.5](../rfc.md#95-economic-security-proofs)
   - Stake requirements create financial incentives for honest behavior
   - Slashing mechanisms deter malicious actions
   - Rewards align participant interests with network health

2. **Scalable Trust** [See RFC Section 9.2](../rfc.md#92-network-effects--economic-scaling)
   - Stake amounts balance security with accessibility
   - Progressive stake requirements for different roles
   - Dynamic adjustment based on network conditions

3. **Progressive Participation** [See RFC Section 9.3](../rfc.md#93-validator-bonds--rewards)
   - Three-tiered stake system (Identity, Sponsor, Validator)
   - Clear progression path for network participants
   - Graduated responsibilities and rewards

4. **Dynamic Governance** [See RFC Section 12.4](../rfc.md#124-custom-rewardslashing-policies)
   - Stake-based voting for protocol changes
   - Transparent slashing conditions
   - Adjustable parameters based on network conditions

### Document Structure

This document is organized into the following sections:
- Stake Types and Requirements: Details of each stake type and their conditions [See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees)
- Stake Operations: Rules for locking and withdrawing stakes [See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees)
- Slashing Conditions: Consequences for protocol violations [See RFC Section 9.5](../rfc.md#95-economic-security-proofs)
- Economic Analysis: Incentive structures and security parameters [See RFC Section 9.2](../rfc.md#92-network-effects--economic-scaling)
- Actor Incentives: Detailed breakdown of participant roles and rewards [See RFC Section 9.3](../rfc.md#93-validator-bonds--rewards)

## Stake Types and Requirements [See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees)

| Stake Type | Actor | Purpose | Amount | Lock Period | Slashing Conditions | Notes |
|------------|-------|---------|--------|-------------|---------------------|-------|
| Identity Minting Stake (S_mint) | Candidate | Required to create a new identity | 100 units | Identity lifetime | - Fraudulent identity creation<br>- Protocol violations<br>- Multiple identity attempts | Cannot be withdrawn while identity is active<br>[See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees) |
| Sponsor Stake (S_sponsor) | Sponsor | Required to vouch for a new identity | 10 units | Duration of sponsorship | - Sponsoring fraudulent identities<br>- Protocol violations during verification<br>- False attestations | Must be maintained for active sponsorships<br>[See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees) |
| Validator Bond (V_bond) | Validator | Required to participate in validation | 1000 units | While active as validator | - Invalid validations<br>- Protocol violations<br>- Censorship<br>- Non-participation | Must exceed S_mint × networkSecurityFactor (10)<br>[See RFC Section 9.3](../rfc.md#93-validator-bonds--rewards) |

## Stake Operations [See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees)

| Operation | Actor | Stake Type | Conditions | Notes |
|-----------|-------|------------|------------|-------|
| Locking | Candidate | S_mint | - Must meet minimum amount (100 units)<br>- Must be available | Required before minting<br>[See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees) |
| Locking | Sponsor | S_sponsor | - Must meet minimum amount (10 units)<br>- Must be available | Required before sponsorship<br>[See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees) |
| Locking | Validator | V_bond | - Must meet minimum amount (1000 units)<br>- Must be available | Required to join validator set<br>[See RFC Section 9.3](../rfc.md#93-validator-bonds--rewards) |
| Withdrawal | Identity Holder | S_mint | - Identity must be revoked<br>- No active sponsorships | Cannot withdraw while active<br>[See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees) |
| Withdrawal | Sponsor | S_sponsor | - Sponsorship period ended<br>- No active violations | Must wait for cooling period<br>[See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees) |
| Withdrawal | Validator | V_bond | - Not in active validator set<br>- No pending violations | Must wait for rotation period<br>[See RFC Section 9.3](../rfc.md#93-validator-bonds--rewards) |

## Slashing Conditions [See RFC Section 9.5](../rfc.md#95-economic-security-proofs)

| Violation | Affected Stake | Slashing Amount | Cascading Effect |
|-----------|---------------|-----------------|------------------|
| Fraudulent Identity Creation | S_mint | 50% (slashFraction) | None |
| False Sponsorship | S_sponsor | 50% (slashFraction) | Direct sponsor loses 50% (sponsorSlashFraction) |
| Invalid Validation | V_bond | 50% (slashFraction) | None |
| Protocol Violation | All stakes | 50% (slashFraction) | Direct sponsor loses 50% (sponsorSlashFraction) |
| Multiple Identity Attempt | S_mint | 50% (slashFraction) | None |
| Censorship | V_bond | 50% (slashFraction) | None |
| Non-participation | V_bond | Proportional to missed validations | None |

## Stake Management Rules [See RFC Section 9.1](../rfc.md#91-identity-minting-stakes--fees)

1. **Locking Requirements**
   - All stakes must be locked before participation
   - Minimum amounts defined by protocol parameters
   - Lock periods enforced by smart contract

2. **Withdrawal Rules**
   - Stakes cannot be withdrawn while active
   - Must wait for rotation period
   - No pending violations

3. **Slashing Rules** [See RFC Section 9.5](../rfc.md#95-economic-security-proofs)
   - Full slashing for protocol violations
   - Partial slashing for sponsors based on violation type
   - Cascading effects defined by protocol

4. **Governance** [See RFC Section 12.4](../rfc.md#124-custom-rewardslashing-policies)
   - Stake amounts configurable via governance
   - Minimum stake ratios must be maintained
   - Changes require quorum approval

## Economic Analysis and Incentives [See RFC Section 9.2](../rfc.md#92-network-effects--economic-scaling)

### Core Parameters [See RFC Section 14.1](../rfc.md#141-parameter-recommendations-δs-stake-sizes-quorum)
- Honest validator reward > dishonest validator penalty
- Sponsor reward > sponsor stake
- Bond size > potential gains from cheating
- Slashing amounts > potential profits from violations

### Actor Incentives and Rewards [See RFC Section 9.3](../rfc.md#93-validator-bonds--rewards)

| Actor | Required Stake | Rewards | Risks | Notes |
|-------|---------------|---------|-------|-------|
| **Sponsor** | 10 units (S_sponsor) | - Base reward per successful sponsorship (0.5 units)<br>- Reputation building for future sponsorships<br>- Potential for higher rewards based on sponsorship history | - 50% stake slashing for false sponsorships<br>- 50% stake slashing for sponsored node violations<br>- Reputation damage for failed sponsorships | - Rewards must exceed opportunity cost of locked stake<br>- Must carefully vet candidates to avoid slashing<br>- Long-term value in building good sponsorship history |
| **Validator** | 1000 units (V_bond) | - Base reward per valid validation (1.0 units)<br>- Performance-based rewards<br>- Honest validator bonus<br>- Additional rewards for high accuracy | - 50% bond slashing for invalid validations<br>- Proportional slashing for missed validations<br>- Removal from validator set for poor performance | - Bond size ensures honest validation more profitable than slashing<br>- Must maintain high uptime and accuracy<br>- Rewards must exceed opportunity cost of locked bond |
| **Identity Holder** | 100 units (S_mint) | - Ability to participate in the network<br>- Potential to become a sponsor<br>- Access to network services and applications | - 50% stake slashing for protocol violations<br>- Loss of identity status if revoked | - Value comes from network participation<br>- Must maintain honest behavior<br>- Long-term value in building reputation |

### Example Scenarios [See RFC Section 9.2](../rfc.md#92-network-effects--economic-scaling)

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

3. **Risk-Reward Balance** [See RFC Section 9.5](../rfc.md#95-economic-security-proofs)
   - Honest behavior must be more profitable than malicious behavior
   - Slashing must exceed potential gains from violations
   - Rewards must justify stake opportunity cost
   - Long-term value in maintaining good reputation 