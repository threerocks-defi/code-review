# Rate Provider: `HETHRateProvider`

## Details
- Reviewed by: @rabmarut
- Checked by: TODO
- Deployed at:
    - [ethereum:0x5A2295f0b8A1f2b9bF26b9549Ce808c68e1a3F5f](https://etherscan.io/address/0x5A2295f0b8A1f2b9bF26b9549Ce808c68e1a3F5f#readProxyContract)
- Audit report(s):
    - [Zokyo - Hord Protocol Audits](https://github.com/zokyo-sec/audit-reports/tree/main/Hord)

## Context
Hord is a Liquid ETH Staking pool that maximizes rewards with auto-compounding, MEV boosts, revenue share, distributed validators and maximum security. This rate provider tracks the price of the `hETH` token in terms of `ETH`.

## Review Checklist: Bare Minimum Compatibility
Each of the items below represents an absolute requirement for the Rate Provider. If any of these is unchecked, the Rate Provider is unfit to use.

- [x] Implements the [`IRateProvider`](https://github.com/balancer/balancer-v2-monorepo/blob/bc3b3fee6e13e01d2efe610ed8118fdb74dfc1f2/pkg/interfaces/contracts/pool-utils/IRateProvider.sol) interface.
- [x] `getRate` returns an 18-decimal fixed point number (i.e., 1 == 1e18) regardless of underlying token decimals.

## Review Checklist: Common Findings
Each of the items below represents a common red flag found in Rate Provider contracts.

If none of these is checked, then this might be a pretty great Rate Provider! If any of these is checked, we must thoroughly elaborate on the conditions that lead to the potential issue. Decision points are not binary; a Rate Provider can be safe despite these boxes being checked. A check simply indicates that thorough vetting is required in a specific area, and this vetting should be used to inform a holistic analysis of the Rate Provider.

### Administrative Privileges
- [x] The Rate Provider is upgradeable (e.g., via a proxy architecture or an `onlyOwner` function that updates the price source address).
    - entry point: ([ethereum:0x5A2295f0b8A1f2b9bF26b9549Ce808c68e1a3F5f](https://etherscan.io/address/0x5A2295f0b8A1f2b9bF26b9549Ce808c68e1a3F5f#readProxyContract))
    - implementation reviewed: ([ethereum:0xeF9767aB19ACEa7622aA2f3ef2747E792505F55e](https://etherscan.io/address/0xef9767ab19acea7622aa2f3ef2747e792505f55e#code))
    - admin address: [ethereum:0x086A6d9FD61758096CF4F394AE7C1F9B6b4EEC14](https://etherscan.io/address/0x086A6d9FD61758096CF4F394AE7C1F9B6b4EEC14#code)
    - admin type: contract
        - contract name: `HordCongress`
        - contract behavior:
            - any congress member can make a proposal to execute calldata on a target contract
            - any member can then vote for/against the proposal
            - if the proposal reaches quorum, any member can execute it
            - if the proposal fails to reach quorum within 3 days, any member can cancel it
            - both the set of members and the quorum value are maintained in the `HordCongressMembersRegistry` ([ethereum:0x29A5f08a38c79a2dD1DF055792822eB1E163d574](https://etherscan.io/address/0x29A5f08a38c79a2dD1DF055792822eB1E163d574#code))
                - the `HordCongress` has sole authority to change these parameters (including adding/removing members from the set)
                - the current set of members is:
                    - EOA: [ethereum:0xDE1C1F83154eF64A8b3AE42b20Ac2DA716d18ABA](https://etherscan.io/address/0xDE1C1F83154eF64A8b3AE42b20Ac2DA716d18ABA)
                    - EOA: [ethereum:0x7027DB53bC5299e0Ee13ca6F594e4215D248c5A8](https://etherscan.io/address/0x7027DB53bC5299e0Ee13ca6F594e4215D248c5A8)
                    - EOA: [ethereum:0x9b9A58EE7443E27871057C2d9C7272599b2957Ef](https://etherscan.io/address/0x9b9A58EE7443E27871057C2d9C7272599b2957Ef)
                - the current quorum value is 2

- [x] Some other portion of the price pipeline is upgradeable (e.g., the token itself, an oracle, or some piece of a larger system that tracks the price).
    - upgradeable component `HordETHStakingManager` (`hETH`)
        - entry point: ([ethereum:0x5bBe36152d3CD3eB7183A82470b39b29EedF068B](https://etherscan.io/address/0x5bBe36152d3CD3eB7183A82470b39b29EedF068B#readProxyContract))
        - implementation reviewed: ([ethereum:0xA09c744117f2aCA714f0CA6fA26260afA972FA65](https://etherscan.io/address/0xa09c744117f2aca714f0ca6fa26260afa972fa65#code))
        - admin: same `HordCongress` contract as above
    - upgradeable component `MaintainersRegistry`
        - entry point: [ethereum:0x8B7819135Fe97aBFDc0c88596509c00FA727eaDc](https://etherscan.io/address/0x8B7819135Fe97aBFDc0c88596509c00FA727eaDc#readProxyContract)
        - implementation reviewed: [ethereum:0xE37B378917EE67166C7B91a28535650aA34682E4](https://etherscan.io/address/0xe37b378917ee67166c7b91a28535650aa34682e4#code)
        - admin: same `HordCongress` contract as above
    - upgradeable component `StakingConfiguration`
        - entry point: [ethereum:0x51B2f83aac13adB9Ed826C4cdb593C88e6B61C92](https://etherscan.io/address/0x51B2f83aac13adB9Ed826C4cdb593C88e6B61C92#readProxyContract)
        - implementation reviewed: [ethereum:0x8E4AF5f932922962612d0ddba7c2E31CBF1C4a0b](https://etherscan.io/address/0x8e4af5f932922962612d0ddba7c2e31cbf1c4a0b#code)
        - admin: same `HordCongress` contract as above

### Oracles
- [x] Price data is provided by an off-chain source (e.g., a Chainlink oracle, a multisig, or a network of nodes).
    - source: any `maintainer` in the `MaintainersRegistry`
    - source address: [ethereum:0x8B7819135Fe97aBFDc0c88596509c00FA727eaDc](https://etherscan.io/address/0x8B7819135Fe97aBFDc0c88596509c00FA727eaDc#readProxyContract)
    - any protections? YES
        - maximum balance delta is configurable by UKNOWN (WARNING: implementation contract not verified. See M01 below.)
            - current value at time of review is 10%
            - if balance delta is too large, the tx succeeds but the contract is paused
                - when paused, token transfers are not possible
                - only the `HordCongress` can unpause
        - WARNING: there is no time-based guard imposed upon price updates. See H01 below.
    - `maintainer` addresses are configurable by `HordCongress`
        - current `maintainer` addresses at time of review:
            - EOA: [ethereum:0x6aCB6B451aAf8a411E2FD68d0497A02Ec43421f3](https://etherscan.io/address/0x6aCB6B451aAf8a411E2FD68d0497A02Ec43421f3)
            - EOA: [ethereum:0x37be0a40A0d2493872e4e4b27C5147B76B554665](https://etherscan.io/address/0x37be0a40A0d2493872e4e4b27C5147B76B554665)
            - EOA: [ethereum:0xbC52775E6F2e0eEB33ce56ec012b3c0bA9fD5eFB](https://etherscan.io/address/0xbC52775E6F2e0eEB33ce56ec012b3c0bA9fD5eFB)
- [ ] Price data is expected to be volatile (e.g., because it represents an open market price instead of a (mostly) monotonically increasing price).

### Common Manipulation Vectors
- [x] The Rate Provider is susceptible to donation attacks.

The exchange rate computation from `HordETHStakingManager#getAmountOfHETHforETH()` utilizes `address(this).balance` which can be manipulated (upward) via donation. The attacker would forfeit the donated funds, but such an attack could potentially create a profit opportunity elsewhere (e.g., an external lending platform).

```solidity
// @audit This enables donating to the contract.
receive() external payable {
    emit EtherReceived(msg.sender, msg.value);
}

function getAmountOfHETHforETH(uint256 amountETH, bool isContractCall, uint256 diffExecLayerRewardsForFeelCalc) public view returns (uint256) {
    if(totalHETHMinted == 0) {
        return amountETH;
    }

    uint256 totalETHInPossession;

    // @audit The live contract balance used in this block can be manipulated via donation.
    if(isContractCall) {
        totalETHInPossession = totalBalanceETHInValidators.add(address(this).balance).sub(amountETH);
    } else {
        totalETHInPossession = totalBalanceETHInValidators.add(address(this).balance).sub(diffExecLayerRewardsForFeelCalc);
    }

    uint256 percentPrecision = 1000000;
    uint256 hethEthRatio = totalHETHMinted.mul(percentPrecision).div(totalETHInPossession);
    uint256 result = amountETH.mul(hethEthRatio).div(percentPrecision);

    return result;
}
```

## Additional Findings
To save time, we do not bother pointing out low-severity/informational issues or gas optimizations (unless the gas usage is particularly egregious). Instead, we focus only on high- and medium-severity findings which materially impact the contract's functionality and could harm users.

### H01: Price updates are effectively unbounded due to lack of time-based constraint
Despite the 10% delta limit imposed upon hETH price updates, there is effectively no limit at all. A `maintainer` can simply issue many consecutive ~10% changes, even within a single block. Keep in mind that all current `maintainers` are EOAs, so this level of power is unacceptable. Such a price increase could be used to liquidate a small `hETH` position for all of the `WETH` liquidity in a Balancer pool.

The code block enabling this attack vector is found in `HordETHStakingManager#setValidatorStats()`. The function is supposed to forcibly `pause()` the contract upon receiving unacceptable parameters. But all parameter checks are value-based, meaning they can easily be overridden across time (or across successive atomic calls to the function).

```solidity
function setValidatorStats(uint256 newRewardsAmount, uint256 newTotalETHBalanceInValidators, uint256 newTotalExecutionLayerRewards) external onlyMaintainer {
    if(!paused()) {
        require(newRewardsAmount >= totalRewardsCollected, "Wrong newRewardsAmount");
        require(newTotalExecutionLayerRewards >= totalExecLayerRewards, "Wrong newTotalExecutionLayerRewards");
    }

    // @audit This block defines the limits imposed on incoming parameters, computed from a configurable percentage (currently 10%).
    uint256 rewardsBoundary = totalRewardsCollected.mul(stakingConfiguration.tolerancePercentageForRewards()).div(100);
    uint256 ethInValidatorsBoundary = totalBalanceETHInValidators.mul(stakingConfiguration.tolerancePercentageForRewards()).div(100);

    // @audit This is the emergency switch.
    //        Note that there is no condition blocking this function from being called repeatedly.
    //        So, it's trivial to issue multiple 10% updates atomically, which can produce considerable deltas (orders of magnitude) within one transaction.
    if (
        (totalRewardsCollected > 0 && (newRewardsAmount > totalRewardsCollected.add(rewardsBoundary) || newRewardsAmount < totalRewardsCollected.sub(rewardsBoundary))) ||
        (totalBalanceETHInValidators > stakingConfiguration.amountETHInValidator() &&
        (newTotalETHBalanceInValidators > totalBalanceETHInValidators.add(ethInValidatorsBoundary) || newTotalETHBalanceInValidators < totalBalanceETHInValidators.sub(ethInValidatorsBoundary)))
    )
    {
        _pause();
    }

    ...
}
```

### M01: Implementation contract for `StakingConfiguration` is unverified on etherscan
This means we are unable to inspect the behavior of the contract, most notably to understand which entities have power to configure system parameters.

## Conclusion
**Summary judgment: UNSAFE**

This review produced two security-related findings which must be addressed before this contract could be deemed safe for Balancer:
- H01: Price updates are effectively unbounded due to lack of time-based constraint
- M01: Implementation contract for `StakingConfiguration` is unverified on etherscan

There is also a potential risk of donation attacks, but these only impact downstream integrations (not Balancer directly). Integrators should be wary of the underlying token's manipulability via donations, which will propagate into the BPT price itself, and assess the unique risk this poses to their respective protocols.

This review also makes no determination as to the security of the `hETH` token itself or the Hord staking system, as it is laser-focused on Balancer integration with the `HETHRateProvider`. Before investing your funds in any DeFi protocol, please consult its source code, documentation, and historical audits.
