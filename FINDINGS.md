## H-1 The Contract Can not be Paused
### Description
The contract support pausing and unpausing of minting of new NFTs by inheriting from `OpenZeppelin::Pausable` contract. the OpenZeppelin pausable required the child contract to implement `pause` and `unpause` public/external function. However there is not such function implemented in `ethereal.sol` contract.

### Impact
The contract can not be paused in case of emergency.

## Recommendation

add External and owner restricted pause/unpause function.


## M-1 ReadOnly Reentrancy in mint function
### Description
Whenever new NFT is minted in `ethereal.sol` the `mint()` function get called. which after certain validation calls either `_mintEth` or `_mintWstEth` base on asset supported by collection. However inside `_mintEth` and `_mintWstEth` before making state updated it first calls `_safeMint` function which is vulnerable to Reentrancy.
The `Redeem` function has Reentrancy guard which protect contract from bypassing the Fee things , however there is still chance for reading the old states.

### impact
Incorrect data will be return from the contract.

### POC
In current test file please add the following code:

```solidity 
function test_MintForContract() public {
        setUpMint();
        vm.deal(address(this), 100 * 1e18); // Ensuring user1 has enough ETH

        vm.prank(address(this));
        uint256 tokenId = ethereal.mint{value: 100 * 1e18}(0, address(this));
        (uint256 balance, uint256 collection, uint256 gem) = ethereal.metadata(tokenId);
        assertEq(balance, 100 * 1e18);
        assertEq(collection, 0);
        assertEq(gem, 0);
    }

    function onERC721Received(address, address, uint256 tokenId, bytes memory) public payable returns (bytes4) {
        uint bal = ethereal.getTokenBalance(tokenId);
        assertEq(bal , 0);
        assertEq(ethereal.getTokenGemId(tokenId) , 0);
        assertEq(ethereal.gemsCirculating() , 0);
        return bytes4(keccak256("onERC721Received(address,address,uint256,bytes)"));
    }
```

run with command:
`forge test --mt test_MintForContract`


## NC-1 incorrect comment `% reward (3 decimals: 100 = 1%)`
### Description
The comments says that the rewards is in 3 decimals where 100 = 1%. this is not correct in case of 3 decimals 1000=1%.

"Green Rabbit"

