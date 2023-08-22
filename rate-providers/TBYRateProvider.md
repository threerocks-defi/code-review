# Rate Provider: `TBYRateProvider`

## Details
- Reviewed by: @baileyspraggins
- Checked by: @\<GitHub handle of secondary reviewer\>
- Deployed at:
    - This review was performed pre-deployment.
    - `TBYRateProvider`: [ethereum:]()
    - `BPSFeed`: [ethereum:0xde1f5f2d69339171d679fb84e4562febb71f36e6](https://etherscan.io/address/0xde1f5f2d69339171d679fb84e4562febb71f36e6)

## Context
The `TBYRateProvider` is a RateProvider for Blueberrey's Bloom Protocol. This RateProvider will be used for future Balancer pools that are created for TBY. TBYs are Term Bound Yield Tokens, which are treasury-backed debt tokens that follow the interest rate of Blackrock's ib01 1 year treasury bond. This RateProvider will be used to calculate the price of TBYs using Bloom's `ExchangeRateRegistry`. This registry is used to keep track of all TBYs in circulation as well as providing current exchange rates for each token in terms of USD. The exchange rate is calculated using `BPSFeed` and the time to maturaty for a given token. `BPSFeed`'s owner will update the price feed periodically using the average interest rate of ib01. This value is calculated off-chain, using Chainlink's [ib01 oracle](https://data.chain.link/ethereum/mainnet/indexes/ib01-usd), scaled down to four decimals. Users can view the current price by calling `getWeightedRate`, which returns the interest rate of the token, including interest gained since `currentRate` was updated.

## Review Checklist: Bare Minimum Compatibility
Each of the items below represents an absolute requirement for the Rate Provider. If any of these is unchecked, the Rate Provider is unfit to use.

- [x] Implements the [`IRateProvider`](https://github.com/balancer/balancer-v2-monorepo/blob/bc3b3fee6e13e01d2efe610ed8118fdb74dfc1f2/pkg/interfaces/contracts/pool-utils/IRateProvider.sol) interface.
- [x] `getRate` returns an 18-decimal fixed point number (i.e., 1 == 1e18) regardless of underlying token decimals.

## Review Checklist: Common Findings
Each of the items below represents a common red flag found in Rate Provider contracts.

If none of these is checked, then this might be a pretty great Rate Provider! If any of these is checked, we must thoroughly elaborate on the conditions that lead to the potential issue. Decision points are not binary; a Rate Provider can be safe despite these boxes being checked. A check simply indicates that thorough vetting is required in a specific area, and this vetting should be used to inform a holistic analysis of the Rate Provider.

### Administrative Privileges
- [ ] The Rate Provider is upgradeable (e.g., via a proxy architecture or an `onlyOwner` function that updates the price source address).

- [ ] Some other portion of the price pipeline is upgradeable (e.g., the token itself, an oracle, or some piece of a larger system that tracks the price).

### Oracles
- [x] Price data is provided by an off-chain source (e.g., a Chainlink oracle, a multisig, or a network of nodes).
    - source: Multisig
        - source address: [ethereum:0x91797a79fEA044D165B00D236488A0f2D22157BC](https://etherscan.io/address/0x91797a79fEA044D165B00D236488A0f2D22157BC)
        - any protections? YES 
            - multisig threshold/signers: 2/3
            - trustworthy signers? NO - All non-ENS addresses
                - [ethereum:0x21c2bd51f230D69787DAf230672F70bAA1826F67](https://etherscan.io/address/0x21c2bd51f230D69787DAf230672F70bAA1826F67)
                - [ethereum:0x4850D609D34389F5E3d8A61c41091f2f3de595C3](https://etherscan.io/address/0x4850D609D34389F5E3d8A61c41091f2f3de595C3)
                - [ethereum:0xCe64ACD38C96c3C8ed0522Af01B4fAd082BEaCeF](https://etherscan.io/address/0xCe64ACD38C96c3C8ed0522Af01B4fAd082BEaCeF)
            - rate update protections:
                - Interest rate cannot be less than 0% and has a ceiling of 50%. 
    - source: Chainlink
        - source address: [ethereum:0x32d1463eb53b73c095625719afa544d5426354cb](https://etherscan.io/address/0x32d1463eb53b73c095625719afa544d5426354cb)
        - any protections? NO

- [ ] Price data is expected to be volatile (e.g., because it represents an open market price instead of a (mostly) monotonically increasing price).
    - While the price data tracks the average interest rate of ib01, the tokens exchange rate will monotonically increase until its maturity date.

### Common Manipulation Vectors
- [ ] The Rate Provider is susceptible to donation attacks.

## Additional Findings
To save time, we do not bother pointing out low-severity/informational issues or gas optimizations (unless the gas usage is particularly egregious). Instead, we focus only on high- and medium-severity findings which materially impact the contract's functionality and could harm users.

- N/A

## Conclusion
**Summary judgment: SAFE**

The `TBYRateProvider` has been confirmed to be safe given some fundamental trust assumptions. The rateProvider satisfies the minimum requirements and there are reasonable protections in place on rate updates, but due to interest rate calculations occuring off-chain, the user must assume that the Bloom team's multisig correctly calculates this average rate off-chain, using Chainlink's price feed, and accurately updates the rate.

