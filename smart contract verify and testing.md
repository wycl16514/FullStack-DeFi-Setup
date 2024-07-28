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

Let's go to create the second test which is transfering some fund from the deployer which is Account#0 to other which is Account#1:
```js
it("should transfer token from one to other", async () => {
        expect(await token.balanceOf(deployer.address)).to.equal(toWei(1000000))
        await token.connect(deployer).transfer(user1.address, toWei(5))
        expect(await token.balanceOf(user1.address)).to.equal(toWei(5))
        expect(await token.balanceOf(deployer.address)).to.equal(toWei(999995))
    })
```
In the test case aboved, we first check the balance of deployer, then transfer five coins(5 wei) to user1, and make sure 5 coin is reduce from the balance of deployer and 5 coin is increased in the account of user1. As we 
can see the token.connect, this is setting the caller for the following contract methods which is the msg.sender. Here is a homework, try to connect user1 instead of deplyer and send 5 coins to user2 and see what happend.
And we need to make sure when error happens in the transaction, then we can revert the transaction and prevent fund lossing:
```js
 it("should revert for transaction fail", async () => {
        await token.connect(deployer).transfer(user1.address, toWei(5))
        try {
            await token.connect(user1).transfer(user2.address, toWei(10))
        } catch (err) {
            console.log("err: ", err)
            expect(await token.balanceOf(user1.address)).to.equal(toWei(5))
        }
    })
```

Comparing with python or java, the smart contract written by solidity is diffcult to debugged. We can't set breakpoints and stop at the line with breakpoint and check the state of the program, we can only debugging by printing
some message, let's see how to do it, we first define a new function in the contract as following:
```js
function transferWithAutoBurn(address to, uint256 amount) public {
        /*
        Destroy given amount of money of the sender and send the remaining
        to other
        */
        require(balanceOf(msg.sender) >= amount, "Not enough tokens");
        uint256 burnAmount = amount / 10;
        //provided by openzeppelin
        //there is a bug for the call
        _burn(to, burnAmount);
        //transfer remainning fund to other
        transfer(to, amount - burnAmount);
    }
```
Don't worry if you don't fully understand aboved code we will dive deep into solidity in later section. Remember to deploy the contract after chaning it,  And we add a test case for it:
```js
it("should burn the right amount of transferWithAutoBurn", async () => {
        await token.connect(deployer).transfer(user1.address, toWei(1))
        await token.connect(user1).transferWithAutoBurn(user2.address, toWei(1))
    })
```
Then run the test you will get error like following:

![截屏2024-07-28 16 44 25](https://github.com/user-attachments/assets/10bc550d-e994-48f6-af9c-7225555eace3)

When something wrong happend, since we can't set breakpoints and stop the programm, then we need to print info to help us debugging,and hardhat provides console.log to help us print important info for debugging, and we can 
change the code in the contract as following:
```js
...
import "hardhat/console.sol";
...
 function transferWithAutoBurn(address to, uint256 amount) public {
        /*
        Destroy given amount of money of the sender and send the remaining
        to other
        */
        require(balanceOf(msg.sender) >= amount, "Not enough tokens");
        uint256 burnAmount = amount / 10;
        console.log("Buring %s from %s, balance is %s", 
        burnAmount, to, balanceOf(to))
        //provided by openzeppelin
        //there is a bug for the call
        _burn(to, burnAmount);
        //transfer remainning fund to other
        transfer(to, amount - burnAmount);
    }
```
Remember to recompile and redeploy after changing to the contract. Then run the test again and you will see the following:

![截屏2024-07-28 16 52 05](https://github.com/user-attachments/assets/7d9dac10-57a0-4d59-8627-8ae4c79bb921)

As you can see from aboved, after buring the balance of the account is 0 which is no fund left for transfer, the expection is there is some amount left, then by looking into the details we found we burn the wrong guy, we should
burn to the caller of this method which is msg.sender instead of the recipient, such kind of bug is desaster for smart contract which lead to destroying a huge amount of assets, therefore we can fix the bug as:
```js
 function transferWithAutoBurn(address to, uint256 amount) public {
        /*
        Destroy given amount of money of the sender and send the remaining
        to other
        */
        require(balanceOf(msg.sender) >= amount, "Not enough tokens");
        uint256 burnAmount = amount / 10;
        // console.log(
        //     "Buring %s from %s, balance is %s",
        //     burnAmount,
        //     to,
        //     balanceOf(to)
        // );
        //provided by openzeppelin
        //there is a bug for the call
        //_burn(to, burnAmount);
        _burn(msg.sender, burnAmount);
        //transfer remainning fund to other
        transfer(to, amount - burnAmount);
    }
```
