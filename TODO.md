## TODO: Document Consistency Tasks

- [x] Rename `S_sponsor` to `S_endorse` and align the stake threshold rule in § 8.2
- [x] Harmonize `V_bond` requirement (must exceed S_mint×10) with default `validatorBond` in Appendix 13.1
- [x] Reconcile **Attestation** data type in § 2.2 with actual `SponsorAttestation` fields in § 4.3
- [x] Update JSON-Schemas for `MintRequest` and `RevocationRequest` to include optional `blockHeight` alongside `timestamp`
* [ ] Define or remove the `*Ack` response schemas (`ChallengeAck`, `SensorAck`, `AttestationAck`, `MintResponse`, `ValidateAck`, `RevokeAck`) referenced in § 13.2
* [ ] Standardize `evidence` vs `evidenceHash` field names between `RevocationRequest` (§ 4.6) and `RevocationRecord` (§ 6.2)
* [ ] Align fields in off-chain `RewardDistribution` message and on-chain `RewardSlashRecord` (e.g., `contractSig` vs `txHash`)
* [ ] Re-add or remove Section 9.7 Implementation Enforcement Mechanisms as needed 