# Solana - Learn the Metaplex SDK by Minting an NFT

## 1

### --description--

In this project, you will learn how to use the Metaplex JavaScript SDK to mint a <dfn title="A digitally transferable asset whose supply is 1">Non-Fungible Token (NFT)</dfn> on the Solana blockchain. You will learn how to add metadata to your NFT, and how to upload your NFT somewhere it can be viewed by others.

For the duration of this project, you will be working in the `learn-the-metaplex-sdk-by-minting-an-nft/` directory.

Change into the above directory in a new bash terminal.

### --tests--

You can use `cd` to change into the `learn-the-metaplex-sdk-by-minting-an-nft/` directory.

```js
const cwdFile = await __helpers.getCWD();
const cwd = cwdFile.split('\n').filter(Boolean).pop();
assert.include(cwd, 'learn-the-metaplex-sdk-by-minting-an-nft');
```

## 2

### --description--

So far, you have interacted with the _System Program_, created and deployed your own program, and interacted with the _Token Program_. Now, you will interact with the _Metaplex Token Program_ to mint your NFT.

Starting from the same code as the previous project, you will change the the way the mint account is created, and the way the token is minted such that it is an NFT.

Create a new keypair in a file called `wallet.json`.

### --tests--

You should have a `wallet.json` file in the `learn-the-metaplex-sdk-my-minting-an-nft/` directory.

```js
const walletJsonExists = __helpers.fileExists(
  'learn-the-metaplex-sdk-by-minting-an-nft/wallet.json'
);
assert.isTrue(walletJsonExists, 'The `wallet.json` file should exist');
const walletJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/wallet.json'
  )
);
assert.isArray(
  walletJson,
  'The `wallet.json` file should be an array of numbers.\nRun `solana-keygen new --outfile wallet.json` to create a new keypair.'
);
```

## 3

### --description--

Start a local Solana cluster. Ensure the RPC URL is set to `http://localhost:8899`.

### --tests--

Your Solana config RPC URL should be set to `http://localhost:8899`.

```js
const command = `solana config get json_rpc_url`;
await new Promise(res => setTimeout(() => res(), 2000));
const { stdout, stderr } = await __helpers.getCommandOutput(command);
assert.include(
  stdout,
  'http://localhost:8899',
  'Try running `solana config set --url localhost`'
);
```

You should run `solana-test-validator` in a separate terminal.

```js
await new Promise(res => setTimeout(() => res(), 2000));
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 4

### --description--

Use the Solana CLI to get the public key of the wallet you created. Put this public key in the `env.WALLET_ADDRESS` property of the `package.json` file.

### --tests--

The `package.json` file should have a `env.WALLET_ADDRESS` property that is not empty.

```js
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);
assert.property(
  packageJson.env,
  'WALLET_ADDRESS',
  'The `package.json` file should have an `env.WALLET_ADDRESS` property.'
);
assert.notEmpty(
  packageJson.env.WALLET_ADDRESS,
  'The `env.WALLET_ADDRESS` property should have a value.'
);
```

The `env.WALLET_ADDRESS` property should match the public key of `wallet.json`.

```js
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
const command =
  'solana address -k learn-the-metaplex-sdk-by-minting-an-nft/wallet.json';
