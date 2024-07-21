# Unstoppable

There's a tokenized vault with a million DVT tokens deposited. It’s offering flash loans for free, until the grace period ends.

To catch any bugs before going 100% permissionless, the developers decided to run a live beta in testnet. There's a monitoring contract to check liveness of the flashloan feature.

Starting with 10 DVT tokens in balance, show that it's possible to halt the vault. It must stop offering flash loans.


# solution

**Scenario**: 

There is a tokenized vault containing one million DVT tokens. This vault offers flash loans for free until a grace period ends. The developers are testing the vault on a testnet with a monitoring contract to check the flash loan feature's functionality.

**Objective**:

Demonstrate how to halt the vault so it no longer offers flash loans, using an initial balance of 10 DVT tokens.

### Challenge 1: Unstoppable

**Goal**: 

Perform a Denial of Service (DOS) attack by exploiting a vulnerability in the `flashLoan` function.

### Vulnerability in the `flashLoan` Function

Here's the critical part of the code:
```solidity
uint256 balanceBefore = totalAssets();
if (convertToShares(totalSupply) != balanceBefore) revert InvalidBalance(); // enforce ERC4626 requirement
```

- **Function Explanation**:
  - **`totalAssets()`**: Returns the vault’s balance of DVT tokens.
  - **`convertToShares(totalSupply)`**: Calculates how many share tokens (`oDVT`) should be minted based on the total supply of vault tokens.

### Key Concepts

1. **ERC4626 Standard**:
   - Defines how tokenized vaults manage user deposits and calculate rewards.
   - Vault mints "share" tokens (`oDVT`) to represent deposits of the underlying asset (`DVT`).

2. **Accounting Functions**:
   - **`convertToShares()`**: Converts deposited assets (`DVT`) into share tokens (`oDVT`).
   - **`totalAssets()`**: Shows the total balance of the underlying asset in the vault.

### Issues in the Code

1. **Mismatch Condition**:
   - The check `(convertToShares(totalSupply) != balanceBefore)` ensures the vault's token supply matches the asset balance. 
   - If tokens are diverted or if there’s a discrepancy, this condition fails, making the `flashLoan` function inactive.

2. **Separate Accounting**:
   - The vault uses two separate accounting systems: one for the underlying tokens (`DVT`) and one for the share tokens (`oDVT`).

### The Attack

**Steps to Perform the DOS Attack**:

1. **Transfer Token**:
   - Manually transfer 1 DVT token to the vault to create an inconsistency.
   ```solidity
   function test_unstoppable() public checkSolvedByPlayer {
       // Transfer tokens to the vault to trigger the inconsistency
       token.transfer(address(vault), 1e18);
   }
   ```

**Result**:

- **Conflict Creation**:
  - **Before Transfer**:
    - `totalSupply` = 1,000,000
    - `balanceBefore` = 1,000,000
  - **After Transfer**:
    - `totalSupply` remains 1,000,000
    - `balanceBefore` becomes 1,000,001

  - This mismatch (`1,000,000 != 1,000,001`) causes the `flashLoan` function to revert due to `InvalidBalance()`.

- **DOS Attack**:
  - This failure prevents the `flashLoan` function from executing, effectively halting all flash loan operations in the vault.

By causing this conflict, you can exploit the vulnerability to disable the vault’s flash loan feature, achieving a successful DOS attack.
