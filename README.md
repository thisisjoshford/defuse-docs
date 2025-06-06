---
description: >-
  NEAR Intents is a protocol for multichain financial products that replaces
  centralised exchanges. It requires market participants to deposit liquidity in
  order to trade.
---

# Overview

{% hint style="info" %}
The NEAR intents protocol and the documentation are under active development.

The protocol has been renamed from Defuse to "NEAR Intents".
{% endhint %}

```mermaid
sequenceDiagram
    participant User
    participant Message Bus
    participant NEAR

    User->>Message Bus: Request a quote
    note right of Message Bus: Market Makers <br> commit <br> to fill the quote
    Message Bus-->>Message Bus: 
    Message Bus-->>User: Return quotes
    User->>Message Bus: User commits <br> to execute the best quote
    Message Bus->>NEAR: Call the verifier contract


    note over NEAR: Smart contract <br> verifies signatures <br> and <br> settles matched <br> commitments
    NEAR ->>User: 
    note right of User: Swap Settled! ✅

```

## Terminology

1. Entities:
   1. Distribution channels. Applications that have the users, who are interested in decentralised spot trading.
   2. Market Makers. Active market participants that deposit liquidity in order to fill quotes issued by users
2. Intent Settlement:
   1. [Message Bus.](market-makers/bus/) an off chain message bus used for sending quotes and signed intents (commitments) between market makers and users. Each distribution channel can run their own Message Bus with their own set of market makers.
   2. [Verifier](intents-smart-contract/verifier/). Smart contract that verifies intents expressed as state changes (“diffs”) signed by corresponding owners. The combination of state changes is committed as long as the invariant (total delta is zero) was kept for each token after these changes were applied. Deployed on NEAR mainnet.
   3. [1 Click](integration/distribution-channels/1click-api.md). Swapping agent that makes it easy for distribution channels to use NEAR intents.
