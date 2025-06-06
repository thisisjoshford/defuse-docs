---
description: Frequently Asked Questions
---

# FAQs

## Is there a testnet deployment?

There's no testnet deployment and no plans for it. We recommend testing on NEAR mainnet with using separate dev/test NEAR accounts.&#x20;

## Is there support for native NEAR deposits?

Only  `ft_transfer_call` can be used to deposit NEP-141 tokens from Near to `intents.near` :\


```
<TOKEN_ACCOUNT_ID>::ft_transfer_call({
  "receiver_id": "intents.near",
  "amount": "1234",
  "msg": "{\"receiver_id\": \"<ACCOUNT_ID>\"}"
})
```

Here is an example [receipt](https://nearblocks.io/txns/EwmeXzZJStA6e5JB49vgxNYJDemqeYCFGvPH7zapP1Fw#execution#4tyaF4MnMcNQVqrg3kXzsH9277ErDeCXS9g3c2keV38G) for that.\
Parameter `msg` can also be empty, so that funds will be deposited to `sender_id` (i.e. caller of `ft_transfer_call`). Here is an example of such [tx](https://nearblocks.io/txns/HoWpAR8dF5azsUVaQWrBW5VsRve5X4dwr9GGiHWj3R1P#execution)

## `tx_hash` in the `recent_deposit` response for all SOL deposits is empty

This information is not available for Solana because the mechanism of deposit tracking works a bit differently there.

## Is there a reason why my UTXOs aren't being swept on the BTC ? I sent 5,000 sats

This is a very small amount that is considered to be "dust" and there is a special business logic to process such small amount.

## How does the deposit process work?

The deposit process begins once the transfer transaction on the foreign network has been completed.  When the balance of the user's unique deposit address has become positive our indexer generates a deposit event and assigns it a PENDING status.

The next step is collecting the current tokens in storage. The result of this process will be either a COMPLETED or FAILED status. Deposits with a FAILED status are currently handled manually and eventually updated to the COMPLETED status.

On EVM networks, deposits can bypass the PENDING status due to the faster processing and transfer completion times.

The data structure for the PENDING and FAILED statuses is identical to that of the COMPLETED status.

Regarding BTC deposits: If you want to make a deposit to an account that hasn’t yet been connected to the application - this is possible but requires extreme caution. You can request a deposit address by calling the bridge API (deposit\_address) and specifying the account\_id parameter. The account\_id can be a NEAR account, an EVM address, or a SOL address to which you have access.

It is recommended starting with a small amount for experimentation. After the deposit is completed, you can connect wallet and check the tokens.

## ETH-connector migration

Because of the split of ETH-connector, `aurora`contract now [acts](https://github.com/aurora-is-near/aurora-engine/blob/3416d0d170bf3dcaaac3ad3b9ba751d002ea1b3f/engine/src/contract_methods/connector/mod.rs#L209-L210) as a NEP-141 proxy to `eth.bridge.near`. This leads to some changes:

* Firstly, **ALL** new deposits will be [treated](https://nearblocks.io/txns/ExLxkbRjKbSDSTVozPpSsTfRL8Aqfs8kvLxFLEx4qDzt#enhanced) as `eth.bridge.near`, even if triggered from `aurora` smart-contract. Even from inside of Aurora Mainnet (example [tx](https://explorer.aurora.dev/tx/0x9c20d9f76443ec3c12f8eb41a65caa0c1391210c539a5924215b9bdf9e0b1fd2?tab=index))! ALL withdrawals of legacy `aurora` will be received on Near as new `eth.bridge.near`.
* Secondly, the migration of already deposited Eth@Near (`aurora`) can be done **permissionlessly** by withdrawing from `intents.near` and depositing it back right away. Luckily, it's possible to do that in a single tx thanks to [this](https://github.com/Near-One/aurora-eth-connector/blob/58d3f39cebcf6266514de3dd04efec5bafb6274e/eth-connector/src/lib.rs#L132-L143) patch on eth-connector side.

Users can migrate in two ways:

*   Via `ft_withdraw()` [tx](https://nearblocks.io/txns/GHrRbGsDuv86u72jHNqofZYTUssfuXyYwcJ8mYyLrq2v#enhanced) with following params:

    ```
    {
      "token": "aurora", // legacy token
      "receiver_id": "intents.near", // deposit back
      "amount": "1234",
      "memo": "Migrate ETH: aurora -> eth.bride.near",
      "msg": "<USER_ACCOUNT_ID>", // recipient inside intents.near for eth.bridge.near tokens
    }
    ```
* Via `ft_withdraw` intent with the _same_ params as in above ^

The [front-end](https://app.near-intents.org) automatically detects legacy tokens on user's balance and prompts him to sign such migration intent.







