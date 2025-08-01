# Step 1 - Deposit

Depositing tokens is the first step in using NEAR Intents. Whether you are transferring assets from NEAR or an external blockchain, this process ensures your funds are accessible within the NEAR ecosystem. The deposit mechanism depends on whether the source chain is NEAR or another blockchain.

Before making any deposits, it’s essential to check your current token balance. This ensures that your funds are appropriately accounted for after each transaction.

### Retrieving Available Tokens

To check your balance, you first need to obtain the correct `token_id` that corresponds to the asset you are querying. This information can be retrieved by calling the following endpoint:

```sh
https://api-mng-console.chaindefuser.com/api/tokens
```

Try it out with this cURL command:

```
curl -s "https://api-mng-console.chaindefuser.com/api/tokens" | jq '.'
```

This will return a JSON response containing token information. An example response is:

```json
{
  "tokens": [
    {
      "defuse_asset_id": "nep141:wrap.near",
      "decimals": 24,
      "blockchain": "near",
      "symbol": "wNEAR",
      "price": 3.36,
      "price_updated_at": "2025-02-15T12:06:00.106Z",
      "contract_address": "wrap.near"
    },
    {
      "defuse_asset_id": "nep141:eth.omft.near",
      "decimals": 18,
      "blockchain": "eth",
      "symbol": "ETH",
      "price": 2671.25,
      "price_updated_at": "2025-02-15T12:06:00.106Z"
    }
  ]
}
```

Each token has a unique `defuse_asset_id`, which serves as the `token_id` required for querying balances.

### Checking Balance on NEAR

Once you have obtained the `defuse_asset_id` for the token, you can retrieve your balance by calling the `mt_balance_of` view function on the `intents.near` contract. The request payload should be structured as follows:

```json
{
  "account_id": "your-account.near",
  "token_ids": "nep141:wrap.near"
}
```

In this example, `nep141:wrap.near` represents NEAR (as wNEAR FungibleToken) and `nep141:eth.omft.near` represents Ethereum

```sh
near contract call-function as-read-only intents.near mt_balance_of json-args '{ "account_id": "your-account.near", "token_id": "nep141:wrap.near" }' network-config mainnet now
```

```ts
import { JsonRpcProvider } from "near-api-js/lib/providers";
import { Connection, InMemorySigner } from "near-api-js";
import { InMemoryKeyStore } from "near-api-js/lib/key_stores";
import { viewFunction } from "@near-js/accounts/lib/utils";

const provider = new JsonRpcProvider({
  url: "https://rpc.mainnet.near.org",
});

// signer without key as we're only reading from contract
const keyStore = new InMemoryKeyStore();
const signer = new InMemorySigner(keyStore);

const connection = new Connection("mainnet", provider, signer, "");

const args = {
  account_id: "your-account.near",
  token_ids: ["nep141:wrap.near", "nep141:eth.omft.near"],
};

const balances = await viewFunction(connection, {
  contractId: "intents.near",
  methodName: "mt_batch_balance_of",
  args: args,
});

console.log(balances);
```

For the first interaction with NEAR Intents, the response will look like the following, meaning we're empty on NEAR and ETH:

```json
▹▸▹▹▹ Getting a response to a read-only function call ...                                                                                                                                        Function execution return value (printed to stdout):
"0"
```

### Depositing from NEAR

If the source chain is NEAR, deposits are made by directly transferring tokens using the standard Fungible Token contract. For example, to deposit NEAR tokens, you must send a signed transaction to `wrap.near` (wNEAR) with the following payload

```json
{
  "receiver_id": "intents.near",
  "amount": "50000000000000000000000", // equivalent of 0.05 NEAR as yoctoNear
  "msg": ""
}
```

```sh
near transaction construct-transaction your-account.near wrap.near add-action function-call near_deposit json-args {} prepaid-gas '30.0 Tgas' attached-deposit '0.05 NEAR' add-action function-call ft_transfer_call json-args '{
  "receiver_id": "intents.near",
  "amount": "50000000000000000000000",
  "msg": ""
}' prepaid-gas '50.0 Tgas' attached-deposit '1 yoctoNEAR' skip network-config mainnet sign-with-legacy-keychain send
```

