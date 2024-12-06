
# 背景信息


攻击交易：[https://app.blocksec.com/explorer/tx/eth/0x9a1d02a7cb9fef11fcec2727b1f9e0b01bc6bcf5542f5b656c84d6400a1b4604](https://github.com)
漏洞合约：[https://etherscan.io/address/0x8a30d684b1d3f8f36b36887a3deca0ef2a36a8e3\#code](https://github.com):[FlowerCloud机场订阅官网](https://hanlianfangzhi.com)


`LockedStaking` 合约提供质押功能，用户调用 `stake` 函数质押时会根据质押时长立即计算收益 `yield`，并且记录在用户的收益 `user.yield` 上。等到该笔质押的时间过后，用户可以调用 `unStake` 函数取回本金和收益。


# Trace 分析


攻击者不断用同一笔资金进行 `start` 和 `unStake` 操作，攻击者创建新合约 strat 存入 500000 VSTR，然后通过 0x1f2c 合约 调用 `unStake` 取回 520000 VSTR。


![image](https://img2024.cnblogs.com/blog/1483609/202412/1483609-20241205151543465-1444231247.png)


`start` 操作就是通过新创建的合约进行 `stake`


![image](https://img2024.cnblogs.com/blog/1483609/202412/1483609-20241205151604676-1973613202.png)


# 漏洞分析


问题出在 `LockedStaking` 合约的 `unStake` 函数，`unStake` 函数对质押的状态检查与更新存在问题，导致在用户的质押到期后，可以无限次进行取款。


1. `require` 检查的 `user.stakeAmount` 参数，在 `unstake` 操作后不更新
2. 在 `unstake` 操作后更新的 `user.isActive` 参数，却不检查。


![image](https://img2024.cnblogs.com/blog/1483609/202412/1483609-20241205151732734-589993685.png)


那么攻击者可以无限次 `unstake` 来获取 VSTR，为什么还需要创建新合约进行 `stake` 操作呢？
因为在 `unstake` 的时候会更新 `data.totalStaked -= stakeAmount;` ，如果不创建新合约进行 `stake` 操作增加 `data.totalStaked` 的值，就会发生下溢出。


![image](https://img2024.cnblogs.com/blog/1483609/202412/1483609-20241205151810377-1505524739.png)


![image](https://img2024.cnblogs.com/blog/1483609/202412/1483609-20241205151818351-642290052.png)