const { stdout, stderr } = await __helpers.getCommandOutput(command);
const walletAddress = stdout.trim();
console.debug('walletAddress', walletAddress, packageJson.env.WALLET_ADDRESS);
assert.notEmpty(
  walletAddress,
  'The `wallet.json` file should have a public key.'
);
assert.notEmpty(
  packageJson.env.WALLET_ADDRESS,
  'The `env.WALLET_ADDRESS` property should have a value.'
);
assert.equal(
  packageJson.env.WALLET_ADDRESS,
  walletAddress,
  'The `env.WALLET_ADDRESS` property should match the public key of `wallet.json`.'
);
```

## 5

### --description--

<!-- 1. Create mint account -->

Following the same steps to create a fungible token, now a mint account for your NFT needs to be created.

Seeing as NFT's are not divisible - you cannot own 0.1 of an NFT - the decimal argument of the `createMint` call needs to be changed. Within `spl-program/create-mint-account.js`, change the decimal place argument to `0`.

### --tests--

The `createMint` call within `spl-program/create-mint-account.js` should have a decimal place argument of `0`.

```js
const mintDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'mint';
});
assert.exists(mintDeclaration, 'A variable named `mint` should exist');
const mintAwaitExpression = mintDeclaration.declarations?.[0]?.init;
assert.equal(
  mintAwaitExpression.type,
  'AwaitExpression',
  'The `createMint` call should be awaited'
);
const createMintCallExpression = mintAwaitExpression.argument;
assert.equal(
  createMintCallExpression.callee.name,
  'createMint',
  'The `mint` variable should be initialised with the `createMint` function'
);
const createMintArguments = createMintCallExpression.arguments;
assert.equal(
  createMintArguments.length,
  5,
  'The `createMint` function should be called with 5 arguments'
);
const [
  connectionArgument,
  payerArgument,
  mintAuthorityArgument,
  freezeAuthorityArgument,
  decimalsArgument
] = createMintArguments;
assert.equal(
  connectionArgument.name,
  'connection',
  'The first argument to `createMint` should be `connection`'
);
assert.equal(
  payerArgument.name,
  'payer',
  'The second argument to `createMint` should be `payer`'
);
assert.equal(
  mintAuthorityArgument.name,
  'mintAuthority',
  'The third argument to `createMint` should be `mintAuthority`'
);
assert.equal(
  freezeAuthorityArgument.name,
  'freezeAuthority',
  'The fourth argument to `createMint` should be `freezeAuthority`'
);
assert.equal(
  decimalsArgument.value,
  0,
  'The fifth argument to `createMint` should be `0`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/spl-program/create-mint-account.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 6

### --description--

Another properties of NFT's is that there is only one of each NFT. This means that the `mintTo` call needs to be changed to mint only one token.

Within `spl-program/mint.js`, change the `amount` argument of the `mintTo` call to `1`.

### --tests--

The `mintTo` call within `spl-program/mint.js` should have an `amount` argument of `1`.

```js
const expressionStatement = babelisedCode
  .getExpressionStatements()
  .find(
    e =>
      e.expression.type === 'AwaitExpression' &&
      e.expression.argument?.callee?.name === 'mintTo'
  );
assert.exists(
  expressionStatement,
  'An `await mintTo(...)` expression should exist'
);
const mintToArguments = expressionStatement.expression.argument.arguments;
const [
  connectionArgument,
  payerArgument,
  mintAddressArgument,
  tokenAccountArgument,
  mintAuthorityArgument,
  amountArgument
] = mintToArguments;
assert.equal(
  connectionArgument?.name,
  'connection',
  'The first argument to `mintTo` should be `connection`'
);
assert.equal(
  payerArgument?.name,
  'payer',
  'The second argument to `mintTo` should be `payer`'
);
assert.equal(
  mintAddressArgument?.name,
  'mintAddress',
  'The third argument to `mintTo` should be `mintAddress`'
);
assert.equal(
  tokenAccountArgument?.name,
  'tokenAccount',
  'The fourth argument to `mintTo` should be `tokenAccount`'
);
assert.equal(
  mintAuthorityArgument?.name,
  'mintAuthority',
  'The fifth argument to `mintTo` should be `mintAuthority`'
);
assert.equal(
  amountArgument?.value,
  1,
  'The sixth argument to `mintTo` should be `1`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/spl-program/mint.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 7

### --description--

Once the NFT has been minted to an account, the mint authority needs to be removed from the mint account. The mint authority is the only account that can mint more tokens. If the mint authority is removed, then no more tokens can be minted.

Within `spl-program/mint.js`, import the `setAuthority` function from `@solana/spl-token`, to change the mint authority.

```typescript
setAuthority(
  connection: Connection,
  payer: Signer,
  account: PublicKey,
  currentAuthority: Signer | PublicKey,
  authorityType: AuthorityType, // Import this enum from @solana/spl-token!
  newAuthority: PublicKey | null
): Promise
```

Call and await the `setAuthority` function, passing in the `connection`, `payer`, `mintAddress`, `mintAuthority`, `AuthorityType.MintTokens`, and `null` arguments.

### --tests--

You should have `await setAuthority(connection, payer, mintAddress, mintAuthority, AuthorityType.MintTokens, null);` in `spl-program/mint.js`.

```js
const expressionStatement = babelisedCode
  .getExpressionStatements()
  .find(
    e =>
      e.expression.type === 'AwaitExpression' &&
      e.expression.argument?.callee?.name === 'setAuthority'
  );
assert.exists(
  expressionStatement,
  'An `await setAuthority(...)` expression should exist'
);
const setAuthorityArguments = expressionStatement.expression.argument.arguments;
const [
  connectionArgument,
  payerArgument,
  mintAddressArgument,
  mintAuthorityArgument,
  authorityTypeMemberExpressionArgument,
  nullArgument
] = setAuthorityArguments;
assert.equal(
  connectionArgument?.name,
  'connection',
  'The first argument to `setAuthority` should be `connection`'
);
assert.equal(
  payerArgument?.name,
  'payer',
  'The second argument to `setAuthority` should be `payer`'
);
assert.equal(
  mintAddressArgument?.name,
  'mintAddress',
  'The third argument to `setAuthority` should be `mintAddress`'
);
assert.equal(
  mintAuthorityArgument?.name,
  'mintAuthority',
  'The fourth argument to `setAuthority` should be `mintAuthority`'
);
assert.equal(
  authorityTypeMemberExpressionArgument?.object?.name,
  'AuthorityType',
  'The fifth argument to `setAuthority` should be `AuthorityType.MintTokens`'
);
assert.equal(
  authorityTypeMemberExpressionArgument?.property?.name,
  'MintTokens',
  'The fifth argument to `setAuthority` should be `AuthorityType.MintTokens`'
);
assert.equal(
  nullArgument?.type,
  'NullLiteral',
  'The sixth argument to `setAuthority` should be `null`'
);
```

You should import `setAuthority` from `@solana/spl-token`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@solana/spl-token';
});
assert.exists(
  importDeclaration,
  'An import from `@solana/spl-token` should exist'
);
const importSpecifiers = importDeclaration.specifiers.map(s => {
  return s.imported.name;
});
assert.include(
  importSpecifiers,
  'setAuthority',
  '`setAuthority` should be imported from `@solana/spl-token`'
);
```

You should import `AuthorityType` from `@solana/spl-token`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@solana/spl-token';
});
assert.exists(
  importDeclaration,
  'An import from `@solana/spl-token` should exist'
);
const importSpecifiers = importDeclaration.specifiers.map(s => {
  return s.imported.name;
});
assert.include(
  importSpecifiers,
  'AuthorityType',
  '`AuthorityType` should be imported from `@solana/spl-token`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/spl-program/mint.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 8

### --description--

With those small changes, you can now mint an NFT!

Run the following to create the mint account:

```bash
node spl-program/create-mint-account.js
```

**Note:** You should see an error (similar to below) in the terminal.

<details>
  <summary>Error</summary>

```bash
FailedToSendTransactionError: The transaction could not be sent successfully to the network. Please check the underlying error below for more details.

Source: RPC

Caused By: Error: failed to send transaction: Transaction simulation failed: Attempt to debit an account but found no record of a prior credit.
```

</details>

### --tests--

You should run `node spl-program/create-mint-account.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node spl-program/create-mint-account.js',
  'Try running `node spl-program/create-mint-account.js` in the terminal'
);
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 9

### --description--

Remember to airdrop some SOL to the wallet 🤦‍♂️

### --tests--

The `wallet.json` account should have at least 2 SOL.

```js
const { stdout } = await __helpers.getCommandOutput(
  `solana balance ./learn-the-metaplex-sdk-by-minting-an-nft/wallet.json`
);
const balance = stdout.trim()?.match(/\d+/)?.[0];
assert.isAtLeast(
  parseInt(balance),
  2,
  'Try running `solana airdrop 2 ./wallet.json`'
);
```

The validator should be running at `http://localhost:8899`.

```js
const command = `curl http://localhost:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 10

### --description--

You can now mint an NFT!

Run the following to create the mint account:

```bash
node spl-program/create-mint-account.js
```

### --tests--

You should run `node spl-program/create-mint-account.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node spl-program/create-mint-account.js',
  'Try running `node spl-program/create-mint-account.js` in the terminal'
);
```

The command should succeed.

```js
// Test terminal-out includes value that can be parsed as a PublicKey.
const terminalOut = await __helpers.getTerminalOutput();
const base58String = terminalOut.match(/[\w\d]{32,44}/)?.[0];
assert.exists(
  base58String,
  'The terminal output should include a base58 string'
);
try {
  console.log('TEST: ', terminalOut.match(/[\w\d]{32,44}/));
  const { PublicKey } = await import('@solana/web3.js');
  new PublicKey(base58String);
} catch (e) {
  assert.fail(e);
}
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 11

### --description--

Put the mint address in `env.MINT_ACCOUNT_ADDRESS` in the `package.json` file.

### --tests--

The `package.json` file should have a `env.MINT_ACCOUNT_ADDRESS` property.

```js
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);
assert.property(
  packageJson.env,
  'MINT_ACCOUNT_ADDRESS',
  'The `package.json` file should have an `env.MINT_ACCOUNT_ADDRESS` property.'
);
```

The `env.MINT_ACCOUNT_ADDRESS` property should match an NFT owned by `wallet.json`.

```js
const command =
  'solana address -k learn-the-metaplex-sdk-by-minting-an-nft/wallet.json';
const { stdout, stderr } = await __helpers.getCommandOutput(command);
const walletAddress = stdout.trim();

const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);

assert.equal(
  packageJson.env.WALLET_ADDRESS,
  walletAddress,
  'The `env.WALLET_ADDRESS` property should match the public key of `wallet.json`.'
);

try {
  const { Connection } = await import('@solana/web3.js');
  const { TOKEN_PROGRAM_ID } = await import('@solana/spl-token');
  const connection = new Connection('http://127.0.0.1:8899');

  const mintAccounts = await connection.getParsedProgramAccounts(
    TOKEN_PROGRAM_ID,
    {
      filters: [
        {
          dataSize: 82
        },
        {
          memcmp: {
            offset: 4,
            bytes: packageJson.env.WALLET_ADDRESS
          }
        }
      ]
    }
  );

  const mintAccount = mintAccounts.find(
    ({ pubkey }) => pubkey?.toBase58() === packageJson.env.MINT_ACCOUNT_ADDRESS
  );
  assert.exists(
    mintAccount,
    'The `env.MINT_ACCOUNT_ADDRESS` property should match an NFT owned by `wallet.json`.'
  );
} catch (e) {
  assert.fail(e);
}
```

## 12

### --description--

Run the following to create the token account:

```bash
node spl-program/create-token-account.js
```

### --tests--

You should run `node spl-program/create-token-account.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node spl-program/create-token-account.js',
  'Try running `node spl-program/create-token-account.js` in the terminal'
);
```

The command should succeed.

```js
// Test terminal-out includes value that can be parsed as a PublicKey.
const terminalOut = await __helpers.getTerminalOutput();
const base58String = terminalOut.match(/[\w\d]{32,44}/)?.[0];
const error = terminalOut.match('Error')?.[0];
assert.notExists(error);
assert.exists(
  base58String,
  'The terminal output should include a base58 string'
);
try {
  const { PublicKey } = await import('@solana/web3.js');
  new PublicKey(base58String);
} catch (e) {
  assert.fail(e, 'Public key should be printed to the terminal');
}
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 13

