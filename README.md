# YieldHunt

Yieldhunt is a sample application demonstrating how to build a dapp using DeFindex 

(COMING SOON!)

**Resources**:
- [DeFindex Documentation](https://docs.defindex.io/)
- [DeFindex GitHub](https://github.com/paltalabs/defindex)

## Using the DeFindex SDK

### SDK Configuration
The DeFindex TypeScript SDK can be used to create and manage vaults. The SDK can be configured and initialized like this:

```ts
import { 
  DefindexSDK, 
  SupportedNetworks, 
  CreateDefindexVault,
  ...
} from '@defindex/sdk';

const sdk = new DefindexSDK({
  apiKey: <API_KEY>
});
```
Obtain an API key from the DeFindex team - request a key in their Discord server: [DeFindex](https://discord.gg/urV2Ysf7)

### Create Vault
Creating a new vault is a three step process. First define the configuration: 

```ts
const vaultConfig: CreateDefindexVault = {
  roles: {
    0: "G...", // Emergency Manager
    1: "G...", // Fee Receiver  
    2: "G...", // Vault Manager
    3: "G..."  // Rebalance Manager
  },
  vault_fee_bps: 100, // 1% fee (100 basis points)
  assets: [{
    address: "CDLZFC3SYJYDZT7K67VZ75HPJVIEUVNIXF47ZG2FB2RMQQVU2HHGCYSC", // XLM asset
    strategies: [{
      address: "CCSPRGGUP32M23CTU7RUAGXDNOHSA6O2BS2IK4NVUP5X2JQXKTSIQJKE", // Strategy contract example
      name: "xlm_blend_autocompound_fixed_xlm_usdc",
      paused: false
    }]
  }],
  name_symbol: { 
      name: //Max 20 characters
      symbol: // Tag
  },
  upgradable: true,
  caller: // Public key of the signer account
};
```
See strategy addresses here [DeFindex Strategy Addresses](https://github.com/paltalabs/defindex/tree/main/public)

Use the vault configuration to create a new vault using the SDK:

```ts
const createResponse = await sdk.createVault(vaultConfig, SupportedNetworks.TESTNET);
```
The `createVault()` function will return transaction data, including the XDR, and the next step is to submit the transaction to the network. 

If [Launchtube](https://launchtube.xyz) is used to submit the transaction to the network, it will look like this.

```ts
let createVaultResult = await server.send(createResponse.xdr!);
```
The vault's contract address can be retrieved from the returned transaction value:       

```ts
const xdrResponse = createVaultResult.returnValue; //"AAAA...";  // your XDR string

// Decode into ScVal
const ResultScVal = xdr.ScVal.fromXDR(xdrResponse, "base64");

// Convert into native JS
const vaultContractId = scValToNative(ResultScVal);
```
**Next step***:
Next steps are to first deposit funds into the vault, and then rebalance the vault to define how the deposited funds are distributed to the strategies. Read more about that process here: [Create a Vault](https://docs.defindex.io/wallet-developer/creating-a-defindex-vault).

## Smart Contract Functions
DeFindex provides an option to make contract calls to the vauls as an alternative to using the SDK or API, and that can be useful for managing the vault through smart contracts on Stellar.

In this project a user is creating the vault in the frontend, and providing the needed information, but management of the vault is done from smart contracts. 

### How to call vault contracts
An easy way to get an overview of the vault functions, use [Stellar.Expert](https://stellar.expert). First go to [Stellar.Expert](https://stellar.expert), select `Testnet` (assuming you are developing on Testnet), enter your vault's contract address and click the Interface tab.

Since calling the vault functions are just like any other cross-contract call in Stellar smart contracts, `invoke_contract()` - which is a standard function - will work well. In this example the vault's `balance()` function is called, it takes the vault's contract address as a parameter and returns the balance in the `i128` format:

```rust
let info_params = vec![&env, vault_address.into_val(&env)];

let result: i128 = env.invoke_contract(
  &vault_address,
  &Symbol::new(&env, "balance"),
  params
);

```


## Balance
The contract function `get_vault_balance()` can be used to get the current balance of the vault.

```rust
pub fn get_vault_balance(
  env: Env,
  vault_address: Address,
  from: Address
) -> i128
```

**Returns**: `i128`
- the vault's balance

## Supply
The contract function `get_vault_supply()` can be used to retrieve the total token supply managed by the vault, either invested or idle.

```rust
pub fn get_vault_supply(
  env: Env,
  vault_address: Address
) -> i128
```

**Returns**: `i128`
- amount of tokens/stablecoins

## Assets
The contract function `get_vault_assets()` retrieves the list of assets managed by the vault.

```rust
pub fn get_vault_assets(
  env: Env,
  vault_address: Address
) -> Vec<AssetStrategySet>
```

**Returns**: `Vec<AssetStrategySet>`
- returns a vector of strategies by token/currence
- strategies are defined by their `address`, `name` and `paused` status


## Report
The contract function `get_vault_report()` generates reports for all strategies in the vault, tracking their performance and fee accrual. 

```rust
pub fn get_vault_report(
  env: Env,
  vault_address: Address
) -> Vec<Report>
```
**Results**: `Vec<Report>`
- the report contains metrics about `gains_or_losses`, `locked_fee` and `prev_balance`


