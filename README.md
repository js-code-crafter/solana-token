# SolanaToken - A JavaScript Library for Solana SPL Token Management

Welcome to the **SolanaToken** npm package, a comprehensive library designed to simplify the interaction with the Solana blockchain's SPL tokens. It provides an easy-to-use interface for wallet management, token transfers, and interaction with different Solana clusters.

# Installation

You can install the **solana-token** package using npm. Run the following command in your terminal:

```bash
npm install solana-token
```

# Basic Usage

This package provides utility methods to interact with Solana\'s SPL token ecosystem. You can manage wallets, carry out token transfers, and interact with various clusters such as devnet, testnet, and mainnet.

To use this package, simply import it into your JavaScript application:

```js
const { splFactory, Spl, walletFactory, Wallet } = require("solana-token");
```

## Wallet Management

You can create or restore wallets using the provided methods. For example, to restore a wallet from a mnemonic:

```js
const wallet = Wallet.restoreFromMnemonic("your mnemonic seed phrase here");
```

## Token Management

### Create Token

```js
const token = await spl.createToken(wallet);
```

### Get Token Balance

```js
const balance = await spl.getTokenAccountBalance(accountPublicKey);
```

### Transfer Tokens

You can transfer SPL tokens between two wallets:

```js
await spl.transferToken(tokenPublicKey, fromWallet, toWallet, amount);
```

## Transaction Management

The library allows you to manage transactions easily:

### Begin a Transaction

```js
const transaction = spl.beginTransaction();
```

### End a Transaction

```js
const confirmation = await spl.endTransaction(transaction);
```

# Example Usage

Here’s a simple example of how to use the SolanaToken package with an Express.js application to manage wallets and transfer tokens:

```js
const express = require("express");
const router = express.Router();
const { splFactory, Spl, walletFactory, Wallet } = require("solana-token");
const { PublicKey } = require("@solana/web3.js");

router.get("/", async (req, res) => {
  const w1 = walletFactory.restoreFromMnemonic("your mnemonic here");
  const w2 = walletFactory.restoreFromMnemonic("another mnemonic here");
  const s = splFactory.create(splFactory.clusters().dev);

  // Airdrop some SOL to the wallets
  await s.getSomeSol(w1);
  await s.getSomeSol(w2);

  // Getting balances
  const w1Balance = await s.getSolBalance(w1);
  const w2Balance = await s.getSolBalance(w2);

  // Token actions began
  const tokenPublicKey = new PublicKey("your_token_address_here");
  const account1 = await s.getOrCreateTokenAccount(w1, tokenPublicKey);
  const account2 = await s.getOrCreateTokenAccount(w2, tokenPublicKey);

  // Transfer tokens
  await s.transferToken(tokenPublicKey, w1, w2, "0.1");

  res.send({
    w1Balance: w1Balance,
    w2Balance: w2Balance,
    account1: account1.tokenAccountPublicKey.toString(),
    account2: account2.tokenAccountPublicKey.toString(),
  });
});

module.exports = router;
```

# Wallet.js Documentation

## Wallet Class

Solana wallet management class for key generation and restoration.

### Methods

#### `create(pass = null, derivedPath = null)`

- **Description**: Creates new wallet with mnemonic and derived keypair
- **Parameters**:
  - `pass`: (Optional) Password for mnemonic encryption
  - `derivedPath`: (Optional) Custom derivation path (default: BIP44Change)
- **Returns**: Wallet instance

#### `restoreFromMnemonic(mnemonic, pass = null, derivedPath = null)`

- **Description**: Restores wallet from mnemonic phrase
- **Parameters**:
  - `mnemonic`: 24-word recovery phrase
  - `pass`: (Optional) Encryption password
  - `derivedPath`: (Optional) Derivation path
- **Returns**: Wallet instance

#### `restoreFromSeed(seed)`

- **Description**: Restores wallet from raw seed bytes
- **Parameters**:
  - `seed`: Uint8Array seed value
- **Returns**: Wallet instance

#### `restoreFromPrivateKey(privateKey)`

- **Description**: Restores wallet directly from private key
- **Parameters**:
  - `privateKey`: Uint8Array private key
- **Returns**: Wallet instance

### Properties

#### `json`

- Returns wallet data as JSON object containing:
  - mnemonic
  - publicKey (base58 string)
  - bn (public key bytes)
  - privateKey (string array format)

#### `mnemonic`

- Returns the 24-word mnemonic phrase (if available)

#### `seed`

- Returns raw seed bytes (Uint8Array)

#### `publicKey`

- Returns Solana PublicKey object

#### `privateKey`

- Returns raw private key bytes (Uint8Array)

#### `keypair`

- Returns complete Solana Keypair object

---

## WalletFactory Class

