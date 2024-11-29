### Kết luận
- Hàm `unchecked` bỏ qua check tràn số nên sẽ là lỗ hổng nếu bị tràn số.
- Ưu diểm: Tối ưu phí gas (không check overflow và underflow).
- Nhược điểm: Sử dụng `underflow` giảm số dư token xuống giá trị cao nhất để khai thác token không giới hạn.
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    unchecked {
    require(balances(msg.sender) - _value >=0);
    balaces(msg.sender) -= _value;
    balances(_to) += _value;
    }
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```
### Contract 
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;
// Từ solidity 0.8.0 trở lên đã tự dộng check lỗi tràn số.

contract Token {
    mapping(address => uint256) balances;
    uint256 public totalSupply;

    constructor(uint256 _initialSupply) public {
        balances[msg.sender] = totalSupply = _initialSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return balances[_owner];
    }
}
```
