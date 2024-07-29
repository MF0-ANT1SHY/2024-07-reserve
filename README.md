# Reserve audit details
- Total Prize Pool: $187,500 in USDC
  - HM awards: $120,000 in USDC
  - Pro League Side Pool $30,000 in USDC
  - QA awards: $5,000 in USDC
  - Judge awards: $11,500 in USDC
  - Validator awards: $6,000 USDC
  - Scout awards: $1,000 in USDC
  - Mitigation Review: $14,000 in USDC (*Opportunity goes to top 3 backstage wardens based on placement in this audit who RSVP.*)
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2024-07-reserve/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts July 29, 2024 20:00 UTC
- Ends August 19, 2024 20:00 UTC

### Pro League Side Pool
- Two Pro Leaguers are designated as leads for the audit.
- 60/40 split among leads based on their [Gatherer score](https://docs.code4rena.com/awarding/incentive-model-and-awards#bonuses-for-top-competitors).
- If a lead ranks outside the top 2 (by [HM award algorithm](https://docs.code4rena.com/awarding/incentive-model-and-awards#high-and-medium-risk-bugs))
    - 33.3% of their share of the side pool goes to the Dark Horse bonus pool
- If a lead ranks outside the top 5 (by [HM award algorithm](https://docs.code4rena.com/awarding/incentive-model-and-awards#high-and-medium-risk-bugs))
    - 33.3% of their share of the side pool goes to the Dark Horse bonus pool
    - 66.6% of their share of the side pool is refunded to sponsor
- Pro League Auditors also compete for a portion of HM award and are eligible for Hunter/Gatherer bonuses

### Dark Horse bonus
- The Dark Horse bonus pool is distributed to the ranked participants beating or tying a lead auditor (using the QA curve). (If no lead ranks outside the top 2, no Dark Horse bonus is awarded.)

## Automated Findings / Publicly Known Issues

The 4naly3er report can be found [here](https://github.com/code-423n4/2024-07-reserve/blob/main/4naly3er-report.md).

_Note for C4 wardens: Anything included in this `Automated Findings / Publicly Known Issues` section is considered a publicly known issue and is ineligible for awards._

- Any acknowledged issues reported in previous audits: [/audits](https://github.com/code-423n4/2024-07-reserve/tree/main/audits)
- Low-value tokens, details here: [system-design.md#collateral-decimals-restriction](https://github.com/code-423n4/2024-07-reserve/blob/main/docs/system-design.md#collateral-decimals-restriction)
- Distributor rounding error at small DAO fees: non-negligible but deemed acceptable
- Changing distributions can send unintended amounts of funds to the veRSR DAO
- require(req.sellAmount > 1) in RevenueTrader no longer ensures sellAmount in {tok} units is greater zero
- BackingManager.forwardRevenue() will leave few revenues due to precision loss
- Distributor._setDistribution() will update the revenue distribution percentages without calling forwardRevenue() first
- Economic risks that arise from using RSR as an overcollateralization layer
- Individual RTokens should assume they can trust veRSR governance. The governing veRSR body should be assumed to be non-malicious
- Guardian censorship risk
- Any known issues / behavior documented directly in the code
- Upgrade


# Overview

The Reserve Protocol enables a class of token called RToken: self-issued tokens backed by a rebalancing basket of collateral. While the protocol enables any number of RTokens to be created, further discussion is limited to the characterization of a single RToken instance.



RTokens can be minted by depositing a basket of _collateral tokens_, and redeemed for the basket as well. Thus, an RToken will tend to trade at the market value of the entire basket that backs it, as any lower or higher price could be arbitraged.

The definition of the issuance/redemption basket is set dynamically on a block-by-block basis with respect to a _reference basket_. While the RToken often does its internal calculus in terms of a single unit of account (USD), what constitutes appreciation is entirely a function of the reference basket, which is a linear combination of reference units.

RTokens can be over-collateralized, which means that if any of their collateral tokens default, there's a pool of value available to make up for the loss. RToken over-collateralization is provided by Reserve Rights (RSR) holders, who may choose to stake their RSR on an RToken instance. Staked RSR can be seized in the case of a default, in a process that is entirely mechanistic based on on-chain price-feeds, and does not depend on governance votes or human judgment.

But markets do not over-collateralize holders for free. In order to incentivize RSR holders to stake in an RToken instance, each RToken instance can choose to offer an arbitrary portion of its revenue to be directed towards its RSR over-collateralization pool. This encourages staking in order to provision over-collateralization.

As with any smart contract application, the actual behavior may vary from the intended behavior. It's safest to observe an application in use for a long period of time before trusting it to behave as expected. This overview describes its _intended_ behavior.

For a much more detailed explanation of the economic design, including an hour-long explainer video (!) see [the Reserve website](https://reserve.org/protocol/2021_version/).

See also [README-sponsor.md](https://github.com/code-423n4/2024-07-reserve/blob/main/README-sponsor.md) for more details.

## Links

- **Previous audits:**  See: [/audits](https://github.com/code-423n4/2024-07-reserve/tree/main/audits)
- **Documentation:** https://reserve.org/protocol/ and [/docs](https://github.com/code-423n4/2024-07-reserve/tree/main/docs)
- **Website:** https://reserve.org
- **X/Twitter:** https://x.com/reserveprotocol
- **Discord:** https://discord.gg/reserveprotocol

---


# Scope

*See [scope.txt](https://github.com/code-423n4/2024-07-reserve/blob/main/scope.txt)*

### Files in scope


| File   | Logic Contracts | Interfaces | nSLOC | Purpose | Libraries used |
| ------ | --------------- | ---------- | ----- | -----   | ------------ |
| /contracts/libraries/Allowance.sol | 1| 1 | 18 | ||
| /contracts/libraries/Array.sol | 1| **** | 20 | |@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/libraries/Fixed.sol | 1| **** | 342 | ||
| /contracts/libraries/Permit.sol | 1| **** | 23 | |@openzeppelin/contracts-upgradeable/utils/AddressUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/utils/cryptography/SignatureCheckerUpgradeable.sol|
| /contracts/libraries/String.sol | 1| **** | 15 | ||
| /contracts/libraries/Throttle.sol | 1| **** | 40 | ||
| /contracts/mixins/Auth.sol | 1| **** | 91 | |@openzeppelin/contracts-upgradeable/access/AccessControlUpgradeable.sol|
| /contracts/mixins/ComponentRegistry.sol | 1| **** | 80 | |@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/mixins/Versioned.sol | 1| **** | 8 | ||
| /contracts/p1/AssetRegistry.sol | 1| **** | 153 | |@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/p1/BackingManager.sol | 1| **** | 170 | |@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/p1/BasketHandler.sol | 1| **** | 376 | |@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/utils/structs/EnumerableMap.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/p1/Broker.sol | 1| **** | 179 | |@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol<br>@openzeppelin/contracts/proxy/Clones.sol|
| /contracts/p1/Deployer.sol | 1| **** | 189 | |@openzeppelin/contracts/proxy/Clones.sol<br>@openzeppelin/contracts/proxy/ERC1967/ERC1967Proxy.sol<br>@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol|
| /contracts/p1/Distributor.sol | 1| **** | 168 | |@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol|
| /contracts/p1/Furnace.sol | 1| **** | 35 | ||
| /contracts/p1/Main.sol | 1| **** | 98 | |@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/access/OwnableUpgradeable.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/p1/RToken.sol | 1| **** | 229 | |@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol|
| /contracts/p1/RevenueTrader.sol | 1| **** | 115 | |@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/p1/StRSR.sol | 1| **** | 433 | |@openzeppelin/contracts-upgradeable/interfaces/IERC1271Upgradeable.sol<br>@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol<br>@openzeppelin/contracts-upgradeable/utils/cryptography/SignatureCheckerUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/utils/cryptography/draft-EIP712Upgradeable.sol<br>@openzeppelin/contracts-upgradeable/utils/CountersUpgradeable.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/p1/StRSRVotes.sol | 1| **** | 173 | |@openzeppelin/contracts-upgradeable/utils/math/SafeCastUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/utils/math/MathUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/interfaces/IERC5805Upgradeable.sol|
| /contracts/p1/mixins/BasketLib.sol | 1| **** | 159 | |@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/utils/structs/EnumerableSet.sol<br>@openzeppelin/contracts/utils/structs/EnumerableMap.sol|
| /contracts/p1/mixins/Component.sol | 1| **** | 44 | |@openzeppelin/contracts-upgradeable/proxy/utils/Initializable.sol<br>@openzeppelin/contracts-upgradeable/proxy/utils/UUPSUpgradeable.sol<br>@openzeppelin/contracts-upgradeable/utils/ContextUpgradeable.sol|
| /contracts/p1/mixins/RecollateralizationLib.sol | 1| **** | 163 | |@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/p1/mixins/RewardableLib.sol | 1| **** | 15 | |@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol<br>@openzeppelin/contracts/utils/Address.sol|
| /contracts/p1/mixins/TradeLib.sol | 1| **** | 78 | |@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/p1/mixins/Trading.sol | 1| **** | 65 | |@openzeppelin/contracts-upgradeable/security/ReentrancyGuardUpgradeable.sol<br>@openzeppelin/contracts/token/ERC20/IERC20.sol|
| /contracts/plugins/governance/Governance.sol | 1| **** | 100 | |@openzeppelin/contracts/governance/Governor.sol<br>@openzeppelin/contracts/governance/extensions/GovernorCountingSimple.sol<br>@openzeppelin/contracts/governance/extensions/GovernorSettings.sol<br>@openzeppelin/contracts/governance/extensions/GovernorTimelockControl.sol<br>@openzeppelin/contracts/governance/extensions/GovernorVotes.sol<br>@openzeppelin/contracts/governance/extensions/GovernorVotesQuorumFraction.sol|
| /contracts/plugins/trading/DutchTrade.sol | 1| 1 | 164 | |@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol<br>@openzeppelin/contracts/token/ERC20/utils/SafeERC20.sol|
| /contracts/plugins/trading/GnosisTrade.sol | 1| **** | 110 | |@openzeppelin/contracts/token/ERC20/extensions/IERC20Metadata.sol<br>@openzeppelin/contracts/utils/math/Math.sol<br>@openzeppelin/contracts-upgradeable/token/ERC20/utils/SafeERC20Upgradeable.sol|
| /contracts/registry/AssetPluginRegistry.sol | 1| **** | 85 | |@openzeppelin/contracts/access/Ownable.sol|
| /contracts/registry/DAOFeeRegistry.sol | 1| **** | 67 | ||
| /contracts/registry/RoleRegistry.sol | 1| **** | 17 | |@openzeppelin/contracts/access/AccessControlEnumerable.sol|
| /contracts/registry/VersionRegistry.sol | 1| **** | 57 | ||
| **Totals** | **34** | **2** | **4079** | | |

### Files out of scope

Any file not listed in the scope table above.

*See also [out_of_scope.txt](https://github.com/code-423n4/2024-07-reserve/blob/main/out_of_scope.txt)*

## Scoping Q &amp; A

### General questions

| Question                                | Answer                       |
| --------------------------------------- | ---------------------------- |
| ERC20 used by the protocol              |       Any             |
| Test coverage                           | ✅ SCOUTS: Please populate this after running the test coverage command                          |
| ERC721 used  by the protocol            |            None              |
| ERC777 used by the protocol             |           None                |
| ERC1155 used by the protocol            |              None            |
| Chains the protocol will be deployed on | Ethereum,Arbitrum,Base,Optimism |

### ERC20 token behaviors in scope

| Question                                                                                                                                                   | Answer |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| [Missing return values](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#missing-return-values)                                                      |   In scope  |
| [Fee on transfer](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#fee-on-transfer)                                                                  |  Out of scope  |
| [Balance changes outside of transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#balance-modifications-outside-of-transfers-rebasingairdrops) | Out of scope    |
| [Upgradeability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#upgradable-tokens)                                                                 |   In scope  |
| [Flash minting](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#flash-mintable-tokens)                                                              | In scope    |
| [Pausability](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#pausable-tokens)                                                                      | Out of scope    |
| [Approval race protections](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#approval-race-protections)                                              | In scope    |
| [Revert on approval to zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-approval-to-zero-address)                            | Out of scope    |
| [Revert on zero value approvals](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-approvals)                                    | Out of scope    |
| [Revert on zero value transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                    | Out of scope    |
| [Revert on transfer to the zero address](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-transfer-to-the-zero-address)                    | In scope    |
| [Revert on large approvals and/or transfers](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-large-approvals--transfers)                  | In scope    |
| [Doesn't revert on failure](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#no-revert-on-failure)                                                   |  In scope   |
| [Multiple token addresses](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#revert-on-zero-value-transfers)                                          | Out of scope    |
| [Low decimals ( < 6)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#low-decimals)                                                                 |   In scope  |
| [High decimals ( > 18)](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#high-decimals)                                                              | In scope    |
| [Blocklists](https://github.com/d-xo/weird-erc20?tab=readme-ov-file#tokens-with-blocklists)                                                                | In scope    |

### External integrations (e.g., Uniswap) behavior in scope:


| Question                                                  | Answer |
| --------------------------------------------------------- | ------ |
| Enabling/disabling fees (e.g. Blur disables/enables fees) | Yes   |
| Pausability (e.g. Uniswap pool gets paused)               |  Yes   |
| Upgradeability (e.g. Uniswap gets upgraded)               |   Yes  |


### EIP compliance checklist

| Contract                                | EIP                          |
| --------------------------------------- | ---------------------------- |
| contracts/p1/StRSR.sol                  | ERC20                        |
| contracts/p1/RToken.sol                 | ERC20                        |


# Additional context

## Main invariants

Any invariants that hold within a contract are documented within the contract itself. 

## Attack ideas (where to focus for bugs)
The protocol has mostly received differential audits that examine the impact of the marginal change. Our intent with this audit is for the auditors to survey the protocol generally, in order to look for issues that may have been missed along the way via the differential structure.

That said, some particular areas our attention has recently been on:
- Impact of oracleErrors on recollateralization
- Addition of issuance premium to prevent toxic issuance during partial de-pegs of collateral
- Stability of individual collateral markets as it relates to collateral plugin configuration

(This is not an exhaustive list.)


## All trusted roles in the protocol

✅ TODO: Description

| Role                             | Description                                      |
|----------------------------------|--------------------------------------------------|
| Issuance pauser                  |                                                  |
| Trading pauser                   |                                                  |
| Short freeze                     |                                                  |
| Long freeze                      |                                                  |
| Owner                            |                                                  |
| Guardian (ie timelock canceller) |                                                  |
| RoleRegistry.owner               |                                                  |
| RoleRegistry.emergencyCouncil    |                                                  |

## Describe any novel or unique curve logic or mathematical models implemented in the contracts:

RecollateralizationLib implements a bespoke recollateralization algorithm that tends to have lots of complexity in it and deserves special attention. 

Otherwise the math everywhere else in the protocol is fairly straightforward. 


## Running tests

See [docs/dev-env.md](https://github.com/code-423n4/2024-07-reserve/blob/main/docs/dev-env.md) for more details

```bash
git clone https://github.com/code-423n4/2024-07-reserve
cd 2024-07-reserve

cp .env.example .env # fill out the MAINNET_RPC_URL variable
yarn

yarn test:p1
yarn test:plugins
yarn test:integration
```
To run code coverage
```bash
yarn test:coverage
# if you get 'FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory' then increase the heap limit
# see https://stackoverflow.com/a/59572966
```


## Miscellaneous
Employees of Reserve Protocol and employees' family members are ineligible to participate in this audit.
