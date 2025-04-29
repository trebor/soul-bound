# Soul Bound Protocol - Stake System Design

## Stake Types and Requirements

| Stake Type | Actor | Purpose | Amount | Lock Period | Slashing Conditions | Notes |
|------------|-------|---------|--------|-------------|---------------------|-------|
| Identity Minting Stake (S_mint) | Candidate | Required to create a new identity | S_mint units | Identity lifetime | - Fraudulent identity creation<br>- Protocol violations<br>- Multiple identity attempts | Cannot be withdrawn while identity is active |
| Sponsor Stake (S_sponsor) | Sponsor | Required to vouch for a new identity | S_sponsor units | Duration of sponsorship | - Sponsoring fraudulent identities<br>- Protocol violations during verification<br>- False attestations | Must be maintained for active sponsorships |
| Validator Bond (V_bond) | Validator | Required to participate in validation | V_bond units | While active as validator | - Invalid validations<br>- Protocol violations<br>- Censorship<br>- Non-participation | Must exceed S_mint × networkSecurityFactor |

## Stake Operations

| Operation | Actor | Stake Type | Conditions | Notes |
|-----------|-------|------------|------------|-------|
| Locking | Candidate | S_mint | - Must meet minimum amount<br>- Must be available | Required before minting |
| Locking | Sponsor | S_sponsor | - Must meet minimum amount<br>- Must be available | Required before sponsorship |
| Locking | Validator | V_bond | - Must meet minimum amount<br>- Must be available | Required to join validator set |
| Withdrawal | Identity Holder | S_mint | - Identity must be revoked<br>- No active sponsorships | Cannot withdraw while active |
| Withdrawal | Sponsor | S_sponsor | - Sponsorship period ended<br>- No active violations | Must wait for cooling period |
| Withdrawal | Validator | V_bond | - Not in active validator set<br>- No pending violations | Must wait for rotation period |

## Slashing Conditions

| Violation | Affected Stake | Slashing Amount | Cascading Effect |
|-----------|---------------|-----------------|------------------|
| Fraudulent Identity Creation | S_mint | slashFraction | None |
| False Sponsorship | S_sponsor | slashFraction | Direct sponsor loses sponsorSlashFraction |
| Invalid Validation | V_bond | slashFraction | None |
| Protocol Violation | All stakes | slashFraction | Direct sponsor loses sponsorSlashFraction |
| Multiple Identity Attempt | S_mint | slashFraction | None |
| Censorship | V_bond | slashFraction | None |
| Non-participation | V_bond | Proportional to missed validations | None |

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

5. **Cascading Slashing Example**
   ```
   A -> B -> C -> D
   ```
   If Node D is slashed:
   - Node D loses slashFraction of stake
   - Node C (direct sponsor) loses sponsorSlashFraction of stake
   - Node B (grand-sponsor) loses (sponsorSlashFraction/2) of stake
   - Node A (great-grand-sponsor) loses (sponsorSlashFraction/4) of stake

## Economic Analysis

1. **Economic Security**
   - Stake amounts must be high enough to deter malicious behavior
   - Slashing must exceed potential gains from violations
   - Rewards must exceed opportunity cost

2. **Network Stability**
   - Minimum stake requirements scale with network size
   - Validator bonds exceed attack incentives
   - Quorum requirements prevent centralization

3. **Implementation Considerations**
   - Efficient slashing execution
   - Proper handling of partial slashing
   - Clear documentation of stake requirements

4. **Future Considerations**
   - Dynamic stake requirements based on network size
   - Adaptive slashing based on violation severity
   - Additional stake types for specialized roles 

## Actor Incentives

| Actor | Required Stake | Rewards | Risks | Notes |
|-------|---------------|---------|-------|-------|
| **Sponsor** | S_sponsor | - Base reward per successful sponsorship (F_sponsor)<br>- Reputation building for future sponsorships<br>- Potential for higher rewards based on sponsorship history | - slashFraction stake slashing for false sponsorships<br>- sponsorSlashFraction stake slashing for sponsored node violations<br>- Reputation damage for failed sponsorships | - Rewards must exceed opportunity cost of locked stake<br>- Must carefully vet candidates to avoid slashing<br>- Long-term value in building good sponsorship history |
| **Validator** | V_bond | - Base reward per valid validation (F_mint)<br>- Performance-based rewards<br>- Honest validator bonus<br>- Additional rewards for high accuracy | - slashFraction bond slashing for invalid validations<br>- Proportional slashing for missed validations<br>- Removal from validator set for poor performance | - Bond size ensures honest validation more profitable than slashing<br>- Must maintain high uptime and accuracy<br>- Rewards must exceed opportunity cost of locked bond |
| **Identity Holder** | S_mint | - Ability to participate in the network<br>- Potential to become a sponsor<br>- Access to network services and applications | - slashFraction stake slashing for protocol violations<br>- Loss of identity status if revoked | - Value comes from network participation<br>- Must maintain honest behavior<br>- Long-term value in building reputation |

