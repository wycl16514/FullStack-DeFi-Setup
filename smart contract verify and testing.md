Actually grammar of solidity is quit simple, the difficult part is how to use solidity to implement bussinese logic that is sound and secure. Since smart contract always involve finacial transaction, any loophopes in the design
logic will be leveraged and casues huge finacial loss. Therefore ensuring the correctness and precautionary is up most importance.

That's why before deloying smart contract, we need testing and debugging to make sure as less vulnerabilities as possible. The first step we need to do is verifing the contract, before depolying let's change the code of contract
a little bit as following:

```js
// SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract SimpleDeFiToken is ERC20 {
    constructor() ERC20("Simple DeFi Token", "SDFT") {
        _mint(msg.sender, 1e24);
    }
}

```
What worth of noticing is the number 1e24, the _mint function used to create given number of coins, and the second parameter is used to set the number, but this number is in unit of 1e18, which means our second parameter 1e24
mapping to number of coins is 1e24 / 1e18 = 1e6 which means we are creating 1e6 coins.

make sure you deploy the SimpleDeFiToken.sol contract to local network, that is you running the hardhat node by using "npx hardhat node", then open a new console and run "npx hardhat run scripts/deploy.js --network localhost". 
After successful deploying the contract, record the address of the contract for later usage, for example my result is:
```js
Simple DeFi token contract address:  0x5FbDB2315678afecb367f032d93F642f64180aa3
Deployer address:  0xf39Fd6e51aad88F6F4ce6aB8827279cffFb92266
Deployer ETH balance:  9999998932358933593750
```

And run the following command to open hardhat console:

```js
npx hardhat console --network localhost
```

When Seeing the ">" sign which means we are in the hardhat console. Now in the console input the following code to get the instance of the deployed contract:
```js
const contract = await ethers.getContractAt("SimpleDeFiToken", "0x5FbDB2315678afecb367f032d93F642f64180aa3")
```
You will see an "undefined" returned after running the aboved code that's because we are calling async function and return immediately, the instance of contract will be valid in a moment and we can verify it as following:
```js
contract.target
```
then we get following:
![截屏2024-07-28 14 08 52](https://github.com/user-attachments/assets/b69b54f1-a55c-4d7c-a2b6-e2b66f650b34)

Then let's check infos related to the contract:

![截屏2024-07-28 14 12 55](https://github.com/user-attachments/assets/070da543-81ce-454e-b32b-482f1ff6c48d)

As we can see the number of totalSupply() is 1e24 the value of second parameter we set to _mint function. In order to convert this value to the number of coins, we can do the following:
```js
ethers.formatEther(await contract.totalSupply())
```
Then you will get the number of coints:'1000000.0'. Now we know how to intract with contract by using lib etherjs, then we can do unit testing base on it instead of inputting each command mannuly. Run .exit on the console to
exit from it. We need to install pacakge of "chai" for unit testing:
```js
npm install chai@4.3.7
```
After installing the package, we go to the test directory and create a new file named "SimpleDeFiToken.test.js" as following:
```js
const { ethers } = require("hardhat")
const { expect } = require("chai")

const toWei = (num) => ethers.parseEther(num.toString())
const fromWei = (num) => ethers.formatEther(num)

//create a test suit which is a package wrapping 
//several unit tests
describe("SimpleDeFiToken", () => {
  let deployer, user1, user2, token
    //following will be runned before executing each unit test
    beforeEach(async () => {
        //user1 is Account#0, user2 is Account#1 
        [deployer, user1, user2] = await ethers.getSigners()
        const tokenContractFactory = await ethers.getContractFactory("SimpleDeFiToken")
        token = await tokenContractFactory.deploy()
    })

    it("should have correct name, symbol, and total supply", async () => {
        expect(await token.name()).to.equal("Simple DeFi Token")
        expect(await token.symbol()).to.equal("SDFT")
        expect(await token.totalSupply()).to.equal(toWei(1000000))
    })
})
```
Then we can run command to run the test:
```js
npx hardhat test
```
Then you will get following result:


![截屏2024-07-28 14 51 12](https://github.com/user-attachments/assets/a1f9becc-ead3-4634-8ee1-abf372d24864)
