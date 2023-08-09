# Rate Provider: `TBYRateProvider`

## Details
- Reviewed by: @baileyspraggins
- Checked by: @\<GitHub handle of secondary reviewer\>
- Deployed at:
    - This review was performed pre-deployment.

## Context
The `TBYRateProvider` is a RateProvider for Blueberrie's Bloom Protocol. This RateProvider will be used for future Balancer pools that are created for TBY. TBYs are Term Bound Yield Tokens, which are treasury-backed Debt tokens that pegged to Blackrock's ib01, 1 year treasury bond. stUSD is a stable coin backed by TBYs. This RateProvider will be used to calculate the price of TBYs using Chainlinks [ib01](https://data.chain.link/ethereum/mainnet/indexes/ib01-usd). The `owner` of `BPSFeed` will update the price feeds as needed, and any user can view the current price of the corresponding token by calling `getWeightedRate`, which returns the total value of the token with up to date appreciation based on the current block timestamp. The `TBYRateProvider` works as a wrapper around a `BPSFeed` to allow for Balancer Pool integration. `TBYRateProvder`'s `getRate` function will return the current weighted rate of the TBY token.

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
    - upgradeable component: `EACAggregatorProxy` ([ethereum:address\>](https://etherscan.io/address/0x32d1463eb53b73c095625719afa544d5426354cb))
     - admin address: [ethereum:0x21f73D42Eb58Ba49dDB685dc29D3bF5c0f0373CA>](https://etherscan.io/address/0x21f73D42Eb58Ba49dDB685dc29D3bF5c0f0373CA)
    - admin type: Multisig
        - multisig threshold/signers: 4
        - multisig timelock? NO
        - trustworthy signers? Yes - Chainlink

### Oracles
- [x] Price data is provided by an off-chain source (e.g., a Chainlink oracle, a multisig, or a network of nodes).
    - source: Chainlink
    - source address: [ethereum:0x32d1463eb53b73c095625719afa544d5426354cb](https://etherscan.io/address/0x32d1463eb53b73c095625719afa544d5426354cb)
    - any protections? NO

- [x] Price data is expected to be volatile (e.g., because it represents an open market price instead of a (mostly) monotonically increasing price). 
    - Price data mirrors the price of a 1 year treasury bond, from Blackrocks IB01, which are expected monotonically increases longterm with slight short term volitility.

### Common Manipulation Vectors
- [ ] The Rate Provider is susceptible to donation attacks.

## Additional Findings
To save time, we do not bother pointing out low-severity/informational issues or gas optimizations (unless the gas usage is particularly egregious). Instead, we focus only on high- and medium-severity findings which materially impact the contract's functionality and could harm users.

### \<H-01: Example High-severity Finding\>
### \<H-02: Example High-severity Finding\>
### \<M-01: Example Medium-severity Finding\>
### \<M-02: Example Medium-severity Finding\>

## Conclusion
**Summary judgment: \<SAFE/UNSAFE\>**

\<Delete this hint: Formulate a nuanced conclusion here. Remember, it's okay if some of the boxes above are checked as long as reasonable protections are in place. If the Rate Provider is very obviously safe, say so. If it's very obviously not, say so: what specifically needs to change before it can be considered safe? If the conclusion is hazy, explain why, and leave the final determination up to the reader. Examples of completely unacceptable conditions include, but are not limited to: EOA admins, EOA price sources, market prices (instead of deposit/redemption prices).\>
