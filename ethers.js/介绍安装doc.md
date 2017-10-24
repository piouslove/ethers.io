# 介绍
这个库（由ethers.io制作和使用）旨在使编写客户端JavaScript的钱包变得更加容易，并且始终保持私钥在所有者机器上。
`ethers.js`库是一个紧凑且完整的Ethereum JavaScript库。
# Getting Started
## `Node.js`中安装
从项目目录：
```
/Users/ricmoo/my-project> npm install --save ethers
```
然后在相关应用文件中：
```
var ethers = require('ethers');
```
## 在WEB应用中导入
为了安全起见，通常最好将这个[脚本](https://cdn.ethers.io/scripts/ethers-v2.0.min.js)复制一份放到应用服务器上，但是使用Ethers CDN(内容分发网络)的一个快速原型也应该可以：
```
<!-- This exposes the library as a global variable: ethers -->
<script src="https://cdn.ethers.io/scripts/ethers-v2.0.min.js" type="text/javascript">
</script>
```
