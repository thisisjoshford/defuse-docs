# Events

In the NEAR blockchain, [events](https://docs.near.org/tutorials/nfts/events) are strings that are logged as a result of executing a smart contract function. Smart contracts can emit any events they wish to alert users to important on-chain actions.

The `Verifier` smart contract uses events to emit information about what happened in every intent and/or transaction function call. In this section, we will list some of the events used.

### Structure of event strings

In the NEAR blockchain, events, based on [NEP-171](https://docs.near.org/tutorials/nfts/events), are prefixed with the string `EVENT_JSON`. Smart contracts can then define custom event content freely. The `Verifier` contract uses strictly JSON objects in order to maintain easy programmability with detecting events.

#### Intents events

Below is a [list of the available events](https://near.github.io/intents/defuse_core/events/enum.DefuseEvent.html) and their explanation. These events are directly related to the [intent submitted](intent-types-and-execution.md):

* **PublicKeyAdded**: Emitted when a public key is added to an account with an intent
* **PublicKeyRemoved**: Emitted when a public key was removed from an account with an intent
* **Transfer**: Emitted when a transfer between two accounts happens with an intent
* **TokenDiff**: Emitted when a `TokenDiff` intent executes successfully
* **IntentsExecuted**: Emitted after successfully having executed the listed intents
* **FtWithdraw**, **NftWithdraw**, **MtWithdraw**, **NativeWithdraw**: Emitted when a withdraw intent is made. Note that many times it is difficult to emit only on success because this depends on cross-contract calls.
* **StorageDeposit**: Emitted when a storage deposit intent is submitted.

#### Multi Token events

Multi Token events are emitted to indicate a change in the internal balances of the `Verifier` contract. The following are the [available events](https://near.github.io/intents/defuse_nep245/enum.MtEvent.html).

* [MtMintEvent](https://near.github.io/intents/defuse_nep245/struct.MtMintEvent.html): Is emitted when the balance of a given account in the `Verifier` smart contract increases, due to deposits.
* [MtBurnEvent](https://near.github.io/intents/defuse_nep245/struct.MtBurnEvent.html): Is emitted when the balance of a given account in the `Verifier` smart contract decreases, due to withdrawals.
* [MtTransferEvent](https://near.github.io/intents/defuse_nep245/struct.MtTransferEvent.html): After executing intents like `TokenDiff`, the requests inside of it are converted eventually to transfers between accounts by fulfilling all the requests in it. This event is emitted to indicate that tokens have "changed hands", or moved between accounts.

To understand the reasoning behind the naming, one can imagine the `Verifier` as a set of isolated balances that can either increase, decrease or move between accounts. Although balances are conserved and simply moved between accounts, from the Verifier contract’s perspective, increasing token amounts is equivalent to minting, and decreasing them is akin to burning.

This picture stems primarily from a discussion in [this section](deposits-and-withdrawals/balances-and-identifying-your-token.md), where all tokens use the Multi Token standard for storage inside the `Verifier` smart contract.



