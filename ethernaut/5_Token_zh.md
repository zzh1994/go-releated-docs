# Token
关于代币合约的整数溢出漏洞（注：大部分代币合约出现的漏洞均为此漏洞）
```solidity
pragma solidity ^0.4.18;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  function Token(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

### 代码分析
1. `require(balances[msg.sender] - _value >= 0);`代码存在溢出漏洞，当_value>balances[msg.sender]时
因截断问题会发出溢出，会产生一个非常大的值，从而msg.sender获得了大量的token

###　攻击方法
1. 直接调用transfer(address,uint256)方法，msg.sender为你自己的地址，to填写其他的地址即可，保证
_value > balances[msg.sender]既可完成攻击