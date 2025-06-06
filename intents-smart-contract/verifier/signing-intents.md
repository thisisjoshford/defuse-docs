# Signing Intents

After creating intents, they must be signed before submission to the `Verifier` contract via the `execute_intents` function. This section explains the signing process.

### Account abstraction

As discussed in the [account abstraction section](account-abstraction.md), accounts in the `Verifier` contract are identified by their NEAR account (whether implicit, being derived from a public key, or named, like `alice.near`). Every account can add an arbitrary number of public keys. Every public key in a user's account can be used to produce signatures that authorize intents for that user. Proper key management is essential.

### Signing philosophy and where our choice of algorithms come from

Digital signature algorithms vary significantly in design and application. Key differences include:

* **Key Generation:** The type of the public/private key generation mechanisms, like RSA vs elliptic curves
* **Curve types:** The type of curve within the elliptic curve, being a NIST curve, like secp256k1 from Bitcoin and Ethereum, or some Montgomery curve, like Ed25519
* **Message (payload) construction for verification:** Given a payload and a specific algorithm, how do we construct the message so that anyone can verify it with the public key?

Given that the goal with NEAR Intents, with any choice from the above, is to make it as easy as possible to integrate other wallets and services, we have allowed [multiple choices](https://near.github.io/intents/defuse_core/payload/multi/enum.MultiPayload.html) on how to verify payloads. Each signature type corresponds to a wallet or service capable of verifying it.

For example, from the [available options](https://near.github.io/intents/defuse_core/payload/multi/enum.MultiPayload.html), you see ERC-191. This exists because Ethereum wallets, like Metamask, support the [ERC-191 standard](https://eips.ethereum.org/EIPS/eip-191) for off-chain data signing.

In the following subsections, we will discuss more available signature types.

## Signature types

A signed [Transfer](https://near.github.io/intents/defuse_core/intents/tokens/struct.Transfer.html) intent that is ready to be submitted to the blockchain looks something like this:

```
{
  "standard": "nep413",
  "payload": {
    "message": "{\"signer_id\":\"alice.near\",\"deadline\":\"2025-05-21T10:34:04.254392Z\",\"intents\":[{\"intent\":\"transfer\",\"receiver_id\":\"bob.near\",\"tokens\":{\"nep141:usdc.near\":\"10\"}}]}",
    "nonce": "Op47m39Q/NzWWi8jYe4umk96OTSnY4Ao0FB/B9aPB98=",
    "recipient": "intents.near"
  },
  "public_key": "ed25519:Gxa24TGbJu4mqdhW3GbvLXmf4bSEyxVicrtpChDWbgga",
  "signature": "ed25519:52oc2FD4rMsAPNSBSx6eNYrF4atreXTZxWFhAPfmZFn1eF7jbE3BrRTL3ey1M1sAKSdK8qriZiHQnhnNBCh8vVMJ"
}
```

This object conforms to the [MultiPayload](https://near.github.io/intents/defuse_core/payload/multi/enum.MultiPayload.html) enum. A signed intent can be one of the possibilities provided by this enum.&#x20;

### NEP-413

The object above follows the NEP-413 [compliant standard](https://github.com/near/NEPs/blob/master/neps/nep-0413.md). This is an off-chain message signing standard that is recognized by NEAR wallets.

#### Encoding information

Nonce: base64\
public\_key: prefixed with the key type, then base58\
signature: prefixed with the key type, then base58

### ERC-191

The object of this type is compliant with the [ERC-191 standard](https://eips.ethereum.org/EIPS/eip-191) for off-chain message signing.

```
{
  "standard": "erc191"
  "payload": "{\"signer_id\": \"0xccaa162a73e6e9dcfdd42c9d97c3b515c1cd34c3\", \"verifying_contract\": \"intents.near\", \"deadline\": \"2025-05-26T13:24:16.983Z\", \"nonce\": \"U3UMmW79FqTMtBx3DYLI2DUxxwAFY+Eo4kY11PEI3PU=\", \"intents\": [{ \"intent\": \"token_diff\", \"diff\": { \"nep141:usdc.near\": \"-1000\", \"nep141:usdt.near\": \"1000\" } }, { \"intent\": \"ft_withdraw\", \"token\": \"usdt.near\", \"receiver_id\": \"bob.near\", \"amount\": \"1000\" }]}",
  "signature": "secp256k1:4jpo5EuztCFUe3gVqWpmwowoorFUmt4ynu3Z8WPo9zw2BSoHB279PZtDz934L1uCi6VfgXYJdTEfRaxyM3a1zaUw1",
}
```

#### Encoding information

signature: prefixed with the key type, then base58

Note that there is no public key, as the public key can be recovered from the signature and the data.

### Raw Ed25519

This is a standard used by [Phantom wallet for Solana off-chain message signing](https://docs.phantom.com/solana/signing-a-message).&#x20;

```
{
    "standard":"raw_ed25519",
    "payload":"{\"signer_id\":\"alice.near\",\"verifying_contract\":\"intents.near\",\"deadline\":{\"timestamp\":1732035219},\"nonce\":\"XVoKfmScb3G+XqH9ke/fSlJ/3xO59sNhCxhpG821BH8=\",\"intents\":[{\"intent\":\"token_diff\",\"diff\":{\"nep141:usdc.near\":\"-1000\",\"nep141:usdt.near\":\"998\"}}]}",
    "public_key":"ed25519:8rVvtHWFr8hasdQGGD5WiQBTyr4iH2ruEPPVfj491RPN",
    "signature":"ed25519:3vtbNQJHZfuV1s5DykzyjkbNLc583hnkrhTz57eDhd966iqzkor6Twgr4Loh2C195SCSEsiGfrd6KcxpjNq9ZbVj"
}
```

#### Encoding information

public\_key: prefixed with the key type, then base58\
signature: prefixed with the key type, then base58

### Passkey, "WebAuthn"

This object type is meant to be used with [passkeys](https://en.wikipedia.org/wiki/WebAuthn) with [Web Authentication standard](https://w3c.github.io/webauthn/). An object of this type looks as follows:

```
{
  "standard": "webauthn",
  "payload": "{\"signer_id\":\"0x3602b546589a8fcafdce7fad64a46f91db0e4d50\",\"verifying_contract\":\"intents.near\",\"deadline\":\"2025-03-30T00:00:00Z\",\"nonce\":\"A3nsY1GMVjzyXL3mUzOOP3KT+5a0Ruy+QDNWPhchnxM=\",\"intents\":[{\"intent\":\"transfer\",\"receiver_id\":\"bob.near\",\"tokens\":{\"nep141:usdc.near\":\"1000\"}}]}",
  "public_key": "p256:2V8Np9vGqLiwVZ8qmMmpkxU7CTRqje4WtwFeLimSwuuyF1rddQK5fELiMgxUnYbVjbZHCNnGc6fAe4JeDcVxgj3Q",
  "signature": "p256:3KBMZ72BHUiVfE1ey5dpi3KgbXvSEf9kuxgBEax7qLBQtidZExxxjjQk1hTTGFRrPvUoEStfrjoFNVVW4Abar94W",
  "client_data_json": "{\"type\":\"webauthn.get\",\"challenge\":\"4cveZsIe6p-WaEcL-Lhtzt3SZuXbYsjDdlFhLNrSjjk\",\"origin\":\"https://defuse-widget-git-feat-passkeys-defuse-94bbc1b2.vercel.app\"}",
  "authenticator_data": "933cQogpBzE3RSAYSAkfWoNEcBd3X84PxE8iRrRVxMgdAAAAAA=="
}
```

The [signature](https://near.github.io/intents/defuse_webauthn/enum.Signature.html) can use either [Ed25519 or P256, aka, secp256r1](https://www.iana.org/assignments/cose/cose.xhtml#algorithms).

#### Encoding information

The object uses the following encoding

authenticator\_data: base64, url-safe\
challenge: base64, url-safe\
signature: prefixed with key type, being `p256` or `ed25519`, following by the signature in base58 encoding\
public\_key: prefixed with the key type, then base58

### TonConnect

TonConnect follows the [standard for data signing](https://docs.tonconsole.com/academy/sign-data) on TON.

```
{
  "address": "EXvSRnDlYHziOJRm1MqGLgQB3EN7319eZLYWVinpoPv7LkBd",
  "domain": "example.com",
  "timestamp": "2025-01-01T00:00:00Z",
  "payload": {
    "type": "text",
    "text": "{\"signer_id\":\"alice.near\",\"verifying_contract\":\"intent.near\",\"deadline\":\"2025-05-26T15:19:43.617898Z\",\"nonce\":\"ZnbiFf4tP4cn65XLuZ6T1H6/Vr3o6ucNftdx3pInLnc=\",\"intents\":[{\"intent\":\"ft_withdraw\",\"token\":\"usdc.near\",\"receiver_id\":\"bob.near\",\"amount\":\"1000\"}]}"
  },
  "public_key": "ed25519:G4HVCaJg9vZb2srcLoWxR9grQ3tGLNFMVrZBhTtBi4Q1",
  "signature": "ed25519:5cwYdTNeGy1mApo9RNor9hSXvcG6GbvVm6di6kuf4frnARtVWRpJoPtvFKHMbt7uDGDtgFfn6bPDFGPK5jamqBwC"
}
```

#### Encoding information

public\_key: prefixed with the key type, then base58\
signature: prefixed with the key type, then base58

## Adding more key and signature types

To support additional key or signature types, please contact the Near Intents team.
