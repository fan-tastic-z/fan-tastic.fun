---
title: "Rust操作Redis"
date: 2024-01-17T16:50:05+08:00
tags: ["Rust", "Redis-rs"]
categories: ["Rust"]
keywords: ["Redis-rs", "Rust操作Redis"]
draft: false
---

在实际项目开发中，Redis 是使用非常多的组件了，在Rust中非常受欢迎的库应该是 <https://github.com/redis-rs/redis-rs>

## redis-rs使用基本使用

在依赖中添加：

```toml
eyre = "0.6.11"
redis = "0.24.0"
```

eyre 和 redis-rs库并没有关系，主要用于下面代码中错误的处理。
下面代码中包含了对redis 的常见的操作，更多的操作可以参考：<https://docs.rs/redis/latest/redis/struct.Cmd.html#>

```rust
use std::collections::{HashMap, HashSet};

use eyre::{OptionExt, Result};
use redis::{Commands, Connection};


fn fetch_an_inter(con: &mut Connection) -> Result<isize> {
    let _ = con.set("fantastic", 18)?;
    let result: isize = con.get("fantastic")?;
    Ok(result)
}

fn get_missing_key(con: &mut Connection) -> Result<String> {
    // 如果key可能不存在，可以通过返回Option处理
    let result: Option<String> = con.get("fan")?;
    result.ok_or_eyre("key not found")
}

fn get_missing_key2(con: &mut Connection) -> Result<i32> {
    // 如果key 可能不存在的情况，可以通过unwrap_or设置默认的值
    let result = con.get("rust").unwrap_or(0i32);
    Ok(result)
}

fn get_list(con: &mut Connection) -> Result<Vec<String>> {
    con.del("language")?;
    let val = ["rust", "go", "vue"];
    let _ = con.rpush("language", &val)?;

    let result = con.lrange("language", 0, -1)?;
    Ok(result)
}

fn get_hashmap(con: &mut Connection) -> Result<HashMap<String, String>> {
    con.del("person")?;
    // let mut p = HashMap::new();
    // p.insert("name", "fan");
    // p.insert("age", "18");
    con.hset("person", "name", "fan")?;
    con.hset("person", "age", "18")?;

    let all_keys:Vec<String> = con.hkeys("person")?;
    println!("all keys is {:?}", all_keys);

    let result = con.hgetall("person")?;
    Ok(result)
}

fn get_set(con: &mut Connection) -> Result<HashSet<i32>> {
    let mut hash_set = HashSet::new();
    hash_set.insert(1);
    hash_set.insert(2);
    hash_set.insert(3);
    con.sadd("peanut", hash_set)?;
    let result = con.smembers("peanut")?;

    Ok(result)
}

type TopThree = Vec<(String, i32)>;

#[derive(Debug)]
struct TopThree2{
    member: String,
    score: i32,
}

fn zset_demo(con: &mut Connection) -> Result<()>{
    con.zadd("scores", "fan", 100)?;
    con.zadd("scores", "peanut", 90)?;
    con.zadd("scores", "fan-tastic", 89)?;
    con.zadd("scores", "z", 10)?;
    con.zadd("scores", "wang", 77)?;

    // 获取 score 60-100之间的
    let names: Vec<String> = con.zrangebyscore("scores", 60, 100)?;
    println!("{:?}", names);

    // 将socres中的 z 的 score 增加20， 返回的结果是增加后的数据
    let new_score:i32 = con.zincr("scores", "z", 20)?;
    println!("{}", new_score);

    // 获取分数最高的前三个
    let res:TopThree = con.zrevrange_withscores("scores", 0, 2)?;
    println!("{:?}", res);

    // 可以根据自己的需要非常方便的转换为结构体
    let res2:Vec<(String,i32)>= con.zrevrange_withscores("scores", 0, 2)?;
    let top_three_2:Vec<TopThree2> = res2.into_iter().map(|(member,score)| TopThree2{member,score}).collect();
    for top in top_three_2 {
        println!("member:{} score:{}", top.member, top.score)
    }

    Ok(())

}


fn get_redis_conn() -> Result<Connection> {
    let client = redis::Client::open("redis://127.0.0.1/")?;
    let con = client.get_connection()?;
    Ok(con)
}

fn main() -> Result<()> {
    let mut con = get_redis_conn()?;
    let an_inter = fetch_an_inter(&mut con)?;
    println!("key fantastic value is {}", an_inter); // key fantastic value is 18

    let missing_key_result = get_missing_key(&mut con);

    if missing_key_result.is_err() {
        println!("{:?}", missing_key_result.unwrap_err().to_string()); // "key not found"
    }

    let missing_key2 = get_missing_key2(&mut con)?;
    println!("{}", missing_key2);

    let list = get_list(&mut con)?;
    println!("{:?}", list); // ["rust", "go", "vue"]

    let map = get_hashmap(&mut con)?;
    println!("{:?}", map); // {"age": "18", "name": "fan"}

    let hash_set = get_set(&mut con)?;
    println!("{:?}", hash_set); // {3, 2, 1}

    zset_demo(&mut con)?;
    Ok(())
}

```

## Pipeline

redis-rs也提供了pipeline的操作，一次发送多个命令执行：

```rust
fn pipeline_demo(con: &mut Connection) -> Result<()> {
    redis::pipe()
        .atomic()
        .zadd("Languages", "Rust", 100).ignore()
        .zadd("Languages", "Go", 90).ignore()
        .zadd("Languages", "Python", 87).ignore()
        .query(con)?;

    let res:Vec<String> = con.zrange("Languages", 0, 2)?;
    println!("res is {:?}", res);
    Ok(())
}
```

## Transactions

使用函数`redis::transaction`可以完成事务操作。这个函数会自动监视键，然后进入事务循环直到成功。一旦成功，结果就会返回。

```rust
fn transaction_demo(con: &mut Connection) -> Result<()> {
    let key = "transaction_key";
    con.set(key, 1)?;
    let (new_val,): (isize,) = redis::transaction(con, &[key], |con, pipe| {
        let old_val: isize = con.get(key)?;
        println!("old_val is: {}", old_val);
        pipe.incr(key, 2)
            .ignore()
            .incr(key, 100)
            .ignore()
            .get(key)
            .query(con)
    })?;
    println!("new_val is: {}", new_val);

    Ok(())
}
```

## 相关链接

- <https://redis.com/lp/redis-enterprise-rust/>
- <https://github.com/redis-rs/redis-rs>
- <https://docs.rs/redis/latest/redis/struct.Cmd.html#>
