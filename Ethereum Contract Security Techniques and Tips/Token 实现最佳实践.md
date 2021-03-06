# 以太坊最佳实践翻译五

## Token 实现最佳实践

实现 Token 应该符合其他的最佳实践，但是它有一些独特的考虑。

### 遵守最新的标准

通常来说，token 的智能合约应该遵循公认的稳定标准。

当前公认的标准有：

* [EIP20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md)
* [EIP721](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md)（不可替代的 token）

### 注意 EIP-20 的前期攻击

EIP-20 的 token 的 `approve()` 函数可能会让第三方提币量超过设置的预定数量。一个 [前期攻击](https://consensys.github.io/smart-contract-best-practices/known_attacks/#transaction-ordering-dependence-tod-front-running) 可以使得一个经过授权的用户不论是在调用方法 `approve()` 之前还是之后都可以调用 `transferFrom()` 函数。更多细节可见 [EIP](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md#approve) 和 [此文档](https://docs.google.com/document/d/1YLPtQxZu1UAvO9cZ1O2RPXBbT0mooh4DYKjA_jp-RLM/edit)。

### 防止将 token 转到地址 0x0

在写本文的此刻，“零”地址（[0x0000000000000000000000000000000000000000](https://etherscan.io/address/0x0000000000000000000000000000000000000000)）所拥有的 token 的价值超过了 8000,000 美元。

### 防止将 token 转到合约地址

注意不要将 token 转到和智能合约地址相同的地址。

该情况下损失的一个例子就是 EOS token 的智能合约地址中有超过 90000 个 token。

**例子：**

实现上面的这些推荐的一个例子会创建下面这个修饰器，验证 `to` 地址不会是 0x0 也不会是该 token 自己的合约地址：
```
modifier validDestination(address to) {
	require(to != address(0x0));
	require(to != address(this));
	_;
}
```

该修饰器被添加到 `transfer` 和 `transferFrom` 方法：
```
function transfer(address _to, uint _value) validDestination(_to) returns (bool) {
	//some code
}

function transferFrom(address _from, address _to, uint _value) validDestination(_to) returns (bool) {
	//some code
}
```