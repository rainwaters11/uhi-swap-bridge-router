## Multi-Hop Architecture & Flash Accounting

This router implements the advanced **Flash Accounting** pattern introduced in Uniswap v4. Unlike traditional routers that perform multiple transfers for every "hop" in a trade, this router treats a multi-pool path as a single atomic operation.

### How it Works
1. **The Unlock Window:** The router initiates a single `manager.unlock()` call for the entire trade path ($Token A \to Token B \to Token C$).
2. **Intermediate Netting:** Inside the `unlockCallback`, the router executes multiple `manager.swap()` calls. 
    - After Swap 1 ($A \to B$), the router is "owed" $B$ by the PoolManager.
    - After Swap 2 ($B \to C$), the router "owes" $B$ to the PoolManager.
    - Because both swaps happen within the same unlock window, the deltas for $Token B$ cancel each other out internally.
3. **The Final Settlement:** Using `manager.currencyDelta()`, the router identifies the only two currencies that have a non-zero balance: the original input ($Token A$) and the final output ($Token C$). It then performs exactly two transfers:
    - **Settle:** Sends the initial input from the user to the PoolManager.
    - **Take:** Sends the final output from the PoolManager to the user.


### Technical Benefits
- **Gas Savings:** By bypassing intermediate transfers, we significantly reduce the number of ERC-20 `transfer` and `transferFrom` calls, which are among the most expensive operations in a swap.
- **Slippage Security:** The router calculates the final `outputDelta` of the very last token in the path. If this amount is less than the user-defined `minAmountOut`, the router reverts the entire transaction before any tokens are transferred.
- **Atomic Integrity:** Because the PoolManager requires all deltas to be zero before the `unlock` window closes, this router ensures that no tokens are ever "leaked" or left stuck in the contract.

## Foundry

**Foundry is a blazing fast, portable and modular toolkit for Ethereum application development written in Rust.**

Foundry consists of:

- **Forge**: Ethereum testing framework (like Truffle, Hardhat and DappTools).
- **Cast**: Swiss army knife for interacting with EVM smart contracts, sending transactions and getting chain data.
- **Anvil**: Local Ethereum node, akin to Ganache, Hardhat Network.
- **Chisel**: Fast, utilitarian, and verbose solidity REPL.

## Documentation

https://book.getfoundry.sh/

## Usage

### Build

```shell
$ forge build
```

### Test

```shell
$ forge test
```

### Format

```shell
$ forge fmt
```

### Gas Snapshots

```shell
$ forge snapshot
```

### Anvil

```shell
$ anvil
```

### Deploy

```shell
$ forge script script/Counter.s.sol:CounterScript --rpc-url <your_rpc_url> --private-key <your_private_key>
```

### Cast

```shell
$ cast <subcommand>
```

### Help

```shell
$ forge --help
$ anvil --help
$ cast --help
```
