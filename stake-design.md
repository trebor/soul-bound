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