### --description--

Put the token account address in `env.TOKEN_ACCOUNT_ADDRESS` in the `package.json` file.

### --tests--

The `package.json` file should have a `env` property.

```js
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);
```

The `package.json` file should have a `env.TOKEN_ACCOUNT_ADDRESS` property.

```js
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson.env,
  'TOKEN_ACCOUNT_ADDRESS',
  'The `package.json` file should have an `env` property.'
);
assert.notEmpty(packageJson.env.TOKEN_ACCOUNT_ADDRESS);
```

The `env.TOKEN_ACCOUNT_ADDRESS` property should match an NFT owned by `wallet.json`.

```js
const command =
  'solana address -k learn-the-metaplex-sdk-by-minting-an-nft/wallet.json';
const { stdout, stderr } = await __helpers.getCommandOutput(command);
const walletAddress = stdout.trim();

const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);

assert.equal(
  packageJson.env.WALLET_ADDRESS,
  walletAddress,
  'The `env.WALLET_ADDRESS` property should match the public key of `wallet.json`.'
);

try {
  const { Connection } = await import('@solana/web3.js');
  const { TOKEN_PROGRAM_ID } = await import('@solana/spl-token');
  const connection = new Connection('http://127.0.0.1:8899');

  const tokenAccounts = await connection.getParsedProgramAccounts(
    TOKEN_PROGRAM_ID,
    {
      filters: [
        {
          dataSize: 165
        },
        {
          memcmp: {
            offset: 32,
            bytes: packageJson.env.WALLET_ADDRESS
          }
        }
      ]
    }
  );

  const tokenAccount = tokenAccounts.find(
    ({ pubkey }) => pubkey.toBase58() === packageJson.env.TOKEN_ACCOUNT_ADDRESS
  );
  assert.exists(
    tokenAccount,
    'The `env.TOKEN_ACCOUNT_ADDRESS` property should match an NFT owned by `wallet.json`.'
  );
  assert.equal(
    tokenAccount.account.data.parsed.info.mint,
    packageJson.env.MINT_ACCOUNT_ADDRESS,
    'The `env.TOKEN_ACCOUNT_ADDRESS` should be associated with the `env.MINT_ACCOUNT_ADDRESS`.'
  );
} catch (e) {
  assert.fail(e);
}
```

## 14

### --description--

Run the following to mint the NFT to the token account:

```bash
node spl-program/mint.js
```

### --tests--

You should run `node spl-program/mint.js` in the terminal.

```js
// Test command is run in correct directory
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node spl-program/mint.js',
  'Try running `node spl-program/mint.js` in the terminal'
);
```

The command should succeed.

```js
// Test authority is null
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);

assert.notEmpty(
  packageJson.env.MINT_ACCOUNT_ADDRESS,
  'The `env.MINT_ACCOUNT_ADDRESS` property should be set.'
);

try {
  const { Connection, PublicKey } = await import('@solana/web3.js');
  const { getMint } = await import('@solana/spl-token');
  const connection = new Connection('http://127.0.0.1:8899');

  const mint = await getMint(
    connection,
    new PublicKey(packageJson.env.MINT_ACCOUNT_ADDRESS)
  );
  assert.equal(mint.mintAuthority, null, 'The mint authority should be null.');
} catch (e) {
  assert.fail(e);
}
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 15

### --description--

Run the following to confirm the NFT has no mint authority, and has a total supply of 1:

```bash
node spl-program/get-token-info.js
```

### --tests--

You should run `node spl-program/get-token-info.js` in the terminal.

```js
// Test command is run in correct directory
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node spl-program/get-token-info.js',
  'Try running `node spl-program/get-token-info.js` in the terminal'
);
```

The command should succeed.

```js
// Script should be idempotent
const command = `node spl-program/get-token-info.js`;
const { stdout, stderr } = await __helpers.getCommandOutput(
  command,
  'learn-the-metaplex-sdk-by-minting-an-nft'
);
assert.include(stdout, 'mintAuthority: null');
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 16

### --description--

You often hear about NFTs being odd pictures of a cat, but what if you wanted to associate more information with the NFT? For example, you could associate a name, description, and image with the NFT. This is where metadata comes in.

The metadata file is stored on IPFS, and the NFT is associated with the metadata file by storing the IPFS hash in the NFT's data field.

There are many accounts and files involved in the process of creating an NFT with metadata. Metaplex has a JavaScript SDK which provides common functionality for creating NFTs with metadata.

Install version `0.18.1` of `@metaplex-foundation/js`.

### --tests--

You should install `@metaplex-foundation/js` version `0.18.1`.

```js
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson.dependencies,
  '@metaplex-foundation/js',
  'The `package.json` file should have a `@metaplex-foundation/js` dependency.'
);
assert.equal(
  packageJson.dependencies['@metaplex-foundation/js'],
  '0.18.1',
  'Try running `npm install --save-exact @metaplex-foundation/js@0.18.1` in the terminal.'
);
```

## 17

### --description--

Create a file called `create-nft.js`.

### --tests--

You should create a file called `create-nft.js`.

```js
const fileExists = __helpers.fileExists(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
assert.isTrue(
  fileExists,
  'The `learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js` file should exist.'
);
```

## 18

### --description--

Within `create-nft.js`, create a `connection` variable set to a new `Connection` of your local Solana validator.

### --tests--

You should have `const connection = new Connection('http://127.0.0.1:8899');` in `create-nft.js`.

```js
const connectionVariableDeclaration = babelisedCode
  .getVariableDeclarations()
  .find(v => v.declarations?.[0]?.id?.name === 'connection');
assert.exists(
  connectionVariableDeclaration,
  'You should declare a variable named `connection`'
);
const newExpression = connectionVariableDeclaration.declarations[0].init;
assert.equal(
  newExpression.callee.name,
  'Connection',
  'You should initialise `connection` with a new `Connection`'
);
assert.equal(
  newExpression.arguments[0].value,
  'http://127.0.0.1:8899',
  "You should create a new connection with `new Connection('http://127.0.0.1:8899')`"
);
```

You should import `Connection` from `@solana/web3.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@solana/web3.js';
});
assert.exists(importDeclaration, 'You should import from `@solana/web3.js`');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'Connection',
  '`Connection` should be imported from `@solana/spl-token`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 19

### --description--

Import the `Metaplex` class from the Metaplex SDK, create a `metaplex` variable, and set it to:

```js
Metaplex.make();
```

### --tests--

You should have `const metaplex = Metaplex.make();` in `create-nft.js`.

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const variableDeclaration = babelisedCode
  .getVariableDeclarations()
  .find(v => v.declarations?.[0]?.id?.name === 'metaplex');
assert.exists(
  variableDeclaration,
  'You should declare a variable named `metaplex`'
);
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(minifiedCode, /metaplex=Metaplex\.make\(\)/);
```

You should import `Metaplex` from `@metaplex-foundation/js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@metaplex-foundation/js';
});
assert.exists(
  importDeclaration,
  'An import from `@metaplex-foundation/js` should exist'
);
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'Metaplex',
  '`Metaplex` should be imported from `@metaplex-foundation/js`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 20

### --description--

The `make` method on the `Metaplex` class expects a `Connection` that will be used to communicate with the cluster.

Pass the `connection` variable to the `make` method.

### --tests--

You should have `const metaplex = Metaplex.make(connection);` in `create-nft.js`.

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'metaplex';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(minifiedCode, /metaplex=Metaplex\.make\(connection\)/);
```

## 21

### --description--

The `Metaplex` class has chainable `use` methods that allow you to configure the `Metaplex` instance with `MetaplexPlugin` implementations.

