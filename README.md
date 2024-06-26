# HyPCjs

_basic, pure-frontend implementation of HyPC node interaction_

## Purpose

Letting you write front-end applications, deployable anywhere (including github pages), that use HyPC nodes to perform work. 

_**Note:** This API assumes the use of MetaMask for crypto payment management. If you're using the `eth` interface, your users will need to use that interface for this to work._

You can see examples of apps using this API at

- [serverless-groq](https://hypercycle-development.github.io/serverless-groq-demo/) ([repo](https://github.com/hypercycle-development/serverless-groq-demo/))
- [serverless-voiceboard](https://hypercycle-development.github.io/pure-client-voiceboard-demo/) ([repo](https://github.com/hypercycle-development/pure-client-voiceboard-demo))

## Usage

- Include the JS files from this repo in your HTML. You can either do this by checking the repo out and including them locally, or by hosting them directly from https://hypercycle-development.github.io/hypc-js/

To host directly, add

```
<script src="https://hypercycle-development.github.io/hypc-js/metamask-sdk.js"></script>
<script src="https://hypercycle-development.github.io/hypc-js/web3.min.js"></script>
<script src="https://hypercycle-development.github.io/hypc-js/hypc.js"></script>
```

to your `html`.

- In your application, initialize a client by choosing the payment route (currently only `eth` is available), and then calling `init` on the client.

```
const client = HyPC.eth("https://target.node.address", "Your Application Name");
client.init().then(_ => client.fetchBalance()).then(console.log);
```

You can now call your node, and invoke its' AIMs as you please. See the API write-up below for a more detailed description of what's possible.

## API

### Public API

_Functions exposed as the primary public interface. These might change in minor ways on major versions of the API, but will likely always be present and maintain type signatures, and be consistent across payment processors. Newer versions will be backward-compatible unless that's literally impossible._

_Use with confidence._

#### `init()`

Initializes the HyPC library. This method must be called before using other functionalities. Returns a Promise that resolves once initialization is complete.

#### `aims()`

Returns a map of AIM interfaces on the given node. The AIMs are mapped by addressable name, so you can do things like `client.aims().my_aim_name.fetchEstimate("my_endpoint")`. This will work whether the actual aim is named `my_aim_name`, `my-aim-name` or `my/aim/name`.

Each AIM has its own interface. The common elements guaranteed to be exposed for all of them are

- `info`: a map of image and hardware info for the given aim. You can check what version of the aim is running here, as well as its', name, tag, image ID, current status and GPU resources.

- `fetchManifest()`: Fetches the manifest object from the AIM. Returns a promise that will resolve to the AIM manifest map. It'll let you do things like programmatically figuring endpoints/methods/data requirements and the AIM license.

- `fetchEstimate(endpoint, data, options)`: Fetches a cost estimate for executing a specific task. Returns a Promise that resolves with the estimated cost for the task.
  - `endpoint`: The endpoint or task to execute.
  - `data` (optional, but rarely omitted): Data required for the task.
  - `options` (optional): Additional options for the request. This is `fetch` options, so you can specify `method` and `headers` if you like.

- `fetchResult(endpoint, data, options)`: Executes a task through the AIM and fetches the result. Handles nonce-signing through the MetaMask SDK, and returns a promise that resolves with the JSON-decoded task result.
  - `endpoint`: The endpoint or task to execute.
  - `data` (optional, but rarely omitted): Data required for the task.
  - `options` (optional): Additional options for the request. This is `fetch` options, so you can specify `method` and `headers` if you like.
  

#### `sendToNode(value)`

Sends a specified value of HyPC tokens to the node address. Returns a Promise that resolves with an object containing the updated balance and status of the transaction.

#### `fetchBalance()`

Fetches the balance of HyPC tokens for the current user account. Returns a Promise that resolves with the balance object.

#### `intAsHyPC(hypc_int)`

Converts an integer representing HyPC tokens to its decimal representation. Takes an integer value and returns a string with the decimal representation.

#### `version`

Returns the current version of the HyPC library as a string.

### Utility API

_Functions needed internally by the public API that you might also find useful in some contexts. So they're exported. Not guaranteed to be around forever._

#### `utils.ASCIItoHex(ascii)`

Converts ASCII characters to their hexadecimal representation.

#### `utils.toSnakeCase(str)`

Converts a string to snake case format.

#### `utils.insertDecimal(integer, decimal)`

Inserts a decimal point into an integer at the specified position.


### Internals API

_Functions needed for debugging/development purposes. Exposed to the API user, but not guaranteed to remain exposed, or retain the same signatures. Use in end-user code at your own risk._

#### `internals.contract`

Provides access to the HyPCContract instance.

#### `internals.nodeFetch(endpoint, options)`

Performs a fetch request to the specified endpoint on the node. Returns a Promise that resolves with the JSON response.

#### `internals.aimFetch(aimSlot, endpoint, options)`

Performs a fetch request to the specified endpoint for the given AIM slot. Returns a Promise that resolves with the JSON response.

#### `internals.nodeInfo()`

Retrieves information about the connected node. Returns an object containing node information.
