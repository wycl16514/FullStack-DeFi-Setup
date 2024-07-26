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

If we want to run the code of solidity locally we need to setup a virtual machine in local by using hardhat, run the following command:

```js
npx hardhat node
```
![截屏2024-07-26 16 43 03](https://github.com/user-attachments/assets/ee6db49c-91db-4641-b70a-fbeaf2b4f5bc)

We need to communicate with the virtual machine with RPC call at port 8545, we need to create a script to help us send the bytecode of smart contract to the virtual machine, create 
a new folder in the project root directory with name script, then add a file with name deploy.js and have following code:

```js
const { ethers } = require("hardhat")
async function main() {
    //get deploy account, normally is Account #0
    const [deployer] = await ethers.getSigners()
    const tokenContractFactory = await ethers.getContractFactory("SimpleDeFiToken")
    //send bytecode in artfacts directory to virutal machine
    const token = await tokenContractFactory.deploy()
    //wait the complete of deployment then we can get the deploy address
    await token.waitForDeployment()
    console.log("Simple DeFi token contract address: ", token.target)
    console.log("Deployer address: ", deployer.address)
    const balance = await deployer.provider.getBalance(deployer.address)
    console.log("Deployer ETH balance: ", balance.toString())

}

try {
    main()
} catch (err) {
    console.error(err)
    process.exitCode = 1
}
```
Then create a new console and cd to the project root then run the following command:
```js
npx hardhat run scripts/deploy.js --network localhost
```
And you will get the following result:
```js
Simple DeFi token contract address:  0x5FC8d32690cc91D4c39d9d3abcBD16989F875707
Deployer address:  0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Deployer ETH balance:  9999995199641698486942
```
The contract address is just like the ip address of server, and the Deployer address is the ID for who deploys this contract, balance is how much "money" saved in the contract. Switch
to to the console which is running the hardhat node you will find following output:


![截屏2024-07-26 17 48 23](https://github.com/user-attachments/assets/de94e843-f9e8-4d28-9362-9d63b1a7a934)

Let's see how to deploy contract to testnet, this time we need the help of RPC endpoint to do the chores,  there are several testnet for Ethereum and we choose Sepolia, and following
are RPC info for the testnet:

https://www.alchemy.com/chain-connect/chain/sepolia

In order to interact with testnet, we need an account as our ID to perform all operations, at this time we need to install metamask:

 https://metamask.io/

Then click the icon at the right of you chrome browser, then click the button on the left to select network:

![截屏2024-07-26 18 13 00](https://github.com/user-attachments/assets/2ab7b7da-fab3-4bdb-9baf-07350c180684)

Then click at the middle of the top to create an account, then click the three points at the right of account and select "account details":

![截屏2024-07-26 18 15 08](https://github.com/user-attachments/assets/d0293da3-d7b8-4ce5-89a8-6687e9f0008f)

Then click "show private key" and copy the private key

![截屏2024-07-26 18 15 21](https://github.com/user-attachments/assets/0c626c49-44f7-458c-ae13-920961fa5b61)

Remember never leak your private key to anyone, then create a .env file at the project root, and add a field with PRIVATE_KEY then past your private key there:

![截屏2024-07-26 18 19 31](https://github.com/user-attachments/assets/38a2835b-9cae-4336-b63e-fcc4a6dbd67a)

And select a RPC endpoint url and save in the .env file:
```js
API_URL = https://eth-sepolia.g.alchemy.com/v2/demo
```

Then we need to goto hardhat.config.js to add following code:
```js
require("@nomicfoundation/hardhat-toolbox");
require('dotenv').config()
const SEPOLIA_API_URL = process.env.API_URL
const SEPOLIA_PRIVATE_KEY = process.env.PRIVATE_KEY
/** @type import('hardhat/config').HardhatUserConfig */
module.exports = {
  solidity: "0.8.24",
  paths: {
    sources: "./src/backend/contracts",
    artifacts: "./src/backend/artifacts",
    cache: "./src/backend/cache",
    tests: "./src/backend/test"
  },
  networks: {
    sepolia: {
      url: SEPOLIA_API_URL,
      accounts: [SEPOLIA_PRIVATE_KEY]
    }
  }
};
```

Before we can deploy contract to testnetwork, we need some ETH to pay for gas:

[https://sepoliafaucet.com/](https://cloud.google.com/application/web3/faucet/ethereum/sepolia)

Such ETH we got it is only for testing and has 0 value. Then we can run the following command to deploy contract to testnet :

```js
npx hardhat run scripts/deploy.js --network sepolia
```
if you have the following problem, change another endpoint url:

![截屏2024-07-26 19 06 20](https://github.com/user-attachments/assets/aeed01b6-fedb-48b4-aba8-bf5b6c22c049)

then run again ,if it is success you will get something like following:
```js
Simple DeFi token contract address:  0x83b993c8C297491bbCff50aC753CDD69Ae773Ce0
Deployer address:  0x9e8691d81e0c78E5A79f81aec73f30DDE45aCC3d
Deployer ETH balance:  19180029339484784
```
Then we can verify the deployment of the contract by using the contract address at : https://sepolia.etherscan.io/:

![截屏2024-07-26 19 10 43](https://github.com/user-attachments/assets/279ff106-04fa-4ca4-b1f3-0cfac02820bf)

Click "Simple DeFi Token (SDFT)" then we will get the details about the contract:
![截屏2024-07-26 19 13 06](https://github.com/user-attachments/assets/b7effc98-cb8d-43d8-99f3-154ba5494ef8)

Finally let's add a deploymet script to package.json:
```js
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "deploy": "npx hardhat run scripts/deploy.js --network"
  },
```
Then if we want to deploy contract again, we simply run:

npm run deploy sepolia

