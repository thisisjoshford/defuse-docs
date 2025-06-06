# Account Abstraction

The `Verifier` contract currently identifies users via their NEAR `AccountId`, which can be either a Named or Implicit account.

### Named Account: `user.near`

\
The only way to start using a Named Account is to send a real tx `intents.near::add_public_key(pk)` from user's NEAR wallet. Others can still deposit or transfer funds to you before you "claim" the account.

### Implicit Account

Such accounts already have a public key encoded in their names, so they can be used without any "claiming" from users.\
There is 1-to-1 relationship between its format and corresponding signing curve types:\\

* EdDSA: `8c5cba35f5b4db9579c39175ad34a9275758eb29d4866f395ed1a5b5afcb9ffc` (i.e. "Implicit NEAR")
* ECDSA: `0x85d456B2DfF1fd8245387C0BfB64Dfb700e98Ef3` (i.e. "Implicit Eth")

For example, users logging in with a Cosmos (ECDSA) wallet will have an _Implicit Eth address_ in `intents.near`, whereas Solana or Ton (EdDSA) wallets will yield _Implicit NEAR addresses_.

It's not feasible to differentiate these addresses by chain, since we only know the signature and public key. Even when differentiating between them based on the used signing standard (NEP-413, EIP-712), it still results in ambiguity when importing the same seed phrase into multiple wallets.

## Account keys

Once an account is created, multiple additional public keys could be added to it to authorize actions on that account. Each of these keys has full control over the account and can add or remove other keys either directly via NEAR transactions or via signed intents.

Here is an example of adding public keys for Explicit NEAR Accounts via [tx](https://nearblocks.io/txns/FBTRk6jRUSW3E1SjBfYbA71DhN5xTX1yE2foy98TafrM#execution).

You can also do it manually with [near-cli](https://github.com/near/near-cli-rs):

```
near contract call-function as-transaction \
  intents.near add_public_key json-args '{
    "public_key": "ed25519:<base58>"
  }' prepaid-gas '100.0 Tgas' attached-deposit '1 yoctoNEAR' \
  sign-as <ACCOUNT_ID> network-config mainnet sign-with-keychain send
```

In addition to directly calling the `add_public_key`method it's also possible to submit a signed intent:

```
{
  "signer_id": "<ACCOUNT_ID>",
  // ...
  "intents": [
    {
      "intent": "add_public_key",
      "public_key": "<PUBLIC_KEY_OF_USER>" 
    },
  ]
}
```

## Authentication Flow example via Frontends

Users' wallets store their private keys and allow users to rotate them. In order to verify signatures inside of `Verifier`, `Verifier` should know which keys are associated with which "named" accounts. Therefore, `intents.near` contract should maintain a _copy_ of mapping of `account_ids` to their `public_keys` (again, each `account_id` can have multiple `public_keys` registered). This copy should include a subset of valid public keys that were added as Full Access Keys to each NEAR account that wants to interact with Intents.

When a user `user1` opens the Frontend for the first time and clicks "Connect Wallet", the frontend prompts the user's wallet to `signMessage("Authenticate")` . As a result, we get a signature and, more importantly, the `public_key` as a counterpart of a Full Access Key that was used to sign this message and the `account_id` which the public\_key corresponds to. The frontend would store this pair `(account_id, public_key)` in browser's local storage.

Now, when a user wants to swap tokens, i.e. sign a `state_change`:

1. First, check if this pair `(account_id, public_key)` is already registered in Intents contract on-chain by calling `get_account_public_keys(account_id) -> Vec<Pubkey>` method on `intents.near` contract.
   1. If there is no such user or if public\_key was not registered for him, the user should be "registered" in Intents contract. For that we should ask user's wallet to `signMessage("user1 is an owner of public_key ed25519:abcd...")`
   2. This signed message is than included in the transaction (by [Message Bus](../../market-makers/bus/)) to `intents.near` calling `add_public_key({"account_id": "user1", "public_key": "ed25519:abcd...", "signature": "xyz123..."})` method. This transaction would add `ed25519:abcd...` to the list of public\_keys that belongs to `user1`
2. The user's wallet should sign `signMessage({"account_id": "user1", "state_changes": [...] })` and this state\_change along with the returned signature and public\_key would be eventually sent by a market maker to the `intents.near`.
3. When `intents.near` receives transaction with such signature, it validates the signature for given `public_key` and atomically checks whether the `public_key` is registered for `user1`.

A challenge arises when a user removes his `Full Access Key`, it should also be unregistered on `intents.near` contract by calling `remove_public_key(public_key)` method sent from user's NEAR account. This can be automated by adding a `FunctionalKey` added to user's account on NEAR and call by ourselves whenever we detect that our user has deleted a key from his NEAR account on-chain.
