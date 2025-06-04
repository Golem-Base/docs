# 🌌 GolemBase SDK for TypeScript

The [TypeScript SDK](https://github.com/Golem-Base/typescript-sdk) allows you to write code in TypeScript that interfaces with a Golem-base-geth node, allowing you to store entities, update them, and delete them.

In the following instructions, we will:

1. Set up your environment for both node.js and TyepScript

2. Create some TypeScript code that will generate a private key file that's needed for connecting to a the Golem-base-geth node; this private.key file also contains an Ethereum address embedded in it

3. Optionally run the code (unless you already have a private.key file)

4. Add funds using the address embedded in the private.key file. We'll obtain the funds from a "faucet" for our test network called Kaolin.

5. Create TypeScript code that connects to the node on Kaolin, stores two entities, and retrieves them.

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

Now open your browser and go to this link, which is the faucet for Kaolin:

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

	// Print out the balance

	const address = await client.getOwnerAddress();

	const balanceWei = await client.getRawClient().httpClient.getBalance({
		address: address,
		blockTag: 'latest'
	});

	console.log(`Balance for address ${address} (Wei, Eth):`);
	console.log(`${balanceWei} Wei, ${balanceWei / 10n ** 18n} Eth`);
	
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

And finally, we check the balance and print it out in both Wei and Eth format.

Ready to explore even more? Check out our full TypeScript SDK example that includes more calls into our API, and complete logging, in the [TypeScript SDK Repository](https://github.com/Golem-Base/typescript-sdk).



# 🚗 Ready for the Rust SDK

Would you prefer to work with Rust? Follow these steps to use our Rust SDK.

Start by cloning our [Rust SDK](https://github.com/Golem-Base/rust-sdk) by typing:

```
git clone https://github.com/Golem-Base/rust-sdk.git
```

Next, inside the root folder for the repository, create a folder called `practice`.

Move into that folder and create a file called `Cargo.toml`, and add the following to it:

```toml
[package]
name = "golem-base-sdk-practice"
version = "0.1.0"
edition = "2021"

[dependencies]
alloy = "0.15"
bytes = "1.0"
dirs = "6.0"
env_logger = "0.11"
log = "0.4"
tokio = { version = "1", features = ["full"] }
ethers = "2"
rand = "0.8"

golem-base-sdk = { workspace = true }
```

Now in the same folder create a subfolder called `src`. Move into `src` and inside `src` create a file called `main.rs`. Add the following to `main.rs`:

```rust
use golem_base_sdk::entity::{Create, EntityResult, Update};
use golem_base_sdk::{Address, Annotation, GolemBaseClient, Hash, PrivateKeySigner, Url};
use log::info;
use std::fs;
use std::path::Path;

async fn log_num_of_entities_owned(client: &GolemBaseClient, owner_address: Address) {
    let n = client
        .get_entities_of_owner(owner_address)
        .await
        .expect("Failed to fetch entities of owner")
        .len();
    info!("Number of entities owned: {}", n);
}

#[tokio::main]
async fn main()  -> Result<(), Box<dyn std::error::Error>> {
    env_logger::Builder::from_env(env_logger::Env::default().default_filter_or("info")).init();

    let private_key_path = Path::new("private.key");

    if !private_key_path.exists() {
        panic!("private.key not found in current directory.");
    }

    let private_key_bytes = fs::read(&private_key_path).expect("Failed to read private key file");
    let private_key = Hash::from_slice(&private_key_bytes);

    let signer = PrivateKeySigner::from_bytes(&private_key)
        .map_err(|e| format!("Failed to parse private key: {}", e))?;
    let url = Url::parse("http://localhost:8545").unwrap();
    let client = GolemBaseClient::builder()
        .wallet(signer)
        .rpc_url(url)
        .build();

    info!("Fetching owner address...");
    let owner_address = client.get_owner_address();
    info!("Owner address: {}", owner_address);
    log_num_of_entities_owned(&client, owner_address).await;

    info!("Creating entities...");
    let creates = vec![
        Create {
            data: "foo".into(),
            btl: 25,
            string_annotations: vec![Annotation::new("key", "foo")],
            numeric_annotations: vec![Annotation::new("ix", 1u64)],
        },
        Create {
            data: "bar".into(),
            btl: 2,
            string_annotations: vec![Annotation::new("key", "bar")],
            numeric_annotations: vec![Annotation::new("ix", 2u64)],
        },
        Create {
            data: "qux".into(),
            btl: 50,
            string_annotations: vec![Annotation::new("key", "qux")],
            numeric_annotations: vec![Annotation::new("ix", 3u64)],
        },
    ];
    let receipts: Vec<EntityResult> = client.create_entities(creates).await?;
    info!("Created entities: {:?}", receipts);
    log_num_of_entities_owned(&client, owner_address).await;

    info!("Deleting first entity...");
    client.delete_entities(vec![receipts[0].entity_key]).await?;
    log_num_of_entities_owned(&client, owner_address).await;

    info!("Updating the third entity...");
    let third_entity_key = receipts[2].entity_key;
    let metadata = client.get_entity_metadata(third_entity_key).await?;
    info!("... before the update: {:?}", metadata);
    client
        .update_entities(vec![Update {
            data: "foobar".into(),
            btl: 40,
            string_annotations: vec![Annotation::new("key", "qux"), Annotation::new("foo", "bar")],
            numeric_annotations: vec![Annotation::new("ix", 2u64)],
            entity_key: third_entity_key,
        }])
        .await?;
    let metadata = client.get_entity_metadata(third_entity_key).await?;
    info!("... after the update: {:?}", metadata);

    info!("Deleting remaining entities...");
    let remaining_entities = client
        .query_entity_keys("ix = 1 || ix = 2 || ix = 3")
        .await?;
    info!("Remaining entities: {:?}", remaining_entities);
    client.delete_entities(remaining_entities).await?;
    log_num_of_entities_owned(&client, owner_address).await;

    Ok(())
}
```

Now inside the `src` folder create another folder called `bin`:

```
mkdir bin
cd bin
```

Here we'll add code for creating a wallet. Create a new file called `wallet.rs` inside `bin` and add the following:

```rust
use ethers::signers::{LocalWallet, Signer};
use ethers::core::k256::ecdsa::SigningKey;
use std::fs::File;
use std::io::Write;
use std::path::Path;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let key_path = "private.key";

    if Path::new(key_path).exists() {
        println!("File \"{}\" already exists. Aborting.", key_path);
        return Ok(());
    }

    let wallet: LocalWallet = LocalWallet::new(&mut rand::thread_rng());
    let signing_key: &SigningKey = wallet.signer();
    let private_key_bytes = signing_key.to_bytes();

    let mut file = File::create(key_path)?;
    file.write_all(&private_key_bytes)?;

    println!("Address: {:?}", wallet.address());
    Ok(())
}
```

Now move back to the root folder of the repository. Let's make the current Cargo.toml aware of our practice project. Open Cargo.toml and find the section that looks like this:

```toml
[workspace]
members = ["api", "cli", "demo", "test-utils"]
resolver = "2"
```

Add a comma after `test-utils` and add an entry for `practice`:

```toml
[workspace]
members = ["api", "cli", "demo", "test-utils", "practice"]
resolver = "2"
```

Now we have two programs we can use; the second one we created, `wallet.rs`, we'll build and run first. This will create a new private.key file for us that will hold our wallet.

## Creating a private.key file

While still in the root folder of the repo, type:

```
cargo build -p golem-base-sdk-practice --bin wallet
```

Note: If you get an error regarding finding openssl via pkg-config, you'll need to run the following:

```
sudo apt update
sudo apt install pkg-config libssl-dev
```

Then try running the build command again. After it builds, run it:

```
cargo run -p golem-base-sdk-practice --bin wallet
```

This will generate the private.key file in the root folder of the repo. The code will also print out the address. Copy the address into your clipboard, including the 0x and everything after it.

Now you're ready to add funds to it.

## Adding Funds

Open your browser and go to this link, which is the faucet for Kaolin:

[https://faucet.kaolin.holesky.golem-base.io/](https://faucet.kaolin.holesky.golem-base.io/)

Paste in your account Address that was printed out earlier and click Request Funds. After a moment, your account will have a couple Eth available in our testnet.

## Creating Entities

And now you're ready to create and edit some entities with the code you created in the second file, main.rs. Here's how you build and run it:

```
cargo build -p golem-base-sdk-practice
cargo run -p golem-base-sdk-practice --bin golem-base-sdk-practice
```

(You can see a full example by looking in the `/example` folder as well as more in the `/examples` folder.)


# 🐍 Using our Python SDK

We also have a Python SDK if that's your language of choice. Start by following the steps above under [Configuration](#configuration).

> Most systems already have python version 3 installed, available directly through the python command. Make sure by typing the following: 
>
> ```bash
> python --version
> ```
>
> You should see something similar to this:
>
> ```text
> Python 3.12.3
> ```
>
> If not, you'll need to install it. The official Python documentation provides [downloadable installations](https://www.python.org/downloads/).

First, create a new folder to hold you project:

```
mkdir golem-python-practice
cd golem-python-practice
```

Next, create and activate a virtual environment.

```
python3 -m venv venv
```

This creates a virtual environment. Note: If you get an error message regarding "ensurepip not being available" type the following to add the python3-venv package:

```
sudo apt install python3.12-venv
```

and then repeat the `python3 -m venv venv` command.

Now activate the virtual environment. On Mac and Linux, type:

```
source venv/bin/activate 
```

Now install the GolemBase SDK and XDG:

```
pip install golem-base-sdk eth-account anyio
```

## Code to create a private.key file

Now we'll create a python app that can create a wallet stored in a private.key file. Create a file called wallet.py and add the following to it:

```python
import os
from eth_account import Account

key_path = 'private.key'

if os.path.exists(key_path):
    print(f'File "{key_path}" already exists. Aborting.')
    exit(0)

acct = Account.create()

with open(key_path, 'wb') as f:
    f.write(bytes.fromhex(acct.key.hex()))

print('Address:', acct.address)
```

Then run the above by typing:

```
python wallet.py
```

You should see output similar to the following:

```
Address: 0x10882A8BD4C79ED56Fca4Cd9AD73E3D234365547
```

This is the address portion of the private key you just created. Copy this to your clipboard, as you'll need it in the next step.

## Adding Funds

Open your browser and go to this link, which is the faucet for Kaolin:

[https://faucet.kaolin.holesky.golem-base.io/](https://faucet.kaolin.holesky.golem-base.io/)

Paste your account Address in and click Request Funds. After a moment, your account will have a couple Eth available in our testnet.


## Creating entities in Golem-base

Now create a file called main.py and insert the following into it:

```python
"""GolemBase Python SDK."""

import argparse
import asyncio
import logging
import logging.config

import anyio
from golem_base_sdk import (
    Annotation,
    GolemBaseClient,
    GolemBaseCreate,
    GolemBaseDelete,
    GolemBaseExtend,
    GolemBaseUpdate,
)
# from xdg import BaseDirectory

logging.config.dictConfig(
    {
        "version": 1,
        "formatters": {
            "default": {
                "format": "[%(asctime)s] %(levelname)s in %(module)s: %(message)s",
            }
        },
        "handlers": {
            "console": {
                "class": "logging.StreamHandler",
                "formatter": "default",
                "level": "DEBUG",
                "stream": "ext://sys.stdout",
            }
        },
        "loggers": {
            # "": {"level": "DEBUG", "handlers": ["console"]},
            "": {"level": "INFO", "handlers": ["console"]},
        },
        # Avoid pre-existing loggers from imports being disabled
        "disable_existing_loggers": False,
    }
)
logger = logging.getLogger(__name__)

INSTANCE_URLS = {
    "demo": {
        "rpc": "https://api.golembase.demo.golem-base.io",
    },
    "local": {
        "rpc": "http://localhost:8545",
        "ws": "ws://localhost:8545",
    },
    "kaolin": {
        "rpc": "https://rpc.kaolin.holesky.golem-base.io",
        "ws": "wss://ws.rpc.kaolin.holesky.golem-base.io",
    },
}


async def run_example(instance: str) -> None:  # noqa: PLR0915
    """Run the example."""
    async with await anyio.open_file(
        "./private.key",
        "rb",
    ) as private_key_file:
        key_bytes = await private_key_file.read(32)

    client = await GolemBaseClient.create(
        rpc_url=INSTANCE_URLS[instance]["rpc"],
        ws_url=INSTANCE_URLS[instance]["ws"],
        private_key=key_bytes,
    )

    watch_logs_handle = await client.watch_logs(
        label="first",
        create_callback=lambda create: logger.info(
            """\n
Got create event: %s
        """,
            create,
        ),
        update_callback=lambda update: logger.info(
            """\n
Got update event: %s
        """,
            update,
        ),
    )

    await client.watch_logs(
        label="second",
        delete_callback=lambda deleted_key: logger.info(
            """\n
Got delete event: %s
        """,
            deleted_key,
        ),
        extend_callback=lambda extension: logger.info(
            """\n
Got extend event: %s
        """,
            extension,
        ),
    )

    if await client.is_connected():
        logger.info("""\n
        *****************************
        * Checking basic methods... *
        *****************************
        """)

        block = await client.http_client().eth.get_block("latest")
        logger.info("Retrieved block %s", block["number"])

        logger.info("entity count: %s", await client.get_entity_count())

        logger.info("""\n
        *************************
        * Creating an entity... *
        *************************
        """)

        create_receipt = await client.create_entities(
            [GolemBaseCreate(b"hello", 60, [Annotation("app", "demo")], [])]
        )
        entity_key = create_receipt[0].entity_key
        logger.info("receipt: %s", create_receipt)
        logger.info("entity count: %s", await client.get_entity_count())

        logger.info("created entity with key %s", entity_key)
        logger.info("storage value: %s", await client.get_storage_value(entity_key))
        metadata = await client.get_entity_metadata(entity_key)
        logger.info("entity metadata: %s", metadata)

        logger.info("""\n
        ***********************************
        * Extend the BTL of the entity... *
        ***********************************
        """)

        logger.info(
            "entities to expire at block %s: %s",
            metadata.expires_at_block,
            await client.get_entities_to_expire_at_block(metadata.expires_at_block),
        )

        [extend_receipt] = await client.extend_entities(
            [GolemBaseExtend(entity_key, 60)]
        )
        logger.info("receipt: %s", extend_receipt)

        logger.info(
            "entities to expire at block %s: %s",
            metadata.expires_at_block,
            await client.get_entities_to_expire_at_block(metadata.expires_at_block),
        )
        logger.info(
            "entities to expire at block %s: %s",
            extend_receipt.new_expiration_block,
            await client.get_entities_to_expire_at_block(
                extend_receipt.new_expiration_block
            ),
        )
        logger.info("entity metadata: %s", await client.get_entity_metadata(entity_key))

        logger.info("""\n
        ************************
        * Update the entity... *
        ************************
        """)

        logger.info(
            "block number: %s", await client.http_client().eth.get_block_number()
        )
        [update_receipt] = await client.update_entities(
            [GolemBaseUpdate(entity_key, b"hello", 60, [Annotation("app", "demo")], [])]
        )
        logger.info("receipt: %s", update_receipt)
        entity_key = update_receipt.entity_key

        logger.info("entity metadata: %s", await client.get_entity_metadata(entity_key))

        logger.info("""\n
        *************************
        * Query for entities... *
        *************************
        """)

        query_result = await client.query_entities('app = "demo"')
        logger.info("Query result: %s", query_result)

        logger.info("""\n
        ************************
        * Delete the entity... *
        ************************
        """)

        logger.info("entity metadata: %s", await client.get_entity_metadata(entity_key))
        logger.info(
            "block number: %s", await client.http_client().eth.get_block_number()
        )

        receipt = await client.delete_entities([GolemBaseDelete(entity_key)])
        logger.info("receipt: %s", receipt)

        logger.info(
            "My entities: %s",
            await client.get_entities_of_owner(client.get_account_address()),
        )

        logger.info(
            "All entities: %s",
            await client.get_all_entity_keys(),
        )

        await watch_logs_handle.unsubscribe()

        logger.info("""\n
        *********************************************
        * Creating an entity to test unsubscribe... *
        *********************************************
        """)

        create_receipt = await client.create_entities(
            [GolemBaseCreate(b"hello", 60, [Annotation("app", "demo")], [])]
        )
        entity_key = create_receipt[0].entity_key
        logger.info("receipt: %s", create_receipt)
        logger.info("entity count: %s", await client.get_entity_count())

        logger.info("created entity with key %s", entity_key)
        logger.info("storage value: %s", await client.get_storage_value(entity_key))
        metadata = await client.get_entity_metadata(entity_key)
        logger.info("entity metadata: %s", metadata)

        await client.delete_entities([GolemBaseDelete(entity_key)])
    else:
        logger.warning("Could not connect to the API...")

    await client.disconnect()


def main() -> None:
    """Run the example."""
    parser = argparse.ArgumentParser(description="GolemBase Python SDK Example")
    parser.add_argument(
        "--instance",
        choices=INSTANCE_URLS.keys(),
        default="kaolin",
        help="Which instance to connect to (default: kaolin)",
    )
    args = parser.parse_args()
    logger.info("Starting main loop")
    asyncio.run(run_example(args.instance))


if __name__ == "__main__":
    main()
```

Note that this example includes a command line option to choose between different nodes:

* A local node running in a container

* Our testnet server

The testnet server is the default, which is what we want. So run the file by simply typing:

```
python main.py
```

To see more examples and learn about our Python SDK, visit our [Python SDK's repo on GitHub](https://github.com/Golem-Base/python-sdk).

# ⌨️ Using Our Command-Line Shell Tool

We also offer a simple-to-use tool you can run right from the command line shell (such as bash or sh). Our CLI tool can create a wallet create entities, and update and delete the entities.

To get started, you need to make sure you have the Go tools installed. To install them, visit the [Go installation page](https://go.dev/doc/install)).

Next, you need to clone our golem-base-op-geth repo:

```
git clone https://github.com/Golem-Base/golembase-op-geth.git
```

Then move into the cloned folder's cmd/golembase folder:

```
cd golembase-op-geth/cmd/golembase
```

Next, build the tool by typing:

```
go build .
```

Give it a few moments to build. Make sure it exists:

```
ls
```
and you should see the executable file called golembase listed.

## Creating a Wallet

Create a wallet by typing:

```
./golembase account create
```

Note: This will create a private.key file and install it in a standard config directory; the tool will show you where in its output:

```
Private key generated and saved to /home/<username>/.config/golembase/private.key
Address: 0x1cdE196136e2418605d526978dF2837d1fF09D21
```

Copy the address it created to your clipboard, including the 0x part, as you'll need that next.

## Adding Funds

Now open your browser and go to this link, which is the faucet for Kaolin:

[https://faucet.kaolin.holesky.golem-base.io/](https://faucet.kaolin.holesky.golem-base.io/)

Paste your account Address in and click Request Funds. After a moment, your account will have a couple Eth available in our testnet.

## Creating an Entity

Now you can create an entity. Note, however, that you need to specify the URL of the testnet, as by default this tool connects to a locally-running instance.

To create an entity, type the following:

```
./golembase entity create --data 'Hello, Golem World!' --node-url https://rpc.kaolin.holesky.golem-base.io
```

You can learn more by visiting the CLI tool's README here: https://github.com/Golem-Base/golembase-op-geth/tree/main/cmd/golembase

