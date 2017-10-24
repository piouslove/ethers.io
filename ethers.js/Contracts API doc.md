# Contracts智能合约
这个API提供了一个与区块链上部署的合约的平滑连接，简化了调用和查询其功能，并且处理所有的二进制协议和必要的转换。
Contract对象是一个元类，所以很多功能只有在通过编译器，例如Solidity，生成的应用程序二进制接口（ABI）实例化之前才会定义。
为了更好地看到这一点，参见下面的例子。
```
var Contract = ethers.Contract;
```
## Connecting to a Contract连接到一个合约
```
new ethers.Contract ( addressOrName , interface , providerOrSigner )
```
连接到由可能是JSON字符串或被解析的对象的接口定义的在`addressOrName`处的合约。
`providerOrSigner`可以是以下任何实例：
### Wallet
钱包将用于签收和发送交易，估计`gas`和调用将使用钱包地址。
### Provider
可以调用估计`gas`和常量函数（但不包含地址），并且可以注册事件回调。
### Custom Signer
对于如何和何时签署和发送交易发生并延迟地址行为进行更多的控制。
## Prototype原型
原型将包含接口中定义的所有方法和事件。
The result of all contant methods are a Promise which resolve to the result as a tuple, optionally with the parameters accessible by name, if named in the ABI.
所有调用非常量方法的结果是一个解析发送到网络的交易的`promise`，与内置属性（下面）的名称冲突不会被覆盖。 相反，它们必须通过函数或事件属性进行访问。
由于签名重载，多个函数可以具有相同的名称。 JavaScript类型系统无法确定这些，因此只有具有给定名称的第一个函数才可用。（将来这将通过添加参数显式调用来解决）。
```
prototype.address
```
The address (or ENS name) of the contract.
```
prototype.interface
```
The Interface meta-class of the parsed ABI. Generally, this should not need to be accessed directly.
```
prototype.functions.functionName
```
An object that maps each ABI function name to a function that will either call (for contant functions) or sign and send a transaction (for non-constant functions)将每个ABI函数名称映射到将要调用（用于连续函数）或者签名并发送交易的函数（用于非常量函数）的对象
```
prototype.estimate.functionName
```
An object that maps each ABI function name to a function that will estimate the cost the provided parameters.
```
prototype.events.oneventname

```
An object that maps each ABI event name (lower case, with the “on” prefix) to a callback that is triggered when the event occurs.
### 示例
Example Contract and Interface
```
/**
 *  contract SimpleStore {
 *
 *      event valueChanged(address author, string value);
 *
 *      address _author;
 *      string _value;
 *
 *      function setValue(string value) {
 *          _author = msg.sender;
 *          _value = value;
 *          valueChanged(msg.sender, value);
 *      }
 *
 *      function getValue() constant returns (address author, string value) {
 *          return (_author, _value);
 *      }
 *  }
 */

 // The interface from the Solidity compiler
 var abi = [
     {
         "constant":true,
         "inputs":[],
         "name":"getValue",
         "outputs":[{"name":"author","type":"address"},{"name":"value","type":"string"}],
         "payable":false,
         "type":"function"
     },
     {
         "constant":false,
         "inputs":[{"name":"value","type":"string"}],
         "name":"setValue",
         "outputs":[],
         "payable":false,
         "type":"function"
     },
     {
         "anonymous":false,
         "inputs":[
             {"indexed":false,"name":"author","type":"address"},
             {"indexed":false,"name":"value","type":"string"}
         ],
         "name":"valueChanged",
         "type":"event"
     }
 ];

 var address = "";
 var provider = ethers.providers.getDefaultProvider();

 var contract = new ethers.Contract(address, abi, provider);
 ```
 Example Constant Function – `getValue ( ) returns ( address author , string value )`
