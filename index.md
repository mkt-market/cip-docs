---
nav_order: 0
title: Overview
layout: default
---

# Overview

Canto Identity Protocol (CIP) is an on-chain identity protocol that aggregates identity from Subprotocols. Anyone can permissionlessly create a Subprotocol to represent any kind of identity trait -- textual, visual, or otherwise. End users freely choose which traits to include in their on-chain identity.

At launch, the core protocol consists of three smart contracts:
- [`CidNFT.sol`](https://github.com/mkt-market/canto-identity-protocol/blob/master/src/CidNFT.sol)
- [`AddressRegistry.sol`](https://github.com/mkt-market/canto-identity-protocol/blob/master/src/AddressRegistry.sol)
- [`SubprotocolRegistry.sol`](https://github.com/mkt-market/canto-identity-protocol/blob/master/src/SubprotocolRegistry.sol)

Within Canto Identity Protocol, ERC721 NFTs called [CIDs](cidNFTs.md) (**C**anto **Id**entities) represent individual on-chain identities. Users can mint CIDs for free by calling the `mint` method on `CidNFT.sol`.

Users must [register](registry.md) a CID as their canonical on-chain identity with `AddressRegistry.sol`.

## Subprotocols

Canto Identity Protocol is an *Open Aggregating Registration System* (OARS) -- a new primitive that aggregates data from subprotocols. CIP itself has no notion of identity traits, such as display name or profile picture. Instead, it aggregates identity from granular, trait-specific protocols called Subprotocols.

[Subprotocol creation](subprotocols.md) is permissionless. However, Subprotocols must be [registered](subprotocols.md#subprotocol-registration) with `SubprotocolRegistry.sol` for a one-time fee in order to be used within Canto Identity Protocol.

### subprotocolNFTs

subprotocolNFTs represent individual identity traits. Users choose which traits to include in their on-chain identity by adding references to those subprotocolNFTs to their CID using the `add` method on `CidNFT.sol`.

Fees and conditions associated with minting subprotocolNFTs are determined by the Subprotocol's creator. Since Subprotocols are loosely defined smart contracts, Subprotocol creators can introduce arbitrary logic to NFT minting and ownership.