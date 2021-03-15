# BNS 协议
## 介绍
BNS（Blockchain Name Service）是区块链域名服务，是一个基于BHP网络的分布式、开放和可扩展的命名系统。

BNS的工作是将可读的域名（比如"amy.abc"）解析为计算机可以识别的标识符，如BHP地址、内容的散列、元数据等。

![BNS时序图](https://user-images.githubusercontent.com/32643286/111112553-5193d500-859b-11eb-9942-7f60b67106e3.png)

- 注册表合约：BNS官方部署的合约，该合约维护所有域名和子域名列表，是BNS的核心组件并存储着BNS域名的关键信息，主要包括域名的拥有权等。
- 解析器合约：存储用户域名信息的合约，可以存储每个域名对应的各个区块链网络的代币地址以及其他信息。
- 注册中心合约：顶级域名的拥有者合约（一般为服务商拥有），可以注册相应的子域名。
- 服务商：注册中心合约的部署和拥有者，可以通过注册中心合约为用户注册域名和配置相关信息，查询解析记录。
- 用户：服务商的用户，可以在服务商端注册账户，配置信息。

## 操作角色

### 管理员操作

1. BNS管理员需要部署BNS注册表合约以及公共的解析器合约
2. 管理员可以调用注册表中的setSubnodeOwner方法给服务商部署的注册中心分配顶级域名（如.abc和.xyz）

### 服务商操作

1. 服务商需部署自己的注册中心合约
2. 服务商需要向BNS管理员申请自己的顶级域名
3. 通过申请后服务商可以为用户调用registerWithConfig注册用户的子域名（如1XXXX.abc）
4. 服务商需要将用户的信息写入解析器合约，通过调用setAddr设置各个区块链网络的代币地址以及setText设置其他信息
5. 服务商可以调用解析器的addr和text解析用户的地址信息和其他信息（其他服务商的用户也可以解析）

### 用户操作

用户只需在服务商系统注册账户，设置信息即可

## 合约方法和参数

### 注册表合约

1. setSubnodeOwner(bytes32 node, bytes32 label, address owner)：注册子域名或者顶级域名

>node：顶级域名的namehash(如abc 注册顶级域名的话此值为0x0000000000000000000000000000000000000000000000000000000000000000)
>
>label：子域名的sha3（如1xxxxxxxxx的sha3）
>
>owner：域名的拥有者

2. owner(bytes32 node)：返回域名的拥有者

> node：域名的namehash（如1xxxxxxx.abc）

3. resolver(bytes32 node)：返回域名设置的解析器

> node：域名的namehash（如1xxxxxxx.abc）

### 注册中心合约

1. constructor(BNS _bns, bytes32 _baseNode)：注册中心的构造函数

> _bns:注册表合约地址
> 
> _baseNode:顶级域名的namehash
2. addController(address controller)：增加controller地址（可以注册子域名的地址）
3. removeController(address controller)：减少controller地址
4. registerWithConfig(bytes32 label, address owner, address resolver, address addr)：为用户注册子域名

> label：子域名的sha3（如1xxxxxxxxx的sha3）
> 
> owner：子域名的拥有者
> 
> resolver：子域名的解析器地址
> 
> addr：子域名在解析器对应的BHP2.0地址

### 解析器合约

1. setAddr(bytes32 node, uint coinType, bytes memory a)：给域名设置地址信息

>node：域名的namehash（如1xxxxxx.abc的namehash）
>
>coinType：对应地址的cointype
>
>a：地址的hex编码

2. setText(bytes32 node, string calldata key, string calldata value)：给域名设置用户信息，使用键值对的方式

3. addr(bytes32 node, uint coinType)：获取域名各个链的地址

4. text(bytes32 node, string key)：获取域名的用户信息

5. multicall(bytes[] calldata data)：将各个方法的字节码打包在一起执行（如将setAddr和setText的方法打包在一起执行）

## 名词解释

1. namehash：bns使用的hash加密

2. sha3：web3的工具包的sha3加密

3. coinType：大部分币都有自己的coinType，具体可参考：https://github.com/satoshilabs/slips/blob/master/slip-0044.md

4. hex编码：bns根据coinType实现的编码，解码
