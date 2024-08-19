# Uniswap V3 and Aave DeFi Script

## Overview

This project demonstrates a DeFi script that interacts with Uniswap V3 to swap USDC for LINK and then deposits the LINK into Aave to start earning interest. The entire process is executed on the Ethereum Sepolia testnet.

### Features:
- **Token Swap:** Swaps USDC for LINK using Uniswap V3.
- **Aave Integration:** Deposits the swapped LINK into Aave for interest earning.
- **Public Testnet Execution:** The script is deployed and tested on the Ethereum Sepolia testnet.

## Project Structure

- **`index.js`**: Main script where the logic for token swapping and Aave deposit is implemented.
- **ABI Files (`*.json`)**:
  - **`factory.json`**: ABI for the Uniswap Factory contract.
  - **`lendingpool.json`**: ABI for the Aave Lending Pool contract.
  - **`pool.json`**: ABI for the Uniswap Pool contract.
  - **`swaprouter.json`**: ABI for the Uniswap Swap Router contract.
  - **`token.json`**: ABI for the ERC20 Token contract.
- **`.env`**: Contains environment variables such as private keys and API keys (not included in the repository).
- **`package.json`**: Lists the dependencies required for the Node.js project.
- **`package-lock.json`**: Locks the versions of the dependencies used.

## Installation and Setup

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/yourusername/defi-script.git
   cd defi-script
   ```

2. **Install Dependencies:**
   ```bash
   npm install
   ```

3. **Set Up Environment Variables:**
   - Create a `.env` file in the root directory and include your private key, Infura project ID, and other necessary environment variables:
     ```
     PRIVATE_KEY=your_private_key_here
     INFURA_PROJECT_ID=your_infura_project_id_here
     AAVE_LENDING_POOL_ADDRESS=your_lending_pool_address_here
     ```

4. **Run the Script:**
   ```bash
   node index.js
   ```

## Code Explanation

### 1. Setup and Configuration

We start by loading the required packages and setting up the provider and wallet using the private key.

```javascript
const ethers = require('ethers');
require('dotenv').config();

const provider = new ethers.providers.JsonRpcProvider('https://sepolia.infura.io/v3/YOUR_INFURA_PROJECT_ID');
const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);
```

### 2. Uniswap V3 Token Swap

The script interacts with Uniswap V3 to swap USDC for LINK. The following section shows how to send the swap transaction:

```javascript
async function swapUSDCForLINK() {
    const uniswapRouter = new ethers.Contract(SWAP_ROUTER_ADDRESS, swapRouterAbi, wallet);
    const transaction = await uniswapRouter.exactInputSingle({
        tokenIn: USDC_ADDRESS,
        tokenOut: LINK_ADDRESS,
        fee: 3000,
        recipient: wallet.address,
        deadline: Math.floor(Date.now() / 1000) + 60 * 20,
        amountIn: ethers.utils.parseUnits('100', 6),
        amountOutMinimum: 0,
        sqrtPriceLimitX96: 0,
    });
    await transaction.wait();
    console.log('Token Swap Complete:', transaction.hash);
}
```

### 3. Aave Deposit Logic

After swapping USDC for LINK, the script proceeds to deposit the LINK into Aave. Here’s a detailed explanation of the Aave deposit process:

#### a. Lending Pool Contract

We interact with Aave’s LendingPool contract to deposit tokens. This is done by initializing the contract with its ABI and address:

```javascript
const lendingPool = new ethers.Contract(LENDING_POOL_ADDRESS, lendingPoolAbi, wallet);
```

#### b. Approving the Transfer

Before depositing tokens into Aave, we must approve the LendingPool contract to transfer our LINK tokens on our behalf. This is done using the `approve` method of the LINK token contract:

```javascript
const linkToken = new ethers.Contract(LINK_ADDRESS, linkTokenAbi, wallet);
const approvalTransaction = await linkToken.approve(LENDING_POOL_ADDRESS, ethers.utils.parseUnits('10', 18));
await approvalTransaction.wait();
console.log('Approval Transaction Confirmed!');
```

#### c. Depositing into Aave

Finally, we deposit the LINK tokens into Aave using the `deposit` method of the LendingPool contract. This method is essential for interacting with Aave:

```javascript
const depositTransaction = await lendingPool.deposit(LINK_ADDRESS, ethers.utils.parseUnits('10', 18), wallet.address, 0);
await depositTransaction.wait();
console.log('Deposit Transaction Confirmed!');
```

**Important Note:** When viewing this transaction on Etherscan, it should show the method name `deposit`. However, in some cases, the method may appear as a hexadecimal string (e.g., `0xe8eda9df`). This string represents the method identifier in the Ethereum ABI, which corresponds to the `deposit` function.

### 4. Running the Script

Once everything is set up, run the script to perform the token swap and deposit the tokens into Aave.

```bash
node index.js
```

## Diagram Illustration

The below diagram is to visualize the token swap and deposit process, it's the workflow of the script:

![Script Workflow](https://github.com/Amar5623/token_swap/blob/main/workflow_diagram.svg?raw=true)

## Testing

The script has been tested on the Ethereum Sepolia testnet. Ensure your wallet is funded with Sepolia ETH to cover gas fees before running the script.
