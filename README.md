dex-on-ethereum/
├── contracts/
│   ├── DEX.sol
│   ├── Token.sol
│   ├── Migrations.sol
├── migrations/
│   ├── 1_initial_migration.js
│   ├── 2_deploy_contracts.js
├── test/
│   ├── DEX.test.js
│   ├── Token.test.js
├── scripts/
│   ├── deploy.js
├── .gitignore
├── README.md
├── truffle-config.js
├── package.json
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract DEX is Ownable {
    event Bought(uint256 amount);
    event Sold(uint256 amount);

    IERC20 public token;

    constructor(address _tokenAddress) {
        token = IERC20(_tokenAddress);
    }

    function buy() payable public {
        uint256 amountTobuy = msg.value;
        uint256 dexBalance = token.balanceOf(address(this));
        require(amountTobuy > 0, "You need to send some ether");
        require(amountTobuy <= dexBalance, "Not enough tokens in the reserve");
        token.transfer(msg.sender, amountTobuy);
        emit Bought(amountTobuy);
    }

    function sell(uint256 amount) public {
        require(amount > 0, "You need to sell at least some tokens");
        uint256 allowance = token.allowance(msg.sender, address(this));
        require(allowance >= amount, "Check the token allowance");
        token.transferFrom(msg.sender, address(this), amount);
        payable(msg.sender).transfer(amount);
        emit Sold(amount);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract Token is ERC20 {
    constructor(uint256 initialSupply) ERC20("MyToken", "MTK") {
        _mint(msg.sender, initialSupply);
    }
}
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Migrations {
    address public owner;
    uint public last_completed_migration;

    constructor() {
        owner = msg.sender;
    }

    modifier restricted() {
        require(msg.sender == owner, "This function is restricted to the contract's owner");
        _;
    }

    function setCompleted(uint completed) public restricted {
        last_completed_migration = completed;
    }
}
const Migrations = artifacts.require("Migrations");

module.exports = function (deployer) {
  deployer.deploy(Migrations);
};
const Token = artifacts.require("Token");
const DEX = artifacts.require("DEX");

module.exports = async function (deployer) {
  await deployer.deploy(Token, 1000000);
  const tokenInstance = await Token.deployed();
  await deployer.deploy(DEX, tokenInstance.address);
};
const Token = artifacts.require("Token");
const DEX = artifacts.require("DEX");

contract("DEX", accounts => {
  let tokenInstance;
  let dexInstance;

  before(async () => {
    tokenInstance = await Token.deployed();
    dexInstance = await DEX.deployed();
  });

  it("should allow users to buy tokens", async () => {
    const amount = web3.utils.toWei("1", "ether");
    await dexInstance.buy({ from: accounts[0], value: amount });
    const balance = await tokenInstance.balanceOf(accounts[0]);
    assert.equal(balance.toString(), amount, "Balance should be 1 ether worth of tokens");
  });

  it("should allow users to sell tokens", async () => {
    const amount = web3.utils.toWei("1", "ether");
    await tokenInstance.approve(dexInstance.address, amount, { from: accounts[0] });
    await dexInstance.sell(amount, { from: accounts[0] });
    const balance = await tokenInstance.balanceOf(accounts[0]);
    assert.equal(balance.toString(), "0", "Balance should be 0 after selling tokens");
  });
});
const Token = artifacts.require("Token");

contract("Token", accounts => {
  it("should put 1000000 tokens in the first account", async () => {
    const instance = await Token.deployed();
    const balance = await instance.balanceOf(accounts[0]);
    assert.equal(balance.toString(), "1000000", "1000000 wasn't in the first account");
  });
});
const Token = artifacts.require("Token");
const DEX = artifacts.require("DEX");

module.exports = async function (callback) {
  try {
    const token = await Token.deployed();
    const dex = await DEX.deployed();
    console.log(`Token deployed at: ${token.address}`);
    console.log(`DEX deployed at: ${dex.address}`);
  } catch (error) {
    console.error(error);
  }
  callback();
};
node_modules
build
.env
# Decentralized Exchange (DEX) on Ethereum

This project demonstrates the creation of a decentralized exchange (DEX) on the Ethereum blockchain using Solidity, Truffle, and OpenZeppelin. The DEX allows users to trade ERC-20 tokens in a decentralized manner.

## Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/dex-on-ethereum.git
   cd dex-on-ethereum
npm install
truffle compile
truffle migrate
truffle test
truffle exec scripts/deploy.js
truffle console

#### truffle-config.js
```javascript
module.exports = {
  networks: {
    development: {
      host: "127.0.0.1",
      port: 8545,
      network_id: "*"
    }
  },
  compilers: {
    solc: {
      version: "0.8.0"
    }
  }
};
{
  "name": "dex-on-ethereum",
  "version": "1.0.0",
  "description": "A decentralized exchange (DEX) on Ethereum",
  "main": "truffle-config.js",
  "scripts": {
    "test": "truffle test"
  },
  "author": "",
  "license": "MIT",
  "dependencies": {
    "@openzeppelin/contracts": "^4.0.0",
    "truffle": "^5.0.0",
    "web3": "^1.0.0"
  }
}
Initial commit: Add basic structure and DEX smart contracts

- Added `contracts/DEX.sol` for the decentralized exchange contract
- Added `contracts/Token.sol` for the ERC-20 token contract
- Added `contracts/Migrations.sol` for migration management
- Added migration scripts in `migrations/1_initial_migration.js` and `migrations/2_deploy_contracts.js`
- Added test cases in `test/DEX.test.js` and `test/Token.test.js`
- Added deployment script in `scripts/deploy.js`
- Added `.gitignore` to exclude unnecessary files
- Added `README.md` with project description, installation, and usage instructions
- Added `truffle-config.js` for Truffle configuration
- Added `package.json` with project dependencies