## Reward Structure

1. **Sponsor Rewards**
   - Base reward: F_sponsor
   - Performance bonus: sponsorBonus
   - Reputation multiplier: reputationMultiplier
   - Slashing for:
   - False sponsorships (slashFraction stake)
   - Sponsored node violations (sponsorSlashFraction stake)

2. **Validator Rewards**
   - Base reward: F_mint
   - Performance bonus: validatorBonus
   - Honest validator reward
   - Accuracy bonus: accuracyBonus
   - Slashing for:
   - Invalid validations (slashFraction bond)
   - Missed validations (proportional to missed count)

## Economic Incentives and Rewards

| Role | Investment Required | Potential Rewards | Risk Factors | Profitability Considerations |
|------|---------------------|-------------------|--------------|-----------------------------|
| **Sponsor** | S_sponsor (S_sponsor units) | - Base reward per successful sponsorship (F_sponsor)<br>- Reputation building for future sponsorships<br>- Potential for higher rewards based on sponsorship history | - slashFraction stake slashing for false sponsorships<br>- sponsorSlashFraction stake slashing for sponsored node violations<br>- Reputation damage for failed sponsorships | - Rewards must exceed opportunity cost of locked stake<br>- Must carefully vet candidates to avoid slashing<br>- Long-term value in building good sponsorship history |
| **Validator** | V_bond (V_bond units) | - Base reward per valid validation (F_mint)<br>- Performance-based rewards<br>- Honest validator bonus<br>- Additional rewards for high accuracy | - slashFraction bond slashing for invalid validations<br>- Proportional slashing for missed validations<br>- Removal from validator set for poor performance | - Bond size ensures honest validation more profitable than slashing<br>- Must maintain high uptime and accuracy<br>- Rewards must exceed opportunity cost of locked bond |
| **Identity Holder** | S_mint (S_mint units) | - Ability to participate in the network<br>- Potential to become a sponsor<br>- Access to network services and applications | - slashFraction stake slashing for protocol violations<br>- Loss of identity status if revoked | - Value comes from network participation<br>- Must maintain honest behavior<br>- Long-term value in building reputation |

### Reward Distribution Mechanics

1. **Sponsor Rewards**
   - Base reward: F_sponsor
   - Performance bonus: sponsorBonus
   - Reputation multiplier: reputationMultiplier
   - Slashing for:
     - False sponsorships (slashFraction stake)
     - Sponsored node violations (sponsorSlashFraction stake)

2. **Validator Rewards**
   - Base reward: F_mint
   - Performance bonus: validatorBonus
   - Honest validator reward
   - Accuracy bonus: accuracyBonus
   - Slashing for:
     - Invalid validations (slashFraction bond)
     - Missed validations (proportional to missed count)

3. **Economic Security Parameters**
   - Honest validator reward > dishonest validator penalty
   - Sponsor reward > sponsor stake
   - Bond size > potential gains from cheating
   - Slashing amounts > potential profits from violations

### Example Scenarios

1. **Successful Sponsor**
   ```
   Investment: S_sponsor units
   Rewards per sponsorship: F_sponsor
   Successful sponsorships needed to break even: 20
   Additional rewards from reputation: Variable
   ```

2. **Active Validator**
   ```
   Investment: V_bond units
   Rewards per validation: F_mint
   Additional honest validator reward: 1.0 units
   Validations needed to break even: ~500
   Performance bonuses: Variable
   ```

3. **Risk-Reward Balance**
   - Honest behavior must be more profitable than malicious behavior
   - Slashing must exceed potential gains from violations
   - Rewards must justify stake opportunity cost
   - Long-term value in maintaining good reputation

### Key Economic Principles

1. **Alignment of Incentives**
   - Rewards structured to make honest behavior most profitable
   - Slashing designed to make violations economically irrational
   - Long-term value in building good reputation

2. **Risk Management**
   - Clear rules for reward distribution
   - Transparent slashing conditions
   - Predictable economic outcomes

3. **Sustainability**
   - Reward pool funded by:
     - Minting fees
     - Slashed stakes
     - Network participation fees
   - Dynamic adjustment based on network conditions
   - Long-term economic viability

4. **Participation Requirements**
   - Minimum stake amounts to participate
   - Performance requirements for rewards
   - Clear rules for reward distribution
   - Transparent slashing conditions 