# Withdrawals

After a successful [deposit](deposits.md), tokens are assigned to a user's account in the `Verifier` contract. Users retain full ownership and control of their tokens. Besides executing trades through intents, the user has the liberty to withdraw their tokens whenever they want.

Given how tokens are stored in the `Verifier` smart contract (see [balances section](balances-and-identifying-your-token.md)), there are two ways to withdraw a fungible token from the contract. The same applies to non-fungible tokens, but the difference shall become clear.

1. Using the `Verifier` smart contract [function call](https://near.github.io/intents/defuse/tokens/nep141/trait.FungibleTokenWithdrawer.html#tymethod.ft_withdraw) `ft_withdraw`, where the token id is used without a prefix (see the [balances section](balances-and-identifying-your-token.md)).
2. Using a signed intent, where the token id is also prefixed with `nep141`.

### Examples

1. If Alice has sufficient balance, she can initiate the following withdrawal using `near-cli`:

```
near contract call-function as-transaction intents.near ft_withdraw json-args '{ "token": "wrap.near", "receiver_id": "alice.near", "amount": "1000" }' prepaid-gas '100.0 Tgas' attached-deposit '1 yoctoNEAR' sign-as alice.near network-config mainnet sign-with-keychain send
```

2. To withdraw using a signed intent, here’s an example for an NEP-141 token:

```json
{
  "standard": "nep413",
  "payload": {
    "message": "{\"signer_id\":\"user1.test.near\",\"deadline\":\"2025-05-20T13:29:34.360380Z\",\"intents\":[{\"intent\":\"ft_withdraw\",\"token\":\"wrap.near\",\"receiver_id\":\"alice.near\",\"amount\":\"1000\"}]}",
    "nonce": "D9JkDUi93utMQZ1hPAOV7gfnzx3fjehp6oNoCQURPe4=",
    "recipient": "defuse.test.near"
  },
  "public_key": "ed25519:Gxa24TGbJu4mqdhW3GbvLXmf4bSEyxVicrtpChDWbgga",
  "signature": "ed25519:52yFEvWpttuFiPwjJR2PdMzBrT34UCH4DX4rtuVeFBdVXt3C4ndA8Xxe7BczHuyC1ACqB24de97oW5U8YUty7omF"
}
```

### Withdrawal of other types of tokens

As with NEP-141 tokens, the same withdrawal methods apply to other token types. Any token type can be withdrawn via a signed intent or a direct function call on NEAR. The following is a table with this information summarized

| Token type/standard          | Withdraw function                                                                                                                | Withdraw Intent                                                                                        |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| NEP-141 - Fungible Token     | [ft\_withdraw](https://near.github.io/intents/defuse/tokens/nep141/trait.FungibleTokenWithdrawer.html#tymethod.ft_withdraw)      | [FtWithdraw](https://near.github.io/intents/defuse_core/intents/tokens/struct.FtWithdraw.html)         |
| NEP-171 - Non-fungible Token | [nft\_withdraw](https://near.github.io/intents/defuse/tokens/nep171/trait.NonFungibleTokenWithdrawer.html#tymethod.nft_withdraw) | [NftWithdraw](https://near.github.io/intents/defuse_core/intents/tokens/struct.NftWithdraw.html)       |
| NEP-245 - Multi Token        | [mt\_withdraw](https://near.github.io/intents/defuse/tokens/nep245/trait.MultiTokenWithdrawer.html#tymethod.mt_withdraw)         | [MtWithdraw](https://near.github.io/intents/defuse_core/intents/tokens/struct.MtWithdraw.html)         |
| Native NEAR Token            | ❌                                                                                                                                | [NativeWithdraw](https://near.github.io/intents/defuse_core/intents/tokens/struct.NativeWithdraw.html) |

## Refunds on failed withdrawals - Warning

This discussion applies to all withdraw types in the previous section, except for NativeWithdraw, since NEAR transfers cannot fail.

A refund restores the user’s balance when a withdrawal fails on the target smart contract. For example. Say Alice makes a request to withdraw 100 USDC from her `Verifier` smart contract account. There are two modes of operation for this request, whether it is done through a function call or an intent. Notice the [withdraw function and its signature](https://near.github.io/intents/defuse/tokens/nep141/trait.FungibleTokenWithdrawer.html#tymethod.ft_withdraw). The key parameter in the function for this discussion is `msg`, stemming from the [NEP-141 standard](https://nomicon.io/Standards/Tokens/FungibleToken/Core). There are two possible cases:

1. If `msg` is specified, the contract will use `ft_transfer` to process the withdrawal
2. If `msg` is not specified, or `"msg": null` in json, the withdrawal will be done using an `ft_transfer_call` function call

Note: The same applies for Multi Token and NFT transfers mentioned in the table above, with their corresponding `nft_transfer`, `nft_transfer_call`, `mt_transfer`, and `mt_transfer_call`, and the intents, which eventually call these functrions.

Ideally speaking, if Alice does not receive her tokens to the smart contract in question, i.e., if her balance in the USDC smart contract does not increase by the 100 USDC after withdrawing 100 USDC from the `Verifier` smart contract due to an error in the USDC smart contract, the `Verifier` should return the 100 USDC to her balance in the `Verifier` contract.&#x20;

However, this is not always the case. Due to complexities related to the asynchronous and sharded nature of the NEAR blockchain, refunds will only be processed when `msg` is unspecified or has a value `null`, i.e., when `ft_transfer` is used for processing the withdrawal.

It is important to emphasize that this refund discussion is exclusively relevant for failures occurring in the target smart contract, not in the `Verifier` contract itself. If an error happens within the `Verifier` contract, the balance will not change. But the refund issue happens if an asynchronous call to another smart contract, the USDC smart contract in the above example, leads to an error.

Note that `msg` being empty, i.e., being `""` or an empty string, or being `Some("")` in Rust instead of `None`, still means that `msg` is specified. In the json parameters of your request, make sure you do not specify `msg` at all or specify the value  `null` if you are seeking the behavior of `ft_transfer` and guarantee a refund on failure.

See the summary table below:

| Withdrawal request                          | \`msg\` parameter specified | Is a refund possible? |
| ------------------------------------------- | --------------------------- | --------------------- |
| `FtWithdraw` intent or `ft_withdraw` call   | No (or msg = null)          | Yes                   |
| `FtWithdraw` intent or `ft_withdraw` call   | Yes, as string              | No                    |
| `MtWithdraw` intent or `mt_withdraw` call   | No (or msg = null)          | Yes                   |
| `MtWithdraw` intent or `mt_withdraw` call   | Yes, as string              | No                    |
| `NftWithdraw` intent or `nft_withdraw` call | No (or msg = null)          | Yes                   |
| `NftWithdraw` intent or `nft_withdraw` call | Yes, as string              | No                    |

To ensure a successful withdrawal and refund, avoid specifying `msg` or set it explicitly to `null`. For programmable actions on the receiving contract, perform them from your own account using `ft_transfer_call`.
