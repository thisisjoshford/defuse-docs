# Step 2 - Swap

Now that we have deposited tokens into the NEAR Intents system, the next step is to swap them for another asset. This section will guide you through the process of swapping your previously deposited NEAR tokens into ETH.

{% hint style="info" %}
For a complete working example of token swaps using NEAR Intents, refer to the [GitHub repository](https://github.com/nearuaguild/near-intents-ts-example).
{% endhint %}

### Fetch a Price Quote

To estimate the potential output for a given exact input, or determine the required input for a given exact output, request a price quote from the solver relay.

Use the following endpoint:

```sh
POST https://solver-relay-v2.chaindefuser.com/rpc
```

With the following payload:

```json
{
  "id": "dontcare",
  "jsonrpc": "2.0",
  "method": "quote",
  "params": [
    {
      "defuse_asset_identifier_in": "nep141:wrap.near",
      "defuse_asset_identifier_out": "nep141:eth.omft.near",
      "exact_amount_in": "50000000000000000000000",
      "quote_id": "00000000-0000-0000-0000-000000000000",
      "min_deadline_ms": "60000"
    }
  ]
}
```

The response will look as the following:

```json
{
  "jsonrpc": "2.0",
  "id": "dontcare",
  "result": {
    "quotes": [
      {
        "quote_hash": "00000000000000000000000000000000",
        "defuse_asset_identifier_in": "nep141:wrap.near",
        "defuse_asset_identifier_out": "nep141:eth.omft.near",
        "amount_in": "50000000000000000000000",
        "amount_out": "20000000",
        "expiration_time": "1726184627"
      }
    ]
  }
}
```

```ts
const RPC_URL = "https://solver-relay-v2.chaindefuser.com/rpc";

const body = {
  id: "dontcare",
  jsonrpc: "2.0",
  method: "quote",
  params: [
    {
      defuse_asset_identifier_in: "nep141:wrap.near",
      defuse_asset_identifier_out: "nep141:eth.omft.near",
      exact_amount_in: "50000000000000000000000",
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

const result = json.result;

if (result === null) return undefined;

const quote = result.at(0);
```

If a quote isn't found, try again since solvers may not have provided information within the 1200ms window.

### Sign an Intent Message

Once a valid quote is retrieved, you need to create an intent message using the data from the quote. This involves generating a new nonce, serializing JSON message and signing it with the ED25519 key pair belonging to your NEAR account.

```ts
const standard = "nep413";
const message = {
  signer_id: "<your-account.near>",
  deadline: quote["expiration_time"],
  intents: [
    {
      intent: "token_diff",
      diff: {
        [quote["defuse_asset_identifier_in"]]: `-${quote["amount_in"]}`,
        [quote["defuse_asset_identifier_out"]]: `${quote["amount_out"]}`,
      },
    },
  ],
};

const messageStr = JSON.stringify(message);
const nonce = await generateNonce();
const recipient = "intents.near";
const intent = serializeIntent(messageStr, recipient, nonce, standard);
const { signature, publicKey } = signMessage(intent);
```

It's important to note that the contract deployed at `intents.near` will verify your signature. To do this, it must know the public key that was used to sign the intent message. You must call the `add_public_key` function to register your public key with the contract. This step is only required once before sending your first intent message.

```ts
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

const publicKey = await account.connection.signer.getPublicKey(
  account.accountId,
  "mainnet"
);

// returns boolean whether this public_key is already added
const hasPublicKey = await account.viewFunction({
  contractId: "intents.near",
  methodName: "has_public_key",
  args: {
    account_id: account.accountId,
    public_key: publicKey.toString(),
  },
});

if (hasPublicKey === true) return;

// adds public_key if it wasn't there yet
await account.functionCall({
  contractId: "intents.near",
  methodName: "add_public_key",
  args: {
    public_key: publicKey.toString(),
  },
  attachedDeposit: 1n,
});
```

### Publish Swap Intent

The final step is to craft the intent payload and publish it for execution, enabling the actual swap to take place. Use the following endpoint:

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
      "quote_hashes": [quote["quote_hash"]],
      "signed_data": {
        "standard": "nep413",
        "payload": {
          "message": "{\"signer_id\":\"<your_account.near>\",\"deadline\":\"2025-02-18T18:25:25.836Z\",\"intents\":[{\"intent\":\"token_diff\",\"diff\":{\"nep141:wrap.near\":\"-5000000000000000000000\",\"nep141:eth.omft.near\":\"92984\"}}]}", // serialized message from the previous step
          "nonce": "bM0eIaBFfs9GScUJy7K7WcHEA8mK10A2s4/+1CLq4yc=",
          "recipient": "intents.near",
        },
        "signature": "ed25519:2Eh4WwskBappyRcfTSvofq3SraznbsnFkRu49pPZXA2Ag3uPYvuHKobG3cWUVHQW3wxMvGEf8jSqjkeYppFMqji",
        "public_key": "ed25519:3BLaN6qcTXEM6NNiy9AmtoxsbWVPvSHaBNjS2ULsH1M3"
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
      quote_hashes: [quoteHash],
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
}f
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

If you ever decide to move your funds back to another chain, the next chapter will walk you through the withdrawal process.
