# BitYield Protocol

## Bitcoin Yield Aggregation Protocol on Stacks Layer 2

BitYield is a decentralized, secure, and composable yield aggregation protocol that allows Bitcoin holders to earn optimized yields across multiple whitelisted DeFi protocols. Built on [Stacks](https://www.stacks.co), it leverages Bitcoin’s security while providing dynamic APY management, automated rebalancing, and institutional-grade safeguards.

## Features

- **Multi-Protocol Aggregation**: Diversifies user deposits across whitelisted yield protocols using allocation strategies.
- **Dynamic APY Management**: Continuously updates APY per protocol to reflect real-time yield opportunities.
- **Rebalancing Engine**: Automatically adjusts allocations to optimize rewards and reduce risk.
- **Token Whitelisting**: Integrates with any SIP-010 compliant token contract once approved by governance.
- **Emergency Shutdown**: A built-in circuit breaker for pausing operations during critical situations.
- **Security & Compliance First**: Enforces strict minimum/maximum deposit limits, token validation, and authorization checks.

## Smart Contract Architecture

### Constants

- **Owner Authorization**: `contract-owner`
- **Error Codes**: Range from `u1000` to `u1012` for clear, consistent error handling.
- **Deposit Bounds**: `min-deposit` and `max-deposit` ensure reasonable limits.
- **APY Bounds**: `MAX-APY` = 10000 (represents 100% in basis points).

### Data Storage

- **Maps**:
  - `user-deposits`: Tracks user balances and last deposit block.
  - `user-rewards`: Tracks accrued and claimed rewards.
  - `protocols`: Stores protocol metadata including name, status, and APY.
  - `strategy-allocations`: Sets basis point allocations per protocol.
  - `whitelisted-tokens`: Tracks which SIP-010 tokens are approved.

- **Variables**:
  - `total-tvl`: Total value locked across the system.
  - `platform-fee-rate`: Platform fee in basis points.
  - `emergency-shutdown`: Boolean flag to halt deposits.

## Security and Validation

### Access Control

Only the contract owner can:

- Add/update protocols
- Adjust fee rates
- Enable emergency shutdown
- Whitelist new tokens

### Validation Functions

- Ensures valid APY, protocol ID, names, and tokens.
- Prevents unauthorized or malformed transactions.
- Confirms that tokens are SIP-010 compliant and approved.

## Deposit & Withdrawal Workflow

### Deposit

```lisp
(deposit (token-trait <sip-010-trait>) (amount uint))
```

- Validates token, amount, and deposit limits.
- Transfers tokens to the contract via SIP-010.
- Updates user deposit state.
- Triggers rebalancing across protocols.

### Withdraw

```lisp
(withdraw (token-trait <sip-010-trait>) (amount uint))
```

- Verifies token approval and sufficient balance.
- Transfers funds back to the user.
- Updates TVL and user state accordingly.

## Reward Calculation & Claiming

### Reward Formula

```lisp
rewards = (deposit * weighted-apy * blocks) / (10000 * 144 * 365)
```

- Uses the weighted average APY across active strategies.
- Rewards are based on time since last deposit.

### Claim Rewards

```lisp
(claim-rewards (token-trait <sip-010-trait>))
```

- Calculates rewards using current block height.
- Transfers claimable amount to user via SIP-010.
- Updates `user-rewards` map.

## 🧠 Strategy Rebalancing

The protocol dynamically rebalances deposits based on:

- Active protocol list
- Allocation percentages (in basis points)
- Ensuring total allocations ≤ 10000 (100%)

```lisp
(rebalance-protocols)
```

## Admin Functions

- `add-protocol`: Adds a new protocol to the registry.
- `update-protocol-status`: Enables/disables protocol.
- `update-protocol-apy`: Updates the APY for a protocol.
- `set-platform-fee`: Adjusts platform fee.
- `set-emergency-shutdown`: Pauses/resumes protocol activity.
- `whitelist-token`: Approves a SIP-010 token for deposits.

## SIP-010 Token Compatibility

BitYield interacts with SIP-010 tokens via the following required trait:

```clojure
(define-trait sip-010-trait
  ...
)
```

Ensure any token used for deposits conforms to this trait and is whitelisted by the protocol admin.

## Example Protocol List

Currently supports up to 5 preconfigured protocols (IDs: `u1` to `u5`). Future versions may allow dynamic protocol listings.

## Security Considerations

- **Safe Token Transfers**: All transfers are validated and executed within contract scope.
- **Emergency Controls**: Contract can be disabled in case of malicious activity.
- **Hardcoded Limits**: Prevent overflow, abuse, or risk exposure from excessive deposits or misconfigured APYs.

## Future Enhancements

- DAO governance for protocol additions and APY updates.
- More granular strategy controls (e.g. time-weighted APY).
- Cross-chain integration for Lightning and L2 bridges.
- Enhanced analytics on TVL, yield history, and risk metrics.
