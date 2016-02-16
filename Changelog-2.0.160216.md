
Full list of changes for BitShares 2.0.160216

Licensing
---------

- Change to MIT license #496

New features
------------

- Implement top_n special authority #516
- Implement buyback accounts #538
- Allow asset to update permission flags when no supply exists #572
- Implement FBA fee routing for STEALTH #563

API improvements
----------------

- Implement new market API #503
- Add a cryptography API #500

Bugfixes
--------

- Handle exception in open() by re-indexing #492
- Don't update bitasset_data_object force_settled_volume every block unless needed #540
- Cap auto-cancel fees at deferred_fee #549
- Fix integer overflow bug in unit test framework when waiting for zero blocks #559
- Fix for #557: check BTC/PTS addresses on balance import including compressed/uncompressed versions
- Remove active_witnesses from global_property_object #562
- Fix stealth transfer bug #523
- Saves change address in the wallet when transfering from blind to an account #564
- Fix #586 - decoding memo for sender in CLI wallet

Blockchain stability enhancements
---------------------------------

- Take mia as reference, not copy, in clear_expired_orders(), maybe fix #485
- Expose whitelisted_accounts, fix #489
- Implement rough Python regular expression based reflection checker #562
- Fix withdraw_permission_object.hpp reflection #562
- Replace ordered_non_unique indexes with composite keys / ordered_unique, using object_id as tiebreaker.
- Reflect ID of force_settlement_object, fix #575
- Fix #492 - database corruption when closing

Code cleanup
------------
  
- Move account_options::validate() implementation from account_object.cpp #498
- Disable skip_validate #505
- Remove libraries/wallet/cache.cpp #510
- Give different object types their own individual header files #466
- Add break to every case in get_relevant_accounts #513
- Remove unused ancient implementation of operation_get_required_authorities #537
- Remove evaluation_observer #550
- Make some casts more explicit.
- Remove type_serializer, re-implement minimal functionality needed by cli_wallet #553

Build system enhancements
-------------------------

- Optionally disable database unity build #509
- Generate hardfork.hpp from hardfork.d directory #511

Support code
------------

- Improve account_balance indexing #529
- Improve vote counting implementation #533
- Defer something-for-nothing culling for taker orders until the order is unmatched #555
- Make is_authorized_asset a free-floating method #566

Logging
-------

- Log a lot of information if clear_expired_orders() is iterating too much, maybe useful to diagnose #485
