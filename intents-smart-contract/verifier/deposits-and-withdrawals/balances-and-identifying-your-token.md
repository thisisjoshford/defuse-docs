# Balances, and identifying your token

This section explains how to check your token balance and how tokens are organized within the Verifier smart contract.

### Managing Tokens via the Multi Token Standard (NEP-245)

After a successful deposit, the `Verifier` contract manages tokens using the [Multi Token Standard (NEP-245)](https://nomicon.io/Standards/Tokens/MultiToken/Core). This enables uniform handling of all supported token types.

### Token IDs in the Verifier contract

Each token is identified by a string-based token ID, prefixed with its standard type (e.g., `nep141`, `nep171`, or `nep245`).

Examples:

* Wrapped NEAR is an NEP-141 fungible token with id `wrap.near`. In the Verifier contract, it's addressed as `nep141:wrap.near`.
* An NFT with issuer/collection contract id `coolnfts.near` and token id `rock.near` is addressed in the Verifier contract with `nep171:coolnfts.near:rock.near`.
* A generic token that uses the NEP-245 standard, originating from contract `mygame.near` and token id `shield.near` is addressed in the Verifier contract with `nep245:mygame.near:shield.near`&#x20;

### Checking your balance

After a successful deposit of any token, for example `wrap.near`, your balance can be checked [using the function](https://near.github.io/intents/defuse_nep245/trait.MultiTokenCore.html#tymethod.mt_balance_of) `mt_balance_of`, which adheres to the NEP-245 standard. The following are the parameters of this function:

```json
{
    account_id: "alice.near",
    token_id: "nep141:wrap.near"
}
```

Using `near-cli`, this query can be done using the command:

```
near contract call-function as-read-only intents.near mt_balance_of json-args '{ "account_id": "alice.near", "token_id": "nep141:wrap.near" }' network-config mainnet now
```

