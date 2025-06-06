# Example Transactions

Examples of supported transactions are listed:\


* [add\_public\_key](https://nearblocks.io/txns/FBTRk6jRUSW3E1SjBfYbA71DhN5xTX1yE2foy98TafrM#execution)
* [remove\_public\_key](https://nearblocks.io/txns/69XiJcRtipZ2i8bkHHgCaSAkswd8Lt6VxiJuM8dKq9qn#execution)
* [execute\_intents](https://nearblocks.io/txns/FxpbspXjRQg3gDii18ibp3w7yvbe7MWaDHQfVqVfq7xN#enhanced) (1 withdrawal intent)

NEP-141:\


* [deposit](https://nearblocks.io/txns/DWv4AkrLxnbJV6paqSFR3Tt42SdVAtunBbEi2XATdXTW#enhanced)
* [deposit-then-execute-intents](https://nearblocks.io/txns/Bn3iC9B1uUJrX59x7cfxngY1E159HspzTCyyDVwx5DMJ) (1 withdrawal intent)
  * [deposit-swap-withdraw](https://nearblocks.io/txns/BMFcWFReAzbH8okweUio2nNTuVXtMr1hXeaaNR4UhEzS) (\~1 NEAR -> \~5 USDT)
* [withdrawal by transaction](https://nearblocks.io/txns/3E8TDSLq2Xn8JZEAdbeX8NGS7Ps6f1EwLeoFxhT7YY3G#enhanced)

Getting balances via CLI:

```
near contract call-function as-read-only \
  defuse-alpha.near mt_batch_balance_of \
  json-args '{
    "account_id": "defuse-ops.near",
    "token_ids": [
        "nep141:wrap.near",
        "nep141:usdt.tether-token.near"
    ]
  }' network-config mainnet now

--------------
[
  "3000004000004000006",
  "10"
]
--------------
```

