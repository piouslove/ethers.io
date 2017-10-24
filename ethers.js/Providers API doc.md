# Providers
`Provider`提取与Ethereum区块链的连接，用于发出查询并发送状态更改交易。  
与`Web3 Provider`不同，在`Ethers Provider`中没有帐户或签名实体的概念。 它只是一个与网络的连接，不能解锁帐户或签名，也并不知道您的以太坊地址。  
要管理状态更改操作，您必须使用`Wallet`来签署交易。 如果您将一个钱包作为签约人交给合约，则由合约管理。  
创建实例：
```
var providers = ethers.providers;
```
## 连接到以太坊
有以下几种方法可以连接到Ethereum区块链：  
```
new providers.EtherscanProvider( [ testnet ] [ , apiToken ] )
```
Connect to the [Etherscan](https://etherscan.io/apis) blockchain [web service API](https://docs.ethers.io/ethers.js/html/etherscan-api).  
**default:** `testnet=false, apiToken=null`
```
new providers.JsonRpcProvider( [ url ] [ , testnet ] [, chainId ] )
```
Connect to the [JSON-RPC API](https://github.com/ethereum/wiki/wiki/JSON-RPC) url of an Ethereum node, such as Parity or Geth.  
**default:** `url=”http://localhost:8545/”, testnet=false, chainId=network default`
```
new providers.InfuraProvider( [ testnet ] [ , apiAccessToken ] )
```
Connect to the [INFURA](https://infura.io/) hosted network of Ethereum nodes.  
**default:** `testnet=false, apiAccessToken=null`
```
new providers.FallbackProvider( providers )
```
通过依次尝试每个提供者来提高可靠性，如果遇到错误，则返回列表中的下一个。
Improves reliability by attempting each provider in turn, falling back to the next in the list if an error was encountered.
```
providers.getDefaultProvider( [ testnet ] )
```
This automatically creates a FallbackProvider backed by INFURA and Etherscan; recommended若想连接以太坊主网推荐用这个  
**default:** `testnet=false`
### 示例
```
var providers = require('ethers').providers;

// Connect to Ropsten (the test network)
var testnet = true;

// Connect to INFUA
var infuraProvider = new providers.InfuraProvider(testnet);

// Connect to Etherscan
var etherscanProvider = new providersInfuraProvider(testnet);

// Creating a provider to automatically fallback onto Etherscan
// if INFURA is down
var fallbackProvider = new providers.FallbackProvider([
    infuraProvider,
    etherscanProvider
]);

// This is equivalent to using the getDefaultProvider
var provider = providers.getDefaultProvider(testnet)

// Connect to a local Parity instance
var provider = new providers.JsonRpcProvider('http://localhost:8545', testnet);
```
## Prototype原型
所有属性都是不可变的，如果没有指定，则反映其默认值，或者由子对象间接填充。All properties are immutable, and reflect their default value if not specified, or if indirectly populated by child Objects.
### Provider
```
prototype.testnet
```
Whether the provider is on the testnet (Ropsten)
```
prototype.chainId
```
The chain ID (or network ID) this provider is connected as; this is used by signers to prevent [replay attacks](https://docs.ethers.io/ethers.js/html/replay-attack) across compatible networks
### FallbackProvider ( inherits from Provider )
```
prototype.providers
```
A copy of the array of providers (modifying this variable will not affect the providers attached)
### JsonRpcProvider ( inherits from Provider )
```
prototype . url
```
The JSON-RPC URL the provider is connected to
将要连接到的`provider`的`JSON-RPC URL`
```
prototype . send ( method , params )
```
Send the JSON-RPC method with params. This is useful for calling non-standard or less common JSON-RPC methods. A Promise is returned which will resolve to the parsed JSON result.使用参数发送JSON-RPC方法。这对于调用非标准或不常用的JSON-RPC方法非常有用。将返回一个将会解析为JSON结果的`promise`
### EtherscanProvider ( inherits from Provider )
```
prototype . apiToken
```
The Etherscan API Token (or null if not specified)
### InfuraProvider ( inherits from JsonRpcProvider )
```
prototype . apiAccessToken
```
The INFURA API Access Token (or null if not specified)
## Account Actions帐户行为
```
prototype.getBalance ( addressOrName [ , blockTag ] )
```
Returns a `Promise` with the balance (as a `BigNumber`) of `addressOrName` at `blockTag`. (See: Block Tags)  
**default:** `blockTag=”latest”`
```
prototype.getTransactionCount ( addressOrName [ , blockTag ] )
```
Returns a `Promise` with the number of sent transactions (as a Number) from `addressOrName` at `blockTag`. This is also the `nonce` required to send a new transaction. (See: Block Tags)这也是帐户发送新交易时必须的那个`nonce`值  
**default:** `blockTag=”latest”`
```
prototype.lookupAddress ( address )
```
Returns a Promise which resolves to the ENS name (or null) that address resolves to.
```
prototype.resolveName ( ensName )
```
Returns a Promise which resolves to the address (or null) of that the ensName resolves to.
### 示例
```
var provider = providers.getDefaultProvider();

var address = "0x02F024e0882B310c6734703AB9066EdD3a10C6e0";

provider.getBalance(address).then(function(balance) {

    // balance is a BigNumber (in wei); format is as a sting (in ether)
    var etherString = ethers.utils.formatEther(balance);

    console.log("Balance: " + etherString);
});

provider.getTransactionCount(address).then(function(transactionCount) {
    console.log("Total Transactions Ever Send: " + transactionCount);
});

provider.resolveName("test.ricmoose.eth").then(function(address) {
    console.log("Address: " + address);
});
```
## Blockchain Status
```
prototype.getBlockNumber ( )
```
Returns a Promise with the latest block number (as a Number).
```
prototype.getGasPrice ( )
```
Returns a Promise with the current gas price (as a BigNumber).
```
prototype.getBlock ( blockHashOrBlockNumber )
```
Returns a Promise with the block at blockHashorBlockNumber. (See: Block Responses)
```
prototype.getTransaction ( transactionHash )
```
Returns a Promise with the transaction with transactionHash. (See: Transaction Results)
```
prototype.getTransactionReceipt ( transactionHash )
```
Returns a Promise with the transaction receipt with transactionHash. (See: Transaction Receipts)
## 示例
#### Current State
```
var provider = providers.getDefaultProvider();

provider.getBlockNumber().then(function(blockNumber) {
    console.log("Current block number: " + blockNumber);
});

provider.getGasPrice().then(function(gasPrice) {
    // gasPrice is a BigNumber; convert it to a decimal string
    gasPriceString = gasPrice.toString();

    console.log("Current gas price: " + gasPriceString);
});
```
#### Blocks
```
var provider = providers.getDefaultProvider();

// Block Number
provider.getBlock(3346773).then(function(block) {
    console.log(block);
});

// Block Hash
var blockHash = "0x7a1d0b010393c8d850200d0ec1e27c0c8a295366247b1bd6124d496cf59182ad";
provider.getBlock(blockHash).then(function(block) {
    console.log(block);
});
```
#### Transactions
```
var provider = providers.getDefaultProvider();

var transactionHash = "0x7baea23e7d77bff455d94f0c81916f938c398252fb62fce2cdb43643134ce4ed";

provider.getTransaction(transactionHash).then(function(transaction) {
    console.log(transaction);
});

provider.getTransactionReceipt(transactionHash).then(function(transactionReceipt) {
    console.log(transactionReceipt);
});
```
## Ethereum Name Resolution
Ethereum命名服务（ENS）允许轻松记住并使用名称分配给Ethereum地址。 采取地址的任何`provider`操作也可以采用ENS名称。

解决用户输入的名称或执行地址的反向查找以获得更人性化的阅读名称通常很有用。
#### Resolving Names
```
provider.resolveName('ricmoo.firefly.eth').then(function(address) {
    console.log(address);
    // '0x32DEF047DeFd076DB21A2D759aff2A591c972248'
});
```
#### Looking up Addresses
```
provider.lookupAddress('0x32DEF047DeFd076DB21A2D759aff2A591c972248').then(function(name) {
    console.log(name);
    // 'ricmoo.firefly.eth'
});
```
## Contract Execution合约执行
这些是相对低级的调用，通常应该使用Contracts API
```
prototype.call ( transaction )
```
将只读（常量）交易发送到单个Ethereum节点，并返回它的执行结果（十六进制字符串）的Promise。这种调用不用花费`gas`，因为它不会改变区块链上的任何状态。
```
prototype.estimateGas ( transaction )
```
发送一个交易到一个单独的Ethereum节点，并返回一个包含发送它所需的`gas`预估量的Promise（一个BigNumber）。这种调用不用花费`gas`，但也只是一个估计。 提供太少的`gas`将导致交易被拒绝（同时仍然消耗所有已提供的`gas`）。
```
prototype.sendTransaction ( signedTransaction )
```
Send the signedTransaction to the entire Ethereum network and returns a Promise with the transaction hash.

This will consume gas from the account that signed the transaction.
## Contract State合约状态
```
prototype.getCode ( addressOrName )
```
Returns a Promise with the bytecode (as a hex string) at addressOrName.
```
prototype.getStorageAt ( addressOrName , position [ , blockTag ] )
```
Returns a Promise with the value (as a hex string) at addressOrName in position at blockTag. (See Block Tags)  
**default:** `blockTag= “latest”`
```
prototype.getLogs ( filter )
```
Returns a Promise with an array (possibly empty) of the logs that match the filter. (See Filters)
## 事件Events
这些方法允许对区块链和合约上的某些事件管理回调。它们主要基于EventEmitter API。
```
prototype.on ( eventType , callback )
```
Register a callback for any future `eventType`; see below for callback parameters
```
prototype.once ( eventType , callback)
```
Register a callback for the next (and only next) `eventType`; see below for callback parameters
```
prototype.removeListener ( eventType , callback )
```
Unregister the `callback` for eventType; if the same callback is registered more than once, only the first registered instance is removed
```
prototype.removeAllListeners ( eventType )
```
Unregister all callbacks for `eventType`
```
prototype.listenerCount ( [ eventType ] )
```
Return the number of callbacks registered for `eventType`, or if ommitted, the total number of callbacks registered
```
prototype.resetEventsBlock ( blockNumber )
```
Begin scanning for events from `blockNumber`. By default, events begin at the block number that the provider began polling at.
### Event Types
#### “block”
Whenever a new block is mined
```
callback( blockNumber )
```
#### any address
When the balance of the coresposding address changes.
```
callback( balance )
```
#### any transaction hash
When the coresponding transaction is mined; also see Waiting for Transactions
```
callback( transaction )
```
#### an array of topics
When any of the topics are triggered in a block’s logs; when using the Contract API, this is automatically handled;
```
callback( log )
```
### Waiting for Transactions
```
prototype.waitForTransaction ( transachtionHash [ , timeout ] )
```
Return a Promise which returns the transaction once transactionHash is mined, with an optional timeout (in milliseconds)
### 示例
```
// Get notified on every new block
provider.on('block', function(blockNumber) {
    console.log('New Block: ' + blockNumber);
});

// Get notified on account balance change
provider.on('0x46Fa84b9355dB0708b6A57cd6ac222950478Be1d', function(blockNumber) {
    console.log('New Block: ' + blockNumber);
});

// Get notified when a transaction is mined
provider.once(transactionHash, function(transction) {
    console.log('Transaction Minded: ' + transaction.hash);
    console.log(transaction);
);

// OR equivalently the waitForTransaction() returns a Promise

provider.waitForTransaction(transactionHash).then(function(transaction) {
    console.log('Transaction Minded: ' + transaction.hash);
    console.log(transaction);
});


// Get notified when a contract event is logged
provider.on([ eventTopic ], function(log) {
    console.log('Event Log');
    console.log(log);
});
```
## Objects对象
### Block Tag区块标记
A block tag is used to uniquely identify a block’s position in th blockchain
区块标记用于唯一标识区块在区块链中的位置：
#### a Number or hex string:
Each block has a block number, eg. 42 or "0x2a.
#### “latest”:
The most recently mined block.
#### “pending”:
The block that is currently being mined.
### Block Responses区块响应
```
{
    parentHash: "0x3d8182d27303d92a2c9efd294a36dac878e1a9f7cb0964fa0f789fa96b5d0667",
    hash: "0x7f20ef60e9f91896b7ebb0962a18b8defb5e9074e62e1b6cde992648fe78794b",
    number: 3346463,

    difficulty: 183765779077962,
    timestamp: 1489440489,
    nonce: "0x17060cb000d2c714",
    extraData: "0x65746865726d696e65202d20555331",

    gasLimit: utils.bigNumberify("3993225"),
    gasUsed: utils.bigNuberify("3254236"),

    miner: "0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8",
    transactions: [
        "0x125d2b846de85c4c74eafb6f1b49fdb2326e22400ae223d96a8a0b26ccb2a513",
        "0x948d6e8f6f8a4d30c0bd527becbe24d15b1aba796f9a9a09a758b622145fd963",
        ... *[ 49 more transaction hashes ]* ...
        "0xbd141969b164ed70388f95d780864210e045e7db83e71f171ab851b2fba6b730"
    ]
}
```
### Transaction Requests交易请求
任何接受数字的属性也可以被指定为BigNumber或十六进制字符串。Any property which accepts a number may also be specified as a BigNumber or hex string.
```
// Example:
{
    // Required unless deploying a contract (in which case omit)
    // 必须，除非是部署合约
    // 目标地址或名称
    to: addressOrName,  // the target address or ENS name

    // These are optional/meaningless for call and estimateGas
    // 这些对于调用或估计gas是可选或无意义的
    nonce: 0,           // the transaction nonce
    gasLimit: 0,        // 本交易可能花费的最大gas量
    gasPrice: 0,        // the price (in wei) per unit of gas

    // These are always optional (but for call, data is usually specified)
    data: "0x",         // extra data for the transaction, or input for call
    value: 0,           // the amount (in wei) this transaction is sending
    chainId: 3          // the network ID; usually added by a signer
}
```
### Transaction Results
```
// Example:
{
    // Only available for mined transactions
    blockHash: "0x7f20ef60e9f91896b7ebb0962a18b8defb5e9074e62e1b6cde992648fe78794b",
    blockNumber: 3346463,
    transactionIndex: 51,

    // Exactly one of these will be present (send vs. deploy contract)
    creates: null,
    to: "0xc149Be1bcDFa69a94384b46A1F91350E5f81c1AB",

    // The transaction hash
    hash: "0xf517872f3c466c2e1520e35ad943d833fdca5a6739cfea9e686c4c1b3ab1022e",

    // See above (Transaction Requests) for these explained
    data: "0x",
    from: "0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8",
    gasLimit: utils.bigNumberify("90000"),
    gasPrice: utils.bigNumberify("21488430592"),
    nonce: 0,
    value: utils.parseEther(1.0017071732629267),

    // The network ID (or chain ID); 0 indicates replay-attack vulnerable
    // (eg. 1 = Homestead mainnet, 3 = Ropsten testnet)
    networkId: 1,

    // The signature of the transaction
    r: "0x5b13ef45ce3faf69d1f40f9d15b0070cc9e2c92f3df79ad46d5b3226d7f3d1e8",
    s: "0x535236e497c59e3fba93b78e124305c7c9b20db0f8531b015066725e4bb31de6",
    v: 37,

    // The raw transaction
    raw: "0xf87083154262850500cf6e0083015f9094c149be1bcdfa69a94384b46a1f913" +
           "50e5f81c1ab880de6c75de74c236c8025a05b13ef45ce3faf69d1f40f9d15b0" +
           "070cc9e2c92f3df79ad46d5b3226d7f3d1e8a0535236e497c59e3fba93b78e1" +
           "24305c7c9b20db0f8531b015066725e4bb31de6"
}
```
### Transaction Receipts
```
// Example
{
    transactionHash: "0x7dec07531aae8178e9d0b0abbd317ac3bb6e8e0fd37c2733b4e0d382ba34c5d2",

    // The block this transaction was mined into
    blockHash: "0xca1d4d9c4ac0b903a64cf3ae3be55cc31f25f81bf29933dd23c13e51c3711840",
    blockNumber: 3346629,

    // The index into this block of the transaction
    transactionIndex: 1,

    // The address of the contract (if one was created)
    contractAddress: null,

    // Gas
    cumulativeGasUsed: utils.bigNumberify("42000"),
    gasUsed: utils.bigNumberify("21000"),

    // Logs
    log: [ ],
    logsBloom: "0x00" ... [ 256 bytes of 0 ] ... "00",

    // State root
    root: "0x8a27e1f7d3e92ae1a01db5cce3e4718e04954a34e9b17c1942011a5f3a942bf4",
}
```
### Filters
主题过滤支持一个有些复杂的规范，但是对于绝大多数的过滤器，单个主题通常就足够了（参见下面的示例）。`EtherscanProvider`仅支持单个主题。
```
// Example
{
    // Optional; The range of blocks to limit querying (See: Block Tags above)
    fromBlock: "latest",
    toBlock: "latest",

    // Optional; An address (or ENS name) to filter by
    address: addressOrName,

    // Optional; A (possibly nested) list of topics
    topics: [ topic1 ]
}
```
## Provider Specific Extra API Calls
```
EtherscanProvider . getEtherPrice()
```
Returns a Promise with the price of ether in USD.
### Examples
```
provider.EtherscanProvider.getEtherPrice().then(function(price) {
    console.log("Ether price in USD: " + price);
});
```