Configure the `Metaplex` instance to use the wallet keypair as the primary identity for transactions:

```js
Metaplex.make(connection).use(keypairIdentity(WALLET_KEYPAIR));
```

The `keypairIdentity` function takes a `Keypair` and returns a `MetaplexPlugin`, and is exported from `@metaplex-foundation/js`.

The `WALLET_KEYPAIR` variable is a `Keypair` created from the `wallet.json` file, and is already exported from `utils.js`.

### --tests--

You should have `const metaplex = Metaplex.make(connection).use(keypairIdentity(WALLET_KEYPAIR));` in `create-nft.js`.

```js
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'metaplex';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /metaplex=Metaplex\.make\(connection\)\.use\(keypairIdentity\(WALLET_KEYPAIR\)\)/
);
```

You should import `keypairIdentity` from `@metaplex-foundation/js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@metaplex-foundation/js';
});
assert.exists(
  importDeclaration,
  'An import from `@metaplex-foundation/js` should exist'
);
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'keypairIdentity',
  '`keypairIdentity` should be imported from `@metaplex-foundation/js`'
);
```

You should import `WALLET_KEYPAIR` from `utils.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === './utils.js';
});
assert.exists(importDeclaration, 'An import from `./utils.js` should exist');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'WALLET_KEYPAIR',
  '`WALLET_KEYPAIR` should be imported from `./utils.js`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 22

### --description--

The final configuration required is the _storage driver_ that will be used to store the NFT's metadata. This is because the NFT's metadata is not stored on-chain, but rather in a separate, often cheaper, storage system.

Some common metadata storage locations are:

- IPFS
- Arweave
- AWS

For the purpose of testing, and to avoid having to pay for storage, you can use the `localStorage` driver which will generate random URLs and keep track of their content in a local server.

Configure the `Metaplex` instance to use the `localStorage` driver.

### --tests--

You should have `const metaplex = Metaplex.make(connection).use(keypairIdentity(WALLET_KEYPAIR)).use(localStorage());` in `create-nft.js`.

```js
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'metaplex';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /metaplex=Metaplex\.make\(connection\)\.use\(keypairIdentity\(WALLET_KEYPAIR\)\)\.use\(localStorage\(\)\)/
);
```

You should import `localStorage` from `utils.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === './utils.js';
});
assert.exists(importDeclaration, 'An import from `./utils.js` should exist');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'localStorage',
  '`localStorage` should be imported from `./utils.js`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 23

### --description--

The storage driver takes an object with a `baseUrl` property, which is the base URL that will be used to generate the metadata URLs.

Set the `baseUrl` to `http://127.0.0.1:3001/`.

### --tests--

You should have `const metaplex = Metaplex.make(connection).use(keypairIdentity(WALLET_KEYPAIR)).use(localStorage({ baseUrl: 'http://127.0.0.1:3001/' }));` in `create-nft.js`.

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'metaplex';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /metaplex=Metaplex\.make\(connection\)\.use\(keypairIdentity\(WALLET_KEYPAIR\)\)\.use\(localStorage\(\{('|"|`)?baseUrl\1:('|"|`)http:\/\/127\.0\.0\.1:3001\/\2\}\)\)/
);
```

## 24

### --description--

You can use any image you want for your NFT, but one is provided for you in the `assets` folder.

Declare a variable `imageBuffer`, and set it to the contents of the `assets/pic.png` file. You can use the `fs/promises` module to read the file.

### --tests--

You can use `const imageBuffer = await readFile('assets/pic.png');` in `create-nft.js`.

```js
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'imageBuffer';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /imageBuffer=(await )?(readFile|readFileSync|read)\(('|"|`)(\.\/)?assets\/pic\.png\3\)/
);
```

You should import one of the file-reading API.

```js
// one of fs, fs/promises
// one of readFileSync | readFile | read
const importDeclarations = babelisedCode.getImportDeclarations().filter(i => {
  return i.source.value === 'fs' || i.source.value === 'fs/promises';
});
assert.notEmpty(
  importDeclarations,
  'An import from `fs` or `fs/promises` should exist'
);
const importSpecifiers = importDeclarations.flatMap(i =>
  i.specifiers.map(s => s.imported.name)
);
const fileReadingApis = ['readFile', 'readFileSync', 'read'];
let found = false;
out: {
  for (const i of importSpecifiers) {
    for (const f of fileReadingApis) {
      if (i === f) {
        found = true;
        break out;
      }
    }
  }
}
assert.isTrue(found, 'One of the file-reading APIs should be imported');
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 25

### --description--

The `@metaplex-foundation/js` module exports a `toMetaplexFile` function:

```typescript
toMetaplexFile(
  content: MetaplexFileContent,
  fileName: string
): MetaplexFile
```

The `MetaplexFile` type contains useful metadata about the file which is useful for your NFT's metadata.

Declare a `file` variable, and set it to the result of calling `toMetaplexFile` with the `imageBuffer` and the filename `pic.png`.

### --tests--

You should have `const file = toMetaplexFile(imageBuffer, 'pic.png');` in `create-nft.js`.

```js
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'file';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /file=toMetaplexFile\(imageBuffer,('|"|`)?pic\.png\1\)/
);
```

You should import `toMetaplexFile` from `@metaplex-foundation/js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@metaplex-foundation/js';
});
assert.exists(
  importDeclaration,
  'An import from `@metaplex-foundation/js` should exist'
);
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'toMetaplexFile',
  'The `toMetaplexFile` function should be imported'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 26

### --description--

The `Metaplex` class has a `storage` method that returns the storage driver configured earlier. The storage driver has an `upload` method that takes a `MetaplexFile` and returns a `Promise` that resolves to a `string` containing the URL of the uploaded file.

Declare an `image` variable, and set it to the result of awaiting `metaplex.storage().upload(file)`.

### --tests--

You should have `const image = await metaplex.storage().upload(file);` in `create-nft.js`.

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'image';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /\s+image=await metaplex\.storage\(\)\.upload\(file\)/
);
```

## 27

### --description--

With the URL to the uploaded image, you can upload the NFT's metadata. The `Metaplex` class has an `nfts` method that returns many useful methods for working with NFT's. One of these methods is `uploadMetadata`:

<details>
  <summary><code>uploadMetadata</code> Signature</summary>

```typescript
uploadMetadata(
  input: {
    name?: string;
    symbol?: string;
    description?: string;
    seller_fee_basis_points?: number;
    image?: MetaplexFile;
    external_url?: MetaplexFile;
    attributes?: Array<{
      trait_type?: string;
      value?: string;
      [key: string]: unknown;
    }>;
    properties?: {
      creators?: Array<{
        address?: string;
        share?: number;
        [key: string]: unknown;
      }>;
      files?: Array<{
        type?: string;
        uri?: MetaplexFile;
        [key: string]: unknown;
      }>;
      [key: string]: unknown;
    };
    collection?: {
      name?: string;
      family?: string;
      [key: string]: unknown;
    };
    [key: string]: unknown;
  }
): Promise
```

</details>

That is a lot of metadata! For now, you can just set the `name`, `description`, and `image` properties.

Destructure the `uri` property from the result of awaiting `uploadMetadata`. Pass the `image` variable as the `image` property, and give the `name` and `description` properties some sane values.

### --tests--

You should have `const { uri } = await metaplex.nfts().uploadMetadata({ name: 'any string', description: 'any string', image });` in `create-nft.js`.

```js
const uriVariableDeclaration = babelisedCode
  .getVariableDeclarations()
  .find(v => {
    return v.declarations?.[0]?.id?.properties?.find(p => p.key.name === 'uri');
  });
assert.exists(
  uriVariableDeclaration,
  'A variable declaration with a `uri` property should exist'
);
const codeString = babelisedCode.generateCode(uriVariableDeclaration);

