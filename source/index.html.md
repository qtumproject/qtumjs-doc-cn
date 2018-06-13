---
title: QtumJS API Reference

language_tabs:
  - typescript
  - javascript

search: true
---

# 介绍

> 安装 qtumjs

```
npm install qtumjs
```

QtumJS是一个用于在Qtum区块链上开发DApp的JavaScript库。您可以使用此库来开发在浏览器中运行的前端UI以及在NodeJS中运行的后端脚本。

主要的类：
类 | 描述
--------- | -----------
QtumRPCRaw | 使用 JSONRPC 1.0调用合约，直接访问 `qtumd` 的区块链 RPC 服务。
QtumRPC | `QtumRPCRaw` 的封装，提供像 JSONRPC 2.0 这样的接口。
Contract | 与智能合约交互的抽象层。使用 [ABI encoding/decoding](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI).

QtumJS 使用 [TypeScript](https://www.typescriptlang.org/) 开发, 因此为所有 API 提供了健壮的类型定义。 我们建议使用 [VSCode](https://code.visualstudio.com/) 来获得语言支持，例如类型提示和自动完成。

当然，你愿意的话也可以选择使用普通的 JavaScript 和记事本。

本文档是 QtumJS API 及其基本用法的参考文档。有关 QtumJS 的教程式介绍，查看: [QtumBook - ERC20 With QtumJS](https://github.com/qtumproject/qtumbook/blob/master/part2/erc20-js.md).


## 运行 Qtum RPC

> 开发模式运行 qtumd：

```
docker run -it --rm \
  --name myapp \
  -v `pwd`:/dapp \
  -p 3889:3889 \
  hayeah/qtumportal
```

> 测试网络（testnet）运行 qtumd：

```
docker run -it --rm \
  --name myapp \
  -e "QTUM_NETWORK=testnet" \
  -v `pwd`:/dapp \
  -p 3889:3889 \
  hayeah/qtumportal
```

QtumJS 依赖 `qtumd` 提供的访问 QTUM 区块链的 JSON-RPC 服务。

更多细节请查看: [QtumBook - Running QTUM](https://github.com/qtumproject/qtumbook/blob/master/SUMMARY.md#part-1---running-qtum).


<aside class="notice">
默认 JSON-RPC 是 “qtum：test”，运行端口为 3889
</aside>

# ERC20 实例

```ts
import {
  Qtum,
} from "qtumjs"

const repoData = require("./solar.json")
const qtum = new Qtum("http://qtum:test@localhost:3889", repoData)

const myToken = qtum.contract("zeppelin-solidity/contracts/token/CappedToken.sol")

async function transfer(fromAddr, toAddr, amount) {
  const tx = await myToken.send("transfer", [toAddr, amount], {
    senderAddress: fromAddr,
  })

  console.log("transfer tx:", tx.txid)
  console.log(tx)

  await tx.confirm(3)
  console.log("transfer confirmed")
}
```

假设 `solar.json` 包含已部署的合约，则可以使用 qtumjs 调用代币合约的方法来流通代币。

一个实例 [solar.json](https://github.com/qtumproject/qtumbook-mytoken-qtumjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example). 这个可以使用 [solar](https://github.com/qtumproject/solar) 部署工具自动生成。

完整实例: [qtumproject/qtumbook-mytoken-qtumjs-cli](https://github.com/qtumproject/qtumbook-mytoken-qtumjs-cli)

合约开发, 查看 [Solar Smart Contract Deployment Tool](https://github.com/qtumproject/solar).

为了充实教程, 查看 [QtumBook - ERC20 With QtumJS](https://github.com/qtumproject/qtumbook/blob/master/part2/erc20-js.md).

# Qtum

```ts
const repoData = require("./solar.json")
const qtum = new Qtum("http://qtum:test@localhost:3889", repoData)
```

`Qtum` 是 `qtumjs` API 的一个对象. 它提供两个主要功能：

+ 对 `qtumd` RPC 服务的访问. 它是 [QtumRPC](#qtumrpc) 的子类.
+ 实例化 [Contract](#contract-2) 对象的工厂方法, 用于与已部署的合约进行交互。

参数 | 类型
--------- | -----------
url | string
 | qtumd RPC 服务的 URL
repoData | [IContractsRepoData](#icontractsrepodata)
 | 关于 Solidity 合约的信息.

`repoData` 包含所有已部署合约或库的 ABI 定义，以及它们的部署地址。这些信息用于实例化 `Contract` 实例。


使用 `Qtum` 的工厂方法实例化的 `Contract` 对象能够解码所有在 `repoData` 里的事件类型. 但是手动构建的合约只能解码在其范围内定义的事件类型, 这也是 Solidity 编译器输出 ABI 定义的限制。

建议使用 Qtum 来实例化 `Contract` 对象.

## contract

```ts
const myToken = qtum.contract("zeppelin-solidity/contracts/token/CappedToken.sol")
```

> 实例化这个合约使用了 [这些](https://github.com/qtumproject/qtumbook-mytoken-qtumjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example#L3) 信息。

实例化 `Contract` 对象的工厂方法，使用了 `repoData` 中的 ABI 定义和地址。合约对象是使用一个事件 log 解码器来配置的，这个解码器能解码所有 `repoData` 中已知的事件类型。

参数 | 类型
--------- | -----------
name | string
  | 作为 `repoData.contracts` map 的 key，用于获取合约信息。

## rawCall

继承自 [QtumRPC#rawcall](#rawcall-2)

# Contract

与智能合约交互的抽象层。

这比使用 `QtumRPC` 直接调用 RPC 的 `sendcontract` 和 `calltocontract` 方法更方便。它处理 ABI 编码，转换 JS 和 Solidity 值。

* 有 API 用于确认交易。
* 有 API 用于调用合约方法，使用 `call` 或 `send` .
* 有 API 用于获取合约 log 事件。

## 构造函数

```js
const rpc = new QtumRPC("http://qtum:test@localhost:3889")

const myToken = new Contract(rpc, repo.contracts[
  "zeppelin-solidity/contracts/token/CappedToken.sol"
])
```

> 合约 [信息](https://github.com/qtumproject/qtumbook-mytoken-qtumjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example#L3) 可以使用 [solar](https://github.com/qtumproject/solar). 生成

参数 | 类型 | 描述
--------- | ----------- | -----------
rpc | QtumRPC | RPC 对象，用于与合约进行交互
info | [IContractInfo](#icontractinfo) | 信息，用于部署合约

建议使用 [Qtum#contract](#contract) 而不是这个构造器。

## call

```js
async function totalSupply() {
  const result = await myToken.call("totalSupply")

  // supply is a BigNumber instance (see: bn.js)
  const supply = result.outputs[0]

  console.log("supply", supply.toNumber())
}
```

> 实例输出:

```js
{ address: 'a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3',
  executionResult:
   { gasUsed: 21689,
     excepted: 'None',
     newAddress: 'a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3',
     output: '00000000000000000000000000000000000000000000000000000000000036b0',
     codeDeposit: 0,
     gasRefunded: 0,
     depositSize: 0,
     gasForDeposit: 0 },
  transactionReceipt:
   { stateRoot: '5a0d9cd5df18165c75755f4345ca81da94f9247c1c031171fd6e2ce1a368844c',
     gasUsed: 21689,
     bloom: '0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000',
     log: [] },
  outputs: [ <BN: 36b0> ] }
```

> 模拟 "mint" 调用:

```ts
const result = await myToken.call("mint", ["dcd32b87270aeb980333213da2549c9907e09e94", 1000])
```

> 运行结果:

```json
{
  "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "executionResult": {
    "gasUsed": 39306,
    "excepted": "None",
    "newAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
    "output": "0000000000000000000000000000000000000000000000000000000000000001",
    "codeDeposit": 0,
    "gasRefunded": 0,
    "depositSize": 0,
    "gasForDeposit": 0
  },
  "transactionReceipt": {
    "stateRoot": "9922edb770bd700a212427d3bc0764a9fed953a987952b2619b8a78dac7498aa",
    "gasUsed": 39306,
    "bloom": "00000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000020000000000008000000000000000000000000000000000000000000000000020000000020000000000800000000000000400000000010000000000000000000000000000000000000000000000000000000000000000000000000000080000000080000000000000000000000000000000000000000000000000000000002010000000000000000000000000000000200000000000000000020000000000000000000000000000000000000000000000000020000000000000000",
    "log": [
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      },
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
          "0000000000000000000000000000000000000000000000000000000000000000",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      }
    ]
  },
  "outputs": [
    true
  ],
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "3e8"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "3e8"
    }
  ]
}
```

使用 `callcontract` 在你本地 qtumd 节点 “模拟” 执行合约方法。这是免费的，实际上并不修改区块链。

这个免费。

参数 | 类型
--------- | -----------
method | string
  | 合约方法名
args | Array\<any>
  | 调用方法的参数
opts | IContractCallRequestOptions
  | 调用配置项
@return | Promise\<[IContractCallResult](#icontractcallresult)>
  | 调用结果，带有 ABI 解码的输出

## send

```ts
async function mint(toAddr, amount) {
  // Submit a `sendtocontract` transaction, invoking the `mint` method.
  const tx = await myToken.send("mint", [toAddr, amount])

  console.log("tx:", tx)

  // Wait for 3 confirmations. The callback receives the
  // updated transaction info for each additional confirmation.
  //
  // Both arguments are optional. `await tx.confirm()` would do.
  const receipt = await tx.confirm(3, (updatedTx) => {
    console.log("new confirmation", updatedTx.txid, updatedTx.confirmations)
  })
  console.log("tx receipt:", JSON.stringify(receipt, null, 2))
}
```

> 实例输出:

```
mint tx: 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9
{ amount: 0,
  fee: -0.081064,
  confirmations: 0,
  trusted: true,
  txid: '858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9',
  walletconflicts: [],
  time: 1515475961,
  timereceived: 1515475961,
  'bip125-replaceable': 'no',
  details:
   [ { account: '',
       category: 'send',
       amount: 0,
       vout: 0,
       fee: -0.081064,
       abandoned: false } ],
  hex: '0200000001006a977de70014fdc2546ed19a531326086c6c9631cb1c5352db5f09e147736b0100000049483045022100b4ca32770a9f42679c6d20b7ddb5feb160303fceafc2db0fedba18a22f0b643602203c2568eb689fd324e76a12f367552fe4cce36b29f8174738209f881959aadbab01feffffff02000000000000000063010403400d0301284440c10f19000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94000000
00000000000000000000000000000000000000000000000000000003e814a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3c2601e72902e0000001976a914dcd32b87270aeb980333213da2549c9907e09e9488ac212e0000',
  method: 'mint',
  confirm: [Function: confirm] }
```

> 回调打印 3 次，分别对应每次确认

```
new confirmation 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9 1
new confirmation 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9 2
new confirmation 858347704258506012f538b19b9702d636dc350bc25a7e60d404bf3d2c08efd9 3
```

> 确认后返回的交易收据：

```json

{
  "blockHash": "3b53ad132c26f9c30e5be9f664573428dad8b52e167becea4428d6903cb32740",
  "blockNumber": 13917,
  "transactionHash": "79338589bb75e1865be889142890a4e25d3b9dbd454ce3f3c2614587c85e2ed3",
  "transactionIndex": 1,
  "from": "dcd32b87270aeb980333213da2549c9907e09e94",
  "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "cumulativeGasUsed": 39306,
  "gasUsed": 39306,
  "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "7d0"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "7d0"
    }
  ],
  "rawlogs": [
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    },
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    }
  ]
}
```

创建一个交易，在网络全局执行合约方法，会改变区块链。

这要花费 gas.

对一个合约有 2 个异步步骤

1. 你提交交易到网络
2. 一旦提交，等待一个指定的确认数

成功确认后，返回带有ABI解码的事件日志的交易收据 ([IContractSendReceipt](#icontractsendreceipt))

参数 | 类型
--------- | -----------
method | string
  | 合约方法名
args | Array\<any>
  | 所调用方法的参数
opts | [IContractSendRequestOptions](#icontractsendrequestoptions)
  | *可选* 发送配置项
@return | Promise\<[IContractSendResult](#icontractsendresult)>
  | 调用结果, 带有 ABI 解码的输出

## 方法重载

如果没有歧义，使用方法名称来调用/发送方法。 如果相同方法名称具有多个定义，请使用方法签名来调用/发送方法。

> 方法名 foo 可能有多个定义:

```ts
function foo();
function foo(int256 _a);
function foo(uint256 _a, uint256 _b);
function foo(int256 _a, int256 _b);
```

> `foo` 方法有 0 个参数和有 1 个参数没有歧义。可以直接调用。

```ts
contract.call("foo")
contract.call("foo", [1])
```

> `foo` 方法带 2 个参数的有歧义，必须带完整方法签名：

```ts
contract.call("foo(uint256,uint256)", [1, 2])
contract.call("foo(int256,int256)", [1, 2])
```

## logs

```js
async function getLogs(fromBlock=0, toBlock="latest") {
  const logs = await myToken.logs({
    fromBlock,
    toBlock,
    minconf: 1,
  })

  console.log(JSON.stringify(logs, null, 2))
}
```

> 实例输出

```js
{
  "entries": [
    {
      "blockHash": "369c6ded05c27ae7efc97964cce083b0ea9b8b950e67c51e52cb1bf898b9c415",
      "blockNumber": 12184,
      "transactionHash": "d1638a53f38fd68c5763e2eef9d86b9fc6ee7ea3f018dae7b1e385b4a9a78bc7",
      "transactionIndex": 2,
      "from": "dcd32b87270aeb980333213da2549c9907e09e94",
      "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "cumulativeGasUsed": 39306,
      "gasUsed": 39306,
      "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000003e8",
      "event": {
        "type": "Mint",
        "to": "dcd32b87270aeb980333213da2549c9907e09e94",
        "amount": "3e8"
      }
    },
    {
      "blockHash": "369c6ded05c27ae7efc97964cce083b0ea9b8b950e67c51e52cb1bf898b9c415",
      "blockNumber": 12184,
      "transactionHash": "d1638a53f38fd68c5763e2eef9d86b9fc6ee7ea3f018dae7b1e385b4a9a78bc7",
      "transactionIndex": 2,
      "from": "dcd32b87270aeb980333213da2549c9907e09e94",
      "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "cumulativeGasUsed": 39306,
      "gasUsed": 39306,
      "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000003e8",
      "event": {
        "type": "Transfer",
        "from": "0000000000000000000000000000000000000000",
        "to": "dcd32b87270aeb980333213da2549c9907e09e94",
        "value": "3e8"
      }
    }
  ],
  "count": 2,
  "nextblock": 12185
}
```

获取由合约生成的 [Solidity 事件日志](http://solidity.readthedocs.io/en/develop/abi-spec.html#events) 。

通过指定 `fromBlock` 和 `toBlock`，可以将事件日志查询限制块号范围。 例如，可以查询块 1000 到 1500 之间的事件日志。

此外，你可以使用 `minconf` 指定事件日志之前确认的最小数量作为结果返回。


参数 | 类型
--------- | -----------
opts | [IRPCWaitForLogsRequest](#irpcwaitforlogsrequest)
  | 事件日志查询参数
@return | Promise\<[IContractEventLogs](#icontracteventlogs)>
  | 日志查询结果，带有 ABI 解码的输出

## onLogs

```js
myToken.onLog((entry) => {
    console.log(entry)
}, { minconf: 1 })
```

订阅合约新事件。每次收到新事件时都会调用回调。默认情况下，`onLog` 监听来自区块链顶端的日志。 使用 `fromBlock` 也可以接收较早的事件。


参数 | 类型
--------- | -----------
callback | (entry: [IContractEventLog](#icontracteventlog)) => void
opts | [IRPCWaitForLogsRequest](#irpcwaitforlogsrequest)
  | 事件日志查询参数

## logEmitter

```js
this.emitter = myToken.logEmitter({ minconf: 1 })

this.emitter.on("Mint", (event) => {
  // ...
})

this.emitter.on("Transfer", (event) => {
  // ...
})

this.emitter.on("?", (event) => {
  // all un-decodeable events
})
```

使用 [EventsEmitter](https://github.com/primus/eventemitter3) 接口订阅合约新事件。发出的事件是 [IContractEventLog](#icontracteventlog) 对象。

Solidity 事件名作为发出的事件名使用。

缺失 ABI 定义的事件 (即不能解析) 会发送 "?".

参数 | 类型
--------- | -----------
opts | [IRPCWaitForLogsRequest](#irpcwaitforlogsrequest)
  | 事件日志查询参数

## receipt

```ts
const txid = "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97"
const receipt = await qrcToken.receipt(txid)
console.log(JSON.stringify(receipt, null, 2))
```

> 实例输出

```js
{
  "blockHash": "af37cb8d9905521542243005fadc9f18c1498c9823e35fa277ea1c37174c289a",
  "blockNumber": 83981,
  "transactionHash": "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97",
  "transactionIndex": 28,
  "from": "57142e3bcf000f28890b5d979afc7ea90204e1de",
  "to": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
  "cumulativeGasUsed": 37029,
  "gasUsed": 37029,
  "contractAddress": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
  "logs": [
    {
      "type": "Transfer",
      "from": "57142e3bcf000f28890b5d979afc7ea90204e1de",
      "to": "c0ed80283c53c300c31c2bda6eca841e53cb6a21",
      "value": "1ba5add5700"
    }
  ],
  "rawlogs": [
    {
      "address": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "00000000000000000000000057142e3bcf000f28890b5d979afc7ea90204e1de",
        "000000000000000000000000c0ed80283c53c300c31c2bda6eca841e53cb6a21"
      ],
      "data": "000000000000000000000000000000000000000000000000000001ba5add5700"
    }
  ]
}
```

获取已被网络接受的交易收据。 如果交易尚未确认，则返回空值。

交易事件日志是 ABI 编码

参数 | 类型
--------- | -----------
txid | string
  | 交易 ID
@return | Promise\<[IContractSendReceipt](#icontractsendreceipt)>
  | 交易收据, 带有事件日志.

# QtumRPC

```ts
const rpc = new QtumRPC('http://qtum:test@localhost:3889');
```

这是一个用于直接访问 `qtumd` RPC API 的 JSON-RPC 客户端。它不会为你处理任何 ABI 编码或解码。

有需要的话你可以把 RPC 用户名和密码包含到 URL 里。在例子中，用户名是 `qtum` ，密码是 `test`.

QtumRPC类有一些在合约抽象内部使用的未公开的方法。 考虑将来可能会发生变化的任何未经证实的不受支持的内容。 现在，rawCall是唯一的公共API。
注意: `QtumRPC` 类有一些没文档的 public 方法在 `Contract` 抽象层内部使用到了. 你要考虑到之后可能不支持的无文档的内容. 现在 `rawCall` 是唯一发布的 API.

参数 | 类型
--------- | -----------
url | string
    | qtumd RPC 服务的 URL

## rawCall

> 调用 `getinfo` RPC 方法以获取 Qutm 区块链的基本信息：

```ts
const info = await rpc.rawCall("getinfo")
console.log(info)
```

> `getinfo` 的输出:

```js
{ version: 141300,
  protocolversion: 70016,
  walletversion: 130000,
  balance: 0,
  stake: 0,
  blocks: 85685,
  timeoffset: 0,
  connections: 8,
  proxy: '',
  difficulty:
   { 'proof-of-work': 0.0000152587890625,
     'proof-of-stake': 5207642.8878753 },
  testnet: false,
  moneysupply: 100322740,
  keypoololdest: 1513325658,
  keypoolsize: 100,
  paytxfee: 0,
  relayfee: 0.004,
  errors: '' }
```

发起一个 JSON-RPC 1.0 方法调用, 返回调用结果. 如果 JSON API 返回不是 200 HTTP 结果，则抛出错误。

> 使用 `try...catch` 处理错误:

```ts
async function main() {
  try {
    const result = await rpc.rawCall("unknown-method-hohoho")
  } catch (err) {
    console.log("err", err)
  }
}
```

## All RPC 方法

qtumd 支持的所有 RPC 方法 .

```
== Blockchain ==
callcontract "address" "data" ( address )
getaccountinfo "address"
getbestblockhash
getblock "blockhash" ( verbose )
getblockchaininfo
getblockcount
getblockhash height
getblockheader "hash" ( verbose )
getchaintips
getdifficulty
getmempoolancestors txid (verbose)
getmempooldescendants txid (verbose)
getmempoolentry txid
getmempoolinfo
getrawmempool ( verbose )
getstorage "address"
gettransactionreceipt "hash"
gettxout "txid" n ( include_mempool )
gettxoutproof ["txid",...] ( blockhash )
gettxoutsetinfo
listcontracts (start maxDisplay)
preciousblock "blockhash"
pruneblockchain
searchlogs <fromBlock> <toBlock> (address) (topics)
verifychain ( checklevel nblocks )
verifytxoutproof "proof"
waitforlogs (fromBlock) (toBlock) (filter) (minconf)

== Control ==
getinfo
getmemoryinfo
help ( "command" )
stop

== Generating ==
generate nblocks ( maxtries )
generatetoaddress nblocks address (maxtries)

== Mining ==
getblocktemplate ( TemplateRequest )
getmininginfo
getnetworkhashps ( nblocks height )
getstakinginfo
getsubsidy [nTarget]
prioritisetransaction <txid> <priority delta> <fee delta>
submitblock "hexdata" ( "jsonparametersobject" )

== Network ==
addnode "node" "add|remove|onetry"
clearbanned
disconnectnode "node"
getaddednodeinfo ( "node" )
getconnectioncount
getnettotals
getnetworkinfo
getpeerinfo
listbanned
ping
setban "subnet" "add|remove" (bantime) (absolute)
setnetworkactive true|false

== Rawtransactions ==
createrawtransaction [{"txid":"id","vout":n},...] {"address":amount,"data":"hex",...} ( locktime )
decoderawtransaction "hexstring"
decodescript "hexstring"
fromhexaddress "hexaddress"
fundrawtransaction "hexstring" ( options )
gethexaddress "address"
getrawtransaction "txid" ( verbose )
sendrawtransaction "hexstring" ( allowhighfees )
signrawtransaction "hexstring" ( [{"txid":"id","vout":n,"scriptPubKey":"hex","redeemScript":"hex"},...] ["privatekey1",...] sighashtype )

== Util ==
createmultisig nrequired ["key",...]
estimatefee nblocks
estimatepriority nblocks
estimatesmartfee nblocks
estimatesmartpriority nblocks
signmessagewithprivkey "privkey" "message"
validateaddress "address"
verifymessage "address" "signature" "message"

== Wallet ==
abandontransaction "txid"
addmultisigaddress nrequired ["key",...] ( "account" )
addwitnessaddress "address"
backupwallet "destination"
bumpfee "txid" ( options )
createcontract "bytecode" (gaslimit gasprice "senderaddress" broadcast)
dumpprivkey "address"
dumpwallet "filename"
encryptwallet "passphrase"
getaccount "address"
getaccountaddress "account"
getaddressesbyaccount "account"
getbalance ( "account" minconf include_watchonly )
getnewaddress ( "account" )
getrawchangeaddress
getreceivedbyaccount "account" ( minconf )
getreceivedbyaddress "address" ( minconf )
gettransaction "txid" ( include_watchonly ) (waitconf)
getunconfirmedbalance
getwalletinfo
importaddress "address" ( "label" rescan p2sh )
importmulti "requests" "options"
importprivkey "qtum" ( "label" ) ( rescan )
importprunedfunds
importpubkey "pubkey" ( "label" rescan )
importwallet "filename"
keypoolrefill ( newsize )
listaccounts ( minconf include_watchonly)
listaddressgroupings
listlockunspent
listreceivedbyaccount ( minconf include_empty include_watchonly)
listreceivedbyaddress ( minconf include_empty include_watchonly)
listsinceblock ( "blockhash" target_confirmations include_watchonly)
listtransactions ( "account" count skip include_watchonly)
listunspent ( minconf maxconf  ["addresses",...] [include_unsafe] )
lockunspent unlock ([{"txid":"txid","vout":n},...])
move "fromaccount" "toaccount" amount ( minconf "comment" )
removeprunedfunds "txid"
reservebalance [<reserve> [amount]]
sendfrom "fromaccount" "toaddress" amount ( minconf "comment" "comment_to" )
sendmany "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] )
sendmanywithdupes "fromaccount" {"address":amount,...} ( minconf "comment" ["address",...] )
sendtoaddress "address" amount ( "comment" "comment_to" subtractfeefromamount )
sendtocontract "contractaddress" "data" (amount gaslimit gasprice senderaddress broadcast)
setaccount "address" "account"
settxfee amount
signmessage "address" "message"
```

## 实例: getblockcount

返回最长的区块链的块数。

```ts
const result = await rpc.rawCall("getblockcount")
```

> 结果

```
85687
```

## 实例: getnewaddress

返回接收付款的新 Qtum 地址。可能对要为用户生成存款地址的交易所有用。

```ts
const result = await rpc.rawCall("getnewaddress")
```

> 结果

```
QSnrDTj4UNcRwKdhY8sUZEd74VzwqeAddW
```

## 实例: fromhexaddress

把一个 base58 pubkeyhash 地址转化成 16 进制地址用于智能合约。

```ts
const result = await rpc.rawCall("gethexaddress", ["QSnrDTj4UNcRwKdhY8sUZEd74VzwqeAddW"])
```

> 结果

```
43debdac95a0eaa4ff92d6b873944a4d92beae59
```

## 实例: gettransactionreceipt

获得确认交易的收据。

```ts
const txid = "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97"
const result = await rpc.rawCall("gettransactionreceipt", [txid])
```

> 结果

```js
[
  {
    "blockHash": "af37cb8d9905521542243005fadc9f18c1498c9823e35fa277ea1c37174c289a",
    "blockNumber": 83981,
    "transactionHash": "62fecfd27d71ddb260ac48c73c8f0f87e96d0b3a598ed2c2251caa4e6f9a9d97",
    "transactionIndex": 28,
    "from": "57142e3bcf000f28890b5d979afc7ea90204e1de",
    "to": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
    "cumulativeGasUsed": 37029,
    "gasUsed": 37029,
    "contractAddress": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
    "log": [
      {
        "address": "49665919e437a4bedb92faa45ed33ebb5a33ee63",
        "topics": [
          "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
          "00000000000000000000000057142e3bcf000f28890b5d979afc7ea90204e1de",
          "000000000000000000000000c0ed80283c53c300c31c2bda6eca841e53cb6a21"
        ],
        "data": "000000000000000000000000000000000000000000000000000001ba5add5700"
      }
    ]
  }
]
```

# 类型词典

## IContractInfo

```ts
export interface IContractInfo {
  /**
   * 合约的 ABI 定义, solc 生成.
   */
  abi: IABIMethod[]

  /**
   * 合约地址
   */
  address: string

  /**
   * 合约所有者的地址
   */
  sender?: string
}
```

与部署合约交互所需的最少部署信息。

## IContractCallResult

调用一个合约方法的返回结果，带有解码的输出和日志。

```ts
export interface IContractCallResult extends IRPCCallContractResult {
  /**
   * ABI 解码的输出
   */
  outputs: any[]

  /**
   * ABI 解码的日志
   */
  logs: Array<IDecodedSolidityEvent | null>
}

export interface IRPCCallContractResult {
  address: string
  executionResult: IExecutionResult,
  transactionReceipt: {
    stateRoot: string,
    gasUsed: string,
    bloom: string,
    log: any[],
  }
}

export interface IExecutionResult {
  gasUsed: number,
  excepted: string,
  newAddress: string,
  output: string,
  codeDeposit: number,
  gasRefunded: number,
  depositSize: number,
  gasForDeposit: number,
}
```

> 实例:

```js
{
  "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "executionResult": {
    "gasUsed": 39306,
    "excepted": "None",
    "newAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
    "output": "0000000000000000000000000000000000000000000000000000000000000001",
    "codeDeposit": 0,
    "gasRefunded": 0,
    "depositSize": 0,
    "gasForDeposit": 0
  },
  "transactionReceipt": {
    "stateRoot": "9922edb770bd700a212427d3bc0764a9fed953a987952b2619b8a78dac7498aa",
    "gasUsed": 39306,
    "bloom": "00000000000000000000000000000000000000000000000000000000000000000000000000000000004000000000000000000000000000000000000000000000000000000020000000000008000000000000000000000000000000000000000000000000020000000020000000000800000000000000400000000010000000000000000000000000000000000000000000000000000000000000000000000000000080000000080000000000000000000000000000000000000000000000000000000002010000000000000000000000000000000200000000000000000020000000000000000000000000000000000000000000000000020000000000000000",
    "log": [
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      },
      {
        "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
        "topics": [
          "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
          "0000000000000000000000000000000000000000000000000000000000000000",
          "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
        ],
        "data": "00000000000000000000000000000000000000000000000000000000000003e8"
      }
    ]
  },
  "outputs": [
    true
  ],
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "3e8"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "3e8"
    }
  ]
}
```

`Contract#call` 的返回类型.

## IContractSendRequestOptions

[Contract#send](#send) 的配置项

```ts
/**
 * `send` 合约方法的配置项.
 */
export interface IContractSendRequestOptions {
  /**
   * 要发送的 QTUM 数. 例如 0.1, 默认: 0
   */
  amount?: number | string

  /**
   * gasLimit, 默认: 200000, 最大: 40000000
   */
  gasLimit?: number

  /**
   * 每 gas 的 Qtum 价格, 默认: 0.00000001, 最小:0.00000001
   */
  gasPrice?: number | string

  /**
   * 发送者的 quantum 地址
   */
  senderAddress?: string
}
```

## IContractSendResult

```ts
const tx = await contract.send(method, args)
await tx.confirm(3, (updatedTx, receipt) => {
  /// ...
})
```

返回 [Contract#send](#send) 的值。

`confirm` 方法用来等待交易确认。

`confirm` 方法的参数:

参数 | 类型
--------- | -----------
n | number
  | *可选* 须等待的确认数
callback | IContractSendConfirmationHandler
  | *可选* 回调函数，每次确认都会调用


回调值为:

参数 | 类型
--------- | -----------
updatedTx | IRPCGetTransactionResult
  | 关于提交给网络的交易的基本信息
receipt | IContractSendReceipt
  | 关于已确认交易的其他信息

### 参考

+ [IRPCGetTransactionResult](#irpcgettransactionresult)
+ [IContractSendReceipt](#icontractsendreceipt)

## IRPCGetTransactionResult

```ts
export interface IRPCGetTransactionResult {
  amount: number,
  fee: number,
  confirmations: number,
  blockhash: string,
  blockindex: number,
  blocktime: number,
  txid: string,
  walletconflicts: any[],
  time: number,
  timereceived: number,
  "bip125-replaceable": "no" | "yes" | "unknown",
  details: any[]
  hex: string,
}
```

关于提交给网络的交易的基本信息。

## IContractSendReceipt

[Contract#send](#send) 的合约收据, 带有解码的事件日志

```ts
export interface IContractSendReceipt extends IRPCGetTransactionReceiptBase {
  /**
   * 使用 ABI 解码的日志
   */
  logs: IDecodedLog[],

  /**
   * 未解码的日志
   */
  rawlogs: ITransactionLog[],
}

/**
 * 解码的 Solidity 事件日志
 */
export interface IDecodedLog {
  /**
   * 事件日志名称
   */
  type: string

  /**
   * 键值映射作为事件日志参数
   */
  [key: string]: any
}
```

> 实例

```json
{
  "blockHash": "3b53ad132c26f9c30e5be9f664573428dad8b52e167becea4428d6903cb32740",
  "blockNumber": 13917,
  "transactionHash": "79338589bb75e1865be889142890a4e25d3b9dbd454ce3f3c2614587c85e2ed3",
  "transactionIndex": 1,
  "from": "dcd32b87270aeb980333213da2549c9907e09e94",
  "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "cumulativeGasUsed": 39306,
  "gasUsed": 39306,
  "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "logs": [
    {
      "type": "Mint",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "amount": "7d0"
    },
    {
      "type": "Transfer",
      "from": "0000000000000000000000000000000000000000",
      "to": "dcd32b87270aeb980333213da2549c9907e09e94",
      "value": "7d0"
    }
  ],
  "rawlogs": [
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    },
    {
      "address": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
      "topics": [
        "ddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef",
        "0000000000000000000000000000000000000000000000000000000000000000",
        "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
      ],
      "data": "00000000000000000000000000000000000000000000000000000000000007d0"
    }
  ]
}
```

### 参考

* [IRPCGetTransactionReceiptBase](#irpcgettransactionreceiptbase)

## IRPCWaitForLogsRequest

```ts
export interface IRPCWaitForLogsRequest {
  /**
   * 查找日志的开始块号。
   */
  fromBlock?: number | "latest",

  /**
   * 查找日志的停止块号. 如果是 null, 会无限期等待
   */
  toBlock?: number | "latest",

  /**
   * 过滤日志的条件。 地址和主题分别指定为十六进制字符串数组
   */
  filter?: ILogFilter,

  /**
   * 日志返回前的最少确认数
   */
  minconf?: number,
}
```

## IContractEventLogs

```ts
/**
 * 查询合约事件日志的结果。
 */
export interface IContractEventLogs {
  /**
   * 事件日志, ABI 解码.
   */
  entries: IContractEventLog[]

  /**
   * 返回的事件日志数
   */
  count: number

  /**
   * 要开始查询新事件日志的块号
   */
  nextblock: number
}
```

查询合约事件日志的结果。

要查询尚未出现的新日志，请在查询事件日志时将 `nextblock` 用作 `startBlock`：
* [IContractEventLog](#icontracteventlog)

## IContractEventLog

一条解码的合约事件日志

```ts
export interface IContractLogEntry extends ILogEntry {
  /**
   * Solidity 事件, ABI 解码. 如果没有找到 ABI 定义，为 Null
   */
  event?: ISolidityEvent
}

/**
 *  qtumd 返回的原始日志数据，不是 ABI 解码
 */
export interface ILogEntry extends IRPCGetTransactionReceiptBase {
  /**
   * EVM 日志主题
   */
  topics: string[]

  /**
   * EVM 日志数据, 十六进制字符串
   */
  data: string
}

/**
 * qtumd 返回的交易收据
 */
export interface IRPCGetTransactionReceiptBase {
  blockHash: string
  blockNumber: number

  transactionHash: string
  transactionIndex: number

  from: string
  to: string

  cumulativeGasUsed: number
  gasUsed: number

  contractAddress: string
}
```

> 实例

```js
{
  "blockHash": "369c6ded05c27ae7efc97964cce083b0ea9b8b950e67c51e52cb1bf898b9c415",
  "blockNumber": 12184,
  "transactionHash": "d1638a53f38fd68c5763e2eef9d86b9fc6ee7ea3f018dae7b1e385b4a9a78bc7",
  "transactionIndex": 2,
  "from": "dcd32b87270aeb980333213da2549c9907e09e94",
  "to": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "cumulativeGasUsed": 39306,
  "gasUsed": 39306,
  "contractAddress": "a778c05f1d0f70f1133f4bbf78c1a9a7bf84aed3",
  "topics": [
    "0f6798a560793a54c3bcfe86a93cde1e73087d944c0ea20544137d4121396885",
    "000000000000000000000000dcd32b87270aeb980333213da2549c9907e09e94"
  ],
  "data": "00000000000000000000000000000000000000000000000000000000000003e8",
  "event": {
    "type": "Mint",
    "to": "dcd32b87270aeb980333213da2549c9907e09e94",
    "amount": "3e8"
  }
}
```

## IDecodedSolidityEvent

```ts
/**
 * 一个解码的 Solidity 事件日志
 */
export interface IDecodedSolidityEvent {
  /**
   * 事件名称
   */
  type: string

  /**
   * 键值映射作为事件日志参数
   */
  [key: string]: any
}
```

> Example

```js
{
  "type": "Transfer",
  "from": "0000000000000000000000000000000000000000",
  "to": "dcd32b87270aeb980333213da2549c9907e09e94",
  "value": "3e8"
}
```

解码的 Solidity 事件日志。 事件参数存储在键值映射中。

## IRPCGetTransactionReceiptBase

网络接受的交易收据。它由 `gettransactionreceipt` RPC 调用返回。

```ts
export interface IRPCGetTransactionReceiptBase {
  blockHash: string
  blockNumber: number

  transactionHash: string
  transactionIndex: number

  from: string
  to: string

  cumulativeGasUsed: number
  gasUsed: number

  contractAddress: string
}
```

## IContractsRepoData

合约相关信息

```ts
export interface IContractsRepoData {
  /**
   * 部署合约的相关信息
   */
  contracts: {
    [key: string]: IContractInfo,
  },

  /**
   * 部署库的相关信息
   */
  libraries: {
    [key: string]: IContractInfo,
  }

  /**
   * 部署合约/库引用的合约信息，但未部署
   */
  related: {
    [key: string]: {
      abi: IABIMethod[],
    },
  }
}

/**
 * 与部署合约进行交互所需的最少部署信息。
 */
export interface IContractInfo {
  /**
   * 合约的 ABI 定义, solc 生成.
   */
  abi: IABIMethod[]

  /**
   * 合约地址
   */
  address: string

  /**
   * 合约所有者的地址
   */
  sender?: string
}

export interface IABIMethod {
  name: string,
  type: string,
  payable: boolean,
  inputs: IABIInput[],
  outputs: IABIOutput[],
  constant: boolean,
  anonymous: boolean,
}
```

可以使用开发工具 [solar](https://github.com/qtumproject/solar) 自动生成。

样例 [solar.json](https://github.com/qtumproject/qtumbook-mytoken-qtumjs-cli/blob/29fab6dfcca55013c7efa8ee5e91bbc8c40ca55a/solar.development.json.example).
