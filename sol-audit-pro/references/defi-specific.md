# DeFi Protocol-Specific Audit Checks

## AMM / DEX Protocols

### Invariant Checks
- [ ] Core invariant (x*y=k or stableswap) holds after every operation
- [ ] Rounding: does rounding always favor the protocol?
- [ ] Minimum liquidity burned on first deposit (prevents donation attack)
- [ ] Skim/sync functions: can they be used to manipulate reserve accounting?

### Liquidity Management
- [ ] LP token minting: shares calculation correct for initial vs subsequent deposits
- [ ] Sandwich protection: slippage params required and validated
- [ ] Fee calculation: fees cannot be extracted without actually swapping
- [ ] Single-sided liquidity: price impact properly accounted

---

## Lending / Borrowing Protocols

### Collateralization
- [ ] Liquidation threshold < liquidation penalty (protocol stays solvent)
- [ ] Can healthy positions be liquidated? (precision errors in health check)
- [ ] Can unhealthy positions avoid liquidation? (bad debt accumulation)
- [ ] Oracle price used for both collateral AND debt? (should use same)
- [ ] Interest accrual: does compounding work correctly at block granularity?

### Interest Rate Model
- [ ] Kink utilization model: smooth transition at kink?
- [ ] Rate manipulation: can borrower manipulate their own rate?
- [ ] Reserve factor: protocol reserves properly accounted

### Borrow/Repay
- [ ] Repay more than owed: handled gracefully (no underflow)
- [ ] Dust positions: very small positions can still be liquidated
- [ ] Fee-on-transfer repayment: received amount < sent amount → bad debt

---

## Yield Aggregators / Vaults (ERC-4626)

### Share Accounting
- [ ] First depositor inflation attack: donate before first deposit?
  ```solidity
  // VULNERABLE ERC-4626
  function previewDeposit(uint assets) public view returns (uint shares) {
      uint supply = totalSupply();
      return supply == 0 ? assets : assets * supply / totalAssets();
      // If attacker donates to totalAssets() before first user, shares → 0
  }
  // FIX: virtual shares offset (OpenZeppelin 4626 uses 1e3 offset)
  ```
- [ ] Harvest sandwich: MEV bot front-runs harvest to steal yield
- [ ] Strategy migration: can tokens be stuck during migration?
- [ ] Emergency withdrawal: works even when strategy is bricked?

### Reward Tokens
- [ ] Reward drip rate manipulation
- [ ] Epoch boundary: rewards at exact epoch boundary correctly attributed?
- [ ] Compounding: does auto-compound increase share price or balance?

---

## Governance Protocols

### Voting
- [ ] Flash loan voting: `block.number` snapshot vs real-time balance
- [ ] Quorum: can quorum be gamed by voter suppression (DoS on other voters)?
- [ ] Proposal threshold: can attacker spam proposals?
- [ ] Timelock: minimum delay enforced, cancellation requires proposer/guardian
- [ ] Guardian/veto: guardian key is multisig, not EOA

### Execution
- [ ] Arbitrary call execution: does proposal execute arbitrary calldata?
- [ ] Value sent with execution: ETH drained via governance?
- [ ] Re-entrancy during execution: callback from executed action re-enters governance?

---

## NFT / ERC-721 Protocols

### Minting
- [ ] Max supply enforced (especially if mint is public)
- [ ] Re-entrancy via `onERC721Received` during `safeMint`
- [ ] Merkle whitelist: leaf collision possible?
- [ ] Randomness: `block.timestamp` / `blockhash` manipulable by validators
  ```solidity
  // VULNERABLE
  uint tokenId = uint(keccak256(abi.encodePacked(block.timestamp, msg.sender))) % maxSupply;
  // Validator can choose timestamp to get desired tokenId
  
  // BETTER: Chainlink VRF for on-chain randomness
  ```

### Royalties / Marketplace
- [ ] ERC-2981 royalty recipient: can be set to contract that reverts (DoS on sales)?
- [ ] Royalty bypass: direct peer-to-peer transfer bypasses royalty enforcement

---

## Bridge / Cross-Chain Protocols

### Message Verification
- [ ] Signature set: quorum of validators required, not just one
- [ ] Replay protection: nonce or uniqueID per message, per chain
- [ ] Chain ID in message: same message cannot be replayed on different chain
- [ ] Validator rotation: old validator set cannot sign after rotation

### Asset Locking/Minting
- [ ] 1:1 backing: minted tokens on destination = locked tokens on source
- [ ] Wrapped token admin: who can mint? Is it guarded by bridge validation?
- [ ] Emergency pause: bridge can be halted if validator set is compromised
- [ ] Wormhole pattern: verify guardian signatures are non-zero before accepting

### Wormhole 2022 Pattern Specifically
```solidity
// VULNERABLE: used outdated verify_signatures which skipped validation
// for certain guardian indices
function verifySignatures(bytes memory sig, uint index) internal {
    // Bug: if guardianIndex was invalid, it didn't revert — just returned
}

// FIXED: strict validation of every signature, every guardian index
```

---

## Token Contracts

### Standard Compliance
- [ ] `transfer` to address(0): should revert (some tokens allow it)
- [ ] Allowance race condition: `approve(0)` then `approve(N)` pattern
- [ ] `transferFrom` with MAX_UINT allowance: should not decrease allowance
- [ ] `decimals()` returns correct value (especially non-18 decimal tokens)

### Fee/Deflationary Tokens
- [ ] Protocol receiving fee-on-transfer token: accounts for reduced received amount
  ```solidity
  // WRONG: assumes full amount received
  function deposit(uint amount) external {
      token.transferFrom(msg.sender, address(this), amount);
      balances[msg.sender] += amount; // too high if fee-on-transfer!
  }
  
  // CORRECT: measure actual balance change
  function deposit(uint amount) external {
      uint before = token.balanceOf(address(this));
      token.transferFrom(msg.sender, address(this), amount);
      uint received = token.balanceOf(address(this)) - before;
      balances[msg.sender] += received; // actual received
  }
  ```

### Rebase Tokens
- [ ] stETH, AMPL, OHM: balance changes without transfer event
- [ ] Protocol storing `balances[user]` instead of shares will drift from reality
- [ ] Use share-based accounting for rebase tokens
