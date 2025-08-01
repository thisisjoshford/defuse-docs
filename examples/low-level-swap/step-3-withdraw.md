# Step 3 - Withdraw

Withdrawing allows you to transfer your tokens back to their original account, whether on NEAR or another blockchain.

By following this guide, you'll learn how to do it securely and efficiently, ensuring that your funds are returned to your intended destination.

### Build Intent Message

Withdrawing tokens is similar to swapping in that you need to craft an intent message, sign it, and send it to the RPC for execution. However, unlike swapping, fetching a quote isn’t necessary since no asset exchange is involved. Instead, the withdrawal intent message defines the amount and destination chain for the transaction.

The process differs slightly depending on whether you’re withdrawing to NEAR or another chain. Let’s break down both approaches.

#### Build for NEAR

The intent message is going to have the following structure

```json
{
  "deadline": "2025-02-18T20:11:03.0462",
  "signer_id": "<your-account.near>",
  "intents": [
    {
      "intent": "native_withdraw",
      "receiver_id": "<your-account.near>",
      "amount": "50000000000000000000000"
    }
  ]
}
```

```ts
const standard = "nep413";
const message = {
  deadline: "2025-02-18T20:11:03.0462",
  signer_id: "<your-account.near>",
  intents: [
    {
      intent: "native_withdraw",
      receiver_id: "<your-account.near>",
      amount: "50000000000000000000000",
    },
  ],
};

const messageStr = JSON.stringify(message);
const nonce = await generateNonce();
const recipient = "intents.near";
const intent = serializeIntent(messageStr, recipient, nonce, standard);
const { signature, publicKey } = signMessage(intent);
```

#### Build for another chain

The intent message for withdrawing to Ethereum Mainnet is going to have the following structure

```json
{
  "deadline": "2025-02-18T20:11:03.0462",
  "signer_id": "<your-account.near>",
  "intents": [
    {
      "intent": "ft_withdraw",
      "token": "eth.omft.near",
      "receiver_id": "eth.omft.near",
      "amount": "2000000000",
      "memo": "WITHDRAW_TO:0x311F9620Be0fe8Db2d840E2b6145D1CF2975acab" // WITHDRAW_TO:ADDRESS_ON_DESTINATION_CHAIN
    }
  ]
}
```

```ts
const standard = "nep413";
const message = {
  deadline: "2025-02-18T20:11:03.0462",
  signer_id: "<your-account.near>",
  intents: [
    {
      intent: "ft_withdraw",
      token: "eth.omft.near",
      receiver_id: "eth.omft.near",
      amount: "2000000000",
      memo: "WITHDRAW_TO:0x311F9620Be0fe8Db2d840E2b6145D1CF2975acab", // WITHDRAW_TO:ADDRESS_ON_DESTINATION_CHAIN
    },
  ],
};

const messageStr = JSON.stringify(message);
const nonce = await generateNonce();
const recipient = "intents.near";
const intent = serializeIntent(messageStr, recipient, nonce, standard);
const { signature, publicKey } = signMessage(intent);
```

### Publish Withdraw Intent

The final step is to craft the intent payload and publish it for execution, enabling the actual withdraw to take place. Use the following endpoint:

```sh
POST https://solver-relay-v2.chaindefuser.com/rpc
```

With the following payload:

```json
{
  "id": "dontcare",
  "jsonrpc": "2.0",
  "method": "publish_intent",
  "params": [
    {
      "quote_hashes": [], // this array is empty as we're withdrawing
      "signed_data": {
        "standard": "nep413",
        "payload": {
          "message": "{\"deadline\":\"2025-02-18T20:11:03.0462\",\"intents\":[{\"intent\":\"native_withdraw\",\"receiver_id\":\"<your-account.near>\",\"amount\":\"50000000000000000000000\"}],\"signer_id\":\"<your-account.near>\"}", // serialized message from the previous step
          "nonce": "/oZRZR0WGWn73s/evEUOxFH3YHSl5JvJH88m8pp3mMI=",
          "recipient": "intents.near"
        },
        "signature": "ed25519:6f8e5hsBkCu9vt3iPuoLrmgz4GCthFyq9MEKy5xp6FUedQwrtuxq3y81ymfspsNSaTyLMeUQobnXrVvUNqEXGRDH",
        "public_key": "ed25519:14fZjXn7DmM2XrAxZWK7XE6EnZugAusMtkKM9Zbb1P3N"
      }
    }
  ]
}
```

```ts
const RPC_URL = "https://solver-relay-v2.chaindefuser.com/rpc";

const signedData = {
  standard,
  payload: {
    message: messageStr,
    nonce,
    recipient,
  },
  signature: `ed25519:${bs58.encode(signature)}`,
  public_key: publicKey.toString(),
};

const body = {
  id: "dontcare",
  jsonrpc: "2.0",
  method: "publish_intent",
  params: [
    {
      quote_hashes: [], // this array is empty as we're withdrawing
      signed_data: signedData,
    },
  ],
};

const response = await fetch(RPC_URL, {
  method: "POST",
  body: JSON.stringify(body),
  headers: {
    "Content-Type": "application/json",
  },
});

const json: any = await response.json();

if (!response.ok) {
  throw new Error(
    `Request failed ${response.status} ${
      response.statusText
    } - ${JSON.stringify(json)}`
  );
}

const intent = json.result || undefined;
```

### Fetch Intent Status

The previous method returns an `intent_hash`, which can be used to fetch the status of the intent, whether it has been successfully resolved or failed for some reason. Use the following endpoint.

```sh
POST https://solver-relay-v2.chaindefuser.com/rpc
```

With the request payload:

```json
{
  "id": "dontcare",
  "jsonrpc": "2.0",
  "method": "get_status",
  "params": [
    {
      "intent_hash": "AKGznsj9tJPzyC53iweg7bjuVterJm18cGPkMKVPMKZG"
    }
  ]
}
```

The response will look like the following, `SETTLED` status stands up for an intent successfully completed:

```json
{
  "id": "dontcare",
  "jsonrpc": "2.0",
  "result": {
    "intent_hash": "AKGznsj9tJPzyC53iweg7bjuVterJm18cGPkMKVPMKZG",
    "status": "SETTLED",
    "data": { "hash": "3mWZrPWcMWwqvmxmKyCaVcEEHAtSoRQXds5MR3J1P1YU" }
  }
}
```

```ts
const RPC_URL = "https://solver-relay-v2.chaindefuser.com/rpc";

const body = {
  id: "dontcare",
  jsonrpc: "2.0",
  method: "get_status",
  params: [
    {
      intent_hash: intent["intent_hash"],
    },
  ],
};

const response = await fetch(RPC_URL, {
  method: "POST",
  body: JSON.stringify(body),
  headers: {
    "Content-Type": "application/json",
  },
});

const json: any = await response.json();

const intentStatus = json.result || undefined;
```

### Final Thoughts

With that, we’ve successfully completed the full journey of using NEAR Intents — from depositing tokens, executing swaps, and if needed withdrawing funds back to NEAR or another blockchain. By following this process, you now have a solid understanding of how to manage cross-chain assets efficiently.
