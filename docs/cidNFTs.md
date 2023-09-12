---
nav_order: 1
title: Canto Identities (CIDs)
layout: default
permalink: cidNFTs
---
{% seo %}
# Canto Identity NFTs

Canto Identity NFTs (CIDs) represent individual on chain identities. Through nested mappings, they point to subprotocolNFTs representing individual identity traits.

On `CidNFT.sol`, the internal mapping `cidData` maps CID tokenIds to Subprotocols, which in turn map to `SubprotocolData` structs:

```solidity
mapping(uint256 => mapping(string => SubprotocolData)) internal cidData;
```

The `SubprotocolData` struct is composed of three types corresponding to each of the three [Association Types](subprotocols.md#association-types) a Subprotocol may use:

```solidity
    struct SubprotocolData {
        /// @notice Mapping for ordered type
        mapping(uint256 => uint256) ordered;
        /// @notice Value for primary type
        uint256 primary;
        /// @notice List for active type
        IndexedArray active;
    }
```

* The `ordered` type is a mapping which maps indexes (0,1,2...) to subprotocolNFT tokenIds
* The `primary` type is a tokenId for a single subprotocolNFT
* the `active` type is a struct consisting of an array of subprotocolNFT tokenIds and a mapping used for optimization purposes

## Minting CidNFTs

To mint a Bio subprotocolNFT, a user calls `mint`:

```solidity
    function mint(MintAddData[] calldata _addList) external {
        uint256 tokenToMint = ++numMinted;
        _mint(msg.sender, tokenToMint); // We do not use _safeMint here on purpose. If a contract calls this method, he expects to get an NFT back
        for (uint256 i = 0; i < _addList.length; ++i) {
            MintAddData calldata addData = _addList[i];
            add(tokenToMint, addData.subprotocolName, addData.key, addData.nftIDToAdd, addData.associationType);
        }
    }
```

Optionally, the user can pass in an array of `MintAddData` structs, which specify subprotocolNFTs to be added immediately:

```solidity
    struct MintAddData {
        string subprotocolName;
        uint256 key;
        uint256 nftIDToAdd;
        AssociationType associationType;
    }
```

## Adding and Removing subprotocolNFTs

Users can add pointers to subprotocolNFTs to their CID at any time by calling the `add` function on `CidNFT.sol`. They must provide:

* `_cidNFTID`, the tokenId of the cidNFT
* `_subProtocolName`, the name of the Subprotocol
* `_key`, the key to be used for the pointer (only used for the `ordered` association type)
* `_nftIDToAdd`, the tokenId of the subprotocolNFTs
* `_type`, the type of association being made

The smart contract performs numerous checks before adding the pointer to `cidData` and emitting an event.

Users can remove pointers to subprotocolNFTs from their CID by calling the `remove` function on `CidNFT.sol` with the same inputs.

## Reading cidNFTs

To retrieve the subprotocolNFTs a CID points to for a specified Subprotocol, `CidNFT.sol` has three external views -- one for each association type:

* `getOrderedData` takes a cidNFT tokenId, Subprotocol name, and index and returns a subprotocolNFT tokenId
* `getPrimaryData` takes a cidNFT tokenId and Subprotocol name and returns a subprotocolNFT tokenId
* `getActiveData` takes a cidNFT tokenId and Subprotocol name and returns an array of subprotocolNFT tokenIds

### Reverse Lookups

`cidNFT.sol` also implements reverse lookup views for each of the association types. Given a subprotocolNFT tokenId and  Subprotocol, these methods return a CID tokenId and additional context depending on the association type:

* `getOrderedCIDNFT` returns the index of the queried subprotocolNFT in addition to the CID tokenId
* `getPrimaryCIDNFT` only returns the CID tokenId
* `getActiveCIDNFT` returns the position of the queried subprotocolNFT in addition to the CID tokenId