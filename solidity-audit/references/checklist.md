# Exhaustive Smart Contract Audit Checklist

Use this for final quality gate before signing off. Mark each: ✅ CLEAR | 🔴 CRITICAL | 🟠 HIGH | 🟡 MEDIUM | 🔵 LOW | N/A

---

## A. Project Setup & Configuration

- [ ] A1. Solidity version pinned (not floating `^`)
- [ ] A2. Compiler version ≥ 0.8.0 (built-in overflow checks)
- [ ] A3. If <0.8.0: SafeMath used on ALL arithmetic
- [ ] A4. Optimizer runs configured appropriately
- [ ] A5. No experimental ABIEncoderV2 in production (use 0.8.0+)
- [ ] A6. License identifier present (SPDX)
- [ ] A7. Constructor vs initializer pattern correct for proxy vs non-proxy

---

## B. Access Control

- [ ] B1. Every privileged function has access modifier
- [ ] B2. No `tx.origin` used for authentication
- [ ] B3. Role-based access correctly implemented (OpenZeppelin AccessControl)
- [ ] B4. Admin/owner is multisig, not EOA
- [ ] B5. `renounceOwnership` cannot leave contract ownerless unexpectedly
- [ ] B6. `initialize()` protected with `initializer` modifier
- [ ] B7. Implementation contract initialized to prevent takeover
- [ ] B8. Timelock on all admin parameter changes
- [ ] B9. Emergency functions properly guarded
- [ ] B10. No hardcoded privileged addresses

---

## C. Reentrancy

- [ ] C1. All external calls follow CEI (Checks-Effects-Interactions)
- [ ] C2. `nonReentrant` guard on all state-changing external call functions
- [ ] C3. Cross-function reentrancy paths analyzed
- [ ] C4. Cross-contract reentrancy paths analyzed
- [ ] C5. Read-only reentrancy: view functions not exploitable by external callers mid-reentry
- [ ] C6. ERC777/ERC1155 hooks checked for re-entry
- [ ] C7. Callback-accepting functions (flash loans, safe transfers) guarded

---

## D. Arithmetic

- [ ] D1. No integer overflow/underflow possible (0.8.0+ or SafeMath)
- [ ] D2. All `unchecked {}` blocks justified and safe
- [ ] D3. Division never before multiplication
- [ ] D4. Casting truncation checked (uint256 → uint128, etc.)
- [ ] D5. Modulo operations correct (especially for rewards distribution)
- [ ] D6. Off-by-one in array bounds, loop conditions, time windows
- [ ] D7. Exponentiation: `**` operator safe for expected ranges

---

## E. External Calls & Tokens

- [ ] E1. All `.call()` return values checked
- [ ] E2. No `.call()` to user-supplied arbitrary addresses
- [ ] E3. ERC20 `transfer`/`transferFrom` return value checked (or use SafeERC20)
- [ ] E4. Fee-on-transfer tokens: actual received amount measured
- [ ] E5. Rebasing tokens: share-based accounting used
- [ ] E6. ETH sent with `.transfer()` (2300 gas limit) vs `.call()` (all gas)
- [ ] E7. `address(0)` guard on all external call targets
- [ ] E8. `address(this)` not sent as external target accidentally
- [ ] E9. Callback ordering assumptions documented and verified

---

## F. Oracle & Price Feeds

- [ ] F1. No spot price from AMM (getReserves, slot0) used for pricing
- [ ] F2. TWAP used with sufficient window (≥30 min for mainnet)
- [ ] F3. Chainlink: `updatedAt` freshness check
- [ ] F4. Chainlink: `answeredInRound >= roundId` check
- [ ] F5. Chainlink: `price > 0` check
- [ ] F6. Chainlink: min/max answer circuit breaker check
- [ ] F7. Multi-oracle: aggregation logic correct, deviation checked
- [ ] F8. Price manipulation resistant during extreme volatility
- [ ] F9. L2 sequencer uptime oracle checked (for Optimism/Arbitrum)

---

## G. Flash Loan Attack Surface

