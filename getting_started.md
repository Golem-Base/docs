# 🚀 Golem Docs

Ready to get started coding with our Golem tools? Here's how you can do it. We'll explore some examples in different languages. We’ll start with our [TypeScript SDK](https://github.com/Golem-Base/typescript-sdk).

# 🌌 GolemBase SDK for TypeScript

The [TypeScript SDK](https://github.com/Golem-Base/typescript-sdk) allows you to write code either in the backend (server) or within the frontend (browser) using TypeScript that has been transpiled to JavaScript through the `tsc` command.

As a first example, let's build a backend using our TypeScript SDK.

**Tip:** For getting up and running quickly, we recommend the following two steps:

1. Start golembase-op-geth through its [docker-compose](https://github.com/Golem-Base/golembase-op-geth/blob/main/RUN_LOCALLY.md).

2. [Install the demo CLI](https://github.com/Golem-Base/golembase-demo-cli?tab=readme-ov-file#installation) and [create a user](https://github.com/Golem-Base/golembase-demo-cli?tab=readme-ov-file#quickstart).

(Note: As an alternative to installing the demo CLI, you can build the [actual CLI](https://github.com/Golem-Base/golembase-op-geth/blob/main/cmd/golembase/README.md) as it's included in the golembase-op-geth repo.)

Start by creating a user:

```
golembase-demo-cli account create
```

When you create a user, it will generate a private key file called `private.key` and store it in your user's configuration area. (For example, on Linux, it will store it in `~/.config/golembase/`)

You will also need to fund the account. You can do so by typing:

```
golembase-demo-cli account fund 10
```

(You can learn more about golembase-demo-cli [at its GitHub repo](https://github.com/Golem-Base/golembase-demo-cli).)

Here's how you can get going with the SDK. First, create a new folder to hold your project:

```bash
mkdir golem-sdk-practice
cd golem-sdk-practice
```

Then create a new package.json and add the dependencies by typing:

```bash
npm init -y & npm install --save-dev typescript && npm i golem-base-sdk xdg-portable tslib
```

**Important:** Next update your package.json file, changing the `type` member to `"type": "module",` and adding the two script lines for `build` and `start`. (Don't forget to add the comma to the end of the "test" line.)

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

Finally, create a folder called `src`, where you'll put the `index.ts` file as described next.

```
golem-sdk-practice/
├──src
    ├── index.ts 
```

Next, open an editor and create a file called `index.ts` in the `src` folder. Here's what you'll want to put into the folder:

```ts
import * as fs from "fs"
import xdg from "xdg-portable"
import { createClient, type GolemBaseClient, type GolemBaseCreate, type AccountData,
  Annotation, Tagged } from "golem-base-sdk"
import { formatEther } from "viem";

const keyBytes = fs.readFileSync(xdg.config() + '/golembase/private.key');

const encoder = new TextEncoder()
const decoder = new TextDecoder()

async function main() {
  const key: AccountData = new Tagged("privatekey", keyBytes)
  const client: GolemBaseClient = await createClient(
    1337, // This is the chainID of the local dev server running in docker
    key,  // The key information for your user
    'http://localhost:8545', // The http address and port to connect to
    'ws://localhost:8546'    // The ws address and port to connect to
  )

  async function numOfEntitiesOwned(): Promise<number> {
    return (await client.getEntitiesOfOwner(await client.getOwnerAddress())).length
  }

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
  // Ask the client to send the entities to create to the server
  const receipts = await client.createEntities(creates)

  // Have the client ask the server how many entities you now own
  console.log("Number of entities owned:", await numOfEntitiesOwned())

  // Have the client ask to get your new balance after the entity creation
  console.log("Current balance: ", formatEther(await client.getRawClient().httpClient.getBalance({
    address: await client.getOwnerAddress(),
    blockTag: 'latest'
  })))

  // Wait a moment for everything to complete.
  await (new Promise(resolve => setTimeout(resolve, 500)))

}

// We put everything inside a single main function because it's async.
main()

```

Next, build it by typing:

`npm run build`

and run it by typing:

`npm run start`

You will see output similar to the following:

```
freckleface@DESKTOP-N3E6T0C:~/dev/golem-sdk-practice$ npm run start

> typescript_sdk_demo1@1.0.0 start
> node dist/index.js

Creating entities!
Number of entities owned: 4
Current balance:  9.99998713261566488

freckleface@DESKTOP-N3E6T0C:~/dev/golem-sdk-practice$
```

Ready to see more? This example barely cracks the surface. To see a full example:

* Visit our [SDK documentation](https://golem-base.github.io/typescript-sdk/).

* Look at the full [code on GitHub](https://github.com/Golem-Base/typescript-sdk) including more examples.

# 🚗 Ready for the Rust SDK

Would you prefer to work with Rust? We've got you covered!

First, follow the steps above for installing Golem-base Op-Geth, starting it under docker-compose, and creating and funding an account, all as described above under [TypeScript SDK](./README.md#-golembase-sdk-for-typescript).

Now go ahead and clone our [Rust SDK](https://github.com/Golem-Base/rust-sdk).

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
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    env_logger::Builder::from_env(env_logger::Env::default().default_filter_or("info")).init();

    let mut private_key_path = config_dir().ok_or("Failed to get config directory")?;
    private_key_path.push("golembase/private.key");
    let private_key_bytes = fs::read(&private_key_path)?;
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
            ttl: 25,
            string_annotations: vec![Annotation::new("key", "foo")],
            numeric_annotations: vec![Annotation::new("ix", 1u64)],
        }
    ];
    let receipts: Vec<EntityResult> = client.create_entities(creates).await?;
    info!("Created entities: {:?}", receipts);
    log_num_of_entities_owned(&client, owner_address).await;

    Ok(())
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

# 🧑‍🚀 Get Involved

Golem Base is being developed in the open — and we're just getting started.

If you're interested in decentralized infrastructure, Ethereum scalability, or advancing on-chain data systems, we’d be glad to have you onboard.

* 📬 [Subscribe for updates](https://golem-base.io/)

* 💬 [Join the Discord](http://discord.gg/golem)

* 🧵 [Follow us on X (Formerly Twitter)](https://x.com/golemproject)

* 💼 [Come join our team! Explore Careers](https://golem.network/careers)