const metaplex = `
const metaplex = {
  nfts: () => ({
    uploadMetadata: ({name, description, image}) => {
        if (typeof name !== 'string') {
            throw new Error('name must be a string');
        }
        if (typeof description !== 'string') {
            throw new Error('description must be a string');
        }
        return Promise.resolve({ uri: 'any string' });
      }
  })
};
let image, name, description;
`;
const testString = `${metaplex}\n${codeString}`;

try {
  await eval(`(async () => {${testString}})()`);
} catch (e) {
  assert.fail(e.message);
}
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 28

### --description--

Another method on the `nfts` object is `create`:

<details>
  <summary><code>create</code> Signature</summary>

````typescript
create(
  input: {
    /**
     * The authority that will be able to make changes
     * to the created NFT.
     *
     * This is required as a Signer because creating the master
     * edition account requires the update authority to sign
     * the transaction.
     *
     * @defaultValue `metaplex.identity()`
     */
    updateAuthority?: Signer;
    /**
     * The authority that is currently allowed to mint new tokens
     * for the provided mint account.
     *
     * Note that this is only relevant if the `useExistingMint` parameter
     * if provided.
     *
     * @defaultValue `metaplex.identity()`
     */
    mintAuthority?: Signer;
    /**
     * The address of the new mint account as a Signer.
     * This is useful if you already have a generated Keypair
     * for the mint account of the NFT to create.
     *
     * @defaultValue `Keypair.generateCode()`
     */
    useNewMint?: Signer;
    /**
     * The address of the existing mint account that should be converted
     * into an NFT. The account at this address should have the right
     * requirements to become an NFT, e.g. its supply should contains
     * exactly 1 token.
     *
     * @defaultValue Defaults to creating a new mint account with the
     * right requirements.
     */
    useExistingMint?: PublicKey;
    /**
     * Whether or not we should mint one token for the new NFT.
     *
     * @defaultValue `true`
     */
    mintTokens?: boolean;
    /**
     * The owner of the NFT to create.
     *
     * @defaultValue `metaplex.identity().publicKey`
     */
    tokenOwner?: PublicKey;
    /**
     * The token account linking the mint account and the token owner
     * together. By default, the associated token account will be used.
     *
     * If the provided token account does not exist, it must be passed as
     * a Signer as we will need to create it before creating the NFT.
     *
     * @defaultValue Defaults to creating a new associated token account
     * using the `mintAddress` and `tokenOwner` parameters.
     */
    tokenAddress?: PublicKey | Signer;
    /**
     * Describes the asset class of the token.
     * It can be one of the following:
     * - `TokenStandard.NonFungible`: A traditional NFT (master edition).
     * - `TokenStandard.FungibleAsset`: A fungible token with metadata that can also have attrributes.
     * - `TokenStandard.Fungible`: A fungible token with simple metadata.
     * - `TokenStandard.NonFungibleEdition`: A limited edition NFT "printed" from a master edition.
     * - `TokenStandard.ProgrammableNonFungible`: A master edition NFT with programmable configuration.
     *
     * @defaultValue `TokenStandard.NonFungible`
     */
    tokenStandard?: TokenStandard;
    /** The URI that points to the JSON metadata of the asset. */
    uri: string;
    /** The on-chain name of the asset, e.g. "My NFT #123". */
    name: string;
    /**
     * The royalties in percent basis point (i.e. 250 is 2.5%) that
     * should be paid to the creators on each secondary sale.
     */
    sellerFeeBasisPoints: number;
    /**
     * The on-chain symbol of the asset, stored in the Metadata account.
     * E.g. "MYNFT".
     *
     * @defaultValue `""`
     */
    symbol?: string;
    /**
     * {@inheritDoc CreatorInput}
     * @defaultValue
     * Defaults to using the provided `updateAuthority` as the only verified creator.
     * ```ts
     * [{
     *   address: updateAuthority.publicKey,
     *   authority: updateAuthority,
     *   share: 100,
     * }]
     * ```
     */
    creators?: CreatorInput[];
    /**
     * Whether or not the NFT's metadata is mutable.
     * When set to `false` no one can update the Metadata account,
     * not even the update authority.
     *
     * @defaultValue `true`
     */
    isMutable?: boolean;
    /**
     * Whether or not selling this asset is considered a primary sale.
     * Once flipped from `false` to `true`, this field is immutable and
     * all subsequent sales of this asset will be considered secondary.
     *
     * @defaultValue `false`
     */
    primarySaleHappened?: boolean;
    /**
     * The maximum supply of printed editions.
     * When this is `null`, an unlimited amount of editions
     * can be printed from the original edition.
     *
     * @defaultValue `toBigNumber(0)`
     */
    maxSupply?: Option<BigNumber>;
    /**
     * When this field is not `null`, it indicates that the NFT
     * can be "used" by its owner or any approved "use authorities".
     *
     * @defaultValue `null`
     */
    uses?: Option<Uses>;
    /**
     * Whether the created NFT is a Collection NFT.
     * When set to `true`, the NFT will be created as a
     * Sized Collection NFT with an initial size of 0.
     *
     * @defaultValue `false`
     */
    isCollection?: boolean;
    /**
     * The Collection NFT that this new NFT belongs to.
     * When `null`, the created NFT will not be part of a collection.
     *
     * @defaultValue `null`
     */
    collection?: Option<PublicKey>;
    /**
     * The collection authority that should sign the created NFT
     * to prove that it is part of the provided collection.
     * When `null`, the provided `collection` will not be verified.
     *
     * @defaultValue `null`
     */
    collectionAuthority?: Option<Signer>;
    /**
     * Whether or not the provided `collectionAuthority` is a delegated
     * collection authority, i.e. it was approved by the update authority
     * using `metaplex.nfts().approveCollectionAuthority()`.
     *
     * @defaultValue `false`
     */
    collectionAuthorityIsDelegated?: boolean;
    /**
     * Whether or not the provided `collection` is a sized collection
     * and not a legacy collection.
     *
     * @defaultValue `true`
     */
    collectionIsSized?: boolean;
    /**
     * The ruleset account that should be used to configure the
     * programmable NFT.
     *
     * This is only relevant for programmable NFTs, i.e. if the
     * `tokenStandard` is set to `TokenStandard.ProgrammableNonFungible`.
     *
     * @defaultValue `null`
     */
    ruleSet?: Option<PublicKey>;
  }
): Promise
````

</details>

The only required properties are `name`, `uri`, and `sellerFeeBasisPoints`. On top of those, you should also provide a `maxSupply` of `1` if you want to create a limited edition NFT.

The `create` method will take care of creating the mint account, the associated token account, the metadata PDA and the original edition PDA (a.k.a. the master edition) for you.

Declare a variable `createResponse`, and assign to it the result of awaiting the `create` method. Pass in the previously mentioned properties with sane values.

### --tests--

You should have `const createResponse = await metaplex.nfts().create({ name: "any string", uri, sellerFeeBasisPoints: <any_int>, maxSupply: 1 });` in `create-nft.js`.

