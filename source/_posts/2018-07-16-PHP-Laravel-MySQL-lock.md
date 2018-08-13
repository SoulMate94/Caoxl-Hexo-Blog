---
title: 数据库 锁机制
date: 2018-07-16 11:45:51
categories: MySQL
tags: [MySQL, 锁]
---

# 使用锁的场景

> 通常在并发场景下，为了防止某个资源被重复创建／更新，在同时存在查询-更新／创建操作的时候

<!-- more -->

# 直接上源码

```
    $amount = 100;
    \DB::beginTransction();
    
    // 锁住该用户对于的行记录, 防止并发中的其他请求修改该用户状态
    $user = \App\Models\User::lockForUpdate()->find(1); # 不可读不可写
    $user = \App\Models\User::sharedLock()->find(1);    # 可读不可写
    
    $log  = \App\Models\BalanceLog::insert([
        'before'    => $user->balance,
        'create_at' => time(),
        'amount'    => $amount,
    ]); 
    
    $user->balance += $amount;
    $user->save();
    \DB::commit();
```

**说明:**

如果不使用 `lockForUpdate()` 或者 `sharedLock()` 来锁住该用户对应的记录，那么在多个相同请求到来时，该用户的余额和日志会被更新／／创建多条，这明显不是我们想要的。

`lockForUpdate()` 创建的锁将在本次事务结束后释放，且如果记录不存在，`lockForUpdate()` 则不会起到什么作用。