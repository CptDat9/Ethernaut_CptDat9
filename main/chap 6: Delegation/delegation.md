### Kết luận
- `delegatecall` làm một hàm có sẵn trong solidity, nó cho phép gọi một hợp đồng B từ hợp đông A ( không thay đổi trạng thái B ) nhưng nó có thể làm thay đổi trạng thái của A.
- Lỗ hổng: Trong contract `Delegate`, hàm `pwm` cho phép thay đổi biến `owner` thành địa chỉ của người gọi `msg.sender`. Kẻ tấn công chỉ cần gửi dữ liệu `msg.data` tương ứng với hàm `pwm()` thì sẽ thay đổi được quyền sở hữu hợp đồng (`owner`) sang của mình.
- Cách phòng tránh: Sử dụng thêm modifier, require cho các function.

### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {
    address public owner;

    constructor(address _owner) {
        owner = _owner;
    }

    function pwn() public {
        owner = msg.sender;
    }
}

contract Delegation {
    address public owner;
    Delegate delegate;

    constructor(address _delegateAddress) {
        delegate = Delegate(_delegateAddress);
        owner = msg.sender;
    }

    fallback() external {
        (bool result,) = address(delegate).delegatecall(msg.data);
        if (result) {
            this;
        }
    }
}
```