```js
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id.name === 'createResponse';
});
assert.exists(variableDeclaration, 'A variable `createResponse` should exist');

const codeString = babelisedCode.generateCode(variableDeclaration);

const metaplex = `
const metaplex = {
  nfts: () => ({
    create: ({name, sellerFeeBasisPoints, maxSupply}) => {
        if (typeof name !== 'string') {
            throw new Error('name must be a string');
        }
        if (typeof sellerFeeBasisPoints !== 'number') {
            throw new Error('sellerFeeBasisPoints must be a number');
        }
        if (typeof maxSupply !== 'number') {
            throw new Error('maxSupply must be a number');
        } else {
          if (maxSupply !== 1) {
            throw new Error('maxSupply must be 1');
          }
        }
        return Promise.resolve('createResponse');
      }
  })
};
let name, sellerFeeBasisPoints, maxSupply, uri;
`;
const testString = `${metaplex}\n${codeString}`;

try {
  await eval(`(async () => {${testString}})()`);
} catch (e) {
  assert.fail(e.message);
}
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 29

### --description--

Log the `createResponse` variable to the console.

### --tests--

You should have `console.log(createResponse);` in `create-nft.js`.

```js
const consoleLogCallExpression = babelisedCode
  .getType('CallExpression')
  .find(c => {
    return (
      c.callee.object?.name === 'console' && c.callee.property?.name === 'log'
    );
  });
assert.exists(consoleLogCallExpression, 'A `console.log` call should exist');
const consoleLogArguments = consoleLogCallExpression.arguments;
const ident = consoleLogArguments.find(a => {
  return a.name === 'createResponse';
});
assert.exists(
  ident,
  'One of the arguments to `console.log` should be `createResponse`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/create-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 30

### --description--

The local storage driver points to a simple REST API that stores the metadata on your local machine. It is useful for testing purposes.

Start the local storage driver in a new terminal:

```bash
npm run start:server
```

### --tests--

The local storage driver should be running at `http://127.0.0.1:3001`.

```js
try {
  const res = await fetch('http://127.0.0.1:3001/status/ping');
  // Response should be 200 with text "pong"
  if (res.status === 200) {
    const text = await res.text();
    if (text !== 'pong') {
      throw new Error(`Expected response text "pong", got ${text}`);
    }
  } else {
    throw new Error(`Expected status code 200, got ${res.status}`);
  }
} catch (e) {
  assert.fail(e);
}
```

## 31

### --description--

Run the script in the terminal:

```bash
node create-nft.js
```

**Note:** You should see an error (similar to below) in the terminal.

<details>
  <summary>Error</summary>

```bash
FailedToSendTransactionError: The transaction could not be sent successfully to the network. Please check the underlying error below for more details.

Source: RPC

Caused By: Error: failed to send transaction: Transaction simulation failed: Attempt to load a program that does not exist
```

</details>

### --tests--

You should run `node create-nft.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node create-nft.js',
  'Run `node create-nft.js` in the terminal'
);
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

The local storage driver should be running at `http://127.0.0.1:3001`.

```js
try {
  const res = await fetch('http://127.0.0.1:3001/status/ping');
  // Response should be 200 with text "pong"
  if (res.status === 200) {
    const text = await res.text();
    if (text !== 'pong') {
      throw new Error(`Expected response text "pong", got ${text}`);
    }
  } else {
    throw new Error(`Expected status code 200, got ${res.status}`);
  }
} catch (e) {
  assert.fail(e);
}
```

## 32

### --description--

The error comes about because the default Solana test validator does not come with the Metaplex Token program deployed.

So, the program needs to be manually deployed to the local cluster. To do this, first a dump of the program from mainnet needs to be created:

```bash
solana program dump --url mainnet-beta metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s ./mlp_token.so
```

<details>
  <summary>About the Command</summary>

The above command:

- Takes a dump of the program at the provided address
- Uses the mainnet-beta cluster (the cluster that the Metaplex Token program is deployed to)
- Outputs the dump to `mlp_token.so`

The address of the Metaplex Token program is `metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s`. You can find the address of any program you want by searching for it on the Solana Explorer:

`https://explorer.solana.com/address/metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s`

</details>

### --tests--

You should have a file called `mlp_token.so` in the root of the project.

```js
const fileExists = __helpers.fileExists(
  'learn-the-metaplex-sdk-by-minting-an-nft/mlp_token.so'
);
assert.isTrue(fileExists, 'A file called `mlp_token.so` should exist.');
```

## 33

### --description--

Deploy the Metaplex Token program to your local cluster:

```bash
solana program deploy --keypair wallet.json ./mlp_token.so
```

### --tests--

You should run `solana program deploy ./mlp_token.so` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'solana program deploy --keypair wallet.json ./mlp_token.so',
  'Run `solana program deploy ./mlp_token.so` in the terminal'
);
```

## 34

### --description--

Re-run the script to create the NFT.

**Note:** You should see an error in the terminal.

### --tests--

You should run `node create-nft.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node create-nft.js',
  'Run `node create-nft.js` in the terminal'
);
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;

const { stdout, stderr } = await __helpers.getCommandOutput(command);

try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

The local storage driver should be running at `http://127.0.0.1:3001`.

```js
try {
  const res = await fetch('http://127.0.0.1:3001/status/ping');
  // Response should be 200 with text "pong"
  if (res.status === 200) {
    const text = await res.text();
    if (text !== 'pong') {
      throw new Error(`Expected response text "pong", got ${text}`);
    }
  } else {
    throw new Error(`Expected status code 200, got ${res.status}`);
  }
} catch (e) {
  assert.fail(e);
}
```

## 35

### --description--

The error comes about because, internally, the Metaplex SDK is expecting a program address of `metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s`. However, when you deployed the `.so` file, it was deployed at a random public address.

Instead, you can start the local cluster with the program pre-deployed at a specific address:

```bash
solana-test-validator --bpf-program metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s ./mlp_token.so --reset
```

Stop your existing local cluster, and start a new one with the above command.

<details>
  <summary>About the Command</summary>

The `--bpf-program` flag is used to specify the address of the program. This is the same address that the program is deployed to on mainnet-beta. Internally, the Metaplex SDK uses this address in the transaction instructions to tell the cluster which program to use.

The `--reset` flag is used to clear the `test-ledger` directory. This is where the local cluster stores its data. If you don't clear the directory, the cluster will use the old data.

</details>

### --tests--

You should be in the `learn-the-metaplex-sdk-by-minting-an-nft` directory.

```js
const cwdFile = await __helpers.getCWD();
const cwd = cwdFile.split('\n').filter(Boolean).pop();
assert.include(cwd, 'learn-the-metaplex-sdk-by-minting-an-nft');
```

You should run `solana-test-validator --bpf-program metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s ./mlp_token.so --reset` in the terminal.

```js
await new Promise(res => setTimeout(() => res(), 2000));
// TODO: Tests do not watch `.temp.log`
const temp = await __helpers.getTemp();
console.log(temp.slice(0, 500));
assert.include(
  temp,
  'solana-test-validator --bpf-program metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s ./mlp_token.so --reset',
  'Run `solana-test-validator --bpf-program metaqbxxUerdq28cj1RbAWkYQm3ybzjb6a8bt518x1s ./mlp_token.so --reset` in the terminal'
);
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
await new Promise(res => setTimeout(() => res(), 2000));
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 36

### --description--

For ease, this command has been added to the `package.json` file as a script. Stop your existing local cluster, and start a new one with the following command:

```bash
npm run start:validator
```

**Note:** You might need to manually run the tests.

### --tests--

You should run `npm run start:validator` in the terminal.

```js
const lastCommand = await __helpers.getTemp();
assert.include(
  lastCommand,
  'npm run start:validator',
  'Run `npm run start:validator` in the terminal'
);
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 37

### --description--

Restarting the validator with the `--reset` flag deletes any previous ledger data. This means you will need to airdrop some SOL into the account associated with `wallet.json` again.

Do so.

### --tests--

The `wallet.json` account should have at least 2 SOL.

```js
const { stdout } = await __helpers.getCommandOutput(
  `solana balance ./learn-the-metaplex-sdk-by-minting-an-nft/wallet.json`
);
const balance = stdout.trim()?.match(/\d+/)?.[0];
assert.isAtLeast(
  parseInt(balance),
  2,
  'Try running `solana airdrop 2 ./wallet.json`'
);
```

The validator should be running at `http://localhost:8899`.