```
var callPromise = contract.getValue();

callPromise.then(function(result) {

    // Solidity return tuples, which can be accessed by their
    // position or by their name.

    // The first entry of the return result (author)
    console.log('Positional argument (0):' + result[0]);
    console.log('Named argument (author): ' + result.author);

    // The second entry of the return result (value)
    console.log('Positional argument (1):' + result[1]);
    console.log('Named argument (value): ' + result.value);
});

// This is identical to the above call
// var callPromise = contract.functions.getValue();
```
Example Non-Constant Function – `setValue ( string value )`
```
var sendPromise = contract.setValue("Hello World");

sendPromise.then(function(transaction) {
    console.log(transaction);
});

// This is identical to the above send
// 下面这句与上面的功能一样
// var sendPromise = contract.functions.setValue("Hello World");
```
Example Event Registration – `valueChanged ( author , value )`
```
// Register for events
contract.onvaluechanged = function(author, value) {
    console.log('Author: ' + author);
    console.log('Value: ' + value);
};

// This is identical to the above event registry
// 下面这句与上面的功能一样
// contract.events.onvaluechanged = function(authot, value) { ...
```
Example Non-Constant Gas Estimate
```
var estimatePromise = contract.estimate.setValue("Hello World");

estimatePromise.then(function(gasCost) {
    // gasCost is returned as BigNumber
    console.log('Estimated Gas Cost: ' + gasCost.toString());
});
```
## Result Types结果类型
Solidity有许多变量类型，其中一些可以在JavaScript中工作，而其他类型的变量类型不适用于其中。 这里是一些关于合约中传递和返回值的注释。
### Integers整数
solidity整数是固定的位数（与最近的字节对齐），并且可用有符号和无符号的变量。
例如，`uint256`是256位（32字节）和无符号。 `int8`是8位（1字节）并有符号。
当类型为48位（6字节）或更少时，值作为`JavaScript Number`返回，因为`JavaScript Number`最多可以使用53位。
任何具有56位（7字节）或更多的类型将返回为`BigNumber`，即使该值在53位安全范围内。
当传递数值时，`JavaScript Number`，十六进制字符串或任何BigNumber是可以接受的（但是，当使用`JavaScript Number`并对其进行数学计算时要小心）。
`uint`和`int`类型分别是`uint256`和`int256`的别名。
### Strings
字符串工作正常，不需要特别小心。

