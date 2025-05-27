# 🚀 Golem Docs

Ready to get started coding with our Golem tools? We offer several SDKs, each targeting a specific programming language. Below we're providing samples for three of them: TypeScript, Rust, and Python.

Regardless of which you choose, however, you'll need to do some initial configuration. But let's start with a brief overview of what Golem Base is and what our tools are.

# 📚 Brief Overview of Golem Base

Golem Base is a Layer 2 network on Ethereum that solves the problem of how to efficiently store and manage structured data on an Ethereum-compatible node without relying on smart contracts. It offers a simple, low-cost alternative for handling operations like creating, updating, or deleting data.

Golem-base-geth is our customized Ethereum client (based on software called op-geth) that powers Golem Base. It extends op-geth to handle Golem Base transactions, enabling organizations to store structured data directly on the node. Data is written through secure, tamper-evident transactions and stored in the node's internal state. (A node refers to a single server running Ethereum software, whether it's the original op-geth, Golem-base-geth, or any number of Ethereum installations.)

When you're first learning how to use our tools, we recommend working with an existing testnet that we've installed. For those with some experience, however, you have the option of downloading and running our Golem-base-geth software directly on your own computer. We provide instructions for installing and running it [later in this document](#-working-with-a-local-golem-base-geth-installation-optional).

# ⚙️ Configuration

To get started using Golem-Base you need to do some initial setup using our CLI tool, which we describe next.

## Golem-base-geth CLI Tool

As part of our Golem-base-geth software, we've built a command-line tool through which you can interact with a node.

If you're on the Mac, run the following to install it:

```
brew install Golem-Base/demo/golembase-demo-cli
```

If you're on Linux, visit our [Releases page](https://github.com/Golem-Base/golembase-demo-cli/releases/). Scroll down to the first instance of Assets, and click it if it's not already expanded. Under Assets, pick the version you need, either Arm or Intel x86/64. Download the file and extract its contents into a folder of your choosing using the following command:

```
tar -xf golembase-demo-cli_Linux_x86_64.tar.gz
```

or 

```
tar -xf golembase-demo-cli_Linux_arm64.tar.gz
```

depending on which file you downloaded.

Next, copy the executable file, golembase-demo-cli, to a location on your system path; a good one here is /usr/local/bin:

```
sudo cp golembase-demo-cli /usr/local/bin
```

You can now delete the downloaded file as well as the unzipped contents.

## Creating an account and funding it

Next we'll use the CLI tool to create an account. This account will be stored in a file called private.key. The location of the file depends on your operating system:

* `~/.config/golembase/` on **Linux**

* `~/Library/Application Support/golembase/` on **macOS**

* `%LOCALAPPDATA%\golembase\` on **Windows**

(This is a standard folder as per a specification called [XDG](https://specifications.freedesktop.org/basedir-spec/latest/).)

To create the account, type:

```
golembase-demo-cli account create
```

You can verify that the account was created by checking the above mentioned folder; for example, on Linux:

```
ls ~/.config/golembase/
```

should yield:

```
private.key
```

Next you need to fund your account. While creating the account builds a local file, the funding depends on the node you're going to attach to. If you're going to use our test net, you need to open our faucet in your browser; however, first you'll need your Ethereum address. There are different ways to extract it from your private.key file, but the easiest for right now is to ask the CLI tool to get it for you. Simply type:

```
golembase-demo-cli account balance --node-url https://rpc.kaolin.holesky.golem-base.io/
```

You'll see an output that includes your address, along with a zero balance, similar to this:

```
Address: 0x309254f8E8F7E69b77778bE80f3852dB2A0C20D9
Balance: 0 ETH
```

Now open your browser and go to this link:

[https://faucet.kaolin.holesky.golem-base.io/](https://faucet.kaolin.holesky.golem-base.io/)

Enter your account Address in and click Request Funds. After a moment, your account will have a couple Eth available in our testnet.

Now you can check your funds by specifying the address of the node:

```
golembase-demo-cli account balance --node-url https://rpc.kaolin.holesky.golem-base.io/
```

And this time you'll see a non-zero value printed out. Now you're ready to try out any of our SDKs!

# 🌌 GolemBase SDK for TypeScript

The [TypeScript SDK](https://github.com/Golem-Base/typescript-sdk) allows you to write code in the backend (server) or within the frontend (browser) or just as a simple command-line utility using TypeScript that has been transpiled to JavaScript through the `tsc` command.

As a first example, let's build a simple command-line utility using our TypeScript SDK.

Start by following the above steps under [Configuration](#configuration) to create a user and fund it on our testnet.

Next, create a new folder to hold your project:

```bash
mkdir golem-sdk-practice
cd golem-sdk-practice
```

Then create a new package.json and add the dependencies by typing:

```bash
npm init -y & npm install --save-dev typescript && npm i golem-base-sdk xdg-portable tslib
```

**Important:** Next update your package.json file, changing the `type` member to `"type": "module",` and adding the two script lines for `build` and `start`. (Don't forget to add the comma to the end of the "test" line.) It should look like this:

```json
{
  "name": "myapp",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "type": "module",
  "devDependencies": {
    "typescript": "^5.8.3"
  },
  "dependencies": {
    "golem-base-sdk": "^0.1.8"
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

Here's what each setting does:

* **target: "ES2020"**: Tells TypeScript to output JavaScript that’s compatible with ES2020 features.

* **module: "ESNext"**: Specifies that the output should use the latest standardized JavaScript module syntax.

* **moduleResolution: "Node"**: This says to resolve module imports using Node.js-style rules.

* **strict: true**: This says to enables all of TypeScript's strict type-checking options to catch more bugs during development.

* **esModuleInterop: true**: This says to allows default imports from CommonJS modules for better compatibility with many npm packages.

* **outDir: "dist"**: This sets the output folder for where to store compiled JavaScript files.

* **rootDir: "src"**: This tells TypeScript where your source files live.

* **"include": ["src"]**: This tells TypeScript to only compile files in the src directory.

Finally, create a folder called `src`, where you'll put the `index.ts` file as described next.

```
golem-sdk-practice/
├──src
    ├── index.ts 
```

Next, open an editor and create a file called `index.ts` in the `src` folder. Add the following code to index.ts:

```ts
import * as fs from "fs"
import xdg from "xdg-portable"
import { createClient, type GolemBaseClient, type GolemBaseCreate,
  type AccountData, Annotation, Tagged } from "golem-base-sdk"
import { formatEther } from "viem";


async function main() {

  const keyPath = xdg.config() + '/golembase/private.key';
  if (!fs.existsSync(keyPath)) {
    console.log(`Key file not found at: ${keyPath}`);
    return;
  }
  const keyBytes = fs.readFileSync(keyPath);

  const encoder = new TextEncoder()
  const decoder = new TextDecoder()

  const key: AccountData = new Tagged("privatekey", keyBytes)

  const client: GolemBaseClient = await createClient(
    600606, // This is the chainID of the node. If running locally with docker, user 1337
    key,  // The key information for your user
    'https://rpc.kaolin.holesky.golem-base.io', // The http address and port to connect to
    'ws://ws.rpc.kaolin.holesky.golem-base.io'  // The ws address and port to connect to
  )

  async function numOfEntitiesOwned(): Promise<number> {
    return (await client.getEntitiesOfOwner(await client.getOwnerAddress())).length
  }

  // Here are two entities to create.
  const creates: GolemBaseCreate[] = [
    {
      data: encoder.encode("foo"), // Some data to store as a payload
      btl: 25, // Blocks to live
      stringAnnotations: [new Annotation("key", "foo")], // A string value
      numericAnnotations: [new Annotation("ix", 1)] // A numeric value
    },
    {
      data: encoder.encode("bar"),
      btl: 2,
      stringAnnotations: [new Annotation("key", "bar")],
      numericAnnotations: [new Annotation("ix", 2)]
    }
  ]

  console.log('Creating entities!')

  // Ask the client to send the entities to create to the server,
  // and catch any error that it returns. (A common error is 
  // "insufficient funds".)
  try {
    const receipts = await client.createEntities(creates)
  }
  catch (e: unknown) {
    if (e instanceof Error) {
      console.log(e.message);
    } else {
      console.log('An unknown error occurred:', e);
    }
  }
  
  // Have the client ask the server how many entities you now own
  console.log("Number of entities owned:", await numOfEntitiesOwned())

  // Have the client ask to get your new balance after the entity creation
  console.log("Current balance: ", formatEther(await client.getRawClient().httpClient.getBalance({
    address: await client.getOwnerAddress(),
    blockTag: 'latest'
  })))

}

// We put everything inside a single main function to help encapsulate it
main()


```

Next, build it by typing:

`npm run build`

and run it by typing:

`npm run start`

You will see output similar to the following:

```
Creating entities!
Number of entities owned: 4
Current balance:  9.99998713261566488
```

Ready to see more? This example barely cracks the surface. To see a full example:

* Visit our [SDK documentation](https://golem-base.github.io/typescript-sdk/).

* Look at the full [code on GitHub](https://github.com/Golem-Base/typescript-sdk) including more examples.

# 🚗 Ready for the Rust SDK

Would you prefer to work with Rust? Follow these steps to use our Rust SDK.

First, follow the steps above under [Configuration](#configuration).

Now go ahead and clone our [Rust SDK](https://github.com/Golem-Base/rust-sdk) by typing:

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

golem-base-sdk = { workspace = true }
```

Now in the same folder create a subfolder called `src`. Move into `src` and inside `src` create a file called `main.rs`. Add the following to `main.rs`:

```rust
use dirs::config_dir;
use golem_base_sdk::entity::{Create, EntityResult};
use golem_base_sdk::{Address, Annotation, GolemBaseClient, Hash, PrivateKeySigner, Url};
use log::info;
use std::fs;

async fn log_num_of_entities_owned(client: &GolemBaseClient, owner_address: Address) {
    let n = client
        .get_entities_of_owner(owner_address)
        .await
        .expect("Failed to fetch entities of owner")
        .len();
    info!("Number of entities owned: {}", n);
}

#[tokio::main]
async fn main() {
    env_logger::Builder::from_env(env_logger::Env::default().default_filter_or("info")).init();

    let mut private_key_path = config_dir().expect("Failed to get config directory");
    private_key_path.push("golembase/private.key");

    let private_key_bytes = fs::read(&private_key_path).expect("Failed to read private key file");

    let private_key = Hash::from_slice(&private_key_bytes);

    let signer = PrivateKeySigner::from_bytes(&private_key).expect("Failed to parse private key");

    let url = Url::parse("https://rpc.kaolin.holesky.golem-base.io/").expect("Invalid RPC URL");

    let client = GolemBaseClient::builder().wallet(signer).url(url).build();

    info!("Fetching owner address...");
    let owner_address = client.get_owner_address();
    info!("Owner address: {}", owner_address);

    log_num_of_entities_owned(&client, owner_address).await;

    info!("Creating entities...");
    let creates = vec![Create {
        data: "foo".into(),
        ttl: 25,
        string_annotations: vec![Annotation::new("key", "foo")],
        numeric_annotations: vec![Annotation::new("ix", 1u64)],
    }];

    let receipts: Vec<EntityResult> = client
        .create_entities(creates)
        .await
        .expect("Failed to create entities");

    info!("Created entities: {:?}", receipts);
    log_num_of_entities_owned(&client, owner_address).await;
}
```

Now move back to the root folder of the repository. Let's make the current Cargo.toml aware of our practice project. Open Cargo.toml and find the section that looks like this:

```toml
[workspace]
members = [
    "example",
    "cli"
]
```

Add a comma after `cli` and add an entry for `practice`:

```toml
[workspace]
members = [
    "example",
    "cli",
    "practice"
]
```

Save the file. Now back in your shell, you can build the practice app by typing:

```
cargo build -p golem-base-sdk-practice
```

And run the app by typing:

```
cargo run -p golem-base-sdk-practice
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

or on Windows type:

```
venv\Scripts\activate
```

Now install the GolemBase SDK and XDG:

```
pip install golem-base-sdk pyxdg
```

Now create a file called main.py and insert the following into it:

```python
#! /usr/bin/env python

"""GolemBase Python SDK"""

import argparse
import asyncio
import logging
import logging.config

from xdg import BaseDirectory

from golem_base_sdk import (
    Annotation,
    GolemBaseClient,
    GolemBaseCreate,
    GolemBaseDelete,
    GolemBaseExtend,
    GolemBaseUpdate,
)

__version__ = "0.0.1"

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
    "local": {
        "rpc": "http://localhost:8545",
        "ws": "ws://localhost:8545",
    },
    "kaolin": {
        "rpc": "https://rpc.kaolin.holesky.golem-base.io",
        "ws": "wss://ws.rpc.kaolin.holesky.golem-base.io",
    },
}


async def run_example(instance: str) -> None:
    """
    connect
    """

    with open(
        BaseDirectory.xdg_config_home + "/golembase/private.key",
        "rb",
    ) as private_key_file:
        key_bytes = private_key_file.readline()

    client = await GolemBaseClient.create(
        rpc_url=INSTANCE_URLS[instance]["rpc"],
        ws_url=INSTANCE_URLS[instance]["ws"],
        private_key=key_bytes,
    )

    await client.watch_logs(
        lambda create: logger.info(
            """\n
Got create event: %s
        """,
            create,
        ),
        lambda update: logger.info(
            """\n
Got update event: %s
        """,
            update,
        ),
        lambda deleted_key: logger.info(
            """\n
Got delete event: %s
        """,
            deleted_key,
        ),
        lambda extension: logger.info(
            """\n
Got extend event: %s
        """,
            extension,
        ),
    )

    if await client.is_connected():
        block = await client.http_client().eth.get_block("latest")

        logger.info("""\n
        *****************************
        * Checking basic methods... *
        *****************************
        """)

        logger.info("Retrieved block %s", block.number)

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
            "entities to expire at block: %s",
            await client.get_entities_to_expire_at_block(metadata.expires_at_block),
        )

        extend_receipt = await client.extend_entities([GolemBaseExtend(entity_key, 60)])
        logger.info("receipt: %s", extend_receipt)

        logger.info(
            "entities to expire at block: %s",
            await client.get_entities_to_expire_at_block(metadata.expires_at_block),
        )
        logger.info("entity metadata: %s", await client.get_entity_metadata(entity_key))

        logger.info("""\n
        ************************
        * Update the entity... *
        ************************
        """)

        update_receipt = await client.update_entities(
            [GolemBaseUpdate(entity_key, b"hello", 60, [Annotation("app", "demo")], [])]
        )
        logger.info("receipt: %s", update_receipt)
        entity_key = update_receipt[0].entity_key

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
    else:
        logger.warning("Could not connect to the API...")

    await client.disconnect()


def main() -> None:
    """
    main
    """
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

Run the file by simply typing:

```
python main.py
```

Note that this example includes a command line option to choose between different nodes:

* A local node running in a container

* Our testnet server

The testnet server is the default; to use the local add the option as follows:

```
python main.py --instance local
```

To see more examples, such as deleting entities and querying, visit our [Python SDK's repo on GitHub](https://github.com/Golem-Base/python-sdk).

# 🧪 Working with a local Golem-base-geth installation (Optional)

If you prefer to work with a local installation of Golem-base-geth instead of the testnet, you can use the following instructions.

## Installing Golem-base-geth and Docker

Golem-base-geth can be run inside a docker container as an Ethereum node, and optionally as a "development server" called "dev mode." Running in dev mode is a great option for learning how to use our SDKs. To run it, you'll need to make sure Docker is installed.

## Installing Docker

If you're going to run Golem-base-geth inside docker, first you'll need to install docker. Follow the [docker installation instructions here](https://docs.docker.com/engine/install/).

Note: When installing Docker on Linux (or using it from within Windows Subsystem for Linux), after installation you'll need to follow [these post-install instructions](https://docs.docker.com/engine/install/linux-postinstall/) so that you can use docker from your own user without having to use sudo.

Note for those running slightly older versions of docker and docker-compose: Docker recently updated docker-compose, integrating it directly into the docker command. As such, the latest version of docker uses this command:

```
docker compose
```

and doesn't require the installation fo a separate docker-compose command. This is in contrast to  how the earlier versions use the separate tool:

```
docker-compose
```

Tip: If you're running an older version of docker, make sure you either have [docker-compose installed](https://docs.docker.com/compose/install/standalone/), in which case you'll use the `docker-compose` command; or update to the latest version of docker, in which case you'll use the `docker compose` command.

## Installing Golem-base-geth

In order to run Golem-base-geth inside Docker, you'll need to clone the repository from GitHub. For that, you can either clone it, which requires git, or download and unzip it.

If you prefer to clone it, you'll need to have [git installed](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git). Then in your shell, switch to a folder you want to hold the Golem-base-geth folder, and type:

git clone https://github.com/Golem-Base/golembase-op-geth.git

Or, alternatively, you can download and unzip the code; to do so, head over to the [Golem-base-geth repository](https://github.com/Golem-Base/golembase-op-geth) and click the green Code button; in the panel that opens, click Download ZIP. From there, unzip the file into a folder of your choice.

## Running Golem-base-geth

Switch to the root folder of the code you just installed. Verify that you're in the folder that contains the file called `docker-compose.yml`.

To start the software:

If you're using a new version of Docker, type:

```
docker compose up -d
```

If you're using an older version (or if you attempted the above and received an error message), instead type:

```
docker-compose up -d
```

Docker will first build the software, which might take a few minutes.

After it's finished and the prompt returns, make sure it's all running by typing:

```
docker ps
```

You should see a listing of the various containers running. There should be five containers running; the second column, called image, should read (in any order):

* golembase-op-geth_op-geth: This is our main Golem-base-geth software

* mongo:8.0.6: This is the MongoDB database software

* golembase-op-geth_mongodb-etl: This is a data pipeline container for MongoDB

* golembase-op-geth_sqlite-etl: This is the sqlite database software

* dmilhdef/rpcplorer:v0.0.1: This is a software package for exploring the data in Golem-base-geth. (We won't be using this in this tutorial.)


## Funding a local instance of Golem-base-geth

Next you'll need to fund your account on the local node using Golem-base-geth's own built-in command-line tool. This is an expanded form of the golembase-demo-cli tool in that it includes funding capabilities.

To use this tool requires that you first install the Go language tools. Follow the instructions ][here](https://go.dev/doc/install) and verify the installation by running:

```
go version
```

Next, drill down into the Golem-base-geth source code that you downloaded; go to the cmd/golembase folder. Type

```
ls
```

and you should see several files including folders such as `account` and `entity` and `query`. There should also be a main.go file.

Tip: While you could compile this into an executable, we'll simply run the go files directly with the `go run` command.

To begin, run the following to check your local balance; it will likely be zero:

```
go run main.go account balance
```

You should see output similar to this:

```
Address: 0x309254f8E8F7E69b77778bE80f3852dB2A0C20D9
Balance: 0 ETH
```

Now add some funds by typing:

```
go run main.go account fund 10
```

There won't be any output. Now repeat the above `account balance` command and you should see funds present.

Now you're ready to work with the local installation of Golem-base-geth. In the preceding SDK samples (except for the Python example, which already provides both our testnet and local as command lines options), replace the URLs with localhost and the following ports:

* http://localhost:8545 for the http address

* ws://localhost:8546 for the ws address

and replace the chainID (the first parameter to createClient) with 1337.

# 💡 What's Next?

Now is your chance to build something! Pick your SDK from above, and start practicing. Head over to the full documentation for the SDK of your choice and pratice removing and updating entities. Try out the querying system. And as you do, think about what you could build with the Golem-Base SDKs!

# 🧑‍🚀 Get Involved

Golem Base is being developed in the open — and we're just getting started.

If you're interested in decentralized infrastructure, Ethereum scalability, or advancing on-chain data systems, we’d be glad to have you onboard.

* 📬 [Subscribe for updates](https://golem-base.io/)

* 💬 [Join the Discord](http://discord.gg/golem)

* 🧵 [Follow us on X (Formerly Twitter)](https://x.com/golemproject)

* 💼 [Come join our team! Explore Careers](https://golem.network/careers)