```ts
import { JsonRpcProvider } from "near-api-js/lib/providers";
import { Account, Connection, InMemorySigner, KeyPair } from "near-api-js";
import { InMemoryKeyStore } from "near-api-js/lib/key_stores";
import { Action, functionCall } from "near-api-js/lib/transaction";

const provider = new JsonRpcProvider({
  url: "https://rpc.mainnet.near.org",
});

const privateKey =
  "ed25519:1bF8wCjpstE3j5kMS7RASEWz2gw4xj7mTPFsUAUxMC5gqMPKWaTsG2883wW5F5dZ1KUJjmGyaCMw2ym1yRtRSXyn";
const accountId = "<your-account.near>";

const keyPair = KeyPair.fromString(privateKey);
const signer = await InMemorySigner.fromKeyPair("mainnet", accountId, keyPair);

const connection = new Connection("mainnet", provider, signer, "");
const account = new Account(connection, accountId);

const actions: Action[] = [
  functionCall(
    "near_deposit",
    {},
    BigInt(30_000_000_000_000), // 30 TGas
    BigInt(50_000_000_000_000_000_000_000) // 0.05 NEAR
  ),
  functionCall(
    "ft_transfer_call",
    {
      receiver_id: "intents.near",
      amount: "50000000000000000000000", // equivalent of 0.05 NEAR as yoctoNear
      msg: "",
    },
    BigInt(50_000_000_000_000), // 50 TGas
    BigInt(1) // 1 yoctoNear
  ),
];
const result = await account.signAndSendTransaction({
  receiverId: "wrap.near", // wNEAR contract
  actions: actions,
});

console.log(result);
```

### Depositing from Different Chains

Deposits and withdrawals from other chains into NEAR FTs are handled by the bridging service - Omni-Bridge.

Since deposits involve the OmniBridge, ensure that the asset you are transferring is supported. To retrieve the full list of supported assets, use the following endpoint:

```sh
POST https://bridge.chaindefuser.com/rpc
```

With the following request JSON payload:

```json
{
  "id": "dontcare",
  "jsonrpc": "2.0",
  "method": "supported_tokens",
  "params": [
    {
      "chains": [
        "eth:1", // Ethereum Mainnet
        "eth:42161", // Arbitrum Mainnet
        "solana:mainnet" // Solana Mainnet
      ]
    }
  ]
}
```

This will return a JSON response containing a list of supported tokens. An example response is:

```json
{
  "id": "dontcare",
  "jsonrpc": "2.0",
  "result": {
    "tokens": [
      {
        "defuse_asset_identifier": "eth:1:native", // CHAIN_TYPE:CHAIN_ID:ADDRESS
        "near_token_id": "eth.omft.near",
        "decimals": 18,
        "asset_name": "ETH",
        "min_deposit_amount": "1",
        "min_withdrawal_amount": "400000000000000",
        "withdrawal_fee": "0"
      },
      ...
    ]
  }
}
```

If the desired asset is in the list, then retrieve a deposit address on the source chain using:

```sh
POST https://bridge.chaindefuser.com/rpc
```

With the following request payload:

```json
{
  "jsonrpc": "2.0",
  "id": "dontcare",
  "method": "deposit_address",
  "params": [
    {
      "account_id": "<your-account.near>",
      "chain": "eth:1" // CHAIN_TYPE:CHAIN_ID
    }
  ]
}
```

This will return a JSON response containing an address on the source chain (in this example it's Ethereum Mainnet). An example response is:

```json
{
  "jsonrpc": "2.0",
  "id": "dontcare",
  "result": {
    "address": "0xF6972DF0809aE0c8AD0bd468d385eE417561F1cB",
    "chain": "eth:1" // CHAIN_TYPE:CHAIN_ID
  }
}
```

Once you receive the deposit address, send the desired asset from the source chain using your wallet (e.g., MetaMask for Ethereum, Phantom for Solana, etc.).

:::warning

Keep in mind that the received `address` is unique for each `account_id`. **DO NOT** transfer any tokens to the address shown in this example, as you will lose those funds.

:::

After successfully depositing your tokens, query your balance again using the same method described above to confirm the deposit was successful. This time, the response should return non-zero values.

The next step is to swap them for another asset. Continue to the next chapter to learn how.
