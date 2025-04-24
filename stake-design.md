# Soul Bound Protocol - Stake System Design

## Stake Types and Requirements

| Stake Type | Actor | Purpose | Amount | Lock Period | Slashing Conditions | Notes |
|------------|-------|---------|--------|-------------|---------------------|-------|
| Identity Minting Stake (S_mint) | Candidate | Required to create a new identity | 100 units | Identity lifetime | - Fraudulent identity creation<br>- Protocol violations<br>- Multiple identity attempts | Cannot be withdrawn while identity is active |
| Sponsor Stake (S_sponsor) | Sponsor | Required to vouch for a new identity | 10 units | Duration of sponsorship | - Sponsoring fraudulent identities<br>- Protocol violations during verification<br>- False attestations | Must be maintained for active sponsorships |
| Validator Bond (V_bond) | Validator | Required to participate in validation | 1000 units (10×S_mint) | While active as validator | - Invalid validations<br>- Protocol violations<br>- Censorship<br>- Non-participation | Must exceed S_mint × 10 |

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
| Fraudulent Identity Creation | S_mint | 100% | None |
| False Sponsorship | S_sponsor | 100% | Direct sponsor loses 50% |
| Invalid Validation | V_bond | 100% | None |
| Protocol Violation | All stakes | 100% | Direct sponsor loses 50% |
| Multiple Identity Attempt | S_mint | 100% | None |
| Censorship | V_bond | 100% | None |
| Non-participation | V_bond | Proportional to missed validations | None |

## Stake Management Rules

1. **Locking Requirements**
   - All stakes must be locked before participation
   - Minimum amounts must be met
   - Availability must be verified

2. **Withdrawal Rules**
   - Stakes cannot be withdrawn while active
   - Cooling periods must be observed
   - No pending violations allowed

3. **Slashing Rules**
   - Immediate upon violation detection
   - Partial slashing for sponsors based on violation type
   - Cascading effects follow tree structure

4. **Governance**
   - Stake amounts configurable via governance
   - Minimum stake ratios must be maintained
   - Parameters can be adjusted based on network conditions

5. **Cascading Slashing Example**
   ```
   Node A (sponsor)
   └── Node B (sponsored)
       └── Node C (sponsored)
           └── Node D (sponsored)
   ```
   If Node D is slashed:
   - Node D loses 100% of stake
   - Node C (direct sponsor) loses 50% of stake
   - Node B (grand-sponsor) loses 25% of stake
   - Node A (great-grand-sponsor) loses 12.5% of stake

## Notes and Considerations

1. **Economic Security**
   - Stake amounts must be high enough to deter malicious behavior
   - Slashing must exceed potential gains from violations
   - Honest behavior must be more profitable than malicious behavior

2. **Network Stability**
   - Cascading effects must be carefully balanced
   - Unfair impact on honest participants must be minimized
   - System must remain stable under various attack scenarios

3. **Implementation Considerations**
   - Clear rules for violation detection
   - Efficient slashing execution
   - Proper handling of partial slashing
   - Accurate tracking of sponsorship relationships

4. **Future Considerations**
   - Dynamic stake requirements based on network size
   - Adaptive slashing based on violation severity
   - Additional stake types for specialized roles 

## Economic Incentives and Rewards

| Role | Investment Required | Potential Rewards | Risk Factors | Profitability Considerations |
|------|---------------------|-------------------|--------------|-----------------------------|
| **Sponsor** | S_sponsor (10 units) | - Base reward per successful sponsorship (F_sponsor = 0.5 units)<br>- Reputation building for future sponsorships<br>- Potential for higher rewards based on sponsorship history | - 100% stake slashing for false sponsorships<br>- 50% stake slashing for sponsored node violations<br>- Reputation damage for failed sponsorships | - Rewards must exceed opportunity cost of locked stake<br>- Must carefully vet candidates to avoid slashing<br>- Long-term value in building good sponsorship history |
| **Validator** | V_bond (1000 units) | - Base reward per valid validation (F_mint = 1 unit)<br>- Performance-based rewards<br>- Honest validator bonus (1.0 units per validation)<br>- Additional rewards for high accuracy | - 100% bond slashing for invalid validations<br>- Proportional slashing for missed validations<br>- Removal from validator set for poor performance | - Bond size ensures honest validation more profitable than slashing<br>- Must maintain high uptime and accuracy<br>- Rewards must exceed opportunity cost of locked bond |
| **Identity Holder** | S_mint (100 units) | - Ability to participate in the network<br>- Potential to become a sponsor<br>- Access to network services and applications | - 100% stake slashing for protocol violations<br>- Loss of identity status if revoked | - Value comes from network participation<br>- Must maintain honest behavior<br>- Long-term value in building reputation |

### Reward Distribution Mechanics

1. **Sponsor Rewards**
   - Base reward (F_sponsor) for each successful sponsorship
   - Additional rewards based on:
     - Sponsorship success rate
     - Longevity of sponsored identities
     - Number of successful sponsorships
   - Rewards distributed after sponsorship period ends
   - Slashing for:
     - False sponsorships (100% stake)
     - Sponsored node violations (50% stake)

2. **Validator Rewards**
   - Base reward (F_mint) for each valid validation
   - Performance bonuses:
     - Honest validator reward (1.0 units)
     - Accuracy-based multipliers
     - Uptime incentives
   - Rewards distributed per validation
   - Slashing for:
     - Invalid validations (100% bond)
     - Missed validations (proportional)
     - Protocol violations (100% bond)

3. **Economic Security Parameters**
   - Honest validator reward > dishonest validator penalty
   - Sponsor reward > sponsor stake
   - Bond size > potential gains from cheating
   - Slashing amounts > potential profits from violations

### Example Scenarios

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
   Rewards per validation: 1 unit (F_mint)
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