# 钱包
一个钱包管理一个私钥/公钥对，用于密码地签署交易，并在Ethereum网络上证明所有权。
```
// Using the Ethers umbrella package...
var ethers = require('ethers');
var Wallet = ethers.Wallet;

// ... 或者 ...

// Using the wallet specific package...
var Wallet = require('ethers-wallet').Wallet;
```
## 创建实例
```
new ethers.Wallet( privateKey [ , provider ] )
```
依靠`privateKey`参数创建一个实例并依靠一个可选参数连接到一个`provider`
```
Wallet.createRandom ( [ options ] )
```
创建随机钱包，可选指定额外的熵`extraEntropy`来搅拌随机来源（确保这个钱包存储在某个安全地方；如果丢失了没有办法恢复）
```
Wallet.fromEncryptedWallet ( json, password [ , progressCallback ] )
```
Decrypt an encrypted Secret Storage JSON Wallet (由Geth创建,或由`prototype.encrypt`创建的钱包)
```
Wallet.fromMnemonic ( mnemonic [ , path ] )
```
Generate a BIP39 + BIP32 wallet from a mnemonic助记符 deriving path  
**default**: `path=”m/44’/60’/0’/0/0”`
```
Wallet.fromBrainWallet ( username , password [ , progressCallback ] )
```
从用户名和密码生成钱包(脑钱包)
### 示例
#### Private Key私钥
```
var privateKey = "0x0123456789012345678901234567890123456789012345678901234567890123";
var wallet = new Wallet(privateKey);

console.log("Address: " + wallet.address);
// "Address: 0x14791697260E4c9A71f18484C9f997B308e59325"
```
#### Random Wallet随机钱包
```
var wallet = Wallet.createRandom();
console.log("Address: " + wallet.address);
// "Address: ... 每次都不一样 ..."
```
Secret Storage Wallet (e.g. Geth or Parity)来自Geth或Parity的安全存储钱包
```
var data = {
    id: "fb1280c0-d646-4e40-9550-7026b1be504a",
    address: "88a5c2d9919e46f883eb62f7b8dd9d0cc45bc290",
    Crypto: {
        kdfparams: {
            dklen: 32,
            p: 1,
            salt: "bbfa53547e3e3bfcc9786a2cbef8504a5031d82734ecef02153e29daeed658fd",
            r: 8,
            n: 262144
        },
        kdf: "scrypt",
        ciphertext: "10adcc8bcaf49474c6710460e0dc974331f71ee4c7baa7314b4a23d25fd6c406",
        mac: "1cf53b5ae8d75f8c037b453e7c3c61b010225d916768a6b145adf5cf9cb3a703",
        cipher: "aes-128-ctr",
        cipherparams: {
            iv: "1dcdf13e49cea706994ed38804f6d171"
         }
    },
    "version" : 3
};

var json = JSON.stringify(data);
var password = "foo";
Wallet.fromEncryptedWallet(json, password).then(function(wallet) {
    console.log("Address: " + wallet.address);
    // "Address: 0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290"
});
```
#### Mnemonic Phrase助记符
```
var mnemonic = "radar blur cabbage chef fix engine embark joy scheme fiction master release";
var wallet = Wallet.fromMnemonic(mnemonic);

console.log("Address: " + wallet.address);
// "Address: 0xaC39b311DCEb2A4b2f5d8461c1cdaF756F4F7Ae9"
```
#### 脑钱包
```
var username = "support@ethers.io";
var password = "password123";
Wallet.fromBrainWallet(username, password).then(function(wallet) {
    console.log("Address: " + wallet.address);
    // "Address: 0x7Ee9AE2a2eAF3F0df8D323d555479be562ac4905"
});
```
## 原型Prototype
```
prototype.address
```
The public address of a wallet
```
prototype.privateKey
```
The private key of a wallet; keep this secret
```
prototype.provider
```
Optional; a connected Provider which allows the wallet to connect to the Ethereum network to query its state and send transactions
```
prototype.getAddress ( )
```
A function which returns the address; for Wallet, this simple returns the address property
```
prototype.sign ( transaction )
```
Signs transaction and returns the signed transaction as a hex string. See Transaction Requests.
```
prototype.signMessage ( message )
```
Signs message and returns the signature as a hex string.
```
prototype.encrypt ( password [ , options ] [ , progressCallback ] )
```
Returns a Promise with the wallet encrypted as a Secret Storage JSON Wallet; options may include overrides for the scypt parameters.
### 示例
#### Signing Transactions
```
var privateKey = "0x0123456789012345678901234567890123456789012345678901234567890123";
var wallet = new Wallet(privateKey);

console.log('Address: ' + wallet.address);
// "Address: 0x14791697260E4c9A71f18484C9f997B308e59325".

var transaction = {
    nonce: 0,
    gasLimit: 21000,
    gasPrice: utils.bigNumberify("20000000000"),

    to: "0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290",

    value: utils.parseEther("1.0"),
    data: "0x",

    // 下面这句确保交易不能在不同网络上重播
    chainId: providers.Provider.chainId.homestead
};

var signedTransaction = wallet.sign(transaction);

console.log(signedTransaction);
// "0xf86c808504a817c8008252089488a5c2d9919e46f883eb62f7b8dd9d0cc45bc2" +
//   "90880de0b6b3a7640000801ca0d7b10eee694f7fd9acaa0baf51e91da5c3d324" +
//   "f67ad827fbe4410a32967cbc32a06ffb0b4ac0855f146ff82bef010f6f2729b4" +
//   "24c57b3be967e2074220fca13e79"

// 现在可以发送到以太坊网络
provider.sendTransaction(signedTransaction).then(function(hash) {
    console.log('Hash: ' + hash);
    // Hash:
});
```
#### Encrypting
```
var password = "password123";

function callback(percent) {
    console.log("Encrypting: " + parseInt(percent * 100) + "% complete");
}

var encryptPromise = wallet.encrypt(password, callback);

encryptPromise.then(function(json) {
    console.log(json);
});
```
## Blockchain Operations区块链操作
这些操作要求钱包附有`provider`以保证连接到了区块链网络。
```
prototype.getBalance ( [ blockTag ] )
```
Returns a `Promise` with the balance of the wallet (as a `BigNumber`, in `wei`) at the `blockTag`.  
**default**: `blockTag=”latest”`
```
prototype.getTransactionCount ( [ blockTag ] )
```
返回一个包含在指定的`blockTag`中帐户已发送交易的数量的 `Promise` (也叫 `nonce`)  
**default**: `blockTag=”latest”`
```
prototype.estimateGas ( transaction )
```
Returns a Promise with the estimated cost for `transaction` (in gas, as a `BigNumber`)交易的估计成本
```
prototype.sendTransaction ( transaction )
```
Sends the `transaction` to the network and returns a Promise with the transaction details. It is highly recommended to omit省略 `transaction.chainId`, it will be filled in by `provider`.
```
prototype.send ( addressOrName, amountWei [ , options ] )
```
Sends `amountWei` to `addressOrName` on the network and returns a Promise with the transaction details.
### 示例
#### Query the Network
```
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);
wallet.provider = ethers.providers.getDefaultProvider();

var balancePromise = wallet.getBalance(address);

balancePromise.then(function(balance) {
    console.log(balance);
});

var transactionCountPromise = wallet.getTransactionCount(address);

transactionCountPromise.then(function(transactionCount) {
    console.log(transactionCount);
});
```
#### Transfer Ether
```
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);
wallet.provider = ethers.providers.getDefaultProvider();

// We must pass in the amount as wei (1 ether = 1e18 wei), so we use
// this convenience function to convert ether to wei.
// 单位转换
var amount = ethers.parseEther('1.0');

var sendPromise = wallet.send(address, amount);

sendPromise.then(function(transactionHash) {
    console.log(transactionHash);
});


// These will query the network for appropriate values
var options = {
    //gasLimit: 21000
    //gasPrice: utils.bigNumberify("20000000000")
};

var promiseSend = wallet.send(address, amount, options);

promiseSend.then(function(transaction) {
    console.log(transaction);
});
```
#### Sending (Complex) Transactions
```
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);
wallet.provider = ethers.providers.getDefaultProvider();

var transaction = {
    // Recommendation: omit nonce; the provider will query the network
    // 建议：省略nonce Provider将查询网络自动获取
    // nonce: 0,

    // Gas Limit; 21000 will send ether to another use, but to execute contracts
    // larger limits are required. The provider.estimateGas can be used for this.
    gasLimit: 1000000

    // Recommendations: omit gasPrice; the provider will query the network
    // 建议：省略gasPrice Provider将查询网络自动获取
    //gasPrice: utils.bigNumberify("20000000000"),

    // Required; unless deploying a contract (in which case omit)
    to: "0x88a5C2d9919e46F883EB62F7b8Dd9d0CC45bc290",

    // Optional
    data: "0x",

    // Optional
    value: ethers.utils.parseEther("1.0"),

    // Recommendation: omit chainId; the provider will populate this
    // 建议：省略chaindId Provider将自动填充
    // chaindId: providers.Provider.chainId.homestead
};

// Estimate the gas cost for the transaction
//var estimateGasPromise = wallet.estimateGas(transaction);

//estimateGasPromise.then(function(gasEstimate) {
//    console.log(gasEstimate);
//});

// Send the transaction
var sendTransactionPromise = wallet.sendTransaction(transaction);

sendTransactionPromise.then(function(transactionHash) {
    console.log(transactionHash);
});
```
## Parsing Transactions
```
Wallet.parseTransaction ( hexStringOrArrayish )
```
Parses a raw hexStringOrArrayish into a Transaction.
### 示例
```
// Mainnet:
var raw = "0xf87083154262850500cf6e0083015f9094c149be1bcdfa69a94384b46a1f913" +
            "50e5f81c1ab880de6c75de74c236c8025a05b13ef45ce3faf69d1f40f9d15b007" +
            "0cc9e2c92f"
var transaction = {
    to: 0123...
    value: 0123...
};
var signedTransaction = wallet.sign(transaction);
var transaction = Wallet.parseTransaction(signedTransaction);

console.log(transaction);
// {
//     to: "0xc149Be1bcDFa69a94384b46A1F91350E5f81c1AB",
//     from: "0xEA674fdDe714fd979de3EdF0F56AA9716B898ec8",
//
//     chainId: 1,
//
//     gasLimit: utils.bigNumberify("90000"),
//     gasPrice: utils.bigNumberify("21488430592"),
//
//     nonce: 1393250
//     data: "0x",
//     value: utils.parseEther("1.0017071732629267"),
//
//     r: "0x5b13ef45ce3faf69d1f40f9d15b0070cc9e2c92f3df79ad46d5b3226d7f3d1e8",
//     s: "0x535236e497c59e3fba93b78e124305c7c9b20db0f8531b015066725e4bb31de6",
//     v: 37,
// }
```
## Verifying Messages
```
Wallet.verifyMessage ( message , signature )
```
Returns the address that signed message with signature.
### 示例
```
var signature = "0xddd0a7290af9526056b4e35a077b9a11b513aa0028ec6c9880948544508f3c63" +
                  "265e99e47ad31bb2cab9646c504576b3abc6939a1710afc08cbf3034d73214b8" +
                  "1c" +
var address = Wallet.verifyMessage('hello world', signature);
console.log(address);
// '0x14791697260E4c9A71f18484C9f997B308e59325'
```