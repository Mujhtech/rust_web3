# Getting Started with Celo Blockchain using Rust Programming Language:

## Introduction:

In this tutorial, we'll go over the fundamentals of interacting with [Solidity](https://docs.soliditylang.org/en/v0.8.20/) [Smart Contract](https://www.ibm.com/topics/smart-contracts) programming language using [WEB3](https://crates.io/crates/web3). By the end of this tutorial, you will have a fundamental understanding of how to make a contract call in the [Rust](https://www.rust-lang.org/learn) Programming Language.

This tutorial will demonstrate how simple it is to interact with Smart Contracts, call functions and listen to events in [Rust](https://www.rust-lang.org/learn).

## Table of Contents:

- [Getting Started with Celo Blockchain using Rust Programming Language](#getting-started-with-celo-blockchain-using-rust-programming-language)
  - [Introduction](#introduction)
  - [Pre-requisites](#pre-requisites)
  - [How it works?](#how-it-works-?)
  - [Getting Started!](#getting-started-!)
  - [Setup the Smart Contract](#setup-the-smart-contract)
      - [Deploy Smart contract using Remix IDE](#deploy-smart-contract-using-remix-ide)
  - [Rust Implementation](#rust-implementation)
      - [Directory Structure](#directory-structure)
      - [Rust Implmentation](#rust-implementation-1)
      - [Run your Project](#run-your-project)
- [Conclusion](#conclusion)
- [About the Author](#about-the-author)
- [References](#references)

## Pre-requisites:

First, this tutorial assumes that you are already familiar with Solidity and understand how Smart Contracts works and that you already know the basics of using [Rust](https://www.rust-lang.org/learn) Language.

For this project we'll be using a few dependencies:

- [tokio](https://crates.io/crates/tokio)
- [web3](https://crates.io/crates/web3)
- [ethers](https://crates.io/crates/ethers)

![image|333x499](./screenshot-1.jpeg)

## How it works? -

[WEB3](https://crates.io/crates/web3) package is a crate package that lets you to deploy, interact with any blockchain network.

## Getting Started! :

I assume anyone going through this tutorial already understands and uses [Rust](https://www.rust-lang.org/learn), so I will skip the setup involved in getting Rust to work on your development computer. That means I assume you already have [VS Code](https://code.visualstudio.com/)/[Intellij Idea](https://www.jetbrains.com/idea/download/?section=windows)/[Eclipse](https://www.eclipse.org/downloads/)/[Atom](https://atom.en.softonic.com/download) and Rust setup on your PC.

If you are entirely new to Rust, here ( [https://www.rust-lang.org/learn/get-started](https://www.rust-lang.org/learn/get-started) ) is a good tutorial you can learn from or make use of online Rust playground ([https://play.rust-lang.org/](https://play.rust-lang.org/)).

## Set up the Smart Contract:

The next step is to compile our Smart Contract using the Solidity compiler of your choice, such as [Hardhat](https://hardhat.org/), [Truffle](https://trufflesuite.com/docs/truffle/how-to/install/) or any other Solidity compiler.

```solidity
// SPDX-License-Identifier: MIT

// TicTacToeV1

pragma solidity ^0.8.4;

contract TicTacToeV1 {
    struct LeaderBoard {
        address player;
        uint256 win;
    }

    uint32 private idCounter; // Counter for assigning unique IDs to leaderboards

    mapping(uint256 => LeaderBoard) internal leaderboards; // Mapping to store leaderboards

    function start(uint256 win) public {
        leaderboards[idCounter] = LeaderBoard(msg.sender, win); // Create a new leaderboard with the sender's address and specified win count
        idCounter++; // Increment the counter for the next leaderboard
    }

    function getLeaderboard(uint256 _index)
        public
        view
        returns (address player, uint256)
    {
        require(_index < idCounter, "Invalid leaderboard index"); // Ensure the provided index is within the range of existing leaderboards
        
        LeaderBoard storage leaderboard = leaderboards[_index]; // Retrieve the specified leaderboard
        return (leaderboard.player, leaderboard.win); // Return the player's address and win count
    }

    function updateLeaderboard(uint256 index) public {
        require(index < idCounter, "Invalid leaderboard index"); // Ensure the provided index is within the range of existing leaderboards
        
        leaderboards[index].win++; // Increment the win count of the specified leaderboard
    }

    function getLeaderboardLength() public view returns (uint256) {
        return idCounter; // Return the total number of leaderboards created
    }
}

```

### Deploy Smart contract using Remix:

Now that your contract is compiled, you can deploy your Smart Contract to the network. You can deploy to any Ethereum compatible network, and in this case we’ll be deploying the Celo testnet or mainnnet depending on your preference. If you’re brand new to this stick with testnet!

- Click the Deploy and Run Transactions Icon on the left side menu.
- Choose Injected Web3 as your environment.
- [Connect MetaMask to Celo](https://medium.com/@Celo_Academy/3-simple-steps-to-connect-your-metamask-wallet-to-celo-732d4a139587) testnet and verify the network.

![image|690x341](./screenshot-2.jpeg)

## Rust Implementation:

### Directory structure:

![image|568x499](./screenshot-3.jpeg)

Let’s copy our Contract ABIs into our project.

Then, create a folder in the project folder directory "lib" and create a file named "tictactoev1.abi.json".

![image|690x365](./screenshot-4.jpeg)

```javascript
[
  {
    inputs: [
      {
        internalType: "uint256",
        name: "_index",
        type: "uint256",
      },
    ],
    name: "getLeaderboard",
    outputs: [
      {
        internalType: "address",
        name: "player",
        type: "address",
      },
      {
        internalType: "uint256",
        name: "",
        type: "uint256",
      },
    ],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [],
    name: "getLeaderboardLength",
    outputs: [
      {
        internalType: "uint256",
        name: "",
        type: "uint256",
      },
    ],
    stateMutability: "view",
    type: "function",
  },
  {
    inputs: [
      {
        internalType: "uint256",
        name: "win",
        type: "uint256",
      },
    ],
    name: "start",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
  {
    inputs: [
      {
        internalType: "uint256",
        name: "index",
        type: "uint256",
      },
    ],
    name: "updateLeaderboard",
    outputs: [],
    stateMutability: "nonpayable",
    type: "function",
  },
];
```

### Rust Implementation:

![image|690x296](./screenshot-5.jpeg)

```js
use std::str::FromStr;
use web3::types::{Address, U256};
use web3::{Web3, contract::Contract, contract::Options, transports};

#[tokio::main]
async fn main() {

    // Set up the transport for the Celo network
    let provider_url = "https://alfajores-forno.celo-testnet.org";
    let transport = transports::Http::new(provider_url).unwrap();
    let web3 = Web3::new(transport);

    // Define the contract ABI and address
    let contract_abi = include_bytes!("contracts/tictactoe.abi.json");
    let contract_address = Address::from_str("0x0f6E0e3F5df62d4067D9969Cd3c9F34cc2b238C9").unwrap();

    // Connect to the contract
    let contract = Contract::from_json(web3.eth(), contract_address, contract_abi).unwrap();

    // Call a function on the contract
    let result = contract.query("getLeaderboardLength", (), None, Options::default(), None);
    let total_leader_board: U256 = result.await.unwrap();
    // Get the result of the function call
    println!("Total leaderboard: {}", total_leader_board);

}
```

In this example, we first set up a transport for the celo network using the transport Http struct method from Web3. We then create a new Web3 instance from the "http transport" we created. We then define the ABI and address of the Smart Contract we want to interact with. We use the Contract struct from web3 to connect to the contract and then call a function on the contract using the query/call method. Finally, we get the result of the function call and print it out.

Note that you'll need to replace the **_contracts/tictactoe.abi.json_** and **_contract_address_** values with the actual ABI and address of the Smart Contract you want to interact with. You can generate the ABI using a tool like abigen or solc, and you can get the contract address from a blockchain explorer or by deploying the contract yourself.

### Run your project:

```bash
cargo run --color=always --package celo_web3 --bin celo_web3
```
The run your program, we can easily see that we have been able to interact seamlessly with our deployed Smart Contract.

![image|690x147](./screenshot-6.png)

## Conclusion: 

Therefore, getting started with Celo Blockchain using the Rust programming language opens up exciting possibilities for developers. Rust's strong emphasis on safety, performance and concurrency makes it an excellent choice for building secure and efficient applications on the Celo Blockchain. By following the necessary steps and leveraging the available resources, developers can quickly gain the knowledge and skills required to dive into Celo Blockchain development with Rust. This combination empowers developers to explore the world of decentralized finance, mobile applications and other innovative solutions within the Celo ecosystem. With its growing popularity and active community, embarking on this journey promises a rewarding and promising future in blockchain development.

## About the Author:

I am a Software Engineer, Tech Evangelist (Preaching the gospel of flutter & blockchain) and Ex-GDSC Leads.

## References:

- [Github Repo](https://github.com/Mujhtech/rust_web3)
- [Tokio crate package](https://crates.io/crates/tokio)
- [web3 crate package](https://crates.io/crates/web3)
- [Get started with rust](https://www.rust-lang.org/learn/get-started)
- [Learn solidity by example](https://solidity-by-example.org)
