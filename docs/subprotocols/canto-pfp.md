---
nav_order: 0
parent: Subprotocols
title: Canto PFP
layout: default
permalink: subprotocols/canto-pfp
---

# Canto PFP Protocol

Canto PFP Protocol is a barebones Subprotocol that can be used to reference any ERC721 NFT in one's on chain identity. It consists of a single smart contract, [`ProfilePicture.sol`](https://github.com/mkt-market/canto-pfp-protocol/blob/master/src/ProfilePicture.sol).

## Minting

To mint a PFP subprotocolNFT, a user calls `mint` on the smart contract, passing in the address of an NFT collection and the tokenId of an NFT they hold from that collection.

```solidity
    function mint(address _nftContract, uint256 _nftID) external {
        uint256 tokenId = ++numMinted;
        if (ERC721(_nftContract).ownerOf(_nftID) != msg.sender)
            revert PFPNotOwnedByCaller(msg.sender, _nftContract, _nftID);
        ProfilePictureData storage pictureData = pfp[tokenId];
        pictureData.nftContract = _nftContract;
        pictureData.nftID = _nftID;
        _mint(msg.sender, tokenId);
        emit PfpAdded(msg.sender, tokenId, _nftContract, _nftID);
    }
```

The `mint` method reverts if the user is not holding the referenced NFT in their wallet.

## Ownership Checks

The `getPFP` method checks whether the NFT referenced by a PFP subprotocolNFT is still owned by the expected wallet. It uses `CidNFT.sol` to lookup which CID the PFP subprotocolNFT has been added to, after which it performs an ownership check against the address that CID resolves to on `AddressRegistry.sol`.

```solidity
    function getPFP(uint256 _pfpID) public view returns (address nftContract, uint256 nftID) {
        if (_ownerOf[_pfpID] == address(0)) revert TokenNotMinted(_pfpID);
        ProfilePictureData storage pictureData = pfp[_pfpID];
        nftContract = pictureData.nftContract;
        nftID = pictureData.nftID;
        uint256 cidNFTID = cidNFT.getPrimaryCIDNFT(subprotocolName, _pfpID);
        IAddressRegistry addressRegistry = cidNFT.addressRegistry();
        if (cidNFTID == 0 || addressRegistry.getAddress(cidNFTID) != ERC721(nftContract).ownerOf(nftID)) {
            nftContract = address(0);
            nftID = 0; // Strictly not needed because nftContract has to be always checked, but reset nevertheless to 0
        }
    }
```

If the NFT is still owned by the expected wallet, this method returns the address of the NFT collection and the tokenId of the NFT. Otherwise, it returns `address(0)` and `0` respectively.

## PFP Preview and Metadata

The `tokenURI` view calls `getPFP` internally. If the referenced NFT is owned by the expected wallet, it returns the tokenURI of the referenced NFT. Otherwise, it reverts.

```solidity
    function tokenURI(uint256 _id) public view override returns (string memory) {
        (address nftContract, uint256 nftID) = getPFP(_id);
        if (nftContract == address(0)) revert PFPNoLongerOwnedByOriginalOwner(_id);
        return ERC721(nftContract).tokenURI(nftID);
    }
```

In practice, this means that the PFP subprotocolNFT shares the same metadata (including image) as the referenced NFT if that NFT is owned by the expected wallet.