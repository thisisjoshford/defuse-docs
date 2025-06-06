# Near Token treatment

Since NEAR is the native token for the near blockchain, implying the requirement to have special treatment for it, and in order to streamline the usage of NEAR with other assets, NEAR is treated as a fungible token ([NEP-141](https://nomicon.io/Standards/Tokens/FungibleToken/Core)) using its wrapped fungible token, [`wrap.near`](https://nearblocks.io/address/wrap.near). The source code (and interface) for wrapped NEAR (wNEAR) can be found [here](https://github.com/near/core-contracts/).

As a result, there is no way to deposit NEAR directly. The user should exchange their NEAR token and exchange it for wrapped NEAR token in the `wrap.near` contract before depositing them into the `Verifier` contract.
