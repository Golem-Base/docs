# 🌌 GolemBase SDK for TypeScript

The [TypeScript SDK](https://github.com/Golem-Base/typescript-sdk) allows you to write code in TypeScript that interfaces with a Golem-base-geth node, allowing you to store entities, update them, and delete them.

In the following instructions, we will:

1. Set up your environment for both node.js and TyepScript

2. Create some TypeScript code that will generate a private key file that's needed for connecting to a the Golem-base-geth node; this private.key file also contains an Ethereum address embedded in it

3. Optionally run the code (unless you already have a private.key file)

4. Add funds using the address embedded in the private.key file

5. Create TypeScript code that connects to the node, stores two entities, and retrieves them.

This will provide an easy starting point that you can build on for your own applications.

## Configure the TypeScript environment

Let's configure your environment. Start by creating a new folder to hold your project:

```bash
mkdir golem-sdk-practice
cd golem-sdk-practice
```

Then create a new package.json and add the dependencies by typing:

```bash
npm init -y & npm install --save-dev typescript && npm i golem-base-sdk @ethereumjs/wallet tslib
```

**Important:** Next update your package.json file, changing the `type` member to `"type": "module",` and adding the three script lines for `build`, `app`, and `wallet`. (Don't forget to add the comma to the end of the "test" line.) It should look like this:

```json
{
  "name": "typescript",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",

    "build": "tsc",
    "app": "node dist/app.js",
    "wallet": "node dist/wallet.js"

  },
  "keywords": [],
  "author": "",
  "license": "ISC",

  "type": "module",

  "devDependencies": {
    "typescript": "^5.8.3"
  },
  "dependencies": {
    "@ethereumjs/wallet": "^10.0.0",
    "golem-base-sdk": "^0.1.13",
    "tslib": "^2.8.1"
  }
}

```

And next, create a file called tsconfig.json, and add the following to it:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "moduleResolution": "Node",
    "strict": true,
    "esModuleInterop": true,
    "outDir": "dist",
    "rootDir": "src"
  },
  "include": ["src"]
}
```

This file configures TypeScript. Here's what each setting does:

* **target: "ES2020"**: Tells TypeScript to output JavaScript that’s compatible with ES2020 features.

* **module: "ESNext"**: Specifies that the output should use the latest standardized JavaScript module syntax.

* **moduleResolution: "Node"**: This says to resolve module imports using Node.js-style rules.

* **strict: true**: This says to enables all of TypeScript's strict type-checking options to catch more bugs during development.

* **esModuleInterop: true**: This says to allows default imports from CommonJS modules for better compatibility with many npm packages.

* **outDir: "dist"**: This sets the output folder for where to store compiled JavaScript files.

* **rootDir: "src"**: This tells TypeScript where your source files live.

* **"include": ["src"]**: This tells TypeScript to only compile files in the src directory.

Finally, create a folder called `src`, where you'll put the `wallet.ts` and `app.ts` files as described next.

```
golem-sdk-practice/
├──src
    ├── wallet.ts 
    ├── app.ts 
```

## Code to create a private.key file

First, you'll need a private.key file that holds your private key (which includes your address). If you already have one, you can skip this next step; however, we do recommend at least creating this next file so you have it for later in case you need to recreate a private.key file.

Open your editor and create the wallet.ts file, and add the following code to it:

```ts
import fs from 'fs';
import { Wallet } from '@ethereumjs/wallet';

const keyPath = 'private.key';

if (fs.existsSync(keyPath)) {
  console.log(`File "${keyPath}" already exists. Aborting.`);
  process.exit(0);
}

const wallet = Wallet.generate();
const privateKey = wallet.getPrivateKey();

fs.writeFileSync(keyPath, privateKey);
console.log('Address:', wallet.getAddressString());
```

This code will create a brand new private.key file. Exit out of the editor, and return to the command prompt. Make sure you're in the top folder of your app and type:

```
npm run build
npm run wallet
```

The first line builds the wallet code. The second line runs it. As it runs, it will print out the address for the private key it created and saved in the private.key file. It will look something like this:

```
Address: 0x00deea12e4f62de5e5955f76030c621926db8eb6
```

Tip: You'll need to copy this address for the next step of adding funds to your wallet.

## Adding Funds

Now open your browser and go to this link:

[https://faucet.kaolin.holesky.golem-base.io/](https://faucet.kaolin.holesky.golem-base.io/)

Paste your account Address in and click Request Funds. After a moment, your account will have a couple Eth available in our testnet.

## Creating entities in Golem-base

Next, create a file in the src folder called app.ts, and add the following to it:

```ts
import * as fs from "fs"

