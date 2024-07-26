When we first write smart contract, we don't need to do it from scratch, we always rely 
on the framework OpenZeppelin, it can reduce our workload and provide security enhancement for our contract, let's install it first:

```js
npm install @openzeppelin/contracts
```

Then we create our first contract in /src/backend/contracts with name "SimpleDeFiToken.sol" and have following code:

```sol
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract SimpleDeFiToken is ERC20 {
    constructor() ERC20("Simple DeFi Token", "SDFT") {
        _mint(msg.sender, 1e24);
    }
}
```
In the code, the first line : "// SPDX-License-Identifier: MIT" is neccessary even it is in comment state, this tells what kind of license you source code will follow, if you miss
this line, the compiler will give you error. The second line "pragama solidity ^0.8.0" tells what kind of compiler should apply to the following code, the Solidity language has 
improved a lot, some enchancement is not backward compatible, therefore some grammar used in the code may not supported by earlier version of compiler.

The "^0.8.0" indicates we need compiler verision no lower than 0.8.0 to compile the following code, otherwise lower version may fall to compile it. The following "import" is just
like the import of python or java or javascript, it is used to import other codes for using.

The "contract" is just like "class" keyword in java, js, python, and "is" keyword is just like "extend" in java or js, which is used to inherit all functionanlities from contract
ERC20, the later implements a lot of interface to generate crypto coins. The "consctractor" is initializer of the contract, it is exectued when we create an instance of the contract,
it is just like __init__ of python and the same for c++, js and java.

The line following constructor is "ERC20("Simple DeFi Token", "SDFT")" is used to intialized the parent class, it is just like super in java , the "_mint" is function provided by
ERC20, which is used to create coins with number of "1e24", now you just like Fed to print any amount of money you like. And "msg" is an global variable for the running environment, we can get lots of info related to the running environment of the code from it, you can take it as operating system, the sender from the msg is used to represent the user who create
the instance of the contract, it just like an ID number or bank account, when methods of the contract is called, we can use "msg.sender" to identify who is calling the contract.

Now we can compile the code using following command:

```js
npx hardhat compile
```

If running successful, you will see something like following:

```js
Compiled 6 Solidity files successfully (evm target: paris).
```

This indicates the compiling is ok. Notices it generates several files because we import Solidity files from OpenZeppelin, and the compiling results are saved at the artifacts 
directory we setup in hardhat.config.js. Then let's check the compiling result to gain deeper understanding to the structure of Solidity.

The First two important concepts for solidity is bytecode and ABI, lets go to the artifacts folder and look at the SimpleDeFiToken.json, and search for field name of "bytecode":

![截屏2024-07-26 16 16 07](https://github.com/user-attachments/assets/417de499-1465-4656-b8e4-a0b073ab29d1)

Bytecode of solidity is just the same as bytecode of java or assembly for complied result of c/c++, it can by run by EVM a virtual machine. And the ABI field contains infor about how
to call method in solidity contract by using js, it tells to js how to access members of the smart contract correctly, if the member of contract with modifier "private" or "internal",
which means you can't access them directly from oursize, this is the same as "private" or "protected" in c++ and java.


![截屏2024-07-26 16 22 09](https://github.com/user-attachments/assets/197c2546-c6e6-419e-a07c-eb233eb5ab3f)

If we want to run 


