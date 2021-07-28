# 智能合约

## 1. 简介

本文档介绍墨群区块链系统中的智能合约系统。用户可使用系统提供的工具集在本地进行区块链合约开发、编译和测试，并发布到KaaS集群中。

### 1.1 智能合约

智能合约是用户和区块链系统进行交互的主要方式。用户通过发布智能合约定义一套逻辑规则。通过调用智能合约中定义的接口，用户可以与该合约进行交互，执行特定的任务或者查询其中保存的数据。区块链的不可篡改性保证这套合约逻辑的正确执行。

### 1.2 墨群区块链系统上的智能合约

墨群区块链中的智能合约系统是一个两层架构，第一层是去中心化应用程序，即DApp（Decentralized Application）。每个DApp下包含智能合约。与一些其他区块链系统不同，墨群链中的DApp以名字区分，同个DApp内的合约也以合约名区分。因此，一个合约的完整名字是"DApp名.合约名"形式。在DApp名明确、或者不重要的情况下，有时也直接以"合约名"指代某个特定合约。

墨群链中的智能合约分为两类，第一类是内置合约。这些合约在系统发布时已经包含在内，统一放在内置DApp "core"之下，提供了包括DApp注册、合约发布在内的一些系统基础功能。内置合约不可动态添加、修改或删除。以下列出了一部分内置合约。

```
core.credit
core.coins
core.dapp
core.contracts
core.deployment
```

第二类合约是用户可以编写并部署到链上的合约。墨群链使用自有的PREDA语言来开发智能合约。

### 1.3 PREDA智能合约

PREDA智能合约具有高效性、确定性、安全性。在语法层面上提供了对分片区块链系统的支持。与诸多其他智能合约语言最大的不同在于，PREDA合约在部署后被直接编译成本地二进制代码，因此无需庞大低效的虚拟机进行解释执行。取而代之的是一个小巧精简的PREDA执行引擎，提供交互服务层与区块链系统进行通信，保证合约的高效执行。

### 1.4 简单的PREDA智能合约示例

```
contract MyCoin_TaxablePayment
{
	// Shard-shared state and functions
	shard token tax;
	shard function turn_over(address ld, bigint amount) export {
		if(balance.get_amount() >= amount){
			relay@ld (token deposit = fund){
				 deposit.transfer_all(balance);
			}
		}
	}
	// Per-Address state and functions
	token balance;
	function transfer(address to, bigint amount) export {
		if(balance.get_amount() >= amount)
		{   
			token fund;
			balance.transfer(fund, amount);
			relay@to (token deposit = fund){
				bigint t = balance/bigint(100);  // 1% tax rate
				deposit.transfer(tax, t);
				deposit.transfer_all(balance);
			}
		}
	}
}
```

## 2. 开发环境

墨群区块链智能合约开发环境支持包括Windows, Linux和macOS在内的多个操作系统。目前经过测试的系统有

```
Windows 10
Ubuntu 18.04
macOS Catalina
```

以上系统均只支持64位版本。

下文的开发环境以Windows 10为例，其他平台类似。

### 2.1 环境准备

- **Visual Studio Code**。VS Code是微软公司提供的跨平台代码编辑器。用户可到其主页下载对应平台的安装包。
- **Java SE Development Kit**。JDK是Oracle公司提供的Java开发套件。如果用户无需使用Java调用合约，则可跳过此步骤。
- **墨群智能合约开发套件**。其内容包括单机私链环境、PREDA语言VS Code插件以及用于和KaaS系统通信的Connector工具。

### 2.2 搭建单机私链环境

要开完一个合约并发布到KaaS系统，用户首先需要一个测试系统来开发、调试和测试合约。墨群智能合约开发套件中提供了称作Playground的本地私链系统。该系统以本地单节点的形式提供了墨群链正式系统的绝大多数功能，并在此之上提供了一些用于帮助开发者的辅助功能。启动开发套件中的Playground即可启动此系统。

> **注意！**Playground环境每次运行都会启动一条全新的链。所有部署的合约和提交的交易在程序关闭时会被丢弃。

### 2.3 安装PREDA语言VS Code插件

开发包中包含了一个用于VS Code的PREDA插件。安装完后提供以下功能

- PREDA源代码语法高亮
- 编译和部署合约到本地Playground私链
- 用于测试和调试的开发工具Preda Dev Tool
- 通过Connector发布合约到KaaS系统

### 2.4 配置VS Code插件

PREDA语言VS Code插件提供了若干配置选项。其中与Playground相关的如下：

- **PG Dapp Name:** 在本地Playground部署合约时用的DApp名称
- **PG Supplied Assets:** 在本地Playground部署合约时用的资产名称
- **PG URL:** 本地Playground服务地址。默认值为http://127.0.0.1:9908

此外还有若干与KaaS系统相关的配置：

- **KAAS Cert Path:** KaaS证书在本地的保存位置
- **KAAS Dapp Address:** 在KaaS上发布合约时使用的DApp地址
- **KAAS URL:** KaaS服务器地址

## 3. 合约开发

> **注意！**编译、部署和测试合约需要和Playground通讯，请在开发过程中保持此程序一直在运行状态。

### 3.1 编写合约

如同其他语言一样，用户可在VS Code中直接编写PREDA代码。默认的PREDA源代码文件扩展名为.prd。将源代码保存为此格式时可以获得语法高亮以及智能提示。

要学习PREDA语言，可参考下列文档和教程

- PREDA语言规范（

  [https://github.com/jiapw/oxd_preal/blob/master/spec/preal_v1_cn.md]: https://github.com/jiapw/oxd_preal/blob/master/spec/preal_v1_cn.md

  ）

- PREDA培训文档（

  [https://docs.google.com/presentation/d/1Dh2hH8dh9Zpv935tPA-_RqXyerWfy7ZrJVQJ9Yq-qvQ/edit#slide=id.gddc6b1e607_0_5]: https://docs.google.com/presentation/d/1Dh2hH8dh9Zpv935tPA-_RqXyerWfy7ZrJVQJ9Yq-qvQ/edit#slide=id.gddc6b1e607_0_5

  ）

- CryptoPuppy交互式教程（

  [http://bc.moqun.cn/puppy/index.html]: http://bc.moqun.cn/puppy/index.html

  ）

### 3.2 编译合约

### 3.3 部署合约

## 4. 测试

### 4.1 PREDA Dev Tool简介

### 4.2 使用Dev Tool调用和测试合约

### 4.3 合约的SDK

## 5. 发布

