# Incident Quick Reference (2021–2026)

Use this table to pattern-match against known real-world exploits when auditing.
Cross-reference attack type + root cause to strengthen your vulnerability checklist.

| Date | Project | Loss | Attack Type | Root Cause | Source |
|------|---------|------|-------------|------------|--------|
| Oct 2021 | Cream Finance | $130M | Flash loan + oracle | yUSD vault price manipulation via supply reduction | rekt.news |
| Feb 2025 | Bybit | $1.4B | UI injection / supply chain | Safe{Wallet} JS tampered via compromised dev machine | NCC Group |
| Mar 2025 | Abracadabra | $13M | Logic flaw | State tracking error in cauldron liquidation | Halborn |
| Jul 2025 | GMX v1 | $42M | Reentrancy | GLP pool cross-contract reentrancy on Arbitrum | Halborn |
| Sep 2025 | Bunni | $8.4M | Flash loan + rounding | Rounding direction error in withdraw, 44 micro-withdrawals | The Block |
| Oct 2025 | Abracadabra #2 | $1.8M | Logic flaw | `cook()` validation flag reset, uncollateralized MIM borrow | Halborn |
| Jan 2026 | Step Finance | $30M | Key compromise | Treasury wallet private keys stolen via device breach | Halborn |
| Jan 2026 | Truebit | $26.4M | Legacy contract | Solidity 0.6.10 integer overflow in mint pricing | CoinDesk |
| Jan 2026 | SagaEVM | $7M | Supply chain / bridge | Inherited Ethermint precompile bridge vulnerability | The Block |

---

## Pattern Analysis

### By Attack Type (frequency + total loss)

| Attack Type | Incidents | Total Loss | Key Lesson |
|-------------|-----------|-----------|------------|
| Logic flaw | 3 | $44.8M | State machine errors, validation flag bypass — hardest to catch with automated tools |
| Flash loan | 2 | $138.4M | Price oracle + rounding — always check division direction AND oracle freshness |
| Reentrancy | 1 | $42M | Cross-contract variant on L2 — `nonReentrant` must cover all cross-pool call paths |
| Supply chain | 2 | $8.4B | JS frontend tampering + bridge inheritance — off-chain attack surface often ignored |
| Key compromise | 1 | $30M | Operational security failure — multisig + hardware wallet mandatory for treasury |
| Legacy contract | 1 | $26.4M | Solidity <0.8.0 overflow — NEVER deploy unaudited pre-0.8 code in new integrations |

### Emerging Patterns (2025–2026)

**Supply chain is now the largest single loss vector.**
The Bybit $1.4B loss was not a Solidity bug — it was a tampered JavaScript dependency in
Safe{Wallet}'s build pipeline. SagaEVM's $7M loss came from inheriting a precompile vulnerability
from the upstream Ethermint framework. Auditors must now scope:
- Frontend/UI dependency integrity (npm lockfiles, CSP headers, subresource integrity)
- Inherited framework vulnerabilities in EVM-compatible chains
- Third-party SDK versions pinned and verified

**Logic flaws compound across protocol versions.**
Abracadabra was hit twice (Mar 2025 + Oct 2025) with different logic flaws in the same cauldron
system. The `cook()` flag reset in the second attack mirrors the AllianceBlock initialized-flag
reset pattern — a class of bugs where a boolean guard is inadvertently cleared by a code path
that should never touch it.

**Rounding attacks are becoming more systematic.**
The Bunni exploit used 44 sequential micro-withdrawals to accumulate rounding errors. This is a
deliberate amplification technique — single rounding errors are dust, but looped 44x they become
$8.4M. Any `withdraw()` or `redeem()` with division must be checked for directional rounding
AND resistance to repeated small-amount calls.

---

## Audit Triggers from This Table

When you see these patterns, immediately cross-reference the incident table:

- `cook()` or multi-step action functions with internal flags → Abracadabra #2 pattern
- GLP/liquidity pool on L2 with external callbacks → GMX v1 pattern
- Any `withdraw()` with division → Bunni rounding pattern
- Pre-0.8.0 Solidity in any mint/price calculation → Truebit pattern
- Bridge or precompile inherited from upstream framework → SagaEVM pattern
- Treasury controlled by a single hot wallet → Step Finance pattern
- Frontend uses external JS/npm deps → Bybit pattern
