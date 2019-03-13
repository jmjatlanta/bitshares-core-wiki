TESTNET: User Guide for Hashed Time-Lock Contracts (HTLC) on BitShares
=====================================================================

**Note:** __This documentation applies to the BitShares **Public TESTNET only**. HTLC is not yet__
__implemented on MAINNET.__

**Note:** __The BitShares HTLC implementation is **beta software** and should be used for__ 
__experimentation purposes only. The software may not perform as designed and you may lose value__
__as a result.__

**Note:** __The BitShares UI Web Wallet does not yet implement the HTLC features. Therefore, one__
__must use the BitShares CLI (command line wallet) to interact with HTLC directly. The UI Team is__
__working with the Core Team to implement the required graphical interface prior to mainnet release.__

## Introduction

An HTLC is a __conditional transfer__ of value from "depositor" to "recipient" where two distinct
conditions prevent immediate execution. 1) The __hashlock__ requires presenting the proper "preimage"
to the blockchain prior to the 2) __timelock__ expiration, else the value automatically returns to
"depositor".

This allows two parties to exchange tokens on independent platforms trustlessly and securely and thus
enables Atomic Cross-Chain Swaps (ACCS) among other useful functionalities.

Getting Started with HTLC
=========================
There are 5 steps to progress through when working with an HTLC:
1. Generate a "preimage" (depositor)
1. Calculate the "preimage_hash" (depositor)
1. Create the HTLC (depositor)
1. Verify the HTLC (recipient)
1. Redeem the HTLC (recipient)

## 0. Prerequisites 
1. The "depositor" must have an unlocked `cli_wallet` with their account (e.g `alice`) 
 - a. `alice` must have a balance of TEST tokens in her wallet
 - b. anyone may issue this command to verify: $>`list_account_balances alice` 
2. The "recipient" must have an unlocked `cli_wallet` with their account (e.g `bob`)
 - a. `bob` may have a 0 balance

**Note**: __For testing purpose, the accounts may be in the same wallet__

## 1. Generate a "preimage" (depositor)

1. `alice` must generate a "preimage" (think: password) and keep is safe from `bob` until she is 
ready to reveal it him. 
2. Using this [external website](https://passwordsgenerator.net/sha256-hash-generator/) generate a
"preimage"
3. Enter your "preimage" text into the textbox
```
Warning: Using a popular preimage is large security risk. The recipient can redeem the HTLC by comparing the preimage_hash to a dictionary of popular hashes. The best preimage is a long, random one.
```
4. Count how many characters your entered, including spaces, this is your "preimage_length" in bytes
5. Retain your "preimage" and "preimage_length" for later use
6. Example: "preimage" is `My Preimage` and "preimage_length" is `11` bytes

## 2. Calculate the "preimage_hash" (depositor)

1. `alice` must calculate the "preimage_hash" from the __preimage__ she generated above
2. Just below the textbox will be the SHA256 hash of your __preimage__ string
3. Retain this as your "preimage_hash" for later use
4. Example: "preimage_hash" is `650BFCEF53BD8E6E030613E0B75EC0CBA4FCD25C53BF0D15A6283593B269DF79`

## 3. Create the HTLC (depositor)
1. `alice` will create the HTLC using the `htlc_create` command with syntax of `htlc_create <from> <to> <amount> <symbol> <hash_algo> <preimage_hash> <preimage_length> <expiration> <broadcast>`
 - a. <from> is `alice` our depositor
 - b. <to> is `bob` our recipient
 - c. <amount> is the number of tokens to be locked into the contract (e.g. `100`)
 - d. <symbol> is the token symbol (e.g. `TEST`)
 - e. <hash_algo> will be `SHA256` (future releases may support additional hash algorithms)
 - f. <preimage_hash> is the calculated value from #2 (**not the preimage** which remains private)
 - g. <preimage_length> is the number of bytes from #1
 - h. <expiration> is the __timelock__ expressed as the number of seconds the HTLC will remain valid
  upon broadcast to the blockchain (e.g. `300` for 5 minutes)
 - i. <broadcast> is a boolean value which must be set to `true` to be sent by the `cli_wallet` software
2. `alice` will execute this command: $(unlocked)>`htlc_create alice bob 100 TEST SHA256 "650BFCEF53BD8E6E030613E0B75EC0CBA4FCD25C53BF0D15A6283593B269DF79" 11 300 true`
3. The JSON response will be returned (else an assert error - check your parameters)
4. The HTLC is now active and the __hashlock__ expiration is counting down the seconds for `bob` to verify
and redeem.
5. The 100 TEST tokens are deducted from the account balance of `alice` and are locked into the HTLC, 
protected by both the __hashlock__ and __timelock__ conditions.

## 4. Verify the HTLC (recipient)

1. `bob` must validate the HTLC created by `alice` meets his requirements. He can do this by looking 
at the recent __account history operations__ for his account to find the active HTLC.
2. `bob` will execute this command: $(unlocked)>`get_account_history bob 1`
 - a. This will return the most recent operation, which should begin with "Create HTLC to bob" 
 - b. Locate the HTLC "database_id" having the format `1.16.XX` 
   - i. "XX" will also be a numeric value unique to the HTLC `alice` created
   - ii. Retain the complete HTLC "database_id" (e.g `1.16.99`)
3. `bob` will use the "database_id" to execute this command: $(unlocked)>`get_htlc 1.16.99`
4. `bob` will verify the following transfer values meet his expectations, as agreed to previously with `alice`:
 - a. from
 - b. to
 - c. amount
 - d. asset
5. `bob` will now learn about the two conditions `alice` applied to the HTLC. 

 - Within the __hashlock__ he observes:

   - a. hash_algo
   - b. preimage_hash
   - c. preimage_length

 - Within the __timelock__ he observes:

   - d. expiration_string

6. If `bob` validates everything, he may proceed to Redeem the HTLC
7. Else, `alice` will automatically receive the value locked in the HTLC when the __timelock__ expires 

## 5. Redeem the HTLC (recipient)

1. `bob` will obtain the `preimage` from `alice`, likely after performing the agreed upon task
2. The `htlc_redeem` command has the following syntax: `htlc_redeem <htlc_database_id> <fee_paying_account> <preimage> <broadcast>`
3. `bob` will execute the following command: $(unlocked)>`htlc_redeem 1.16.99 bob "My Preimage" true`
4. The JSON response will be returned (else an assert error - check your parameters)
5. The "preimage" `bob` supplied satisfies the __hashlock__ condition on the HTLC and the `100 TEST`
are moved into his account balance (less 0.4 TEST which is paid as the transaction fee, less 0.8 TEST which is paid as the kilobyte fee for storing the "preimage").
6. Anyone may issue this command to verify: $>`list_account_balances bob`

Using Two HTLCs to Perform an Atomic Swap
=========================================

An __atomic swap__ involves a pair of HTLCs related by the same __hashlock__ within both, allowing for 
trustless exchange of distinct tokens between distinct accounts. More on how to perform this on the
TESTNET shortly.

Performing an Atomic Cross Chain Swap (ACCS)
============================================

The ultimate goal for our HTLC implementation is to allow the trustless exchange of tokens on the 
BitShares blockchain for tokens on a foreign blockchain. We will likely start with TESTBTC instructions
in the coming weeks. Please practice on our chain first. 