# Deposits

### Depositing fungible tokens (NEP-141), including wrapped NEAR

The Verifier contract implements the [FungibleTokenReceiver](https://docs.near.org/primitives/ft) interface, part of the [NEP-141 standard](https://nomicon.io/Standards/Tokens/FungibleToken/Core).&#x20;

To deposit tokens into the Verifier contract, users must call the function `ft_transfer_call` (see the function signature [here](https://github.com/near/near-sdk-rs/blob/611e01ebf6c226f4e1e820a2f50f4a9acf8f1215/examples/fungible-token/ft/src/lib.rs#L103)) on the token, with an `msg` parameter specifying information about the user, whose account will be assigned as the owner of the tokens.

#### Format of `msg` parameter to assign ownership of tokens inside the Verifier contract

The `msg` parameter must follow a specific format to ensure proper token deposit within the contract. The `msg` parameter supports three formats:

1. It can be empty, with size 0 bytes, which will assign ownership to the sender of the transaction (i.e., the predecessor in NEAR terminology)
2. A simple string that contains an account id, for example `alice.near`
3. A JSON object, referred to as `DepositMessage`, with the following fields:
   1. `receiver_id`, a string with an account id taking ownership
   2. `execute_intents`, a list of intents to be executed, right after the deposit is complete.
   3. `refund_if_fails`, a boolean, when `false` (the default), will execute the intents as a detached promise, decoupling any failure during the execution of the intents from the deposit operation.

Examples of `ft_transfer_call` parametrization in JSON:

1. With empty `msg` or non-existing `msg`, granting the tokens to the sender/creator of that transaction

```json
{
    receiver_id: "alice.near",
    amount: "1000",
}
```

```json
{
    receiver_id: "alice.near",
    amount: "1000",
    msg: "",
}
```

2. With an `msg`  that explicitly assigns ownership to another account:

```json
{
    receiver_id: "alice.near",
    amount: "1000",
    msg: "bob.near",
}
```

3. With an `msg` that is a `DepositMessage` object

```json
{
    receiver_id: "alice.near",
    amount: "1000",
    msg: "{"\"receiver_id\": \"charlie.near\", \"execute_intents\": [...], \"refund_if_fails\": false}\"",
}
```

or

```json
{
    receiver_id: "alice.near",
    amount: "1000",
    msg: "{"\"receiver_id\": \"charlie.near\", \"execute_intents\": [...]\"",
}
```

Note: msg is always a string—even when it contains a JSON-encoded object. Proper character escaping is required.

### Depositing non-fungible tokens (NEP-171)

The Verifier contract implements the [NonFungibleTokenReceiver](https://docs.near.org/primitives/nft) interface. This is [NEP-171](https://nomicon.io/Standards/Tokens/NonFungibleToken/Core) standard.

To transfer NFTs to the `Verifier`, use `nft_transfer_call`, following the same `msg` format rules as for fungible tokens. The `msg` parameter dictates how ownership of the transferred tokens will be assigned.
