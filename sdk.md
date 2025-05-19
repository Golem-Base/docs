# Platforms

The Golem Base TypeScript SDK can be used either in Node.js or on the web.

## Node applications

The TypeScript SDK for Golem Base can be used in combination with Node.js to create
backend or desktop/CLI applications that can interact with Golem Base.

The SDK repository includes an [example TypeScript application](https://github.com/Golem-Base/typescript-sdk/blob/main/example/index.ts)
showcasing the basic setup that allows you to create/update/delete/extend entities in Golem Base.

Make sure to take a look at the [online documentation](https://golem-base.github.io/typescript-sdk/modules).

## Web applications

The same SDK can also be used in the web browser.
You can include the SDK as a library in your own project and use a JavaScript bundler
to create a bundle that you can then deploy.
Alternatively, you can also directly import the bundled SDK if you want to try it
out quickly, like we do in the [example included in the SDK repo](https://github.com/Golem-Base/typescript-sdk/blob/main/example/index.html).

# Using the SDK

## Managing state using transactions

Creating, updating and deleting entities is done with transactions.

To create your first entity, you need to:
1. Fetch your private key, for instance by reading it from a file
1. Create a Golem Base client, by calling the [`createClient`](https://golem-base.github.io/typescript-sdk/functions/createClient.html) function, passing it
   the private key, the chain ID, and the JSON-RPC URL
1. Make sure you have funds on your account to run transactions
1. Create your first entity in Golem Base:
   ```typescript
     const creates: GolemBaseCreate[] = [
       {
         data: new TextEncoder().encode("foo"),
         btl: 25,
         stringAnnotations: [new Annotation("key", "foo")],
         numericAnnotations: [new Annotation("ix", 1)]
       },
     ]
     const receipts = await client.createEntities(creates)
   ```
   This creates an entity that will live for 25 blocks, has the string `"foo"` as
   content, and has two annotations attached to it.

From here, there are three other operations on existing entities:
1. [Update entities](https://golem-base.github.io/typescript-sdk/interfaces/GolemBaseClient.html#updateentities) to modify their data, annotations, or BTLs
1. [Extend entities](https://golem-base.github.io/typescript-sdk/interfaces/GolemBaseClient.html#extendentities) to simply extend the BTL by a given number of blocks
1. [Delete entities](https://golem-base.github.io/typescript-sdk/interfaces/GolemBaseClient.html#deleteentities)

Each of them have corresponding methods to create and execute a transaction
that will carry out the requested operation.

## Querying state using JSON-RPC methods

To query data on Golem Base, like retrieving entities based on their owner, or on
their annotations, there is a JSON-RPC API.

The client that you created earlier has methods for all the available operations, and the full
overview can be found in the [SDK documentation](https://golem-base.github.io/typescript-sdk/interfaces/GolemBaseClient.html).

For instance, to retrieve all entities with a numeric annotation with key `ix` and value `1`, `2` or `3`,
you would do the following:
```typescript
const results = await client.queryEntities("ix = 1 || ix = 2 || ix = 3"),
```
which would return a list containing for every matching entity its key and the value that the entity stores.

The methods you are probably most interested in are:
* `getAllEntityKeys` to get the keys of all the entities stored in Golem Base
* `getEntitiesOfOwner` to get the keys of all the entities owned by a given address
* `getEntitiesToExpireAtBlock` to get the keys of all the entities that expire at a given block
* `getEntityCount` to get the total count of entities stored in Golem Base
* `getEntityMetaData` to get the meta data of a given entity
* `getStorageValue` to get the storage value of a given entity
* `queryEntities` to retrieve entities with queries based on annotations

For the `queryEntities` method, we currently support querying by annotations.
A query is a set of conditions, separated by either `&&` (logical conjunction)
or `||` (logical disjunction).
Every condition is of the format `key = value`.
For string annotations, you enclose the value in double quotes, for numerical
annotations, the value needs to be an integer.
You can indicate precedence with parentheses.
So an example query, could look like this:
```
foo = "bar" && (index = 5 || class = "test")
```

### Making raw JSON-RPC requests

The recommended way to interact with Golem Base, is by using one of the SDKs.
However, sometimes it can be useful to make a JSON-RPC request yourself and see
the actual result that gets returned, for instance while debugging issues.

For instance, to call the `getAllEntityKeys` method, you can run the following command:
```sh
curl --location --silent --request POST --header "Content-Type: application/json" --data '{"jsonrpc": "2.0", "method": "golembase_getAllEntityKeys", "params": [], "id": 1}' https://api.golembase.demo.golem-base.io | jq
```

Which will return a list of all entity keys present in the demo instance.

For methods that take arguments, you put the arguments in the `params` list.

## Subscribing to events

You can also subscribe to events to follow the activity on Golem Base and take action
when needed.
To do so, you'd use the [`watchLogs`](https://golem-base.github.io/typescript-sdk/interfaces/GolemBaseClient.html#watchlogs-1) method.
This method takes four callbacks, one for each type of transaction on Golem Base,
and these callbacks are invoked whenever a corresponding transaction was added to
a block.

An example invocation could look like this:
```typescript
try {
  const client = createClient(...)
  const block = await client.getRawClient().httpClient.getBlockNumber()
  const unsubscribe = client.watchLogs({
    fromBlock: block,
    onCreated: (args) => {
      log.info("Got creation event:", args)
    },
    onUpdated: (args) => {
      log.info("Got update event:", args)
    },
    onExtended: (args) => {
      log.info("Got extension event:", args)
    },
    onDeleted: (args) => {
      log.info("Got deletion event:", args)
    },
    pollingInterval: 500,
    transport: "http",
  })
} finally {
    if (unsubscribe) {
        unsubscribe()
    }
}
```
Where the `unsubscribe` callback is used to stop the subscription.
