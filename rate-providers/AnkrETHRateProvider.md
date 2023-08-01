# Rate Provider: `AnkrETHRateProvider`

## Details
- Reviewed by: @mkflow27
- Checked by: @rabmarut
- Deployed at:
    - [avalanche:0xd6Fd021662B83bb1aAbC2006583A62Ad2Efb8d4A](https://snowtrace.io/address/0xd6Fd021662B83bb1aAbC2006583A62Ad2Efb8d4A#code)
    - [arbitrum:0xFC8d81A01deD207aD3DEB4FE91437CAe52deD0b5](https://arbiscan.io/address/0xFC8d81A01deD207aD3DEB4FE91437CAe52deD0b5#code)
    - [zkevm:0xFC8d81A01deD207aD3DEB4FE91437CAe52deD0b5](https://zkevm.polygonscan.com/address/0xfc8d81a01ded207ad3deb4fe91437cae52ded0b5#code)

## Context
`ankrETH` is a liquid staking token developed by Ankr. The `ankrETH` tokens are bridged via a custom bridge developed by Ankr (see [audit](https://assets.ankr.com/earn/ankr_bridge_security_audit.pdf)). The bridged version of `ankrETH` is called `InternetBond` and is a `BeaconProxy`.

The overall process of getting the rate to multiple networks is:

1. Custom bridge `ankrETH` to network of choice
2. Daily update of `ratio` (tvl / tvl + rewards) on the network of choice in a contract called `ratioFeed` . This contract is an aggregation of rates and can be queried for the rate.
3. Rate provider returns inverse of `ratio` as `getRate()`

Any addresses provided throughout this review apply to the deployment on Avalanche. The other deployments are assumed to be identical, as suggested by a cursory review of each. In limited cases where they are not identical, a specific note is provided.

## Review Checklist: Bare Minimum Compatibility
Each of the items below represents an absolute requirement for the Rate Provider. If any of these is unchecked, the Rate Provider is unfit to use.

- [x] Implements the [`IRateProvider`](https://github.com/balancer/balancer-v2-monorepo/blob/bc3b3fee6e13e01d2efe610ed8118fdb74dfc1f2/pkg/interfaces/contracts/pool-utils/IRateProvider.sol) interface.
- [x] `getRate` returns an 18-decimal fixed point number (i.e., 1 == 1e18) regardless of underlying token decimals.

## Review Checklist: Common Findings
Each of the items below represents a common red flag found in Rate Provider contracts.

If none of these is checked, then this might be a pretty great Rate Provider! If any of these is checked, we must thoroughly elaborate on the conditions that lead to the potential issue. Decision points are not binary; a Rate Provider can be safe despite these boxes being checked. A check simply indicates that thorough vetting is required in a specific area, and this vetting should be used to inform a holistic analysis of the Rate Provider.

### Administrative Privileges
- [ ] The Rate Provider is upgradeable (e.g., via a proxy architecture or an `onlyOwner` function that updates the price source address).

- [x] Some other portion of the price pipeline is upgradeable (e.g., the token itself, an oracle, or some piece of a larger system that tracks the price).
    - upgradeable component: `ankrETH` ([avalanche:0x12D8CE035c5DE3Ce39B1fDD4C1d5a745EAbA3b8C](https://snowtrace.io/address/0x12d8ce035c5de3ce39b1fdd4c1d5a745eaba3b8c#code))
        - admin address: [avalanche:0xF508AE11De875b2136C580229d1B8291F1EC2B7E](https://snowtrace.io/address/0xf508ae11de875b2136c580229d1b8291f1ec2b7e#code)
        - admin type: multisig
            - multisig threshold/signers: 3/5
            - multisig timelock? NO
            - trustworthy signers? NO (can't identify any)
    - upgradeable component: `InternetBondRatioFeed_R2` ([avalanche:0xEf3C162450E1d08804493aA27BE60CDAa054050F](https://snowtrace.io/address/0xEf3C162450E1d08804493aA27BE60CDAa054050F#code))
        - admin address: [avalanche:0xF508AE11De875b2136C580229d1B8291F1EC2B7E](https://snowtrace.io/address/0xf508ae11de875b2136c580229d1b8291f1ec2b7e#code)
        - admin type: multisig (same as above)
            - multisig threshold/signers: 3/5
            - multisig timelock? NO
            - trustworthy signers? NO (can't identify any)

### Oracles
- [x] Price data is provided by an off-chain source (e.g., a Chainlink oracle, a multisig, or a network of nodes).
    - source: `InternetBondRatioFeed_R2` accepts updates from the 3/5 multisig mentioned above (here called `owner`), or an `operator` designated by the `owner`
    - source address: [avalanche:0xEf3C162450E1d08804493aA27BE60CDAa054050F](https://snowtrace.io/address/0xEf3C162450E1d08804493aA27BE60CDAa054050F#code)
    - any protections? YES **but only for `operators`. The 3/5 multisig `owner` can always override.** For `operators`:
        - rate monotonically increases
        - rate delta is within configurable (by `owner`) threshold
        - 12 hours minimum between updates (**NOTE: this is broken, see below**)
        - **NOTE: the zkEVM deployment seems to use an [older version](https://zkevm.polygonscan.com/address/0x8886d04007871becbffbde6e4b8a66090956e1b1#code) of the implementation which lacks any protections for `operators` (see below)**

- [ ] Price data is expected to be volatile (e.g., because it represents an open market price instead of a (mostly) monotonically increasing price).

### Common Manipulation Vectors
- [ ] The Rate Provider is susceptible to donation attacks.

## Additional Findings
To save time, we do not bother pointing out low-severity/informational issues or gas optimizations (unless the gas usage is particularly egregious). Instead, we focus only on high- and medium-severity findings which materially impact the contract's functionality and could harm users.

### M-01: A privileged `operator` can bypass the 12-hour minimum update period
In the implementation of [`InternetBondRatioFeed_R2`](https://snowtrace.io/address/0x6870edcd297f33ea986d4cf975ee049e668b36fe#code), there is a check in the `_ratioRules()` function that is clearly designed to cap the update frequency at every 12 hours:

```solidity
require(
    block.timestamp - lastUpdated >= 12 hours,
    "ratio was updated less than 12 hours ago"
);
```

However, another check in `updateRatioBatch()` prevents `hisRatio.lastUpdate` (passed as `lastUpdated` above) from being set unless at least **24 hours** has passed:

```solidity
if (block.timestamp - hisRatio.lastUpdate > 1 days - 1 minutes) {
    uint64 latestOffset = hisRatio.historicalRatios[0];
    hisRatio.historicalRatios[
        ((latestOffset + 1) % 8) + 1
    ] = uint64(ratios[i]);
    hisRatio.historicalRatios[0] = latestOffset + 1;
    hisRatio.lastUpdate = uint40(block.timestamp);
}
```

Therefore, between 12 and 24 hours after the most recent update, a designated `operator` can call `updateRatioBatch()` as many times as they want. During this window, the first check will pass but the second will fail. When this happens, `_ratios` is properly updated, but `hisRatio.lastUpdate` is not.

#### Impact
This makes it possible to update `_ratios` many times in a single block or over multiple blocks, effectively bypassing the `_ratioThreshold` and escalating an `operator`'s privilege to be more similar to the `owner`'s. Note that the `owner` can **always** arbitrarily update `_ratios`, but our trust assumptions for an `operator` are meant to be more restricted.

### M-02: The deployment on zkEVM is outdated and unprotected (relative to other networks)
The zkEVM deployment seems to use an older version of `InternetBondRatioFeed_R2` called [`InternetBondRatioFeed_R1`](https://zkevm.polygonscan.com/address/0x8886d04007871becbffbde6e4b8a66090956e1b1#code). This version lacks any protections for `operators`; when it comes to updating the price ratio, an `operator` is identical to the `owner`.

#### Impact
An `operator` can provide updates to `_ratios` at any time and any magnitude, not monotonically increasing or constrained within a threshold.

## Conclusion
**Summary judgment: UNSAFE**

The trustworthiness of this Rate Provider largely hinges on the trustworthiness of a 3/5 multisig whose signers are unclear to the reviewer. Given both M-01 and M-02, it also hinges on the multisig's reasonable discretion in designating `operators`, who possess almost as much power as the multisig itself. It appears that at least one `operator` is an EOA, so this may be a very risky proposition.

In order to be deemed safe, the reviewer would like to see:
- M-01 patched
- M-02 patched
- A transparent report detailing the signers of the 3/5 multisig
