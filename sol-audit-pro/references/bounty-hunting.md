# Bug Bounty Hunting — Recon, Tools & Workflow

## 1. Target Selection Strategy

### Finding High-Value Targets
- **Immunefi leaderboard**: Sort by maximum bounty — protocols paying $1M+ are serious about security
- **Recently deployed**: New protocols (< 6 months) are less battle-tested
- **Recent upgrades**: Check proxy upgrade events on Etherscan — new implementation = new attack surface
- **Forks of audited code**: Forks often introduce bugs when modifying audited base code
- **High TVL + small team**: More funds, less security bandwidth

### Triage: Is This Worth My Time?
```
Quick triage checklist (5 min):
1. Is there an active bounty program? (check Immunefi / protocol docs)
2. What's the max payout for Critical? (< $10k may not be worth deep dive)
3. Are there already public audits? (read them — known issues list)
4. Is the code verified on Etherscan?
5. Rough LOC estimate — is this a focused target?
```

---

## 2. Tooling Setup

### Foundry (Primary)
```bash
curl -L https://foundry.paradigm.xyz | bash
foundryup

# Project setup for bounty hunting
forge init bounty-lab
cd bounty-lab

# Add common interfaces
forge install OpenZeppelin/openzeppelin-contracts
forge install aave/aave-v3-core
forge install Uniswap/v3-core
forge install transmissions11/solmate
```

### Essential Cast Commands
```bash
# Get contract source
cast etherscan-source <address> --chain mainnet

# Read storage slot (find proxy implementation)
cast storage <proxy_address> 0x360894a13ba1a3210667c828492db98dca3e2076

# Decode calldata
cast calldata-decode "deposit(uint256,address)" <calldata>

# Get past events
cast logs --address <contract> --sig "Transfer(address,address,uint256)" \
  --from-block 19000000 --to-block latest

# Simulate transaction
cast call <contract> "balanceOf(address)(uint256)" <user_address>

# Run arbitrary tx without sending
cast run <tx_hash> --rpc-url mainnet --quick

# Trace a transaction
cast trace <tx_hash> --rpc-url mainnet
```

### Tenderly (Transaction Debugging)
- Fork any transaction and re-simulate with modified parameters
- Step through EVM execution line by line
- Best for understanding complex multi-contract interactions

### Phalcon / Blocksec
- Visual transaction flow explorer
- Excellent for understanding flash loan attack transactions post-mortem
- Study past exploits here: `explorer.phalcon.xyz`

---

## 3. Reading Past Exploits (Intelligence Gathering)

### Where to Find Exploit Post-Mortems
- **Rekt.news** — ranked by loss, detailed writeups
- **SunWeb3Sec/DeFiHackLabs** — GitHub repo with PoC for 200+ hacks
- **BlockSec Blog** — technical deep dives
- **Immunefi blog** — official writeups from white hat hunters

### Learning from DeFiHackLabs PoCs
```bash
git clone https://github.com/SunWeb3Sec/DeFiHackLabs
cd DeFiHackLabs

# Run any past exploit PoC
forge test --match-contract <ProtocolName>_exp -vvvv
```

Study these to understand:
- How real exploits chain multiple vulnerabilities
- How flash loans are integrated
- How to write clean, runnable PoC

---

## 4. Common Bounty-Winning Patterns (Recent 2023-2025)

### Pattern A: Protocol Upgrade Introduces Bug
1. Find recent upgrade transaction (proxy upgrade event)
2. Diff old vs new implementation
3. Focus on: changed functions, new state variables, altered access control
4. New code is least reviewed → highest bug density

### Pattern B: Integration Assumption Mismatch
Protocol A assumes Protocol B behaves a certain way, but doesn't:
- Protocol assumes Chainlink always returns fresh price
- Protocol assumes ERC20 transfer always succeeds
- Protocol assumes Uniswap callback can't reenter

Look for: any cross-protocol interaction with implicit assumptions

### Pattern C: Known Vuln in Similar Protocol
If Protocol X was hacked for vuln Y, search for other protocols using similar patterns:
```bash
# Search for similar code patterns across verified contracts
# Use Sourcify, 4byte.directory, or Etherscan code search
```

### Pattern D: Edge Case in Tokenomics
- First/last depositor in empty vault
- Extremely small or large amounts (1 wei, max uint)
- Exact boundary values (liquidation at exactly 100% collateral)
- Empty pool state transitions

---

## 5. Scoping Tricky Situations

### "Is this in scope?"
When in doubt:
1. Check the scope section of the bounty program literally
2. If contract is listed — it's in scope
3. If it's an external dependency (Chainlink, Uniswap) — usually out of scope, BUT if the *integration* is misconfigured, that's in scope
4. If unsure — ask the protocol's security contact BEFORE disclosing

### Out-of-scope but still valuable
Some findings are out-of-scope for bounties but still useful:
- Centralization risks (admin key is EOA) → report anyway, protocol may reward goodwill
- Gas optimizations → not paid, skip
- Front-end vulnerabilities → separate scope, check if listed

---

## 6. Maximizing Payout

### Severity Inflation Defense
Protocols and platforms often downgrade severity. Pre-empt this:
- Include precise dollar calculation of max loss
- Show the attack requires zero special privileges
- Demonstrate it works on a fork with actual numbers
- Reference similar past bugs that received same severity

### Escalation Path
```
Initial submission → Triager review (1-5 days)
→ If downgraded: provide additional evidence, request review
→ If disputed: escalate to platform (Immunefi/Sherlock have arbitration)
→ Document everything in writing throughout
```

### Partial Fixes
If protocol patches only part of the issue:
- Identify remaining attack vectors
- Submit as follow-up (may qualify for additional payment)
- Always retest after announced fix

---

## 7. Legal & Safety

- **Never deploy exploit on mainnet** — fork PoC is sufficient proof
- **Never extract funds** even with intent to return — "white hat" is not a legal defense universally
- **Get safe harbor in writing** if doing anything beyond static analysis
- **Tax implications**: bounty payments are typically taxable income — document receipt
- **Jurisdiction matters**: research your local laws on security research
