### 构建状态和代码覆盖率徽章
- [![Build Status](https://travis-ci.org/nibbstack/erc721.svg?branch=master)](https://travis-ci.org/nibbstack/erc721) &nbsp;[![codecov](https://codecov.io/gh/nibbstack/erc721/branch/master/graph/badge.svg?token=F0tgRHyWSM)](https://codecov.io/gh/nibbstack/erc721) &nbsp;[![NPM Version](https://badge.fury.io/js/@0xcert%2Fethereum-erc721.svg)](https://www.npmjs.com/package/@nibbstack/erc721) &nbsp;[![Bug Bounty](https://img.shields.io/badge/bounty-open-2930e8.svg)](https://github.com/nibbstack/erc721/blob/master/BUG_BOUNTY.md)

### ERC-721 代币 — 参考实现

**注意：此仓库已从 0xcert 转移到 Nibbstack，因此从 0xcert/ethereum-erc721 重命名为 @nibbstack/erc721。**

这是 [ERC-721](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md) 非同质化代币标准的完整参考实现，适用于以太坊和Wanchain区块链。它还兼容其他EVM兼容的链，如币安智能链(BSC)、雪崩(AVAX)等。这是一个开源项目，包含[Hardhat](https://hardhat.org/)测试。

此实现的目的是为任何希望在以太坊和Wanchain区块链上使用和发展非同质化代币的人提供一个良好的起点。你可以直接使用这些经过多次审计的代码，而不是自己重新实现ERC-721。我们希望未来社区将广泛使用它。

请注意，此实现比ERC-721标准更受限制，因为它不直接支持`payable`函数调用。但你完全可以自己添加这个功能。

如果你正在寻找一个功能更丰富、更高级的ERC-721实现，请查看[0xcert Framework](https://github.com/0xcert/framework)。

### 结构

所有合约和测试都在[src](src/)文件夹中。有多种实现，你可以选择：

- [`nf-token.sol`](src/contracts/tokens/nf-token.sol): 这是基础的ERC-721代币实现（支持ERC-165）。
- [`nf-token-metadata.sol`](src/contracts/tokens/nf-token-metadata.sol): 这实现了代币合约的可选ERC-721元数据特性。它实现了代币名称、符号以及指向公开暴露的ERC-721 JSON元数据文件的明确URI。
- [`nf-token-enumerable.sol`](src/contracts/tokens/nf-token-enumerable.sol): 这实现了可选的ERC-721枚举支持。如果你想知道代币的总供应量，按索引查询代币等，这将非常有用。

在[token](src/contracts/tokens)和[utils](src/contracts/utils)目录中命名为`erc*.sol`的其他文件是接口，定义了各自的标准。

在[mocks](src/contracts/mocks)文件夹中提供了展示基本合约用法的模拟合约。

在[src/tests/mocks](src/tests/mocks)中也有专门的测试模拟，它们被设计用来测试不同的边缘情况和行为，不应用于实现参考。

### 要求

* 支持 NodeJS 12+
* 支持 Windows、Linux 或 macOS

### 安装

#### npm

*如果你想在JavaScript项目中使用此包，这是推荐的安装方法。*

此项目作为一个npm模块[发布](https://www.npmjs.com/package/@nibbstack/erc721)。你必须使用`npm`命令安装它：

```
$ npm install @nibbstack/erc721@2.6.1
```

#### 源代码

*如果你想改进`nibbstack/erc721`项目，这是推荐的安装方法。*

克隆此仓库并安装所需的`npm`依赖项：

```
$ git clone git@github.com:nibbstack/erc721.git
$ cd ethereum-erc721
$ npm install
```

确保一切设置正确：

```
$ npm run test
```

### 使用方法

#### npm

在JavaScript代码中与此包的合约交互，你只需要引入此包的`.json`文件：

```js
const contract = require("@nibbstack/erc721/abi/NFTokenEnumerable.json");
console.log(contract);
```

#### Remix IDE（仅限以太坊）

你可以使用[Remix IDE](https://remix.ethereum.org)快速部署此库的合约。这里是一个例子。

你创造了并拥有独特的手工吹制艺术品（每个都有序列号/批次号），你希望使用以太坊或Wanchain主网进行销售。你将销售非同质化代币，买家将能够与其他的人交易这些代币。每个艺术品对应一个代币。你向任何持有这些代币的人承诺，他们可以赎回他们的代币并取得艺术品的实际拥有权。

为此，只需将以下代码粘贴到Remix中并部署智能合约。你将为每件新的艺术品“铸造”一个代币。当你交出艺术品的实物拥有权时，你将“销毁”该代币。

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "https://github.com/nibbstack/erc721/src/contracts/tokens/nf-token-metadata.sol"; 
import "https://github.com/nibbstack/erc721/src/contracts/ownership/ownable.sol"; 

/**
 * @dev 这是带有元数据扩展的NFToken的示例合约实现。
 */
contract MyArtSale is
  NFTokenMetadata,
  Ownable
{
  /**
   * @dev 合约构造函数。设置元数据扩展的`名称`和`符号`。
   */
  constructor()
  {
    nftName = "Frank's Art Sale";
    nftSymbol = "FAS";
  }

  /**
   * @dev 铸造一个新的NFT。
   * @param _to 将拥有铸造的NFT的地址。
   * @param _tokenId 由msg.sender铸造的NFT的tokenId。
   * @param _uri 表示RFC 3986 URI的字符串。
   */
  function mint(
    address _to,
    uint256 _tokenId,
    string calldata _uri
  )
    external
    onlyOwner
  {
    super._mint(_to, _tokenId);
    super._setTokenUri(_tokenId, _uri);
  }
}
```

*在举行拍卖或销售任何物品之前，你应该咨询律师。特别是，拍卖法规因司法管辖区而异。此应用程序仅作为技术的示例提供，不是法律建议。*

### 验证

你可以使用在线验证器[erc721validator.org](https://erc721validator.org/)检查智能合约的有效性、实现的正确性以及相应智能合约支持的功能。

### 游乐场

#### 以太坊 - Ropsten 测试网

我们已经在[Ropsten](https://ropsten.etherscan.io/)网络上部署了一些合约。你现在就可以和它们玩耍。无需安装软件。在这个测试版本的合约中，任何人都可以`铸造`或`销毁`代币，所以不要将其用于任何重要事项。

| 合约 | 代币地址 | 交易哈希 |
| --- | --- | --- |
| [`nf-token.sol`](src/contracts/tokens/nf-token.sol) | [0xd8bbf8ceb445de814fb47547436b3cfeecadd4ec](https://ropsten.etherscan.io/address/0xd8bbf8ceb445de814fb47547436b3cfeecadd4ec) | [0xaac94c9ce15f5e437bd452eb1847a1d03a923730824743e1f37b471db0f16f0c](https://ropsten.etherscan.io/tx/0xaac94c9ce15f5e437bd452eb1847a1d03a923730824743e1f37b471db0f16f0c) |
| [`nf-token-metadata.sol`](src/contracts/tokens/nf-token-metadata.sol) | [0x5c007a1d8051dfda60b3692008b9e10731b67fde](https://ropsten.etherscan.io/address/0x5c007a1d8051dfda60b3692008b9e10731b67fde) | [0x1e702503aff40ea44aa4d77801464fd90a018b7b9bad670500a6e2b3cc281d3f](https://ropsten.etherscan.io/tx/0x1e702503aff40ea44aa4d77801464fd90a018b7b9bad670500a6e2b3cc281d3f) |
| [`nf-token-enumerable.sol`](src/contracts/tokens/nf-token-enumerable.sol) | [0x130dc43898eb2a52c9d11043a581ce4414487ed0](https://ropsten.etherscan.io/address/0x130dc43898eb2a52c9d11043a581ce4414487ed0) | [0x8df4c9b73d43c2b255a4038eec960ca12dae9ba62709894f0d85dc90d3938280](https://ropsten.etherscan.io/tx/0x8df4c9b73d43c2b255a4038eec960ca12dae9ba62709894f0d85dc90d3938280) |

#### Wanchain - 测试网

我们已经在[testnet](http://testnet.wanscan.org/)网络上部署了一些合约。你现在就可以和它们玩耍。无需安装软件。在这个测试版本的合约中，任何人都可以`铸造`或`销毁`代币，所以不要将其用于任何重要事项。

| 合约 | 代币地址 | 交易哈希 |
| --- | --- | --- |
| [`nf-token.sol`](src/contracts/tokens/nf-token.sol) | [0x6D0eb4304026116b2A7bff3f46E9D2f320df47D9](http://testnet.wanscan.org/address/0x6D0eb4304026116b2A7bff3f46E9D2f320df47D9) | [0x9ba7a172a50fc70433e29cfdc4fba51c37d84c8a6766686a9cfb975125196c3d](http://testnet.wanscan.org/tx/0x9ba7a172a50fc70433e29cfdc4fba51c37d84c8a6766686a9cfb975125196c3d) |
| [`nf-token-metadata.sol`](src/contracts/tokens/nf-token-metadata.sol) | [0xF0a3852BbFC67ba9936E661fE092C93804bf1c81](http://testnet.wanscan.org/address/0xF0a3852BbFC67ba9936E661fE092C93804bf1c81) | [0x338ca779405d39c0e0f403b01679b22603c745828211b5b2ea319affbc3e181b](http://testnet.wanscan.org/tx/0x338ca779405d39c0e0f403b01679b22603c745828211b5b2ea319affbc3e181b) |
| [`nf-token-enumerable.sol`](src/contracts/tokens/nf-token-enumerable.sol) | [0x539d2CcBDc3Fc5D709b9d0f77CaE6a82e2fec1F3](http://testnet.wanscan.org/address/0x539d2CcBDc3Fc5D709b9d0f77CaE6a82e2fec1F3) | [0x755886c9a9a53189550be162410b2ae2de6fc62f6791bf38599a078daf265580](http://testnet.wanscan.org/tx/0x755886c9a9a53189550be162410b2ae2de6fc62f6791bf38599a078daf265580) |

### 贡献

查看[CONTRIBUTING.md](./CONTRIBUTING.md)了解如何提供帮助。

### 漏洞赏金

你是那种阅读智能合约文档并理解ERC-721代币参考实现如何工作的人。所以你拥有独特的技能，你的时间很有价值。我们将以漏洞报告的形式为你对本项目的贡献支付报酬。

如果你的项目依赖于ERC-721，或者你希望帮助提高这个项目的保障性，那么你可以承诺一个赏金。这意味着你将承诺向展示问题的研究人员支付费用。如果感兴趣，请通过[bounty@nibbstack.com](mailto:bounty@nibbstack.com)与我们联系。谢谢。

阅读完整的[漏洞赏金计划](BUG_BOUNTY.md)。

### 许可证

查看[LICENSE](./LICENSE)了解详细信息。