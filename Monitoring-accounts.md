General Introduction
====================
In contrast to most cryptocurrency wallets, the BitShares 2.0 has a different
model to represent the blockchain, its transactions and accounts. This chapter
wants to given an introduction to the concepts of *objects* as they are used by
the BitShares 2.0 client. Furthermore, we will briefly introduce the API and
show how to subscribe to object changes (such as new blocks or incoming
deposits). Afterwards, we will show how exchange may monitor their accounts and
credit incoming funds to their corresponding users.

Objects
-------
On the BitShares blockchains there are no addresses, but objects identified by a
unique *id*, an *type* and a *space* in the form:

    space.type.id

Some examples:

    1.2.15   # blockchain/protocol space . account . nr-15
    1.6.105  # blockchain/protocol space . witness . nr-105
    1.14.7   # blockchain/protocol space . worker . nr-7

    2.1.0    # wallet space . dynamic global properties
    2.3.8    # wallet space . asset . nr-8

A programmatic description of all fields can be found in the
(sources)[https://github.com/cryptonomex/graphene/blob/master/libraries/chain/include/graphene/chain/protocol/types.hpp].

Accounts
--------
The BitShares blockchain users are requires to register each account with a
unique username and a public key on the blockchain. The blockchain assigns an
incremental user *id* and offers to resolve the name-to-id pair. For instance
`1.2.15`

    2.6.80    # wallet space . account-balance . nr-80
    2.7.80    # wallet space . account-statistics . nr-80
    2.10.80   # wallet space . account-transactions . nr-80
    2.8.80    # wallet space . transactions . nr-80
    2.9.80    # wallet space . block-summary . nr-80

Wallet & Full node
------------------
We need a client to connect to. Either we run a witness in (monitor mode) or use
a public trusted witness. For exchanges, it is though recommended to run a
full node. We can connect to the network via a seed node:

    programs/witness_node/witness_node -s 104.200.28.117:61705 --rpc-endpoint 127.0.0.1:8090  # FIXME?

This opens up a node that we can connect to via the inluded wallet

    programs/cli_wallet/cli_wallet -s ws://127.0.0.1:8090 -H 127.0.0.1:8091

which will open port `8091` for HTTP-RPC requests *and* has the capabilities to
handle accounts while the witness_node can only answer queries to the
blockchain.

API
---
We provide several different API's. Each API has its own ID. When running
`witness_node`, initially two API's are available:

* API 0: provides read-only access to the database
* API 1 is used to login and gain access to additional, restricted API's.

Here is an example using wscat package from npm for websockets:

    $ npm install -g wscat
    $ wscat -c ws://127.0.0.1:8090
    > {"id":1, "method":"call", "params":[0,"get_accounts",[["1.2.0"]]]}
    < {"id":1,"result":[{"id":"1.2.0","annotations":[],"membership_expiration_date":"1969-12-31T23:59:59","registrar":"1.2.0","referrer":"1.2.0","lifetime_referrer":"1.2.0","network_fee_percentage":2000,"lifetime_referrer_fee_percentage":8000,"referrer_rewards_percentage":0,"name":"committee-account","owner":{"weight_threshold":1,"account_auths":[],"key_auths":[],"address_auths":[]},"active":{"weight_threshold":6,"account_auths":[["1.2.5",1],["1.2.6",1],["1.2.7",1],["1.2.8",1],["1.2.9",1],["1.2.10",1],["1.2.11",1],["1.2.12",1],["1.2.13",1],["1.2.14",1]],"key_auths":[],"address_auths":[]},"options":{"memo_key":"GPH1111111111111111111111111111111114T1Anm","voting_account":"1.2.0","num_witness":0,"num_committee":0,"votes":[],"extensions":[]},"statistics":"2.7.0","whitelisting_accounts":[],"blacklisting_accounts":[]}]}

We can do the same thing using an HTTP client such as curl for API's which do
not require login or other session state:

    $ curl --data '{"jsonrpc": "2.0", "method": "call", "params": [0, "get_accounts", [["1.2.0"]]], "id": 1}' http://127.0.0.1:8090/rpc
    {"id":1,"result":[{"id":"1.2.0","annotations":[],"membership_expiration_date":"1969-12-31T23:59:59","registrar":"1.2.0","referrer":"1.2.0","lifetime_referrer":"1.2.0","network_fee_percentage":2000,"lifetime_referrer_fee_percentage":8000,"referrer_rewards_percentage":0,"name":"committee-account","owner":{"weight_threshold":1,"account_auths":[],"key_auths":[],"address_auths":[]},"active":{"weight_threshold":6,"account_auths":[["1.2.5",1],["1.2.6",1],["1.2.7",1],["1.2.8",1],["1.2.9",1],["1.2.10",1],["1.2.11",1],["1.2.12",1],["1.2.13",1],["1.2.14",1]],"key_auths":[],"address_auths":[]},"options":{"memo_key":"GPH1111111111111111111111111111111114T1Anm","voting_account":"1.2.0","num_witness":0,"num_committee":0,"votes":[],"extensions":[]},"statistics":"2.7.0","whitelisting_accounts":[],"blacklisting_accounts":[]}]}

API 0 is accessible using regular JSON-RPC:

    $ curl --data '{"jsonrpc": "2.0", "method": "get_accounts", "params": [["1.2.0"]], "id": 1}' http://127.0.0.1:8090/rpc

Since restricted APIs requires login, they is **only** accessible over the
websocket RPC. However, to monitor incoming deposits, we only need API 0.

Subsciptions
------------
All objects can be subscribed to in order to get notified by the wallet via
websocket whenever the object changes. You can subscribe to objects with the
following RPC:

    > {"id":4,"method":"call","params":[DATABASE_API_ID,"subscribe_to_objects",[WALLET_DEFINED_CALLBACK_ID,["2.1.0"]]]}

This call above will register a callback `WALLET_DEFINED_CALLBACK_ID` that will
be called every time object "2.1.0" changes. `WALLET_DEFINED_CALLBACK_ID` is a
number assigned by the wallet. Object 2.1.0 is the dynamic global properties
which will change every time a new block is produced. `DATABASE_API_ID` is the
value returned by the following call and may be different on different runs.

    > {"id":2,"method":"call","params":[0,"database",[]]}
    < {"id":2,"result":1}

The `result` will be your `DATABASE_API_ID`!

After calling `subscribe_to_objects` the wallet will start to receive notices
every time the object changes:

    < {
        "method": "notice"
        "params": [
            WALLET_DEFINED_CALLBACK_ID, 
            [
                {
                    "current_witness": "1.7.5", 
                    "head_block_number": 5, 
                    "random": "2033120557c36e278db2eaad818494f791ff4d7b0418858a7ab9b5a8", 
                    "head_block_id": "00000005171f82f1b6bd948e7d58d95e572001fd", 
                    "next_maintenance_time": "2015-05-02T00:00:00", 
                    "time": "2015-05-01T13:05:50", 
                    "id": "2.1.0"
                }
            ]
        ], 
    }

Here is an example of a full session:

    > {"id":1,"method":"call","params":[0,"login",["",""]]}
    < {"id":1,"result":true}
    > {"id":2,"method":"call","params":[0,"database",[]]}
    < {"id":2,"result":1}
    > {"id":3,"method":"call","params":[0,"network",[]]}
    < {"id":3,"result":2}
    > {"id":4,"method":"call","params":[1,"subscribe_to_objects",[0,["2.1.0"]]]}
    < {"id":4,"result":true}
    < {"method":"notice","params":[0,[{"id":"2.1.0","random":"2033120557c36e278db2eaad818494f791ff4d7b0418858a7ab9b5a8","head_block_number":5,"head_block_id":"00000005171f82f1b6bd948e7d58d95e572001fd","time":"2015-05-01T13:05:50","current_witness":"1.7.5","next_maintenance_time":"2015-05-02T00:00:00"}]]}
    < {"method":"notice","params":[0,[{"id":"2.1.0","random":"9d5ff7e453db4815005eb42ddd040e3afb459950f75f4440deb3dec0","head_block_number":6,"head_block_id":"000000060e3369d6feaf330ea9114cd855c93aab","time":"2015-05-01T13:05:55","current_witness":"1.7.3","next_maintenance_time":"2015-05-02T00:00:00"}]]}
    < {"method":"notice","params":[0,[{"id":"2.1.0","random":"cb8686582c40634a0c0834d0f2c4ad19f8ca80598cc3eee2b93c124d","head_block_number":7,"head_block_id":"000000071d0bc8db55d7da75d1d880818d1930fd","time":"2015-05-01T13:06:00","current_witness":"1.7.0","next_maintenance_time":"2015-05-02T00:00:00"}]]}

To monitor accounts, we recommend to use `get_full_accounts` in order to fetch
the current state of an account and *automatically* subscribe to future account
updates including balance update.

A notification after a transaction would take the form:

    [[
        {
          "owner": "1.2.3184", 
          "balance": 1699918247, 
          "id": "2.5.3", 
          "asset_type": "1.3.0"
        }, 
        {
          "most_recent_op": "2.9.74", 
          "pending_vested_fees": 6269529, 
          "total_core_in_orders": 0, 
          "pending_fees": 0, 
          "owner": "1.2.3184", 
          "id": "2.6.3184", 
          "lifetime_fees_paid": 50156232
        }
    ]]

Note that *two* notifications will be sent, one when the transaction is pending,
and another one when the transaction was included into a block.

Once we receive a notification we can either pull the full transaction via the
`most_recent_op` object from the blockchain (the witness node) or get the
transaction history from the wallet (the cli_wallet).

A code example (Python) will be given shortly.
