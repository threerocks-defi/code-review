# Rate Provider: `WrappedUsdPlusRateProvider`

## Details
- Reviewed by: @baileyspraggins
- Checked by: @\<GitHub handle of secondary reviewer\>
- Deployed at:
    - [arbitrum:0x2bA447d4B823338435057571bF70907F8224BB47](https://arbiscan.io/address/0x2bA447d4B823338435057571bF70907F8224BB47)
    - [optimism:0xe561451322a5efC51E6f8ffa558C7482c892Bc1A](https://optimistic.etherscan.io/address/0xe561451322a5efC51E6f8ffa558C7482c892Bc1A)
    - [base:0xe561451322a5efC51E6f8ffa558C7482c892Bc1A](https://basescan.org/address/0xe1b1e024f4Bc01Bdde23e891E081b76a1A914ddd)
- Audit report(s):
    - [USD+](https://2173993027-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F9HhCCgYexXiRot0OWAJY%2Fuploads%2FCKqeV09QHnfTVum3rtBd%2Fabch-ovn-core-report.pdf?alt=media&token=2eb0419d-4695-43a0-ba2f-f3caebfc75b4)

## Context
Overnight Finance is a DeFi protocol that provides various yield-generating stable coins. wUSD+ is the wrapped asset of Overnight's main token, USD+, which allows users to interact with external protocols using the token. USD+ is a rebasing token that is fully collateralized with assets that can be converted to USDC.

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
    - upgradeable component: 
        - `WrappedUsdPlusToken` 
            - Entrypoint: 
                - [arbitrum:0xB86fb1047A955C0186c77ff6263819b37B32440D](https://arbiscan.io/address/0xb86fb1047a955c0186c77ff6263819b37b32440d)
                - [optimism:0xA348700745D249c3b49D2c2AcAC9A5AE8155F826](https://optimistic.etherscan.io/address/0xA348700745D249c3b49D2c2AcAC9A5AE8155F826)
                - [base:0xd95ca61CE9aAF2143E81Ef5462C0c2325172E028](https://basescan.org/address/0xd95ca61CE9aAF2143E81Ef5462C0c2325172E028)
            - Implementation Reviewed:     
                - [arbitrum:0xA7E51DF47dcd98F729a80b1C931aAC2b5194f4A0](https://arbiscan.io/address/0xa7e51df47dcd98f729a80b1c931aac2b5194f4a0)
                - [optimism:0x52A8F84672B9778632F98478B4DCfa2Efb7E3247](https://optimistic.etherscan.io/address/0x52a8f84672b9778632f98478b4dcfa2efb7e3247)
                - [base:0xE3434045a3bE5376e8d3Cf841981835996561f80](https://basescan.org/address/0xe3434045a3be5376e8d3cf841981835996561f80)
        - `UsdPlusToken`
            - Entrypoint: 
                - [arbitrum:0xe80772Eaf6e2E18B651F160Bc9158b2A5caFCA65](https://arbiscan.io/address/0xe80772eaf6e2e18b651f160bc9158b2a5cafca65)
                - [optimism:0x73cb180bf0521828d8849bc8CF2B920918e23032](https://optimistic.etherscan.io/address/0x73cb180bf0521828d8849bc8CF2B920918e23032)
                - [base:0xB79DD08EA68A908A97220C76d19A6aA9cBDE4376](https://basescan.org/address/0xB79DD08EA68A908A97220C76d19A6aA9cBDE4376)
            - Implementation Reviewed:  
                - [arbitrum:0x159f28F598b5C5340D6A902D34eB373D30499660](https://arbiscan.io/address/0x159f28f598b5c5340d6a902d34eb373d30499660)
                - [optimism:0xB79DD08EA68A908A97220C76d19A6aA9cBDE4376](https://optimistic.etherscan.io/address/0xb79dd08ea68a908a97220c76d19a6aa9cbde4376)
                - [base:0x441Df98011aD427C5692418999ba2150e6d84277](https://basescan.org/address/0x441df98011ad427c5692418999ba2150e6d84277)

        - `Exchange`
            - Entrypoint: 
                - [arbitrum:0x73cb180bf0521828d8849bc8CF2B920918e23032](https://arbiscan.io/address/0x73cb180bf0521828d8849bc8cf2b920918e23032)
                - [optimism:0xe80772Eaf6e2E18B651F160Bc9158b2A5caFCA65](https://optimistic.etherscan.io/address/0xe80772Eaf6e2E18B651F160Bc9158b2A5caFCA65)
                - [base:0x7cb1B38591021309C64f451859d79312d8Ca2789](https://basescan.org/address/0x7cb1B38591021309C64f451859d79312d8Ca2789)
            - Implementation Reviewed:  
                - [arbitrum:0x763018d8B4c27a6Fb320CD588e2Bc355D0d3049E](https://arbiscan.io/address/0x763018d8b4c27a6fb320cd588e2bc355d0d3049e)
                - [optimism:0xcf02cf91b5ec8230d6bd26c48a8b762ce6081c0f](https://optimistic.etherscan.io/address/0xcf02cf91b5ec8230d6bd26c48a8b762ce6081c0f)
                - [base:0x083f016e9928a3eaa3aca0ff9f4e4ded5db3b4b7](https://basescan.org/address/0x083f016e9928a3eaa3aca0ff9f4e4ded5db3b4b7)

    - admin address: [\<network:address\>](\<link to contract block explorer\>)
    - admin type: \<EOA/multisig\> \<Delete this hint: If EOA, delete the whole sub-section below.\>
        - multisig threshold/signers: \<X/Y\>
        - multisig timelock? \<YES: minimum duration/NO\>
        - trustworthy signers? \<YES: whom/NO\> \<Delete this hint: Are the signers known entities such as Vitalik, Hudson, samczsun, or Fernando? Or are they random addresses?\>

### Oracles
- [ ] Price data is provided by an off-chain source (e.g., a Chainlink oracle, a multisig, or a network of nodes).

- [ ] Price data is expected to be volatile (e.g., because it represents an open market price instead of a (mostly) monotonically increasing price). 

### Common Manipulation Vectors
- [ ] The Rate Provider is susceptible to donation attacks.

## Additional Findings
To save time, we do not bother pointing out low-severity/informational issues or gas optimizations (unless the gas usage is particularly egregious). Instead, we focus only on high- and medium-severity findings which materially impact the contract's functionality and could harm users.


## Conclusion
**Summary judgment: SAFE

The `WrappedUsdPlusRateProvider` calaculates the exchange rate of wUSD+ in terms of USD+ by multiplying `1e18` by the `liquidityIndex` of the underlying asset, USD+. This liquidityIndex is set by the exchange everytime a payout occurs. We can state that this RateProvider is **SAFE** for Balancer pools given a few trust assumptions. 
