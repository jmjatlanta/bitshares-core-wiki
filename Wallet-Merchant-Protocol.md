# Wallet Merchant Protocol

The purpose of this protocol is to enable a merchant to request payment from the user via a hosted wallet provider or via a browser plugin.  We will assume that the wallet is hosted at https://wallet.org and that the merchant is hosted at https://merchant.org.

To request a payment, the merchant needs to pass the following JSON object to https://wallet.org/invoice?args=...

```
{

}
```