# Wallet Merchant Protocol

The purpose of this protocol is to enable a merchant to request payment from the user via a hosted wallet provider or via a browser plugin.  We will assume that the wallet is hosted at https://wallet.org and that the merchant is hosted at https://merchant.org.

To request a payment, the merchant needs to pass the following JSON object to https://wallet.org/invoice?args=...

```
{
   "to" : "merchant_account_name",
   "to_label" : "Merchant Name",
   "memo" : "Invoice #1234",
   "line_items" : [
        { "label" : "Something to Buy", "price" : "1000.00 SYMBOL" },
        { "label" : "Something to Buy", "price" : "1000.00 SYMBOL" },
        { "label" : "User Specified Price", "price" : "CUSTOM SYMBOL" },
        { "label" : "User Asset and Price", "price" : "CUSTOM" }
    ],
    "note" : "Something the merchant wants to say to the user",
    "callback" : "https://merchant.org/complete"
}
```

When the wallet receives this request it will present the user with a form that allows them to select which account they wish to pay with along with a total due.  If any CUSTOM fields are specified the user will be able to edit the line item, otherwise it will be pre-filled. 

After the user creates the transaction, signs it, and gets confirmation that the transaction has been included in the blockchain then the `callback` url will be called with `signed transaction` as a URL encoded JSON object.

https://merchant.org/complete?block_num=5,trx_num=3,trx=... 

The merchant will then check with the blockchain to see if the transaction has been included and once that confirmation is complete then the merchant is free to ship or enable downloads.  

To check to see if the transaction has been included the merchant will use the HTTP POST api:
```
POST /rpc HTTP/1.1
content-type:application/json
host: https://wallet.org
content-length:100

{  "id":1, "method": "get_transaction","params" : [5,3] }
```

If the response of this request should match the value of `trx` provided in the `https://merchant.org/complete?trx=...` callback.

The merchant should verify that all of the payment details match the expected value and be sure to protect against "replay" attacks by verifying that the invoice number included in the memo is unique and matches the expected value. 


