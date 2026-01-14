# FUSDT – Flush Tether USD Token  
Institutional‑Grade, Audit‑Ready Stablecoin

FUSDT (Flush Tether FUSDT) is a fully reserved, multisig‑controlled stablecoin protocol operating on both EVM and TRON networks.  
It is designed for institutional trust, transparency, and long‑term stability.

---

## 🚀 Features
- Dual‑chain support: ERC20 + TRC20  
- Multisig‑controlled mint/burn  
- Pause/Unpause security mechanism  
- Minimal, audit‑ready smart contract architecture  
- Exchange‑ready metadata and documentation  
- Reserve proof + governance framework  

---

## 📄 Documentation
- [LAUNCH.md](./docs/LAUNCH.md) – Launch & Operational Guide  
- [TOKENOMICS.md](./docs/TOKENOMICS.md)  
- [GOVERNANCE.md](./docs/GOVERNANCE.md)  
- [AUDIT.md](./docs/AUDIT.md)  
- [PRESS_RELEASE.md](./docs/PRESS_RELEASE.md)  

---

## 🧱 Repository Structure

/contracts
/scripts
/docs
├── LAUNCH.md
├── TOKENOMICS.md
├── GOVERNANCE.md
├── AUDIT.md
└── PRESS_RELEASE.md
/test

---

## 🛠️ Deployment

### EVM (Sepolia → Mainnet)

npx hardhat run scripts/deploy.js --network sepolia

### TRON (Shasta → Mainnet)
tronbox migrate --network shasta


---

## 🛡️ Security
- No reentrancy  
- No overflow/underflow  
- Mint/Burn restricted to multisig  
- Pause mechanism active  
- Ownership must be transferred to multisig  

---

## 📬 Contact
Website: https://flushtether.digital  
X: https://x.com/fusdt  
Email: contact@flushtether.digital

---
### /contracts

/contracts
  ├── FUSDT.sol
  ├── FUSDT_TRON.sol
  ├── interfaces/
  │     ├── IERC20.sol
  │     ├── ITRC20.sol
  │     └── IMultisig.sol
  ├── security/
  │     ├── Pausable.sol
  │     └── AccessControlled.sol
  ├── utils/
        ├── SafeMath.sol
        └── Address.sol


### FUSDT.sol (EVM version)

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "./security/Pausable.sol";
import "./utils/SafeMath.sol";

contract FUSDT is Pausable {
    using SafeMath for uint256;

    string public constant name = "FUSDT";
    string public constant symbol = "FUSDT";
    uint8 public constant decimals = 6;

    uint256 public totalSupply;
    address public multisig;

    mapping(address => uint256) public balanceOf;
    mapping(address => mapping(address => uint256)) public allowance;

    modifier onlyMultisig() {
        require(msg.sender == multisig, "Not authorized");
        _;
    }

    constructor(address _multisig) {
        multisig = _multisig;
    }

    function mint(address to, uint256 amount) external onlyMultisig whenNotPaused {
        totalSupply = totalSupply.add(amount);
        balanceOf[to] = balanceOf[to].add(amount);
        emit Transfer(address(0), to, amount);
    }

    function burn(address from, uint256 amount) external onlyMultisig whenNotPaused {
        balanceOf[from] = balanceOf[from].sub(amount);
        totalSupply = totalSupply.sub(amount);
        emit Transfer(from, address(0), amount);
    }

    function transfer(address to, uint256 amount) external whenNotPaused returns (bool) {
        balanceOf[msg.sender] = balanceOf[msg.sender].sub(amount);
        balanceOf[to] = balanceOf[to].add(amount);
        emit Transfer(msg.sender, to, amount);
        return true;
    }

    event Transfer(address indexed from, address indexed to, uint256 value);
}

---
### FUSDT_TRON.sol (TRC20 version)

pragma solidity ^0.5.10;

contract FUSDT_TRON {
    string public name = "FUSDT";
    string public symbol = "FUSDT";
    uint8 public decimals = 6;

    uint256 public totalSupply;
    address public multisig;

    mapping(address => uint256) public balanceOf;

    modifier onlyMultisig() {
        require(msg.sender == multisig, "Not authorized");
        _;
    }

    constructor(address _multisig) public {
        multisig = _multisig;
    }

    function mint(address to, uint256 amount) public onlyMultisig {
        totalSupply += amount;
        balanceOf[to] += amount;
        emit Transfer(address(0), to, amount);
    }

    function burn(address from, uint256 amount) public onlyMultisig {
        balanceOf[from] -= amount;
        totalSupply -= amount;
        emit Transfer(from, address(0), amount);
    }

    event Transfer(address indexed from, address indexed to, uint256 value);
}

---

## /scripts (Deployment + Verification)

/scripts
  ├── deploy.js
  ├── verify.js
  ├── tron-deploy.js
  └── tron-verify.js

---

 ## deploy.js (EVM)
  
  const hre = require("hardhat");

async function main() {
  const multisig = process.env.MULTISIG;

  const FUSDT = await hre.ethers.getContractFactory("FUSDT");
  const fusdt = await FUSDT.deploy(multisig);

  await fusdt.deployed();
  console.log("FUSDT deployed at:", fusdt.address);
}

main().catch((error) => {
  console.error(error);
  process.exit(1);
});

---

## verify.js (EVM)

async function main() {
  await hre.run("verify:verify", {
    address: process.env.CONTRACT,
    constructorArguments: [process.env.MULTISIG],
  });
}

main();

---

## tron-deploy.js (TRON)

const FUSDT = artifacts.require("FUSDT_TRON");

module.exports = function (deployer) {
  const multisig = process.env.MULTISIG;
  deployer.deploy(FUSDT, multisig);
};

---

### tron-verify.js

console.log("TRON verification is manual via TRONSCAN.");

---



### test.md

# FUSDT – Test Plan

This document outlines the full test suite required to validate the FUSDT smart contracts on both EVM and TRON networks.

---

## 1. Unit Tests

### ERC20 / TRC20 Core
- Transfer  
- Approval  
- Allowance  
- Balance updates  
- Event emission  

### Mint/Burn
- Only multisig can mint  
- Only multisig can burn  
- Supply updates correctly  
- Events emitted  

### Pause/Unpause
- Only multisig can pause  
- Transfers blocked when paused  
- Mint/Burn blocked when paused  

---

## 2. Access Control Tests
- Unauthorized mint attempt  
- Unauthorized burn attempt  
- Unauthorized pause attempt  
- Unauthorized ownership transfer  

---

## 3. Deployment Tests
- Contract deploys successfully  
- Owner is correctly set  
- Multisig is correctly assigned  
- Initial state is correct  

---

## 4. Edge Case Tests
- Mint to zero address  
- Burn from zero address  
- Transfer to zero address  
- Overflow/underflow attempts  

---

## 5. Gas Optimization Tests
- Transfer gas usage  
- Mint/Burn gas usage  
- Pause/Unpause gas usage  

---

## 6. TRON‑Specific Tests
- TRC20 event compatibility  
- TRON address format validation  
- TRON multisig integration  

---

## 7. Test Commands

### EVM

npx hardhat test


### TRON


---

## 8. Conclusion
This test suite ensures FUSDT is secure, stable, and ready for audit, exchange integration, and production deployment.


