---
title: Moonbeam Routed Liquidity
description: Learn how to receive Moonbeam Routed Liquidity after establishing a cross-chain integration with a Moonbeam-based network.  
---

# Receiving Moonbeam Routed Liquidity

![XCM Overview Banner](/images/builders/interoperability/xcm/xc-integration/xc-integration-banner.png)

## Introduction {: #introduction }

Moonbeam Routed Liquidity (MRL) refers to a Moonbeam use case in which liquidity that exists in any blockchain ecosystem that Moonbeam is connected to can be routed to Polkadot parachains. This is possible because of general message passing (GMP), where messages with arbitrary data and tokens can be sent across non-parachain blockchains through [chain-agnostic GMP protocols](/builders/interoperability/protocols){target=_blank}.  

These GMP protocols can combine with Polkadot’s XCM messaging system to allow for seamless liquidity routing into parachains, either through the GMP precompile or traditional smart contracts that interact with XCM-related precompiles.

This guide will cover the process of integrating with a GMP provider’s SDKs and interfaces so that your parachain can access liquidity from non-parachain blockchains through Moonbeam. In addition, this guide will provide the necessary data to register the liquidity represented as Moonbeam’s ERC-20 xc-assets on your parachains.

Currently, MRL is available through Wormhole connected chains, but there is nothing stopping a parachain team from implementing a similar pathway through a different GMP provider.  

## Prerequisites {: #prerequisites }

In order to begin an MRL integration with your parachain, you will first need to:  

- [Establish an XC integration with Moonbeam](/builders/interoperability/xcm/xc-integration){target=_blank}
- [Registering the XC-20 token you want to routed to your parachain](/builders/interoperability/xcm/xc-integration#register-local-xc-20s-erc-20s){target=_blank}
- [Register Moonbeam’s asset on your parachain](/builders/interoperability/xcm/xc-integration#moonbeam-native-tokens){target=_blank}

An XC integration is required, because otherwise assets would not be able to be sent from Moonbeam into your parachain.  

GMP protocols typically move assets in a lock/mint or burn/mint fashion. This liquidity exists on Moonbeam normally as ERC-20 tokens. All ERC-20s on Moonbeam are now XCM-enabled, meaning that they can now exist as XC-20s in any other parachain, as long as they register it to enable XCM transfers.  

Registering Moonbeam’s asset is also required due to a temporary drawback of pallets that send XCM messages for asset transfer, making Moonbeam’s native gas asset the only asset that can be used as a cross-chain fee on the way back.  

## MRL through Wormhole {: #mrl-through-wormhole }

While MRL intends to encompass many different GMP providers, Wormhole is the first that has been built for the public. After you have an XC integration with Moonbeam, the steps to receive liquidity through Wormhole:  

1. Register all of the [ERC-20 assets](#tokens-available-through-wormhole) that you desire on the parachain
2. Notify the Moonbeam team of your desire to integrate into the MRL program so that we can help you with the technical implementation
3. Create a forum post on the [Moonbeam Community Forum](https://forum.moonbeam.foundation/){target=_blank} to supply the following information:
    - Parachain ID  
    - The account type that your parachain uses (AccountId32 vs AccountKey20)  
    - The addresses and names of the tokens that you have registered  
    - An endpoint that can be used by the Wormhole Connect frontend  
    - Why you want your parachain to be connected through Wormhole Connect  
4. Connect with the Wormhole team to finalize technical details and sync announcements  

### Sending Tokens Through Wormhole {: #sending-tokens-through-wormhole }

MRL provides a one-click solution that allows you to define a multilocation as a final destination for your assets to arrive from any Wormhole chain with a [Wormhole Connect integration](https://wormhole.com/connect/){target=_blank}.  

To send tokens through Wormhole & MRL, user interfaces will use a mixture of the [WormholeTokenBridge](https://github.com/wormhole-foundation/wormhole/blob/main/ethereum/contracts/bridge/interfaces/ITokenBridge.sol){target=_blank} and [Moonbeam’s GMP precompile](/builders/pallets-precompiles/precompiles/gmp){target=_blank}.  

Users transferring liquidity will invoke the `transferTokensWithPayload` method on the origin-chain's deployment of the WormholeTokenBridge smart contract to send tokens to the GMP precompile. This function requires a bytes payload, which must be formatted as a SCALE encoded multilocation object wrapped within another precompile-specific versioned type. The GMP precompile which will relay tokens to a parachain using XCM messages.  

![Transfering wormhole MRL](/images/builders/interoperability/mrl/mrl-1.png)

You can learn more about how to use the GMP precompile with Wormhole on the GMP precompile’s documentation page, but as a parachain integrating MRL, you will likely not need to implement or use the GMP precompile.  

### Sending Tokens Back Through Wormhole {: #sending-tokens-back-through-wormhole }

To send tokens back from your parachain to Wormhole, a user will need to send a transaction using both the `utility.batchAll`, `xTokens.transferMultiassets`, and `polkadotXcm.send` extrinsics. This is a one-click solution, but for the time being will require xcGLMR as a fee token, requiring the user to also own xcGLMR on the parachain.  

In the future, the xTokens pallet will be updated, allowing for your native gas currency to be used as a fee token instead. Parachains that use a different pallet would need to implement their own solution to transfer reserve and non-reserve assets in a single message.  

![Transfering wormhole MRL out](/images/builders/interoperability/mrl/mrl-2.png)

In the diagram above, the user is sending MRL tokens from a parachain back into Wormhole.

1. The user sends two XCM messages in one transaction using the utility.batchAll extrinsic:
2. The first XCM message sends xcGLMR & the ERC-20 to the user’s multilocation derivative account with xTokens.transferMultiassets, using xcGLMR as the fee token. The multilocation-derivative account is a keyless account on Moonbeam that an account on another parachain has control of via XCM
3. The second XCM message sends a remote EVM call to the batch precompile via polkadotXcm.send extrinsic
    - First, it approves the WormholeTokenBridge smart contract to move the ERC-20 sent over in the second step 
    - Second, it sends tokens across chains with the WormholeTokenBridge’s transferTokens function
4. The guardian network will pick up on the Wormhole transaction and sign it 
5. A wormhole relayer will relay the tokens to the destination chain & destination account

It’s important to note that not every parachain will have xTokens and the other pallets implemented in a way that will allow this path. Substrate-based chains are very flexible, to the point where a standard doesn’t exist. If you believe your parachain does not support this path, please provide an alternative solution in the Moonbeam forum and to the Wormhole team.  

### Tokens Available through Wormhole {: #tokens-available-through-wormhole }

While Wormhole has

The ERC-20 assets that can be bridged through Wormhole's MRL solution are dependent on the tokens that the [xLabs relayer](https://xlabs.xyz/){target=_blank} takes in. A current list of the tokens are available in the table below:  

**TODO: a table of a bunch of tokens**

Please take the time to verify that these assets are still Wormhole assets on Moonbeam by using the [Wormhole asset verifier](https://www.portalbridge.com/#/token-origin-verifier){target=_blank}.  