# Loosely Specified Smart Contracts (LSP) Framework

An experimental architecture developed to address the rigidity of traditional Ethereum Smart Contracts by enabling **Loosely Specified Processes (LSP)**. This project allows for runtime logic adaptation, dynamic function routing, and on-chain contract deployment using low-level Solidity Assembly.

🔗 **[Read the full Project Report (PDF)](./docs/Project_Report_LSP.pdf)**

## ⚡ Key Features

* **Runtime Deployment**: Inject and deploy arbitrary bytecode directly on-chain using the `create` opcode, bypassing standard deployment tools.
* **Dynamic Routing**: Route calls to specific functions based on runtime conditions using optimized Selectors and `staticcall`.
* **Gas Optimization**: Extensive use of Solidity Assembly (`mload`, memory management) to reduce execution costs.
* **Input Decoding**: Handle dynamic input types (strings, integers, booleans) from raw byte streams.

## 📂 Project Structure

### Core Architecture (`/contracts/core`)

* **`ContractDeployer.sol`**: The engine of the framework. It accepts raw bytecode and deploys a new contract instance at runtime. It includes error handling to revert if deployment fails (e.g., zero code size).
* **`DynamicSelector.sol`**: Implements dynamic routing by using assembly to extract the function selector from the first 4 bytes of the calldata (`mload(add(data, 0x20))`) and dispatching execution.
* **`OptimizedSelector.sol`**: An evolution of the selector logic that utilizes `address(this).staticcall(data)` to execute functions dynamically, improving gas efficiency and code readability.
* **`InputDecoder.sol`**: A utility contract to decode raw `bytes[]` arrays into readable Solidity types (`string`, `int`, `bool`) at runtime, essential for handling unspecified input formats.

### Utilities & Examples (`/contracts/utils`, `/contracts/examples`)

* **`ABICoderUtils.sol`**: A helper tool to encode parameters (such as `uint256` for constructors) into ABI format, preparing them for injection into the deployer.
* **`SimpleAdder.sol`**: A basic contract used to verify the runtime deployment capability.
* **`SimpleStorage.sol`**: A stateful contract with a constructor, used to test the deployment of complex logic requiring initialization parameters.

## 🛠️ How it Works

The framework operates through a three-step pipeline designed to handle unspecified processes at runtime, leveraging low-level EVM manipulations:

### 1. Dynamic Deployment

The `ContractDeployer` bypasses standard Solidity deployment tools to instantiate contracts programmatically. It accepts raw bytecode and uses the `create` assembly opcode to deploy a new contract instance on the Ethereum network immediately:

```solidity
// From ContractDeployer.sol
assembly {
    // create(value, offset, length)
    // Deploys a new contract from memory without external transaction tools
    deployedAddress := create(0, add(bytecode, 0x20), mload(bytecode))
}
```

### 2. Runtime Routing (Selector Extraction)

Instead of relying on hardcoded function calls, the system parses incoming raw byte streams. The `DynamicSelector` uses Assembly to manually extract the 4-byte function signature (selector) directly from the calldata memory pointer:

```solidity
// From DynamicSelector.sol
bytes4 selector;
assembly {
    // Load the first 32 bytes starting after the length prefix to isolate the selector
    selector := mload(add(data, 0x20))
}

// Route execution dynamically based on the extracted selector
if (selector == this.A.selector) { ... }
```

### 3. Logic Injection & Parameter Encoding

For contracts requiring initialization (e.g., constructors with arguments), the `ABICoderUtils` prepares the payload. The system allows you to encode parameters (like `uint256` or `strings`) and concatenate them with the target contract's bytecode before injection. This ensures the new contract is initialized correctly with dynamic data known only at runtime.