要在字符串和字节之间进行转换（可能偶尔出现），请使用`utils.toUtf8Bytes()`和`utils.toUtf8String()`等实用工具函数。
### Bytes
字节可以是固定长度或动态长度的变量。在这两种情况下，值作为十六进制字符串返回，并且可以以十六进制字符串或数组形式传入。
要将字符串转换为数组，请使用实用工具函数`utils.arrayify()`
### Arrays
数组工作正常，不需要特别注意。
## Deploying a Contract部署合约
要将合约部署到Ethereum网络，您必须具有其字节码及其应用程序二进制接口（ABI），这些通常由Solidity编译器生成。
```
Contract.getDeployTransaction ( bytecode , interface , … )
```
生成部署由字节码和接口指定的合同所需的交易。 还应该传递构造函数所需要的任何其他参数。
### 示例
```
/**
 *  contract Example {
 *
 *      string _value;
 *
 *      // Constructor
 *      function Example(string value) {
 *          _value = value;
 *      }
 *  }
 */

// The interface from Solidity
var abi = '[{"inputs":[{"name":"value","type":"string"}],"type":"constructor"}]';

// The bytecode from Solidity
var bytecode = "0x6060604052341561000c57fe5b60405161012d38038061012d83398101604052" +
                 "8080518201919050505b806000908051906020019061003f929190610047565b" +
                 "505b506100ec565b828054600181600116156101000203166002900490600052" +
                 "602060002090601f016020900481019282601f1061008857805160ff19168380" +
                 "011785556100b6565b828001600101855582156100b6579182015b8281111561" +
                 "00b557825182559160200191906001019061009a565b5b5090506100c3919061" +
                 "00c7565b5090565b6100e991905b808211156100e55760008160009055506001" +
                 "016100cd565b5090565b90565b6033806100fa6000396000f30060606040525b" +
                 "fe00a165627a7a72305820041f440021b887310055b6f4e647c2844f4e1c8cf1" +
                 "d8e037c72cd7d0aa671e2f0029";

// Notice we pass in "Hello World" as the parameter to the constructor
var deployTransaction = Contract.getDeployTransaction(bytecode, abi, "Hello World");
console.log(deployTransaction);
// {
//    data: "0x6060604052341561000c57fe5b60405161012d38038061012d83398101604052" +
//            "8080518201919050505b806000908051906020019061003f929190610047565b" +
//            "505b506100ec565b828054600181600116156101000203166002900490600052" +
//            "602060002090601f016020900481019282601f1061008857805160ff19168380" +
//            "011785556100b6565b828001600101855582156100b6579182015b8281111561" +
//            "00b557825182559160200191906001019061009a565b5b5090506100c3919061" +
//            "00c7565b5090565b6100e991905b808211156100e55760008160009055506001" +
//            "016100cd565b5090565b90565b6033806100fa6000396000f30060606040525b" +
//            "fe00a165627a7a72305820041f440021b887310055b6f4e647c2844f4e1c8cf1" +
//            "d8e037c72cd7d0aa671e2f002900000000000000000000000000000000000000" +
//            "0000000000000000000000002000000000000000000000000000000000000000" +
//            "0000000000000000000000000b48656c6c6f20576f726c640000000000000000" +
//            "00000000000000000000000000"
// }

// Connect to the network
var provider = ethers.providers.getDefaultProvider();

// Create a wallet to deploy the contract with
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey, provider);

// Send the transaction
var sendPromise = wallet.sendTransaction(deployTransaction);

// Get the transaction
sendPromise.then(function(transaction) {
    console.log(transaction);
});
```
## Custom Signer
指定签名者的最简单方法是简单地使用钱包的实例。 但是，如果需要更细粒度的控制，自定义签名者可以延迟地址，签名和发送交易。
A signer can be any object with:
```
object.getAddress()
```
Required.  
Which must return a valid address or a Promise which will resolve to a valid address or reject an error.
```
object.provider
```
Required.  
A provider that will be used to connect to the Ethereum blockchain to issue calls, listen for events and possibly send transaction.
```
object.estimateGas ( transaction )
```
Optional.  
If this is not defined, the provider is queries directly, after populating the address using getAddress().

The result must be a Promise which resolves to the BigNumber estimated gas cost.
```
object . sendTransaction ( transaction )
```
Optional.  
If this is defined, it is called instead of sign and is expected to populate nonce, gasLimit and gasPrice.

The result must be a Promise which resolves to the sent transaction, or rejects on failure.
```
object . sign ( transaction )
```
Optional.  
If this is defined, it is called to sign a transaction before using the provider to send it to the network.

The result may be a valid hex string or a promise which will resolve to a valid hex string signed transaction or reject on failure.
### 示例
```
var privateKey = '0x0123456789012345678901234567890123456789012345678901234567890123';
var wallet = new ethers.Wallet(privateKey);

function getAddress() {
    return new Promise(function(resolve, reject) {
        // Some asynchronous method; some examples
        //  - request which account from the user
        //  - query a database
        //  - wait for another contract to be mined

        var address = wallet.address;

        resolve(address);
    });
}

function sign(transaction) {
    return new Promise(function(resolve, reject) {
        // Some asynchronous method; some examples
        //  - prompt the user to confirm or decline
        //  - check available funds and credits
        //  - request 2FA over SMS

        var signedTransaction = wallet.sign(transaction);

        resolve(signedTransaction);
    });
}

var customSigner = {
    getAddress: getAddress,
    provider: ethers.providers.getDefaultProvider(),
    sign: sign
}
```