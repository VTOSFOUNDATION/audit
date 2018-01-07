# Summary

1. Disclaimer
2. Abstract
3. Attacks
4. Vulnerabilities found
5. Transparency
6. Summary of the audit

## 1. Disclaimer
The audit makes no statements or warranties about utility of the code, safety of the code, suitability of the business model, regulatory regime for the business model, or any other statements about fitness of the contracts to purpose, or their bug free status. The audit documentation is for discussion purposes only.

## 2. Abstract
The following is an audit made to VTOSToken2.sol smart contract (https://github.com/VTOSFOUNDATION/tokens) and will cover a series of technical and good behavior checks.

At the moment of this audit a previous one were made, use it as complement and reference (https://github.com/VTOSFOUNDATION/audit)

## 3. Attacks

* **Overflow and underflow attacks**
Ethereum Virtual Machine memory works in a 256 bit basis, when a variable is read, it's expanded to 256bit size and when the variable is stored it can be sized from 8 to 256 bits. An overflow/underflow attack can be made if the code doesn't check changes made on variables (add, sub, mult, div, exp), for example, if a 8 bit variable is used, it can represent 256 unsigned integer numbers (uint 0 to 255). Lets use 0 and 255 for the example.

  * 0 in binary is 0000 0000 (0x00 hex), if sub 1 from 0 it will become 1111 1111 (0xFF)
  
  * 255 in binary is 1111 1111 (0xFF hex), if add 1 from 255 it will become 0000 0000 (0x00)

  This contract checks for overflows and underflows with [**OpenZeppelin's** *SafeMath*] library (https://github.com/OpenZeppelin/zeppelin-solidity/blob/master/contracts/math/SafeMath.sol).

  There is one instance of arithmetic operation that doesn't check for overflow at line 250:

  `totalSupply = initialSupply * 10 ** uint256(decimals); //Update total supply with the decimal amount`

  Since this only will be executed during deployment the only risk it to place a number greater than ~ 2^256/10^18 for the initial supply.

* _**Short address attack**_
This will be a mention to this kind of attack since there is no contract-side complete solution, a good reference to understand this is [Smart Contract short address attack mitigation failure] https://blog.coinfabrik.com/smart-contract-short-address-attack-mitigation-failure/

This contract is not directly vulnerable to this kind of attack. 

## 4. Vulnerabilities found
* `LOW` On `function transferOwnership(address newOwner)` an `if` is used to check that new owner is not address `0x0`, if it is, the transaction is not reverted and gas is lost.

* `VERY LOW` Redundant check on line 84 `balances[msg.sender] >= _amount` with line 88 `balances[msg.sender].sub(_amount);` SafeMath will handle this, if `_amount` is greater than `balances[msg.sender]`, sub will throw.

* `VERY LOW` Redundant check on line 85 `balances[_to].add(_amount) > balances[_to]);` with line 89 `balances[_to].add(_amount);` SafeMath will handle this, if `_amount` makes `balances[_to]` overflow, add will throw.

* `VERY LOW` Redundant check on line 125 `balances[_from] >= _amount` with line 129 `balances[_from].sub(_amount);` SafeMath will handle this, if `_amount` is greater than `balances[_from]`, sub will throw.

* `VERY LOW` Redundant check on line 126 `allowed[_from][msg.sender] >= _amount` with line 131 `allowed[_from][msg.sender].sub(_amount)` SafeMath will handle this, if `_amount` is greater than `allowed[_from][msg.sender]`, sub will throw.

* `VERY LOW` Redundant check on line 127 `balances[_to].add(_amount) > balances[_to]);` with line 130 `balances[_to].add(_amount);` SafeMath will handle this, if `_amount` makes `balances[_to]` overflow, add will throw.

* `VERY LOW` Redundant check on line 176 `require(_value <= balances[msg.sender]);` with line 180 `balances[msg.sender].sub(_value);` SafeMath will handle this, if `_value` is greater than `balances[msg.sender]`, sub will throw.

* `VERY LOW` Lack of return variable names on functions:

  `function balanceOf(address _owner) public view returns (uint256);`
  
  `function transfer(address _to, uint256 _amount) public returns (bool);`

  `function allowance(address _owner, address _spender) public view returns (uint256);`
  
  `function transferFrom(address _from, address _to, uint256 _amount) public returns (bool);`
  
  `function approve(address _spender, uint256 _amount) public returns (bool);`

  More than vulnerabilities, these notes are for consistency with the ERC20 Standard definition, consider change them to:

  `function balanceOf(address _owner) public view returns (uint256 balance);`
  
  `function transfer(address _to, uint256 _amount) public returns (bool success);`

  `function allowance(address _owner, address _spender) public view returns (uint256 remaining);`
  
  `function transferFrom(address _from, address _to, uint256 _amount) public returns (bool success);`
  
  `function approve(address _spender, uint256 _amount) public returns (bool success);`

* `VERY LOW` Contract `Ownable` doesn't add real functionalities to the token, `modifier onlyOwner()` is only used for `function transferOwnership(address newOwner)`, consider remove all that section or give it utility.

## 5. Transparency
For this checks lets use some of the check from the list of ICO Transparency Monitor (https://github.com/Neufund/ico-transparency-monitor). Some of these check are subjective so the user must consider this questions, also, some of these must be updated at a later stage.

  * Is smart contract source code available? `YES`

  * Is smart contract source code provided in etherscan? `NO - Not yet deployed`

  * Is instruction provided how to reproduce deployed byte-code? `NO - Not yet deployed`

  * Does smart contract provide all tracking data via events? `NO - Owner change is not reported over an event`

  * Was smart contract code easy to read and properly commented? `YES`

## 6. Summary of the audit
* The ERC20 token standard itself is well implemented, some optimizations can be made considering redundant checks.

* Consistency is important, the ERC20 standard use return variable names on it's functions, in real it doesn't affect functionality but format is important.

* Code that doesn't add any real functionality is only waste of gas, please consider give some use to `Ownable` contract or remove it at all.
