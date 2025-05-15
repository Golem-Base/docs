# 🚀 Golem Docs

Ready to get started coding with our Golem tools? Here's how you can do it. We'll explore some examples in different languages. We’ll start with our [TypeScript SDK](https://github.com/Golem-Base/typescript-sdk).

# 🌌 GolemBase SDK for TypeScript

The [TypeScript SDK](https://github.com/Golem-Base/typescript-sdk) allows you to write code either in the backend (server) or within the frontend (browser) using TypeScript that has been transpiled to JavaScript through the `tsc` command.

As a first example, let's build a backend using our TypeScript SDK.

**Tip:** For getting up and running quickly, we recommend the following two steps:

1. Start golembase-op-geth through its [docker-compose](https://github.com/Golem-Base/golembase-op-geth/blob/main/RUN_LOCALLY.md).

2. [Install the demo CLI](https://github.com/Golem-Base/golembase-demo-cli?tab=readme-ov-file#installation) and [create a user](https://github.com/Golem-Base/golembase-demo-cli?tab=readme-ov-file#quickstart).

(Note: As an alternative to installing the demo CLI, you can build the [actual CLI](https://github.com/Golem-Base/golembase-op-geth/blob/main/cmd/golembase/README.md) as it's included in the golembase-op-geth repo.)

When you create a user, it will generate a private key file called `private.key` and store it in your user's configuration area. (For example, on Linux, it will store it in `~/.config/golembase/`)

You will also need to fund the account. You can do so by typing:

```
golembase-demo-cli account fund 10
```

Here's how you can get going with the SDK. First, create a new folder to hold your project:

```bash
mkdir golem-sdk-practice
cd golem-sdk-practice
```

Then create a new package.json and add the dependencies by typing:

```bash
npm init -y & npm install --save-dev typescript && npm i golem-base-sdk
```

**Important:** Next update your package.json file, changing the `type` member to `"type": "module",` and adding the two script lines for `build` and `start`. (You can leave the `name` member set to whatever it is. And the order of the members doesn't matter.)

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
import { createClient, type GolemBaseClient, type GolemBaseCreate,
  Annotation, Tagged } from "golem-base-sdk"
import { formatEther } from "viem";

const keyBytes = fs.readFileSync(xdg.config() + '/golembase/private.key');

const encoder = new TextEncoder()
const decoder = new TextDecoder()

async function main() {
  const client: GolemBaseClient = await createClient(
    new Tagged("privatekey", keyBytes), 'http://localhost:8545', 'ws://localhost:8546'
  )

  async function numOfEntitiesOwned(): Promise<number> {
    return (await client.getEntitiesOfOwner(await client.getOwnerAddress())).length
  }

  const creates: GolemBaseCreate[] = [
    {
      data: encoder.encode("foo"),
      ttl: 25,
      stringAnnotations: [new Annotation("key", "foo")],
      numericAnnotations: [new Annotation("ix", 1)]
    },
    {
      data: encoder.encode("bar"),
      ttl: 2,
      stringAnnotations: [new Annotation("key", "bar")],
      numericAnnotations: [new Annotation("ix", 2)]
    }
  ]
  console.log('Creating entities!')
  const receipts = await client.createEntities(creates)

  console.log("Number of entities owned:", await numOfEntitiesOwned())

  console.log("Current balance: ", formatEther(await client.getRawClient().httpClient.getBalance({
    address: await client.getOwnerAddress(),
    blockTag: 'latest'
  })))

  await (new Promise(resolve => setTimeout(resolve, 500)))

}

main()
```

Next, build it by typing:

`npm run build`

and run it by typing:

`npm run start`

You will see output similar to the following:

```
Creating entities!
Number of entities owned: 2
Current balance:  99.99999999999814816
```

Ready to see more? This example barely cracks the surface. To see a full example:

* Visit our [SDK documentation](https://golem-base.github.io/typescript-sdk/).

* Look at the full [code on GitHub](https://github.com/Golem-Base/typescript-sdk) including more examples.

# 🚗 Remove the rust with the Rust SDK

Would you prefer to work with Rust? We got you covered!

[Coming soon]

# 🚂 Ready to go go go with Go?

If you download our entire [Golem-base Op-Geth](https://github.com/Golem-Base/golembase-op-geth) project and install Go on your machine, you'll have everything you need to create applications in Go that interact with Geth.

First, make sure you have [Go installed](https://go.dev/doc/install).

Second, you'll want to install Golem-base Op-Geth, start it inside docker-compose, and create and fund an account as described above under [TypeScript SDK](./README.md#-golembase-sdk-for-typescript).

Next, clone our Golem-Base Op-Geth repo, as this contains the Go libraries you'll need:

```
git clone https://github.com/Golem-Base/golembase-op-geth.git
```

You'll probably want to familiarize yourself with the folder structure. 

[Coming soon]

# 🧑‍🚀 Get Involved

Golem Base is being developed in the open — and we're just getting started.

If you're interested in decentralized infrastructure, Ethereum scalability, or advancing on-chain data systems, we’d be glad to have you onboard.

* 📬 [Subscribe for updates](https://golem-base.io/)

* 💬 [Join the Discord](http://discord.gg/golem)

* 🧵 [Follow us on X (Formerly Twitter)](https://x.com/golemproject)

* 💼 [Come join our team! Explore Careers](https://golem.network/careers)