```js
const command = `curl http://localhost:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

## 38

### --description--

Create a new NFT again. Pay attention to the output in the terminal.

### --tests--

You should run `node create-nft.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(
  lastCommand?.trim(),
  'node create-nft.js',
  'Run `node create-nft.js` in the terminal'
);
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

The local storage driver should be running at `http://127.0.0.1:3001`.

```js
try {
  const res = await fetch('http://127.0.0.1:3001/status/ping');
  // Response should be 200 with text "pong"
  if (res.status === 200) {
    const text = await res.text();
    if (text !== 'pong') {
      throw new Error(`Expected response text "pong", got ${text}`);
    }
  } else {
    throw new Error(`Expected status code 200, got ${res.status}`);
  }
} catch (e) {
  assert.fail(e);
}
```

## 39

### --description--

Part of the output should be the mint account's public key. Copy the base58 string and paste it into the `env.MINT_ACCOUNT_ADDRESS` property of the `package.json` file.

_As this is a new NFT, the `env.MINT_ACCOUNT_ADDRESS` property will be different to the previous one._

### --tests--

The `package.json` file should have a `env.MINT_ACCOUNT_ADDRESS` property.

```js
const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);
assert.property(
  packageJson.env,
  'MINT_ACCOUNT_ADDRESS',
  'The `package.json` file should have an `env.MINT_ACCOUNT_ADDRESS` property.'
);
```

The `env.MINT_ACCOUNT_ADDRESS` property should match an NFT owned by `wallet.json`.

```js
const command =
  'solana address -k learn-the-metaplex-sdk-by-minting-an-nft/wallet.json';
const { stdout, stderr } = await __helpers.getCommandOutput(command);
const walletAddress = stdout.trim();

const packageJson = JSON.parse(
  await __helpers.getFile(
    'learn-the-metaplex-sdk-by-minting-an-nft/package.json'
  )
);
assert.property(
  packageJson,
  'env',
  'The `package.json` file should have an `env` property.'
);

assert.equal(
  packageJson.env.WALLET_ADDRESS,
  walletAddress,
  'The `env.WALLET_ADDRESS` property should match the public key of `wallet.json`.'
);

try {
  const { Connection } = await import('@solana/web3.js');
  const { TOKEN_PROGRAM_ID } = await import('@solana/spl-token');
  const connection = new Connection('http://127.0.0.1:8899');

  const mintAccounts = await connection.getParsedProgramAccounts(
    TOKEN_PROGRAM_ID,
    {
      filters: [
        {
          dataSize: 165
        },
        {
          memcmp: {
            offset: 32,
            bytes: packageJson.env.WALLET_ADDRESS
          }
        }
      ]
    }
  );

  const mintAccount = mintAccounts.find(
    ({ account }) =>
      account?.data?.parsed?.info?.mint === packageJson.env.MINT_ACCOUNT_ADDRESS
  );
  assert.exists(
    mintAccount,
    'The `env.MINT_ACCOUNT_ADDRESS` property should match an NFT owned by `wallet.json`.'
  );
} catch (e) {
  assert.fail(e);
}
```

## 40

### --description--

Now that your NFT is deployed, and you have the mint account's public key, you can always find it using the public key.

Create a new file called `get-nft.js`.

### --tests--

You should have a `get-nft.js` file.

```js
const fileExists = __helpers.fileExists(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
assert.isTrue(fileExists, 'You should have a `get-nft.js` file.');
```

## 41

### --description--

Within `get-nft.js`, declare a `connection` variable with a configuration pointing to your local cluster.

### --tests--

You should have `const connection = new Connection('http://127.0.0.1:8899');` in `get-nft.js`.

```js
const connectionVariableDeclaration = babelisedCode
  .getVariableDeclarations()
  .find(v => v.declarations?.[0]?.id?.name === 'connection');
assert.exists(
  connectionVariableDeclaration,
  'You should declare a variable named `connection`'
);
const newExpression = connectionVariableDeclaration.declarations[0].init;
assert.equal(
  newExpression.callee.name,
  'Connection',
  'You should initialise `connection` with a new `Connection`'
);
assert.equal(
  newExpression.arguments[0].value,
  'http://127.0.0.1:8899',
  "You should create a new connection with `new Connection('http://127.0.0.1:8899')`"
);
```

You should import `Connection` from `@solana/web3.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@solana/web3.js';
});
assert.exists(importDeclaration, 'You should import from `@solana/web3.js`');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'Connection',
  '`Connection` should be imported from `@solana/spl-token`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 42

### --description--

Within `get-nft.js`, declare a `metaplex` variable with the same configuration as the `metaplex` variable in `create-nft.js`.

### --tests--

You should have `const metaplex = Metaplex.make(connection).use(keypairIdentity(WALLET_KEYPAIR)).use(localStorage({ baseUrl: 'http://127.0.0.1:3001/' }));` in `get-nft.js`.

```js
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'metaplex';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /metaplex=Metaplex\.make\(connection\)\.use\(keypairIdentity\(WALLET_KEYPAIR\)\)\.use\(localStorage\(\{('|"|`)?baseUrl\1:('|"|`)http:\/\/127\.0\.0\.1:3001\/\2\}\)\)/
);
```

You should import `Metaplex` from `@metaplex-foundation/js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@metaplex-foundation/js';
});
assert.exists(
  importDeclaration,
  'You should import from `@metaplex-foundation/js`'
);
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'Metaplex',
  '`Metaplex` should be imported from `@metaplex-foundation/js`'
);
```

You should import `keypairIdentity` from `@metaplex-foundation/js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@metaplex-foundation/js';
});
assert.exists(
  importDeclaration,
  'You should import from `@metaplex-foundation/js`'
);
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'keypairIdentity',
  '`keypairIdentity` should be imported from `@metaplex-foundation/js`'
);
```

You should import `WALLET_KEYPAIR` from `utils.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === './utils.js';
});
assert.exists(importDeclaration, 'You should import from `./utils.js`');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'WALLET_KEYPAIR',
  '`WALLET_KEYPAIR` should be imported from `./utils.js`'
);
```

You should import `localStorage` from `utils.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === './utils.js';
});
assert.exists(importDeclaration, 'You should import from `./utils.js`');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'localStorage',
  '`localStorage` should be imported from `./utils.js`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 43

### --description--

Within `get-nft.js`, declare a `mintAddress` variable, and assign it the value of an instance of `PublicKey` constructed from the `MINT_ACCOUNT_ADDRESS` property from `pkg.env`.

**Hint:** The `pkg` object is exported from `utils.js`.

### --tests--

You should have `const mintAddress = new PublicKey(pkg.env.MINT_ACCOUNT_ADDRESS);` in `get-nft.js`.

```js
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'mintAddress';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /mintAddress=new PublicKey\(pkg\.env\.MINT_ACCOUNT_ADDRESS\)/
);
```

You should import `pkg` from `utils.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === './utils.js';
});
assert.exists(importDeclaration, 'You should import from `./utils.js`');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'pkg',
  '`pkg` should be imported from `./utils.js`'
);
```

You should import `PublicKey` from `@solana/web3.js`.

```js
const importDeclaration = babelisedCode.getImportDeclarations().find(i => {
  return i.source.value === '@solana/web3.js';
});
assert.exists(importDeclaration, 'You should import from `@solana/web3.js`');
const importSpecifiers = importDeclaration.specifiers.map(s => s.imported.name);
assert.include(
  importSpecifiers,
  'PublicKey',
  '`PublicKey` should be imported from `@solana/web3.js`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 44

### --description--

To get the NFT, use the `findByMint` method on the `Metaplex.nfts()` class:

```typescript
findByMint(
  {
    mintAddress: PublicKey;
    tokenAddress?: PublicKey;
    tokenOwner?: PublicKey;
    loadJsonMetadata?: boolean;
  }
): Promise
```

Pass in the required `mintAddress` property, and assign the awaited result to an `nft` variable.

### --tests--

You should have `const nft = await metaplex.nfts().findByMint({ mintAddress });` in `get-nft.js`.

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'nft';
});
assert.exists(variableDeclaration, 'An `nft` variable should exist');
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /nft=await metaplex\.nfts\(\)\.findByMint\(\{mintAddress\}\)/
);
```

