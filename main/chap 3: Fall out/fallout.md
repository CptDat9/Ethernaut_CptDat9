### Tấn công
- Lỗ hổng:
- Trong hợp đồng này, hàm constructor có tên là Fal1out() thay vì constructor(), vì vậy hợp đồng không sử dụng hàm constructor() đúng chuẩn của Solidity. Hệ quả là hàm Fal1out() được coi là một hàm thông thường, không phải là constructor, và kẻ tấn công có thể gọi lại hàm này để "tạo" hợp đồng và trở thành chủ sở hữu hợp đồng.
### Cách phòng ngừa
- Sử dụng constructor

### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Fallout {
    using SafeMath for uint256;

    mapping(address => uint256) allocations;
    address payable public owner;

    /* constructor */
    function Fal1out() public payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
    } // Bị tấn công chỗ này

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function allocate() public payable {
        allocations[msg.sender] = allocations[msg.sender].add(msg.value);
    }

    function sendAllocation(address payable allocator) public {
        require(allocations[allocator] > 0);
        allocator.transfer(allocations[allocator]);
    }

    function collectAllocations() public onlyOwner {
        msg.sender.transfer(address(this).balance);
    }

    function allocatorBalance(address allocator) public view returns (uint256) {
        return allocations[allocator];
    }
}
```
