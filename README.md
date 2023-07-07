# Table of Content
- [Introduction](#introduction)
     - [How it works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Setup the Smart Contract](#setup-the-smart-contract)
     - [Deploy Smart contract (Remix)](#deploy-smart-contract-remix)
- [Rust Implementation](#rust-implementation)
     - [Directory Structure](#directory-structure)
     - [Rust Implmentation](#rust-implementation-1)
     - [Run your Project](#run-your-project)
- [About the Author](#about-the-author)
- [References](#references)
    

## Introduction

In this tutorial, we'll go over the fundamentals of interacting with Solidity smart contract programming language using web3. You will have a fundamental understanding of how to make a contract call in the rust programming language.

This tutorial will demonstrate how simple it is to interact with smart contracts, call functions, and listen to events in rust.

### How it works

Web3 Package is a pretty cool crate package that let you deploy, interact with any blockchain network.

## Prerequisites

First, This tutorial assumes that you are already familiar with solidity and understand how smart contracts work and also assumes that you already know the basics of using rust language.

For this project we'll be using a few interesting dependencies:

- [tokio](https://crates.io/crates/tokio)
- [web3](https://crates.io/crates/web3)
- [ethers](https://crates.io/crates/ethers)

![image|333x499](./screenshot-1.jpeg)

## Getting Started

I assume anyone going through this tutorial already understands and uses Rust, so I will skip the setup involved in getting Rust to work on your development computer. That means I assume you already have VS Code/Intellij Idea/Eclipse/Atom and Rust setup on your PC.

If you are entirely new to Rust, here ( [https://www.rust-lang.org/learn/get-started](https://www.rust-lang.org/learn/get-started) ) is a good tutorial you can learn from or make use of online rust playground ([https://play.rust-lang.org/](https://play.rust-lang.org/)).

## Setup the Smart Contract

The next step is to compile our smart contract using the solidity compiler of your choice, such as hardhat, truffle, or any other solidity compiler.

```solidity
// SPDX-License-Identifier: MIT

// TicTacToeV1

pragma solidity ^0.8.20;



/**

@title TicTacToeV1
@dev This contract represents a Tic-Tac-Toe leaderboard system.
Players can start a game by calling the start function and passing the number of wins they have.
The player's address and win count are stored in the leaderboards array.
The contract provides functions to retrieve leaderboard information and update the win count.
*/
contract TicTacToeV1 {

    /**
    * @dev Struct representing a leaderboard entry.
    * @param player The address of the player.
    * @param win The number of wins for the player.
    */
    struct LeaderboardEntry  {
        address player;
        uint256 win;
    }

    uint32 private idCounter;

    LeaderboardEntry[] private leaderboards;

    /**
    * @dev Event emitted when a game is started.
    * @param player The player's address who started the game.
    * @param winCount The number of wins the player has.
    */
    event GameStarted(address player, uint256 winCount);

    /**
    * @dev Event emitted when a leaderboard entry is updated.
    * @param index The index of the leaderboard entry being updated.
    * @param newWinCount The updated win count for the leaderboard entry.
    */
    event LeaderboardUpdated(uint256 index, uint256 newWinCount);

    /**
    * @dev Start a game by adding a player to the leaderboard.
    * @param win The number of wins the player has.
    */
    function start(uint256 win) public {
        require(win > 0, "Invalid win count");
        leaderboards.push(LeaderboardEntry(msg.sender, win));
        idCounter++;
        emit GameStarted(msg.sender, win);
    }


    /**
    * @dev Retrieve player and win count for a given leaderboard index.
    * @param _index The index of the leaderboard entry to retrieve.
    * @return player The player's address.
    * @return playerwin The player's win count.
    */
    function getLeaderboard(uint256 _index)
        public
        view
        returns (address player, uint256 playerwin)
    {
        require(_index < idCounter, "Invalid leaderboard index");
        LeaderboardEntry storage leaderboard = leaderboards[_index];
        return (leaderboard.player, leaderboard.win);
    }

    /**
    * @dev Update the win count of a specific leaderboard entry.
    * @param index The index of the leaderboard entry to update.
    */
    function updateLeaderboard(uint256 index) public {
        leaderboards[index].win++;
        emit LeaderboardUpdated(index, leaderboards[index].win);
    }

    /**
    * @dev Get the length of the leaderboard array.
    * @return The length of the leaderboard array.
    */
    function getLeaderboardLength() public view returns (uint256) {
        return (idCounter);
    }
}
```

### Deploy Smart contract (Remix)

Now that your contract is compiled, you can deploy your smart contract to the network. You can deploy to any Ethereum compatible network, and in this case we’ll be deploying the Celo testnet or mainnnet depending on your preference. If you’re brand new to this stick with testnet!

- Click the Deploy and Run Transactions Icon on the left side menu.
- Choose Injected Web3 as your environment.
- [Connect MetaMask to Celo](https://medium.com/@Celo_Academy/3-simple-steps-to-connect-your-metamask-wallet-to-celo-732d4a139587) testnet and verify the network.

![image|690x341](./screenshot-2.png)

## Rust Implementation

### Directory structure

![image|568x499](./screenshot-3.jpeg)

Let’s copy our Contract ABIs into our project.

Then create a folder in the project folder directory lib and create a file named tictactoev1.abi.json.

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

### Rust Implementation

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

In this example, we first set up a transport for the celo network using the transport Http struct method from web3. We then create a new web3 instance from the http transport we created. We then define the ABI and address of the smart contract we want to interact with. We use the Contract struct from web3 to connect to the contract, and then call a function on the contract using the query/call method. Finally, we get the result of the function call and print it out.

Note that you'll need to replace the **_contracts/tictactoe.abi.json_** and **_contract_address_** values with the actual ABI and address of the smart contract you want to interact with. You can generate the ABI using a tool like abigen or solc, and you can get the contract address from a blockchain explorer or by deploying the contract yourself.


### Run your project
```bash
cargo run --color=always --package celo_web3 --bin celo_web3
```

The run your program, we can easily see that we have been able to interact seamlessly with our deployed smart contract.

![image|690x147](./screenshot-6.png)

## About the Author

I am a Software Engineer, Tech Evangelist (Preaching the gospel of flutter & blockchain) also and Ex-GDSC Leads.

## References

- [Github Repo](https://github.com/Mujhtech/rust_web3)
- [Tokio crate package](https://crates.io/crates/tokio)
- [web3 crate package](https://crates.io/crates/web3)
- [Get started with rust](https://www.rust-lang.org/learn/get-started)
- [Learn solidity by example](https://solidity-by-example.org)