## 45

### --description--

Log the `nft` variable to the console.

### --tests--

You should have `console.log(nft);` in `get-nft.js`.

```js
const consoleLogCallExpression = babelisedCode
  .getType('CallExpression')
  .find(c => {
    return (
      c.callee.object?.name === 'console' && c.callee.property?.name === 'log'
    );
  });
assert.exists(consoleLogCallExpression, 'A `console.log` call should exist');
const consoleLogArguments = consoleLogCallExpression.arguments;
const ident = consoleLogArguments.find(a => {
  return a.name === 'nft';
});
assert.exists(ident, 'One of the arguments to `console.log` should be `nft`');
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 46

### --description--

Run the `get-nft.js` script.

```bash
node get-nft.js
```

### --tests--

You should run `node get-nft.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(lastCommand?.trim(), 'node get-nft.js');
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

The local storage driver should be running at `http://127.0.0.1:3001`.

```js
try {
  const res = await fetch('http://127.0.0.1:3001/status/ping');
  // Response should be 200 with text "pong"
  if (res.status === 200) {
    const text = await res.text();
    if (text !== 'pong') {
      throw new Error(`Expected response text "pong", got ${text}`);
    }
  } else {
    throw new Error(`Expected status code 200, got ${res.status}`);
  }
} catch (e) {
  assert.fail(e);
}
```

## 47

### --description--

Notice the output includes a `json` property. This is the metadata for the NFT.

The metadata includes the image URL, but not the image data itself. To get the image data, you can use the `download` method on the `Metaplex.storage()` class:

```typescript
download(
  uri: string
): Promise<MetaplexFile>
```

Declare a variable `imageData`, and assign it the awaited result of calling the `download` method, passing in the `image` property of the NFT metadata.

### --tests--

You should have `const imageData = await metaplex.storage().download(nft.json.image);` in `get-nft.js`.

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
const variableDeclaration = babelisedCode.getVariableDeclarations().find(v => {
  return v.declarations?.[0]?.id?.name === 'imageData';
});
const minifiedCode = babelisedCode.generateCode(variableDeclaration, {
  minified: true
});
assert.match(
  minifiedCode,
  /imageData=await metaplex\.storage\(\)\.download\(nft\.json\.image\)/
);
```

## 48

### --description--

Log the `imageData` variable to the console.

### --tests--

You should have `console.log(imageData);` in `get-nft.js`.

```js
const consoleLogCallExpressions = babelisedCode
  .getType('CallExpression')
  .filter(c => {
    return (
      c.callee.object?.name === 'console' && c.callee.property?.name === 'log'
    );
  });
assert.exists(consoleLogCallExpressions, 'A `console.log` call should exist');
const is_ident = consoleLogCallExpressions.some(c => {
  const consoleLogArguments = c.arguments;
  const ident = consoleLogArguments.find(a => {
    return a.name === 'imageData';
  });
  return ident;
});
assert.exists(
  is_ident,
  'One of the arguments to `console.log` should be `imageData`'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 49

### --description--

Run the `get-nft.js` script.

### --tests--

You should run `node get-nft.js` in the terminal.

```js
const lastCommand = await __helpers.getLastCommand();
assert.equal(lastCommand?.trim(), 'node get-nft.js');
```

The validator should be running at `http://127.0.0.1:8899`.

```js
const command = `curl http://127.0.0.1:8899 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","id":1, "method":"getHealth"}'`;
const { stdout, stderr } = await __helpers.getCommandOutput(command);
try {
  const jsonOut = JSON.parse(stdout);
  assert.deepInclude(jsonOut, { result: 'ok' });
} catch (e) {
  assert.fail(e, 'Try running `solana-test-validator` in a separate terminal');
}
```

The local storage driver should be running at `http://127.0.0.1:3001`.

```js
try {
  const res = await fetch('http://127.0.0.1:3001/status/ping');
  // Response should be 200 with text "pong"
  if (res.status === 200) {
    const text = await res.text();
    if (text !== 'pong') {
      throw new Error(`Expected response text "pong", got ${text}`);
    }
  } else {
    throw new Error(`Expected status code 200, got ${res.status}`);
  }
} catch (e) {
  assert.fail(e);
}
```

## 50

### --description--

Now, with the buffer data, use the `writeFile` function from the `fs/promises` module to reconstruct the image file. Re-run the script to generate the image file.

### --tests--

You should have `await writeFile(<file_name>, imageData.buffer);` in `get-nft.js`.

```js
const expressionStatement = babelisedCode.getExpressionStatements().find(e => {
  return e.expression?.argument?.callee?.name === 'writeFile';
});
assert.exists(expressionStatement, 'An `await writeFile()` call should exist');
const [fileName, imageBufferMemberExpression] =
  expressionStatement.expression.argument.arguments;
assert.isString(fileName?.value, 'The first argument should be a string');
assert.equal(
  imageBufferMemberExpression?.object?.name,
  'imageData',
  'The second argument should be `imageData.buffer`'
);
assert.equal(
  imageBufferMemberExpression?.property?.name,
  'buffer',
  'The second argument should be `imageData.buffer`'
);
```

You should have a `.png` file in the project root.

```js
const dir = await __helpers.getDirectory(
  'learn-the-metaplex-sdk-by-minting-an-nft'
);
assert.exists(
  dir.find(f => f.endsWith('.png')),
  'A .png file should exist in the project root'
);
```

### --before-all--

```js
const codeString = await __helpers.getFile(
  'learn-the-metaplex-sdk-by-minting-an-nft/get-nft.js'
);
const babelisedCode = new __helpers.Babeliser(codeString);
global.babelisedCode = babelisedCode;
```

### --after-all--

```js
delete global.babelisedCode;
```

## 51

### --description--

Contratulations on finishing this project! Feel free to play with your code.

**Summary**

The main differences between NFT's and fungible tokens are:

- NFT's are not divisible
- Only one NFT of a mint can exist
- No more NFT's can be minted once the mint authority is removed

The Metaplex SDK provides a simple interface for minting NFT's and managing the metadata associated with them:

```javascript
// Create a new Metaplex instance
const metaplex = Metaplex.make(connection);
// Attach a wallet to use for transactions
metaplex.use(keypairIdentity(wallet_keypair));
// Attach a storage driver to use for uploading and downloading metadata
metaplex.use(storage_driver);

// Create a MetaplexFile
const file = toMetaplexFile(image_buffer);
// Upload the file to the storage driver
const image_uri = await metaplex.storage().upload(file);
// Upload the metadata to the storage driver
const { uri } = await metaplex.nfts().uploadMetadata({
  name: 'fCC',
  description: 'An image of the freeCodeCamp logo',
  image: image_uri
});
// Create a new NFT mint
const nftInfo = await metaplex.nfts().create({
  name: 'fCC',
  uri,
  sellerFeeBasisPoints: 1000,
  maxSupply: 1
});

// Get nft data
const nft = await metaplex.nfts().findByMint({
  mintAddress: nftInfo.mintAddress
});
// Download image from storage driver
const image = await metaplex.storage().download(nft.json.image);
```

🎆

Once you are done, enter `done` in the terminal.

### --tests--

You should enter `done` in the terminal

```js
const lastCommand = await __helpers.getLastCommand();
assert.include(lastCommand, 'done');
```

## --fcc-end--