import {
  createClient,
  type GolemBaseCreate,
  Annotation,
  Tagged,
  type AccountData,
  type GolemBaseClient,
  type CreateEntityReceipt
} from "golem-base-sdk"

const filename = 'private.key'; // If your private.key file is elsewhere, add its path to this string.
if (!fs.existsSync(filename)) {
	console.log(`File "${filename}" not found. Aborting.`);
	process.exit(0);
}

const encoder = new TextEncoder()
const decoder = new TextDecoder()

const keyBytes = fs.readFileSync(filename);
const key: AccountData = new Tagged("privatekey", keyBytes)

const client: GolemBaseClient = await createClient(
	600606, // The id of  the node.
	key,
	'https://rpc.kaolin.holesky.golem-base.io', // The http address
	'ws://ws.rpc.kaolin.holesky.golem-base.io'  // The web socket address
)

async function createSampleEntities(): Promise<CreateEntityReceipt[]> {
	const creates: GolemBaseCreate[] = [
		{
			data: encoder.encode("Welcome to Golem-base!"), // We need to encode the string to an array of integers
			btl: 25,
			stringAnnotations: [new Annotation("xyz", "zzz"), new Annotation("abc", "def")],
			numericAnnotations: [new Annotation("num", 1), new Annotation("favorite", 2)]
		},
		{
			data: encoder.encode("Welcome back!"),
			btl: 30,
			stringAnnotations: [new Annotation("aaa", "bbb")],
			numericAnnotations: [new Annotation("num", 3)]
		}
	];

	try {
		const receipts: CreateEntityReceipt[] = await client.createEntities(creates);
		return receipts;
	}
	catch (e) {
		console.log('Error:');
		console.log((e as Error).message)
	}

	return [];
}

async function main() {
	// Call our functinon to create the entities. This will return an 
	// array of receipts. Each receipt contains an entity hash code, and a BTL.
	const create_receipts: CreateEntityReceipt[] = await createSampleEntities();

	// Loop through the receipts
	for (const receipt of create_receipts) {
		
		console.log(`Entity ${receipt.entityKey}:`)

		// Obtain the metadata for the entity
		
		const metadata = await client.getEntityMetaData(receipt.entityKey);
		console.log('Metadata:');
		console.log(metadata);
		
		// getStorageValue returns the data saved with the entity as an array of integers;
		// thus we need to decode it using the runtime's built-in TextDecoder class.
		const storage = decoder.decode(await client.getStorageValue(receipt.entityKey))
		console.log('Payload:')
		console.log(storage);

	}
	
}

main()
```

Make sure you're in the root of your app, the same folder containing your private.key file.

> **Tip:** If you already have a private.key file and didn't make one here, adjust the filename variable to include the path to your private.key file.

To build and run the app:

```
npm run build
npm run app
```

The code in main() first calls the createSampleEntities, which builds a structure containing two separate entities to store. Each entity contains:
* a payload to store with the entity (the `data` member)
* an integer value for blocks to live
* an array of any number of string annotations and numeric annotations.

Note that the GolemBaseCreate type requires the payload be in the form of an array of integers. Since we're starting with a string, we use the JavaScript engine's built-in encoder to encode it from a string to an array of integers.

The function then calls client.createEntities to store the entities. client.createEntities returns an array of receipts, with one receipt per entity. Each receipt contains:
* a hash called entityKey,
* an expiration block number.

The createSampleEntities returns this same array back to main.

Back in main, we then loop through the receipts, making two calls to the server:

* **client.getEntityMetaData:** This retrieves the expiresAtBlock, the string and numeric annotations, and the sender address, stored as "owner".
* **client.getStorageValue:** This retrieves only the payload. Note that it's returned as an array of integers, meaning you'll likely have to decode it back to whatever type you started with. We started with a string, so we use the JavaScript engine's built-in decoder to decode it back to a string.

We then print out the results of each call.

Ready to explore even more? Check out our full TypeScript SDK example in the [TypeScript SDK Repository](https://github.com/Golem-Base/typescript-sdk).