Factory pattern implementation for wallet creation.

### Methods

#### `create(pass, derivedPath)`

- Creates new wallet instance (mirrors Wallet.create())

#### `restoreFromMnemonic(mnemonic, pass, derivedPath)`

- Restores wallet from mnemonic (mirrors Wallet method)

#### `restoreFromSeed(seed)`

- Restores from seed bytes (mirrors Wallet method)

#### `restoreFromPrivateKey(privateKey)`

- Restores from private key (mirrors Wallet method)

---

# spl.js Documentation

## Spl Class

Main class for Solana SPL token operations and blockchain interactions.

### Core Methods

#### `connect(cluster = null)`

- **Description**: Connects to Solana cluster
- **Parameters**:
  - `cluster`: 'devnet', 'testnet' or 'mainnet-beta' (default: devnet)
- **Returns**: Spl instance

#### `getSomeSol(publicKey, amount = 1)`

- **Description**: Requests SOL airdrop
- **Parameters**:
  - `publicKey`: Recipient address
  - `amount`: SOL amount (default: 1)
- **Returns**: Promise<TransactionConfirmation>

#### `transferSol(fromKeypair, toPublicKey, solAmount)`

- **Description**: Transfers SOL between accounts
- **Parameters**:
  - `fromKeypair`: Sender keypair
  - `toPublicKey`: Recipient address
  - `solAmount`: Amount to send
- **Returns**: Promise<TransactionSignature>

### Token Management

#### `createToken(ownerKeypair, decimals = 9, hasFreezeAuthority = false)`

- **Description**: Creates new SPL token
- **Parameters**:
  - `ownerKeypair`: Token authority
  - `decimals`: Token precision
  - `hasFreezeAuthority`: Enable freeze authority
- **Returns**: Promise<{token, txInfo}>

#### `mint(tokenPublicKey, ownerKeypair, amount)`

- **Description**: Mints tokens to account
- **Parameters**:
  - `tokenPublicKey`: Token mint address
  - `ownerKeypair`: Mint authority
  - `amount`: Tokens to mint
- **Returns**: Promise<TransactionSignature>

#### `burn(tokenPublicKey, walletKeypair, amount)`

- **Description**: Burns tokens from account
- **Parameters**:
  - `tokenPublicKey`: Token mint address
  - `walletKeypair`: Burner account
  - `amount`: Tokens to burn
- **Returns**: Promise<TransactionSignature>

### Account Management

#### `getOrCreateTokenAccount(walletKeypair, tokenPublicKey)`

- **Description**: Gets/Creates associated token account
- **Parameters**:
  - `walletKeypair`: Owner keypair
  - `tokenPublicKey`: Token mint address
- **Returns**: Promise<{tokenAccountPublicKey, txInfo}>

#### `getTokenAccountBalance(tokenAccountPublicKey)`

- **Description**: Gets token account balance
- **Parameters**:
  - `tokenAccountPublicKey`: Token account address
- **Returns**: Promise<number> (human-readable amount)

### NFT Operations

#### `mintNFT(wallet, uri, name, symbol)`

- **Description**: Mints NFT with metadata
- **Parameters**:
  - `wallet`: Owner keypair
  - `uri`: Metadata URI
  - `name`: NFT name
  - `symbol`: Token symbol
- **Returns**: Promise<Metadata>

### Arweave Integration

#### `uploadFile(arweaveInstance, filePath)`

- **Description**: Uploads file to Arweave
- **Parameters**:
  - `arweaveInstance`: Arweave client
  - `filePath`: Local file path
- **Returns**: Promise<TransactionInfo>

#### `uploadMetadata(arweaveInstance, metadata)`

- **Description**: Uploads JSON metadata to Arweave
- **Parameters**:
  - `arweaveInstance`: Arweave client
  - `metadata`: JSON metadata object
- **Returns**: Promise<TransactionInfo>

### Utility Methods

#### `getAccountLink(accountPublicKey)`

- Returns Solscan URL for account inspection

#### `getTransactionCost(keypair, lamports = 10)`

- Estimates transaction fee cost
- Returns Promise<number> (fee in lamports)

#### `sleep(ms)`

- Utility for async delays

---

## SplFactory Class

Factory for creating pre-configured Spl instances.

### Methods

#### `create(cluster = null)`

- Creates Spl instance connected to specified cluster
- **Parameters**:
  - `cluster`: Target network cluster
- **Returns**: Spl instance

---

**The code is thoroughly documented. Refer to the code for additional details.**

# Contributing

We welcome contributions! If you'd like to enhance the functionality of the `solana-token` package, please fork the repository and submit a pull request. For large changes, please open an issue first to discuss what you would like to change.

# License

This project is licensed under the MIT License - see the LICENSE file for details.
