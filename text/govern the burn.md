# RFC-TBC: Govern the burn

|                 |                                                                                             |
| --------------- | ------------------------------------------------------------------------------------------- |
| **Start Date**  | 28.05.24                                                                    |
| **Description** | Place stream spending feature utilised by [Kappa Sigma Mu Society](https://hackmd.io/@gavwood/ByEiaFgZ8) pot under direct governance oversight by exposing the burn and related parameters to OpenGov.                                                                    |
| **Authors**     | [Decent Partners](https://decent.partners) / tbc                                                                                            |

## Summary

This RFC proposes to expose Kusama's burn based spending structure which is currently solely utilised by Kappa Sigma Mu society to OpenGov for the following reasons:

- As a matter of consistency to ensure all on-chain organisations operate under the same technical and social oversight.
- In recognition that burn derived stream funding is an under-appreciated funding primitive that can be made more accessible and programmable
- As an alternative approach to implementing an easier to implement version of the [Liquid Treasury / Optimistic Fundin](wish for change in Polkadot) in Kusama.

## Motivation

Within the current design of Kusama (and Polkadot) protocols there exists a [treasury and a burn mechanism](https://guide.kusama.network/docs/learn-polkadot-opengov-treasury/), designed to incentivise the spending of accrued funds.

In the current 6 day spend period on Kusama [494 KSM / $16k will be burned](https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Fksm-rpc.stakeworld.io#/treasury) unless it is utilised via spending proposals.

Kusama Society Kappa Sigma Mu, one of the earliest experiments on the network receives funding to its economic game to encourage new members to get a tattoo of a canary.

The pot is funded by 0.2% of the burn which streams in every 6 day spend period with people wishing to enter the society proposing bids in return for getting a tattoo. However the society pot is limited to 16.666 per week for bids.

As a result of this uninterrupted funding, the account now holds [160k+ KSM worth over $5m equivalent](https://kusama.subscan.io/account/F3opxRbN5ZbbNGg1tFTKna9ymddEen74rNVr5JRPb3nRsXX). Even if the burn halts, it would take almost 200 years to spend down the funds.

The society cannot use the funds, but due to the early implementation of the burn funding feature in the network, the ability to amend the spend requires a technical change to the core runtime. 

Whilst a technical solution can address the issue, it should really be a decision that is made through the social layer of the network. By exposing this parameterisation to OpenGov we can address both the Kappa Sigma Mu spend and make the feature more broadly accessible amd feature. 

It is worth noting that the fate of the 160k KSM accrued funds in the society pot is out of scope for this RFC since the account exists as a second order effect of the burn parameterisation and should be addressed through existing technical capabilities by either: 

- Use existing governance calls `ForceSetBalance` and/or `ForceSetTransfer` to either;
  - Burn funds leaving operational overhead for the society game
  - Repatriate funds to the Kusama treasury as combination of economic reset and to boost spend going into the burn.  
- Updating the `society pallet` to make the pot spendable (unlikely)

## Stakeholders

This RFC is relevant to the following stakeholders, listed from high to low impact:

    Token holders who would gain oversight of the burn based spending oversight without need for a runtime upgrade and therefore coding experience. Depending on the parameters, these changes may or may not require a particular governance track.
    On-chain organisations wishing to access a consistent optimistic burn based funding mechanism that previously benefitted Kappa Sigma Mu. 
    Kappa Sigma Mu who would no longer receive top ups to the society pot.

## Explanation

### Existing Order

orded by the standard pallet_timestamp), is passed to EraPayout as an input, as is measured against the full year to determine how much should be inflated.

### New Order

    The naming used in this section is tentative, based on a WIP implementation, and subject to change before finalization of this RFC.

The new order splits the process for inflation into two steps:

    Sourcing the inflation amount: This step merely specifies by how much the chain intends to inflate its token. This amount is not minted right away, and is instead passed over to the next step for distribution.
    Distributing the aforementioned amount: A sequence of functions that decide what needs to be done with the sourced inflation amount. This process is expected to transfer the inflation amount to any account that should receive it. This implies that the staking system should, similar to treasury, have a key-less account that will act as a temporary pot for the inflation amount.

In very abstract terms, an example of the above process can be:

    The chain inflates its token by a fixed 10% per year, an amount called i.
    Pay out 20% of i to the treasury account.
    Pay out 10% of what is left of i to the fellowship account.
    Pay out up to 70% of what is left of i to staking, depending on the staking rate.
    Burn anything that is left.

A proper configuration of this pallet should use pallet_parameters where possible to allow for any of the actual values used to specify Sourcing and Distribution to be changed via on-chain governance. Please see the example configurations section for more details.

In the new model, inflation can happen at any point in time. Since now a new pallet is dedicated to inflation, and it can internally store the timestamp of the last inflation point, and always inflate the correct amount. This means that while the duration of a staking era is 1 day, the inflation process can happen eg. every hour. The opposite is also possible, although more complicated: The staking/treasury system can possibly receive their corresponding income on a weekly basis, while the era duration is still 1 day. That being said, we don't recommend using this flexibility as it brings no clear advantage, and is only extra complexity. We recommend the inflation to still happen shortly before the end of the staking era. This means that if the inflation sourcing or distribution is a function of the staking rate, it can reliably use the staking rate of the last era.

Finally, as noted above, this RFC implies a new accounting system for staking to keep track of its staking reward. In short, the new process is as follows: pallet_inflation will mint the staking portion of inflation directly into a key-less account controlled by pallet_staking. At the end of each era, pallet_staking will inspect this account, and move whatever amount is paid out into it to another key-less account associated with the era number. The actual payouts, initiated by stakers, will transfer from this era account into the corresponding stakers' account.

    Interestingly, this means that any account can possibly contribute to staking rewards by transferring DOTs to the key-less parent account controlled by the staking system.

### Proposed Implementation

## Drawbacks

Description of recognized drawbacks to the approach given in the RFC. Non-exhaustively, drawbacks relating to performance, ergonomics, user experience, security, or privacy.

## Testing, Security, and Privacy

Describe the the impact of the proposal on these three high-importance areas - how implementations can be tested for adherence, effects that the proposal has on security and privacy per-se, as well as any possible implementation pitfalls which should be clearly avoided.

## Performance, Ergonomics, and Compatibility

Describe the impact of the proposal on the exposed functionality of Polkadot.

### Performance

Is this an optimization or a necessary pessimization? What steps have been taken to minimize additional overhead?

### Ergonomics

If the proposal alters exposed interfaces to developers or end-users, which types of usage patterns have been optimized for?

### Compatibility

Does this proposal break compatibility with existing interfaces, older versions of implementations? Summarize necessary migrations or upgrade strategies, if any.

## Prior Art and References

[Rethinking the burn](https://kusama.polkassembly.io/post/2018) - Nov '22

[Kappa Sigma Mu the richest collective you’ve never heard of](https://forum.polkadot.network/t/kusama-issue-1-treasury-burn-kappa-sigma-mu-and-the-richest-collective-youve-never-heard-of/3059) - Aug '23

## Unresolved Questions

Would exposing the burn based spending model to governance require a new track alongside updates to existing pallets - or indeed the creation of a new `burn pallet`. 

## Future Directions and Related Material

The burn model in Kusama could become the pre-ignition phase for a new generation of on-chain collectives kickstarted through novel multi-chain incentives, pledges and rewards. With the governance super-powers of [Krievo's](https://virto.network/blog/1ksm-1dao-create-community-kreivo/) XCM focused organisational design and payment rails, alongsie [native grass-roots stablecoins](https://kusama.polkassembly.io/referenda/362 arriving into Kusama Asset Hub via [Brale](https://brale.xyz), we believe new incentive structures can emerge that better align the incentives of voters, parachains and collectives in positive, rather than zero sum games. 

Related reading:
- [ParaNotes](https://forum.polkadot.network/t/introducing-paranotes-regenerative-defi-for-advancing-the-paraverse/1002)
- [Optimistic Funding](https://polkadot.subsquare.io/referenda/712)








