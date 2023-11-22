# Rate Provider: `WeETH`

## Details
- Reviewed by: @rabmarut
- Checked by: TODO
- Deployed at:
    - [ethereum:0xCd5fE23C85820F7B72D0926FC9b05b43E359b7ee](https://etherscan.io/address/0xCd5fE23C85820F7B72D0926FC9b05b43E359b7ee#readProxyContract)
- Audit report(s):
    - [Ether.fi Audits](https://etherfi.gitbook.io/etherfi/security/audits)

## Context
Stake `ETH`, get `eETH` - a natively restaked liquid staking token that fuels DeFi and decentralizes Ethereum. The wrapper, `weETH`, directly implements `IRateProvider`.

## Review Checklist: Bare Minimum Compatibility
Each of the items below represents an absolute requirement for the Rate Provider. If any of these is unchecked, the Rate Provider is unfit to use.

- [x] Implements the [`IRateProvider`](https://github.com/balancer/balancer-v2-monorepo/blob/bc3b3fee6e13e01d2efe610ed8118fdb74dfc1f2/pkg/interfaces/contracts/pool-utils/IRateProvider.sol) interface.
- [x] `getRate` returns an 18-decimal fixed point number (i.e., 1 == 1e18) regardless of underlying token decimals.

## Review Checklist: Common Findings
Each of the items below represents a common red flag found in Rate Provider contracts.

If none of these is checked, then this might be a pretty great Rate Provider! If any of these is checked, we must thoroughly elaborate on the conditions that lead to the potential issue. Decision points are not binary; a Rate Provider can be safe despite these boxes being checked. A check simply indicates that thorough vetting is required in a specific area, and this vetting should be used to inform a holistic analysis of the Rate Provider.

### Administrative Privileges
- [x] The Rate Provider is upgradeable (e.g., via a proxy architecture or an `onlyOwner` function that updates the price source address).
    - implementation reviewed: [ethereum:0xe629ee84C1Bd9Ea9c677d2D5391919fCf5E7d5D9](https://etherscan.io/address/0xe629ee84c1bd9ea9c677d2d5391919fcf5e7d5d9#code)
    - admin address: [ethereum:0xF155a2632Ef263a6A382028B3B33feb29175b8A5](https://etherscan.io/address/0xf155a2632ef263a6a382028b3b33feb29175b8a5#readProxyContract)
    - admin type: multisig
        - multisig threshold/signers: 2/5
        - multisig timelock? NO
        - trustworthy signers? NO

- [x] Some other portion of the price pipeline is upgradeable (e.g., the token itself, an oracle, or some piece of a larger system that tracks the price).
    - upgradeable component: `LiquidityPool`
        - entry point: [ethereum:0x308861A430be4cce5502d0A12724771Fc6DaF216](https://etherscan.io/address/0x308861A430be4cce5502d0A12724771Fc6DaF216#readProxyContract)
        - implementation reviewed: [ethereum:0x4D784Aa9eacc108ea5A326747870897f88d93860](https://etherscan.io/address/0x4d784aa9eacc108ea5a326747870897f88d93860#code)
        - admin: same multisig as above
    - upgradeable component: `EETH`
        - entry point: [ethereum:0x35fA164735182de50811E8e2E824cFb9B6118ac2](https://etherscan.io/address/0x35fA164735182de50811E8e2E824cFb9B6118ac2#readProxyContract)
        - implementation reviewed: [ethereum:0x1B47A665364bC15C28B05f449B53354d0CefF72f](https://etherscan.io/address/0x1b47a665364bc15c28b05f449b53354d0ceff72f#code)
        - admin: same multisig as above
    - upgradeable component: `MembershipManager`
        - entry point: [ethereum:0x3d320286E014C3e1ce99Af6d6B00f0C1D63E3000](https://etherscan.io/address/0x3d320286E014C3e1ce99Af6d6B00f0C1D63E3000#readProxyContract)
        - implementation reviewed: [ethereum:0x047A7749AD683C2Fd8A27C7904Ca8dD128F15889](https://etherscan.io/address/0x047a7749ad683c2fd8a27c7904ca8dd128f15889#code)
        - admin: same multisig as above
    - upgradeable component: `EtherFiAdmin`
        - entry point: [ethereum:0x0EF8fa4760Db8f5Cd4d993f3e3416f30f942D705](https://etherscan.io/address/0x0EF8fa4760Db8f5Cd4d993f3e3416f30f942D705#readProxyContract)
        - implementation reviewed: [ethereum:0x9D6fC3cBaaD0ef36b2B3a4b4b311C9dd267a4aeA](https://etherscan.io/address/0x9d6fc3cbaad0ef36b2b3a4b4b311c9dd267a4aea#code)
        - admin: **EOA ([ethereum:0xf8a86ea1Ac39EC529814c377Bd484387D395421e](https://etherscan.io/address/0xf8a86ea1Ac39EC529814c377Bd484387D395421e))**
    - upgradeable component: `EtherFiOracle`
        - entry point: [ethereum:0x57AaF0004C716388B21795431CD7D5f9D3Bb6a41](https://etherscan.io/address/0x57AaF0004C716388B21795431CD7D5f9D3Bb6a41#readProxyContract)
        - implementation reviewed: [ethereum:0x698cB4508F13Cc12aAD36D2B64413C302B781d9A](https://etherscan.io/address/0x698cb4508f13cc12aad36d2b64413c302b781d9a#code)
        - admin: **same EOA as above**

### Oracles
- [x] Price data is provided by an off-chain source (e.g., a Chainlink oracle, a multisig, or a network of nodes).
    - source: `LiquidityPool` allows a `membershipManager` to arbitrarily increase the exchange rate via a `rebase()` function.
        - The current `memberShipManager` is the upgradeable `MembershipManager` contract described above.
        - The entry point to `LiquidityPool#rebase()` is `MembershipManager#rebase()`, which is callable only by the `etherFiAdmin`.
            - The current `etherFiAdmin` is the upgradeable `EtherFiAdmin` contract described above.
            - The entry point to `MembershipManager#rebase()` is `EtherFiAdmin#executeTasks()`, which can be called by anyone on the `admins` list.
                - The `admins` list is configurable by the `owner`, which is the **EOA described above**.
    - any protections? YES
        - The current `EtherFiAdmin` performs many checks within the `EtherFiOracle` to determine the validity of an incoming report.
        - **However, the `EtherFiAdmin` and `EtherFiOracle` are both upgradeable by an EOA, meaning that the behavior can completely change on the whims of a single entity.**

```solidity
// Context: LiquidityPool
function rebase(int128 _accruedRewards) public {
    if (msg.sender != address(membershipManager)) revert IncorrectCaller();
    // @audit This increases the exchange rate.
    totalValueOutOfLp = uint128(int128(totalValueOutOfLp) + _accruedRewards);

    emit Rebase(getTotalPooledEther(), eETH.totalShares());
}

// Context: MembershipManager
function rebase(int128 _accruedRewards) external {
    if (msg.sender != address(etherFiAdmin)) revert InvalidCaller();
    uint256 ethRewardsPerEEthShareBeforeRebase = liquidityPool.amountForShare(1 ether);
    // @audit Call here.
    liquidityPool.rebase(_accruedRewards);
    uint256 ethRewardsPerEEthShareAfterRebase = liquidityPool.amountForShare(1 ether);

    // The balance of MembershipManager contract is used to reward ether.fan stakers (not eETH stakers)
    // Eth Rewards Amount per NFT = (eETH share amount of the NFT) * (total rewards ETH amount) / (total eETH share amount in ether.fan)
    uint256 etherFanEEthShares = eETH.shares(address(this));
    uint256 thresholdAmount = fanBoostThresholdEthAmount();
    if (address(this).balance >= thresholdAmount) {
        uint256 mintedShare = liquidityPool.deposit{value: thresholdAmount}(address(this), address(0));
        ethRewardsPerEEthShareAfterRebase += 1 ether * thresholdAmount / etherFanEEthShares;
    }

    _distributeStakingRewardsV0(ethRewardsPerEEthShareBeforeRebase, ethRewardsPerEEthShareAfterRebase);
    _distributeStakingRewardsV1(ethRewardsPerEEthShareBeforeRebase, ethRewardsPerEEthShareAfterRebase);
}

// Context: EtherFiAdmin
function executeTasks(IEtherFiOracle.OracleReport calldata _report, bytes[] calldata _pubKey, bytes[] calldata _signature) external isAdmin() {
    bytes32 reportHash = etherFiOracle.generateReportHash(_report);
    uint32 current_slot = etherFiOracle.computeSlotAtTimestamp(block.timestamp);
    require(etherFiOracle.isConsensusReached(reportHash), "EtherFiAdmin: report didn't reach consensus");
    require(slotForNextReportToProcess() == _report.refSlotFrom, "EtherFiAdmin: report has wrong `refSlotFrom`");
    require(blockForNextReportToProcess() == _report.refBlockFrom, "EtherFiAdmin: report has wrong `refBlockFrom`");
    require(current_slot >= postReportWaitTimeInSlots + etherFiOracle.getConsensusSlot(reportHash), "EtherFiAdmin: report is too fresh");
    require(current_slot < _report.refSlotTo + 1 + etherFiOracle.reportPeriodSlot(), "EtherFiAdmin: report is too old");

    numValidatorsToSpinUp = _report.numValidatorsToSpinUp;

    _handleAccruedRewards(_report);
    _handleValidators(_report, _pubKey, _signature);
    _handleWithdrawals(_report);
    _handleTargetFundsAllocations(_report);

    lastHandledReportRefSlot = _report.refSlotTo;
    lastHandledReportRefBlock = _report.refBlockTo;

    emit AdminOperationsExecuted(msg.sender, reportHash);
}

function _handleAccruedRewards(IEtherFiOracle.OracleReport calldata _report) internal {
    if (_report.accruedRewards == 0) {
        return;
    }

    // compute the elapsed time since the last rebase
    int256 elapsedSlots = int32(_report.refSlotTo - lastHandledReportRefSlot);
    int256 elapsedTime = 12 seconds * elapsedSlots;

    // This guard will be removed in future versions
    // Ensure that thew TVL didnt' change too much
    // Check if the absolute change (increment, decrement) in TVL is beyond the threshold variable
    // - 5% APR = 0.0137% per day
    // - 10% APR = 0.0274% per day
    int256 currentTVL = int128(uint128(liquidityPool.getTotalPooledEther()));
    int256 apr;
    if (currentTVL > 0) {
        apr = 10000 * (_report.accruedRewards * 365 days) / (currentTVL * elapsedTime);
    }
    int256 absApr = (apr > 0) ? apr : - apr;
    require(absApr <= acceptableRebaseAprInBps, "EtherFiAdmin: TVL changed too much");

    // @audit Call here.
    membershipManager.rebase(_report.accruedRewards);
}
```

- [ ] Price data is expected to be volatile (e.g., because it represents an open market price instead of a (mostly) monotonically increasing price).

### Common Manipulation Vectors
- [ ] The Rate Provider is susceptible to donation attacks.

Note that this Rate Provider is not susceptible to donation attacks due to the way the `LiquidityPool` accounts for received ETH:

```solidity
// Context: LiquidityPool

// @audit This function always returns the contract's "balance" as the sum of two variables.
function getTotalPooledEther() public view returns (uint256) {
    return totalValueOutOfLp + totalValueInLp;
}

// @audit Upon receipt, one variable increases while the other decreases: there is no net change to the "balance."
receive() external payable {
    if (msg.value > type(uint128).max) revert InvalidAmount();
    totalValueOutOfLp -= uint128(msg.value);
    totalValueInLp += uint128(msg.value);
}

// @audit Donations are not wholly lost; they can be absorbed into the price by an admin, who is trusted not to make
// large atomic changes to the exchange rate.
function rebase(int128 _accruedRewards) public {
    if (msg.sender != address(membershipManager)) revert IncorrectCaller();
    totalValueOutOfLp = uint128(int128(totalValueOutOfLp) + _accruedRewards);

    emit Rebase(getTotalPooledEther(), eETH.totalShares());
}
```

## Additional Findings
To save time, we do not bother pointing out low-severity/informational issues or gas optimizations (unless the gas usage is particularly egregious). Instead, we focus only on high- and medium-severity findings which materially impact the contract's functionality and could harm users.

- N/A

## Conclusion
**Summary judgment: UNSAFE**

The `EtherFiAdmin` ultimately determines how rebases (wrapped token price hikes) are propagated through the system, and it is upgradeable by an EOA. We should pay no mind to any logic contained within the current implementation because it could be changed at a moment's notice by a single trusted entity. If this entity can find a way to profit off a large price hike, there is nothing stopping it from issuing one atomically.
