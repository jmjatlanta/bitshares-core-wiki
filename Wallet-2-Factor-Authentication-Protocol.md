## Two Factor Authentication Protocol

Two factor authentication is critical for maximizing security and ease of use.  In the case of cryptocurrencies two-factor authentication means gathering multiple signatures on a transaction to authorize a transfer.  The process of gathering these signatures requires passing around a transaction digest to multiple parties who will automatically provide their signature after verifying your identity by some means. 

The purpose of this protocol is to define a standard way for 3rd parties to provide automatic 2-factor authentication services with any standard Graphene wallet.  Users of a Graphene wallet can easily add any number of 2-factor authentication providers to their account and the Graphene wallet will use this protocol to gather the required signatures for the transaction.

For the purpose of this document `https://secondfactor.org` will be the example service provider.

## Step 1 - User Registration

The first thing a user must do is identify themselves with `secondfactor.org` by some means.  This may be as simple as verifying an email address, registering a phone number, or ordering a keyfob. 

## Step 2 - Create Graphene Account for secondfactor.org

A Graphene account is how `secondfactor.org` authenticates itself with a Graphene blockchain. We will assume `secondfactor.org` has registered the Graphene account name `sfactor`.  When a user wishes to add a secondfactor.org authentication to their own account, they will update their [account permissions](https://bitshares.org/technology/dynamic-account-permissions/) to require the approval of `sfactor` in addition to the approval of their own keys to authorize a transaction.

## Step 3 - Register as a 2-factor Provider

There are two ways to register as a 2-factor provider:

1. Ask the user to add https://secondfactor.org as the auth provider for the account `sfactor`
2. Ask the wallet service provide to automatically add the setting for all wallets

## Step 4 - Implement the 2-factor API 

The most basic 2-factor API assumes the provider.



reates an account with `secondfactor.org`

