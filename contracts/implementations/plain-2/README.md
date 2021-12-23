# Plain2RAI

Author: https://github.com/mrazivy-vtak
This file provides documentation for Plain2Pool which is two coin pool with adaption to handle moving redemption price such as RAI.

**Tips**

I've created **_two types of Plain2RAI pool, Plain2Optimized-based and Plain2Basic-based one_**.

On [gitcoin](https://gitcoin.co/issue/reflexer-labs/curve-contract/6/100027296), it was explained to adapt the Plain2Optimized one, but in this case, it can only handle ERC20 that returns True/revert, so I created Plain2Basic-based one just in case, as it can handle ERC20 return True/revert, True/False.

Both has same interface, differences are about how to handle call response.

## Description of modifications

### [contracts/implementations/plain-2/Plain2RAI.vy](contracts/implementations/plain-2/Plain2RAI.vy):

is built from [Plain2Optimized.by](contracts/implementations/plain-2/Plain2Optimized.vy) with adaption to handle moving redemption price.

This contract uses RedemptionPriceSnap (described below) to fetch a moving redemption price, and also utilizes rate multiplier, which is usually used to align the decimals among coins, to work with moving redemption price.
These mechanisms were largely inspired by [MetaRAI.vy](contracts/implementations/meta/MetaRAI.vy).

#### Contract States

In addition to the states implemented in [Plain2Optimized.by](contracts/implementations/plain-2/Plain2Optimized.vy), the followings are also defined.

1. `rate_multiplier: public(uint256)`
   - A rate multiplier for pegged coin.
2. `redemption_price_snap: public(address)`
   - An address where RedemptionPriceSnap contract is located.
3. `redemption_price_scale: public(uint256)`
   - A scale of the redeption price returned by RedemptionPriceSnap.
4. `deployer: public(address)`
   - An address of deployer who deploy Plain2RAI via Factory. This is only used for initializing RedemptionPriceSnap via `initialize_rate_feed` method.

#### Contract methods

In addition to the methods implemented in [Plain2Optimized.by](contracts/implementations/plain-2/Plain2Optimized.vy), the followings are also defined.

```py
@external
def initialize_rate_feed(
    _feed: address,
    _redemption_price_scale: uint256
)
```

Initialize RedemptionPriceSnap address `redemption_price_snap` and `redemption_price_scale`.
This is the same as the one defined in [MetaRAI.vy](contracts/implementations/meta/MetaRAI.vy) and callable by only `deployer`.

```py
@view
@internal
def _get_scaled_redemption_price() -> uint256:
```

This fetchs a redemption price normalized by `redemption_price_scale`.
This is the same as the one defined in [MetaRAI.vy](contracts/implementations/meta/MetaRAI.vy).

```py
@view
@internal
def _rates() -> uint256[N_COINS]:
```

This returns rate multiplier of two coins. Internally `_get_scaled_redemption_price` was called.

#### RedemptionPriceSnap

Plain2RAI uses RedemptionPriceSnap to fetch a moving redemption price.
This interface is the same as the one used in [MetaRAI.vy](contracts/implementations/meta/MetaRAI.vy) and [this](https://github.com/reflexer-labs/geb-redemption-price-snap/blob/master/src/RedemptionPriceSnap.sol).

```python
interface RedemptionPriceSnap:
    def snappedRedemptionPrice() -> uint256: view
```

### Unit Test:

To test Plain2RAI, [tests/conftest.py](tests/conftest.py) and some fixtures are modified.

In [conftest.py](tests/conftest.py), `pool_types` now supports `rai`, and the validation process of test conditions (such as --pool-size, --return-type options) for RAI Pool has been added in `pytest_collection_modifyitems` function.

In [fixtures/deployments.py](tests/fixtures/deployments.py), the initialization process for RAI Pool (e.g. calling `initialize_rate_feed`) has been added.

There are some other minor changes, please refer to the commit log 1d7d459e6539da25a6c058b3a00f4e804fd85e54.

## Unit test results

**All cases are passed!**

To check it, run below command

```sh
brownie test --pool-type rai
```

```sh
Brownie v1.17.2 - Python development framework for Ethereum

==================================== test session starts =====================================
platform darwin -- Python 3.9.9, pytest-6.2.5, py-1.11.0, pluggy-1.0.0
rootdir: ***
plugins: eth-brownie-1.17.2, xdist-1.34.0, hypothesis-6.27.3, web3-5.25.0, forked-1.3.0
collected 251 items

Launching 'ganache-cli --port 8545 --gasLimit 12000000 --accounts 10 --hardfork istanbul --mnemonic brownie --defaultBalanceEther 1000000000'...

tests/test_factory.py ..ssssssssss.s.ss.s...ssssss..ssssssssss.ss                      [ 17%]
tests/gauge/test_add_reward.py ...                                                     [ 18%]
tests/gauge/test_approve.py ..............                                             [ 23%]
tests/gauge/test_boost_delegation.py ..                                                [ 24%]
tests/gauge/test_checkpoint.py ...                                                     [ 25%]
tests/gauge/test_claim_rewards_multiple.py .......                                     [ 28%]
tests/gauge/test_claim_rewards_none.py ..                                              [ 29%]
tests/gauge/test_claim_rewards_transfer.py ...                                         [ 30%]
tests/gauge/test_claim_rewards_unipool.py ....                                         [ 32%]
tests/gauge/test_deposit_for.py ..                                                     [ 33%]
tests/gauge/test_deposit_withdraw.py ......                                            [ 35%]
tests/gauge/test_kick.py .                                                             [ 35%]
tests/gauge/test_mass_exit_reward_claim.py .                                           [ 36%]
tests/gauge/test_set_rewards_receiver.py .                                             [ 36%]
tests/gauge/test_transfer.py .........                                                 [ 40%]
tests/gauge/test_transferFrom.py ..................                                    [ 47%]
tests/pools/common/test_add_liquidity.py .......                                       [ 50%]
tests/pools/common/test_add_liquidity_initial.py ....                                  [ 51%]
tests/pools/common/test_claim_fees.py .....                                            [ 53%]
tests/pools/common/test_exchange.py ..                                                 [ 54%]
tests/pools/common/test_exchange_imbalanced.py ..                                      [ 55%]
tests/pools/common/test_exchange_reverts.py ...............                            [ 61%]
tests/pools/common/test_get_virtual_price.py ........                                  [ 64%]
tests/pools/common/test_ramp_A_precise.py ........                                     [ 67%]
tests/pools/common/test_receiver.py .....                                              [ 69%]
tests/pools/common/test_remove_liquidity.py .......                                    [ 72%]
tests/pools/common/test_remove_liquidity_imbalance.py ...........                      [ 76%]
tests/pools/common/test_remove_liquidity_one_coin.py ..................                [ 84%]
tests/pools/common/test_withdraw_admin_fees.py .                                       [ 84%]
tests/token/test_approve.py .............                                              [ 89%]
tests/token/test_transfer.py .........                                                 [ 93%]
tests/token/test_transferFrom.py .................                                     [100%]

======================== 219 passed, 32 skipped in 312.96s (0:05:12) =========================
```

Plain2Basic-based one is also passing all cases!

```sh
==================================== test session starts =====================================
platform darwin -- Python 3.9.9, pytest-6.2.5, py-1.11.0, pluggy-1.0.0
rootdir: ***
plugins: eth-brownie-1.17.2, xdist-1.34.0, hypothesis-6.27.3, web3-5.25.0, forked-1.3.0
collected 753 items

Launching 'ganache-cli --port 8545 --gasLimit 12000000 --accounts 10 --hardfork istanbul --mnemonic brownie --defaultBalanceEther 1000000000'...

tests/test_factory.py ..ssssssssss.s.ss.s...ssssss..ssssssssss.ss                      [  5%]
tests/gauge/test_add_reward.py ...                                                     [  6%]
tests/gauge/test_approve.py ..............                                             [  7%]
tests/gauge/test_boost_delegation.py ..                                                [  8%]
tests/gauge/test_checkpoint.py ...                                                     [  8%]
tests/gauge/test_claim_rewards_multiple.py .......                                     [  9%]
tests/gauge/test_claim_rewards_none.py ..                                              [  9%]
tests/gauge/test_claim_rewards_transfer.py ...                                         [ 10%]
tests/gauge/test_claim_rewards_unipool.py ....                                         [ 10%]
tests/gauge/test_deposit_for.py ..                                                     [ 11%]
tests/gauge/test_deposit_withdraw.py ......                                            [ 11%]
tests/gauge/test_kick.py .                                                             [ 11%]
tests/gauge/test_mass_exit_reward_claim.py .                                           [ 12%]
tests/gauge/test_set_rewards_receiver.py .                                             [ 12%]
tests/gauge/test_transfer.py .........                                                 [ 13%]
tests/gauge/test_transferFrom.py ..................                                    [ 15%]
tests/pools/common/test_add_liquidity.py .......                                       [ 16%]
tests/pools/common/test_add_liquidity_initial.py ....                                  [ 17%]
tests/pools/common/test_claim_fees.py .....                                            [ 17%]
tests/pools/common/test_exchange.py ..                                                 [ 18%]
tests/pools/common/test_exchange_imbalanced.py ..                                      [ 18%]
tests/pools/common/test_exchange_reverts.py ...............                            [ 20%]
tests/pools/common/test_get_virtual_price.py ........                                  [ 21%]
tests/pools/common/test_ramp_A_precise.py ........                                     [ 22%]
tests/pools/common/test_receiver.py .....                                              [ 23%]
tests/pools/common/test_remove_liquidity.py .......                                    [ 24%]
tests/pools/common/test_remove_liquidity_imbalance.py ...........                      [ 25%]
tests/pools/common/test_remove_liquidity_one_coin.py ..................                [ 28%]
tests/pools/common/test_withdraw_admin_fees.py .                                       [ 28%]
tests/token/test_approve.py .............                                              [ 29%]
tests/token/test_transfer.py .........                                                 [ 31%]
tests/token/test_transferFrom.py .................                                     [ 33%]
tests/test_factory.py ..ssssssssss.s.ss.s...ssssss..ssssssssss.ss                      [ 39%]
tests/gauge/test_add_reward.py ...                                                     [ 39%]
tests/gauge/test_approve.py ..............                                             [ 41%]
tests/gauge/test_boost_delegation.py ..                                                [ 41%]
tests/gauge/test_checkpoint.py ...                                                     [ 41%]
tests/gauge/test_claim_rewards_multiple.py .......                                     [ 42%]
tests/gauge/test_claim_rewards_none.py ..                                              [ 43%]
tests/gauge/test_claim_rewards_transfer.py ...                                         [ 43%]
tests/gauge/test_claim_rewards_unipool.py ....                                         [ 44%]
tests/gauge/test_deposit_for.py ..                                                     [ 44%]
tests/gauge/test_deposit_withdraw.py ......                                            [ 45%]
tests/gauge/test_kick.py .                                                             [ 45%]
tests/gauge/test_mass_exit_reward_claim.py .                                           [ 45%]
tests/gauge/test_set_rewards_receiver.py .                                             [ 45%]
tests/gauge/test_transfer.py .........                                                 [ 46%]
tests/gauge/test_transferFrom.py ..................                                    [ 49%]
tests/pools/common/test_add_liquidity.py .......                                       [ 50%]
tests/pools/common/test_add_liquidity_initial.py ....                                  [ 50%]
tests/pools/common/test_claim_fees.py .....                                            [ 51%]
tests/pools/common/test_exchange.py ..                                                 [ 51%]
tests/pools/common/test_exchange_imbalanced.py ..                                      [ 51%]
tests/pools/common/test_exchange_reverts.py ...............                            [ 53%]
tests/pools/common/test_get_virtual_price.py ........                                  [ 54%]
tests/pools/common/test_ramp_A_precise.py ........                                     [ 55%]
tests/pools/common/test_receiver.py .....                                              [ 56%]
tests/pools/common/test_remove_liquidity.py .......                                    [ 57%]
tests/pools/common/test_remove_liquidity_imbalance.py ...........                      [ 58%]
tests/pools/common/test_remove_liquidity_one_coin.py ..................                [ 61%]
tests/pools/common/test_withdraw_admin_fees.py .                                       [ 61%]
tests/token/test_approve.py .............                                              [ 63%]
tests/token/test_transfer.py .........                                                 [ 64%]
tests/token/test_transferFrom.py .................                                     [ 66%]
tests/test_factory.py ..ssssssssss.s.ss.s...ssssss..ssssssssss.ss                      [ 72%]
tests/gauge/test_add_reward.py ...                                                     [ 72%]
tests/gauge/test_approve.py ..............                                             [ 74%]
tests/gauge/test_boost_delegation.py ..                                                [ 74%]
tests/gauge/test_checkpoint.py ...                                                     [ 75%]
tests/gauge/test_claim_rewards_multiple.py .......                                     [ 76%]
tests/gauge/test_claim_rewards_none.py ..                                              [ 76%]
tests/gauge/test_claim_rewards_transfer.py ...                                         [ 76%]
tests/gauge/test_claim_rewards_unipool.py ....                                         [ 77%]
tests/gauge/test_deposit_for.py ..                                                     [ 77%]
tests/gauge/test_deposit_withdraw.py ......                                            [ 78%]
tests/gauge/test_kick.py .                                                             [ 78%]
tests/gauge/test_mass_exit_reward_claim.py .                                           [ 78%]
tests/gauge/test_set_rewards_receiver.py .                                             [ 78%]
tests/gauge/test_transfer.py .........                                                 [ 80%]
tests/gauge/test_transferFrom.py ..................                                    [ 82%]
tests/pools/common/test_add_liquidity.py .......                                       [ 83%]
tests/pools/common/test_add_liquidity_initial.py ....                                  [ 83%]
tests/pools/common/test_claim_fees.py .....                                            [ 84%]
tests/pools/common/test_exchange.py ..                                                 [ 84%]
tests/pools/common/test_exchange_imbalanced.py ..                                      [ 85%]
tests/pools/common/test_exchange_reverts.py ...............                            [ 87%]
tests/pools/common/test_get_virtual_price.py ........                                  [ 88%]
tests/pools/common/test_ramp_A_precise.py ........                                     [ 89%]
tests/pools/common/test_receiver.py .....                                              [ 89%]
tests/pools/common/test_remove_liquidity.py .......                                    [ 90%]
tests/pools/common/test_remove_liquidity_imbalance.py ...........                      [ 92%]
tests/pools/common/test_remove_liquidity_one_coin.py ..................                [ 94%]
tests/pools/common/test_withdraw_admin_fees.py .                                       [ 94%]
tests/token/test_approve.py .............                                              [ 96%]
tests/token/test_transfer.py .........                                                 [ 97%]
tests/token/test_transferFrom.py .................                                     [100%]

======================== 657 passed, 96 skipped in 2614.07s (0:43:34) ========================
```

## License

All changes are made under the (c) Curve.Fi, 2021 - [licence](LICENSE)

---

https://github.com/mrazivy-vtak
