---
author: Blank
date: 2017-10-11 20:10:07
title: 比特币入门四
tags: Bitcoin
category: 区块链
status: publish
summary: 比特币API。[...]
---

### Bitcoin API

交易信息json

<!--more-->

字段说明：

    blockhash   =>  块哈希
    
    blockheight =>  所在区块高度
    
    blocktime    =>  在区块内的时间
    
    confirmations    =>  交易的确认数
    
    fees    =>  交易矿工费
    
    locktime    =>  接收时间
    
    size    =>  交易大小
    
    time    =>  广播时间
    
    txid    =>  交易索引id
    
    valueIn    =>  输入方的总金额
    
    valueOut    =>  输出方的总金额
    
    version    =>  版本

输入方：`vin`，可能有多个。

    addr    =>  输入方的地址
    
    scriptSig    =>  输入脚本
    
    asm    =>  脚本asm
    
    hex    =>  脚本十六进制
    
    sequence    =>  输入方的序列号
    
    txid    =>  每个输入方的前输出的交易索引id（with each inputs prevout txid）
    
    value    =>  输入的金额
    
    vout    =>  前输出的输出索引位置（prevout output index number）

输出方：`vout`，可能有多个。

    scriptPubKey    =>  公钥脚本
    
    addresses    =>  输出方的地址
    
    asm    =>  脚本asm
    
    hex    =>  脚本十六进制
    
    type    =>  公钥哈希
    
    value    =>  输出的金额


