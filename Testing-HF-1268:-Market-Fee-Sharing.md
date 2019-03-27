# Preface


The instructions below are intended to assist with manual testing of changes made to [share market fees between an asset owner and a trader's registrar and referrer](https://github.com/bitshares/bsips/blob/master/bsip-0043.md) in [BitShares Core v.3.0.0](https://github.com/bitshares/bitshares-core/milestone/16?closed=1).

These test instructions should be executed from a command line interface wallet (CLI) that has been **built for your test environment**.  For example, testing performed with the public testnet requires the CLI built for the [BitShares Public Testnet](https://testnet.bitshares.eu).  The following instructions were executed on a private testing environment where TEST was the core token.  These exact instructions may differ on your test environment in the following ways:

- The core token may be different than "TEST" (e.g. "BTS").  Modify the commands to use your core token symbol.
- The account names that are created might already exist in your test environment.  Check for their existence by running `get_account <ACCOUNT_NAME>`.  Modify the commands to use alternate account names).
- The asset names that are created might already exist in your test environment.  Check for the existence by running `get_asset <ACCOUNT_NAME>`.  Modify the commands to use alternate account names).


# Overview

From [BSIP-43](https://github.com/bitshares/bsips/blob/master/bsip-0043.md):

_"At this time unfortunately an assets owner is not able to share market fees with registrars and referrers to stimulate them to market the asset trading, so we suggest to add this possibility. Furthermore, enabling this feature for MPAs (e.g. bitCNY or bitUSD) can provide additional bounty for Bitshares registrars and referrals which can lead to more traders joining to the ecosystem._

_An asset owner defines the **market_fee_reward_percent**  in asset options - what percentage of the market fee he wants to share with the registrar.
**market_fee** * **market_fee_reward_percent** goes the registrar.
**market_fee** * (1 - **market_fee_reward_percent**) goes to the asset owner._

_Registrar splits the reward between itself and its referrers by defining **reward_percent**, which defines referrer's percentage. 
It is defined per each other using the already existing BitShares mechanism._

_Market fee reward is accumulated on the user account._

_Each user decides when he wants to claim the market fee reward and move it to their active balance."_

The distribution variables are summarized in the table below.

|Distribution Variable|Determined by|Distributed to|Description|
|-|-|-|-|
|market_fee_percent|Asset Owner|Asset Owner|Portion of the market trade that is collected as a fee|
|market_fee_reward_percent|Asset Owner|Registrar of trader that is receiving the asset from the filled order|Portion of the market_fee that is shared with the registrar|
|reward_percent|Registrar of trader that is receiving the asset from the filled order|Referrer of trader that is receiving the asset from the filled order|Portion of the registrar's portion that is shared with referrers|

The formulas for calculating the fee distribution are in the table below.

|Distribution Recipient|<div id="distribution-formula" /> Formula for Distribution Amount|
|-|-|
|Asset Owner|filled_trade &times; market_fee_percent &times; (1 - market_fee_reward_percent)|
|Registrar|filled_trade &times; market_fee_percent &times; market_fee_reward_percent &times; (1 - reward_percent)|
|Referrer|filled_trade &times; market_fee_percent &times; market_fee_reward_percent &times; reward_percent|

**<div id="example-1" /> Example 1 </div>**

100 units of an asset, called ASSET, is traded and filled on the DEX.  The market fee percent for the asset is 2%.  The resulting market fee is 2 ASSET.

The market_fee_reward_percent is 40%, and the reward_percent is 80%.

|Distribution Recipient|Distribution Calculation per [Formula](#distribution-formula)|Distribution Amount|
|-|-|-|
|Asset Owner|100 ASSET &times; 0.02 &times; (1 - 0.40)|1.20 ASSET|
|Registrar|100 ASSET &times; 0.02 &times; 0.40 &times; (1 - 0.80)|0.16 ASSET|
|Referrer|100 ASSET &times; 0.02 &times; 0.40 &times; 0.80|0.64 ASSET|
|**Total**|-|**2.00 ASSET**|



# Test Scenario: Shared Market Fees for a UIA

This test scenario will demonstrate the sharing of market fees of a user-issued asset (UIA) by using the same settings that are described in [Example 1](#example-1).

## <div id="initialize_accounts" /> Initialize Accounts

The following test scenario portrays the interaction of six actors: a faucet account ("faucet"), an account registrar ("registrar"), a referrer ("referrer"), an asset owner ("asset-owner"), and two trading accounts ("trader-80" and "trader-30").  Each will require their own wallet with their own keys (see [Tips for Creating New Keys](#tips-for-creating-keys)).  Certain steps must be performed by specific actors from their respective wallet.  Each step of the instructions describe which actor is performing that step (e.g. "Registrar: ..." indicates that the action should be performed from the wallet of the registrar account).  The reader should use the respective actor's wallet.

|Tip|
|-|
|Testers who prefer convenience over strict separation of keys may prefer to keep the private keys of every actor in a single wallet.|


### Faucet: Register the registrar and referrers

Create the accounts of the registrar, and the referrers from the **faucet** account (or any other LTM account)

```
register_account registrar TEST... TEST... faucet faucet 0 true
transfer faucet registrar 20000 TEST "" true

register_account referrer TEST... TEST... faucet faucet 0 true
transfer faucet referrer 1000 TEST "" true

register_account asset-owner TEST... TEST... faucet faucet 0 true
transfer faucet asset-owner 50000 TEST "" true

```


### Referrer: Upgrade to Lifetime Membership

```
upgrade_account referrer true
```


### Registrar: Upgrade to Lifetime Membership

```
upgrade_account registrar true
```


### Asset Owner: Upgrade to Lifetime Membership

```
upgrade_account asset-owner true
```


### Registrar: Create the trader accounts

Trader 80 will be created to be consistent with [Example 1](#example-1) such that 80% of all registrar-destined fees are distributed to the referrer.

```
register_account trader-80 TEST... TEST... registrar referrer 80 true
```

Trader 30 will be created as another trader such that 30% of all registrar-destined fees are distributed to the referrer.

```
register_account trader-30 TEST... TEST... registrar referrer 30 true
```

### Faucet: Fund the trader accounts

```
transfer faucet trader-80 5000 TEST "" true
transfer faucet trader-30 5000 TEST "" true
```


## Create and Issue the UIA

### Asset Owner: Create the Asset Type

The asset, called ASSET, will be created to be consistent with [Example 1](#example-1) with a 20% market fee (expressed as 2000 in terms of hundredths of a percent) and a 40% reward fee (expressed as 4000 in terms of the hundredths of a percent) that is shared with the registrar of the trading account that receives the asset.

```
create_asset asset-owner ASSET 2 {"max_supply":"1000000000000000","market_fee_percent":2000,"max_market_fee":"1000000000000000","issuer_permissions":1,"flags":1,"core_exchange_rate":{"base":{"amount":1,"asset_id":"1.3.0"},"quote":{"amount":1,"asset_id":"1.3.1"}},"whitelist_authorities":[],"blacklist_authorities":[],"whitelist_markets":[],"blacklist_markets":[],"description":"","extensions":{"reward_percent":4000}} null 
```

### Asset Owner: Issue the Asset

The asset will be issued to one of the trading accounts, "trader-30".

```
issue_asset trader-30 5000 ASSET "" true
```


## Place Orders

Two orders will be constructed such that Trader 80 should receive 100 ASSET minus the market fee that is dictated by [Example 1](#example-1).

Trader 30 will offer to sell 100 ASSET in exchange for 50 TEST.

Trader 80 will offer to sell 50 TEST in exchange for 100 ASSET.

Those orders will be matched and filled and the market fee of 2% will be deducted such that Trader 80 will finally receive 98 ASSET.


### Trader 30: Sell 100 ASSET

Place an order to sell 100 ASSET for at least 50 TEST.

```
sell_asset trader-30 100 ASSET 50 TEST 600 false true
```

Check the order book to find the newly placed order.

```
get_order_book ASSET TEST 1
```

The order will be waiting on the books for a match.


### Trader 80: Sell 50 TEST

Place an order to sell 50 TEST for at least 100 ASSET.

```
sell_asset trader-80 50 TEST 100 ASSET 600 false true
```

Check the order book and find the order

```
get_order_book ASSET TEST 1
```

The original order by trader-30 will no longer be in the book because it was matched and filled against the order from trader-80.

Check the account history for more details

```
get_account_history trader-80 2
```

The fee for the filled transaction should be 2% of the filled order of 100 ASSET = 2 ASSET.

Check the balance of Trader 80 to confirm reception of 98 ASSET.

```
list_account_balances trader-80
```


## Check the Distribution of the Market Fee

The market fee of 2 ASSET should now be distributed between the asset owner, the registrar, and the referrer per the description from [Example 1](#example-1).

### <div id="check-vesting-balances-of-asset-owner" /> Any Wallet: Check the fees distributed to the Asset Owner

First find the `dynamic_asset_data_id` of the asset

```
get_asset ASSET
```

Use the value for `dynamic_asset_data_id` field in an invocation of `get_object`

```
get_object 2.3.x
```

Inspect the value found in the output for `accumulated_fees` which will be listed in satoshis.  This asset has a precision of 2 which means that the actual fees collected should be divided by 10<sup>2</sup>.  _The result should be 1.20 ASSET._


### Any Wallet: Check the fees distributed to the Registrar

```
get_vesting_balances registrar
```

There might be many vesting balances listed.  Find the vesting balance of type `market_fee_sharing` with `asset_id` that matches the `id` of ASSET (Call `get_asset ASSET` to identify its `id` which will be of the form "1.3.x").

Inspect the value for the `balance` field for the ASSET.  This asset has a precision of 2 which means that the actual fees collected should be divided by 10<sup>2</sup>.  _The result should be 0.16 ASSET._


### Any Wallet: Check the fees distributed to the Referrer

```
get_vesting_balances referrer
```

There might be many vesting balances listed.  Find the vesting balance of type `market_fee_sharing` with `asset_id` that matches the `id` of ASSET (Call `get_asset ASSET` to identify its `id` which will be of the form "1.3.x").

Inspect the value for the `balance` field for the ASSET.  This asset has a precision of 2 which means that the actual fees collected should be divided by 10^2.  _The result should be 0.64 ASSET._


## (Optional) Withdraw the Distributed Fees

Withdrawal of fees may optionally be tested by following [these instructions](#tips-for-withdrawing-distributed-fees).


# Reference: Unit Tests

For reference, the following unit tests that have been prepared to test the new functionality ([1419](https://github.com/bitshares/bitshares-core/pull/1419)) for the issue ([BSIP-43](https://github.com/bitshares/bsips/blob/master/bsip-0043.md), [1268](https://github.com/bitshares/bitshares-core/issues/1270)).

- Market Fee Rewards
	- [asset_rewards_test for Smart Asset](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L214)
	- [asset_claim_reward_test for UIA](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L328)
- Asset Configuration
	- [cannot_create_asset_with_additional_options_before_hf](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L119)
	- [create_asset_with_additional_options_after_hf](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L144)
	- [cannot_update_additional_options_before_hf](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L176)
	- [update_additional_options_after_hf](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L192)
- Tests with no rewards for registrar and referrer
	- [white_list_asset_rewards_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L875)
	- [white_list_is_empty_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L416)
	- [white_list_doesnt_contain_registrar_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L450)
	- [white_list_contains_referrer_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L485)
- Sanity checks
	- [create_vesting_balance_object_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L947)
	- [issue_asset](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L655)
- Blockchain Integration Tests
	- [create_asset_via_proposal_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L551)
	- [update_asset_via_proposal_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L605)
	- [create_vesting_balance_with_instant_vesting_policy_before_hf1268_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L764)
	- [create_vesting_balance_with_instant_vesting_policy_after_hf1268_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L787)
	- [create_vesting_balance_with_instant_vesting_policy_via_proposal_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L830)
- Asset Fee accumulation
	- [accumulated_fees_before_hf_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L681)
	- [accumulated_fees_after_hf_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L705)
	- [accumulated_fees_with_additional_options_after_hf_test](https://github.com/bitshares/bitshares-core/blob/bf7ff54d9a17aa43f4663521e371b8c0ddfc2284/tests/tests/market_fee_sharing_tests.cpp#L731)

# <div id="tips-for-creating-keys" /> Reference: Creating New Keys

Registering a new account in BitShares requires the provision of at least two public keys to the registrar: one for the owner key and another for the active key.  Some registrars require a third public key for the memo.

Random keys for a BitShares account can be created by using the CLI Wallet command `suggest_brain_key`.


```
locked >>> suggest_brain_key 
{
  "brain_priv_key": "XXXXXX ...... ...... ...... ...... ...... ...... ...... ...... ...... ...... ...... ...... ...... ...... ......",
  "wif_priv_key": "5......",
  "pub_key": "TEST......"
}
```

The public key is provided to a registrar during account registration.  The WIF private key should be securely archived.

After account registration, a key can be imported into a CLI wallet with the command `import_key`.

```
locked >>> gethelp import_key

usage: import_key ACCOUNT_NAME_OR_ID  WIF_PRIVATE_KEY

example: import_key "1.3.11" 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
example: import_key "usera" 5KQwrPbwdL6PhXujxW37FSSQZ1JiwsST4cqQzDeyXtP79zkvFD3
```


# <div id="tips-for-withdrawing-distributed-fees" /> Reference: Withdraw the Distributed Fees

## Registrar and Referrer: Withdraw from Vesting Balances

The vesting balances for either the registrar or the referrer can be withdrawn with the following steps.

Three identifiers will be needed to withdraw the vesting balance.

- The identifier of the account
- The identifier of the asset
- The identifier of the vesting balance

The identifier of the account can be found by inspecting the `id` ("1.3.x") in the output of `get_account`

```
get_account <ACCOUNT_NAME>
```

The identifier of the asset can be found by inspecting the `id` in the output of `get_asset` ("1.2.y").

```
get_asset <ACCOUNT_NAME>
```

The identifier of the vesting balance can be found by inspecting the `id` in the output of `get_vesting_balances` ("1.13.z").

```
get_vesting_balances <ACCOUNT_NAME>
```

There might be many vesting balances listed.  Find the vesting balance of type `market_fee_sharing` with `asset_id` that matches the `id` of ASSET.   Note the `id` of the vesting balance (e.g. "1.13.z") to use in the next step.


Withdraw the vesting balance with the `vesting_balance_withdraw_operation` which has an [operation identifier of 33](https://github.com/bitshares/bitshares-core/blob/8c1a78b7f3f0083abee695c1b8a9b944f2c4a1be/libraries/chain/include/graphene/chain/protocol/operations.hpp#L84).  This will require building a transaction in the CLI Wallet.

```
begin_builder_transaction
add_operation_to_builder_transaction 0 [33, {"vesting_balance":"1.13.z", "owner":"1.2.x", "amount":{"amount":16, "asset_id":"1.3.y"}}]
set_fees_on_builder_transaction 0 1.3.0
```

Optionally preview the transaction before signing.

```
preview_builder_transaction 0
```

Sign and broadcast the transaction.

```
sign_builder_transaction 0 true
```

## Asset Owner: Withdraw from Accumulated Fees

The vesting balances for either the registrar or the referrer can be withdrawn with the following steps.

Three identifiers will be needed to withdraw the vesting balance.

- The identifier of the account
- The identifier of the asset

The identifier of the account can be found by inspecting the `id` ("1.3.x") in the output of `get_account`

```
get_account <ACCOUNT_NAME>
```

The identifier of the asset can be found by inspecting the `id` in the output of `get_asset` ("1.2.y").

```
get_asset <ACCOUNT_NAME>
```

Withdraw the vesting balance with the `asset_claim_fees_operation` which has an [operation identifier of 43](https://github.com/bitshares/bitshares-core/blob/8c1a78b7f3f0083abee695c1b8a9b944f2c4a1be/libraries/chain/include/graphene/chain/protocol/operations.hpp#L94).  This will require building a transaction in the CLI Wallet.

```
begin_builder_transaction
add_operation_to_builder_transaction 0 [43, {"issuer":"1.2.x", "amount_to_claim":{"amount":N, "asset_id":"1.3.y"}}]
set_fees_on_builder_transaction 0 1.3.0
```

Optionally preview the transaction before signing.

```
preview_builder_transaction 0
```

Sign and broadcast the transaction.

```
sign_builder_transaction 0 true
```