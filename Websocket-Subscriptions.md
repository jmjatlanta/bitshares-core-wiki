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