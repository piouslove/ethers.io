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