- [ ] G1. No single-tx governance actions (flash loan voting)
- [ ] G2. No same-block price manipulation possible
- [ ] G3. Collateral valuation uses TWAP not spot
- [ ] G4. Emergency functions not abusable with flash-loaned governance power
- [ ] G5. Donation attacks to empty markets impossible

---

## H. Upgradability

- [ ] H1. Storage layout consistent between proxy versions
- [ ] H2. `_gap[]` array included for future storage slots
- [ ] H3. `selfdestruct` not in implementation
- [ ] H4. `delegatecall` to user-supplied address impossible
- [ ] H5. EIP-1967 storage slots used (not slot 0/1)
- [ ] H6. UUPS: `_authorizeUpgrade` properly guarded
- [ ] H7. Transparent proxy: admin cannot call implementation functions
- [ ] H8. Initialized flag not reset during upgrades

---

## I. Signatures & Cryptography

- [ ] I1. `chainId` in all signed messages
- [ ] I2. Contract address in signed messages (anti-replay across contracts)
- [ ] I3. Nonce in all signed messages (anti-replay per user)
- [ ] I4. `ecrecover` return value != address(0) checked
- [ ] I5. `v` value validated (27 or 28 only)
- [ ] I6. EIP-712 domain separator correct
- [ ] I7. Merkle proof: leaf/node distinction (no second preimage attack)
- [ ] I8. Signature deadline/expiry enforced

---

## J. Denial of Service

- [ ] J1. No unbounded loops over user-controlled arrays
- [ ] J2. Pull-over-push for ETH distribution
- [ ] J3. No single-point-of-failure external dependency
- [ ] J4. Griefing: attacker cannot permanently break protocol state
- [ ] J5. Gas estimation: all paths within block gas limit
- [ ] J6. Self-destruct: cannot brick contract by force-sending ETH

---

## K. Front-Running & MEV

- [ ] K1. Slippage parameters validated (non-zero minimum)
- [ ] K2. Deadline parameters validated
- [ ] K3. Commit-reveal for sensitive operations (randomness, sensitive prices)
- [ ] K4. Sandwich-resistant swaps (AMM protocols)
- [ ] K5. `block.timestamp` not critical for exact logic

---

## L. Business Logic

- [ ] L1. Protocol invariants documented and verified
- [ ] L2. Reward accounting: no double-claiming, no missed epochs
- [ ] L3. Liquidation: healthy positions cannot be liquidated
- [ ] L4. Liquidation: unhealthy positions can always be liquidated
- [ ] L5. Fee accumulation: fees cannot be extracted without service
- [ ] L6. Rounding: always rounds in protocol's favor, not user's
- [ ] L7. State transitions: no invalid state reachable
- [ ] L8. Emergency mode: all critical paths work in paused/emergency state

---

## M. Events & Logging

- [ ] M1. All state changes emit events
- [ ] M2. Events include indexed fields for key lookup params
- [ ] M3. Events not used as primary storage (off-chain only)
- [ ] M4. No sensitive data in events (private keys, etc.)

---

## N. Code Quality (Informational)

- [ ] N1. No floating pragma
- [ ] N2. No unused variables or imports
- [ ] N3. No shadowed variables
- [ ] N4. NatSpec comments on all public functions
- [ ] N5. No magic numbers (use named constants)
- [ ] N6. Error messages on all requires
- [ ] N7. Custom errors preferred over string revert (gas efficient)
- [ ] N8. No dead code paths

---

## O. Deployment & Operations

- [ ] O1. Deployment scripts tested on fork
- [ ] O2. Constructor arguments validated
- [ ] O3. Initial state valid after deployment
- [ ] O4. Admin handoff procedure documented
- [ ] O5. Bug bounty program in place (Immunefi recommended)
- [ ] O6. Incident response plan documented
- [ ] O7. Circuit breakers / pause mechanisms tested
- [ ] O8. Multisig signers use hardware wallets with tx preview (anti-blind-signing)

---

## P. Protocol-Specific (check relevant section in defi-specific.md)

- [ ] P1. AMM invariant checks
- [ ] P2. Lending collateralization
- [ ] P3. Vault share accounting
- [ ] P4. Governance voting snapshot
- [ ] P5. NFT randomness
- [ ] P6. Bridge message verification
- [ ] P7. Fee-on-transfer token handling
