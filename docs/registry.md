---
nav_order: 2
title: Subprotocol Registry
layout: default
permalink: registry
---

# Address Registry

`AddressRegistry.sol` allows users to register a CID as their canonical on chain identity. It also offers address lookup and reverse lookup functionality, allowing applications to find an address' registered CID and vice versa.

## Registering cidNFTs

To register a cidNFT to an address, users must call the `register` method on `AddressRegistry.sol`, passing in the CID's tokenId.

```solidity
    function register(uint256 _cidNFTID) external {
        if (ERC721(cidNFT).ownerOf(_cidNFTID) != msg.sender)
            // ownerOf reverts if non-existing ID is provided
            revert NFTNotOwnedByUser(_cidNFTID, msg.sender);
        cidNFTs[msg.sender] = _cidNFTID;
        emit CIDNFTAdded(msg.sender, _cidNFTID);
    }
```

Note that a wallet can only have one CID registered to its address at any given time. Calling the `register` method from a wallet address that has already registered a CID will overwrite its existing registration.

 To remove a registration, users can call the `remove` method. Transferring a CID to another wallet will also remove its existing registration.

## Address Lookup

The `getCid` method takes an address and returns the tokenId of the CID registered to that address. If no CID is registered to the address, this method returns `0`.

### Reverse Lookup

The `getAddress` method takes a tokenId (of a CID) and returns the address to which it is registered. If the queried CID is not registered to an address, this method returns `address(0)`.