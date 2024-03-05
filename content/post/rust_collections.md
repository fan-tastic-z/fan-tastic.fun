---
title: "Rust笔记----集合"
date: 2023-07-24T22:02:04+08:00
tags: ["Rust"]
categories: ["Rust"]
keywords: ["rust","Vec", "VecDeque", "LinkedList", "HashMap", "HashSet"]
draft: false
---

rust中有8个标准的集合,它们全部都是泛型类型.

- `Vec<T>`
- `VecDeque<T>`
- `LinkedList<T>`
- `BinaryHeap<T>`
- `HashMap<K, V>和BTreeMap<K, V>`
- `HashSet<T>和BTreeSet<T>`

## `Vec<T>`

Vec类型和数组类型的区别在于,Vec的长度动态可变.Vec的数据类型描述方式为`Vec<T>`

创建向量的方式多种多样：

- `let v = vec![1,2,3,4];`
- `let v = vec!["fan-tastic";10];`
- `let v = Vec::new();`
- `let v: Vec<i32> = (0..5).collect();` 这种情况通常需要写明类型
- `let v = Vec::with_capacity()` 创建空的vec, 容量可以根据需要自己设定

### Vec 内存布局

对于 `let n = vec![1,2,3]` 的内存布局为:

![Vec内存布局](/images/rust_base_data_type/Vec内存布局.png)

一个`Vec<T>` 包含3个值：

- 对分配在堆上用于保存元素缓冲区的引用
- 缓冲区可以存储的元素的个数，即容量
- 当前实际存储的元素的个数，即长度

当缓冲区达到容量上限后，再给向量添加元素会导致：

1. 重新分配一个更大的缓冲区
2. 将现有的内容复制过去
3. 基于新的缓冲区更新向量的指针和容量
4. 释放旧的缓冲区

注意： 如果提前知道向量中需要保存的元素的个数,可以使用`Vec::with_capacity`先创建一个有足够大缓冲区的向量.

```rust
fn main() {
    let mut v = Vec::with_capacity(2);
    v.push(1);
    v.push(2);
    v.push(3);
    println!("{}",v.capacity()) // 4
}
```

可以通过代码简单验证,如果没有空闲容量,则会重新申请一块内存,大小为原来vec内存大小的两倍.

### `Vec<T>` 常用方法

与数组类似,向量也可以使用切片的相关的方法.

```rust
fn main() {
    let mut names = vec!["fan-tastic", "fan", "cc"];
    let mut buffer = vec![0, 100];
    // 通过索引获取元素
    let first_name = names[0];
    // 取得一个切片引用
    let my_ref = &buffer[10..20];
    // 取得一个切片的副本
    let my_copy = buffer[10..20].to_vec();
    if let Some(item) = names.first() {
        println!("first {}", item);
    }
    if let Some(item) = names.last() {
        println!("last {}", item);
    }

    if let Some(item) = names.get(2) {
        println!("get index 2 value is {}", item);
    }
    println!("len is {}", names.len());
    println!("is empty: {}", names.is_empty());
    println!("capacity is {}", names.capacity());
    // 扩容
    names.reserve(20);
    // vec 末尾追加
    names.push("aa");
    //  移除并返回最后一个元素.返回值的类型 是 `Option<T>`.如果最后一个元素是 x 则返回 Some(x),如果向量为
    // 空则返回 None.
    names.pop();
    // 在指定索引位置添加元素
    names.insert(2, "peanut");
    //  移除并返回 vec[index], 将 vec[index+1..]中的值向左顺移一个位置
    // 如果 index >= vec.len() 则panic,因为此时 vec[index] 没有要移除的元素
    names.remove(2);
    // 清空所有元素
    names.clear();
    // names.truncate(new_len)将向量长度减少为 new_len，清除范围在vec[new_len..] 之内的元素
    names.truncate(2);
    // vec.extend(iterable) 按顺序将 iterable 的所有项添加到 vec 末尾
    names.extend(["a", "b", "c"]);
    // vec.split_off(index) 与 vec.truncate(index) 类似, 返回一个包含从 vec 末尾移除值的 Vec<T>
    names.split_off(3);
    // vec.append(&mut vec2) 把 vec2 的所有元素转移到 vec, 之后 vec2 变空
    names.append(&mut vec!["a", "b", "c"]);
}
```

更多方法参考: <https://doc.rust-lang.org/std/vec/struct.Vec.html>

关于向量的迭代需要注意:

- 迭代 `Vec<T>` 产生 `T` 类型的项,元素逐个从向量中转移出来并被消费.
- 迭代 `&Vec<T>` 类型的值.产生 `&T` 类型的项,即对个别元素的引用,不会转移.
- 迭代 `&mut Vec<T>` 类型的值产生 `&mut T` 类型的项.

数组、切片和向量还有 `.iter()` 和 `.iter_mut()` 方法, 这两个方法创建的迭代器产生对它们元素的引用.
通常我们对向量进行for循环迭代时都推荐使用 `.iter()` 和 `.iter_mut()`.

## 切片

切片（`[T]`）是数组或向量中的一个区域，由于切片可以是任意长度，因此不能直接存储在变量中或者作为函数参数传递，切片总是通过引用传递。
对切片的引用是一个胖指针： 包括指向切片第一个元素的指针和切片中元素的数量。

```rust
let v:Vec<i32> = vec![18,19,20,31];
let a:[f64;4] = [0.0,0.1,0.2,0.3];

let sv: &Vec<i32> = &v;
let sa: &[f64; 4] = &a;
```

这里Rust 就会自动把 `&Vec<i32>` 的引用和`&[f64; 4]`的引用转换成直接指向数据的切片引用。

![切片](/images/collections/slice.jpg)

## `VecDeque<T>`

`VecDeque<T>` 双端队列,支持前端和后端的高效添加和移除操作.

创建`VecDeque<T>`的方法:

- `VecDeque::new()`
- `VecDeque::with_capacity(n)`
- `VecDeque::from(vec)`

虽然没有提供 `vec_deque![]` 宏的来快速创建创建双端队列, 但是 `VecDeque<T>` 实现了 `From<Vec<T>>`, 因此 `VecDeque::from(vec)`可以快速将向量转换为队列.

### VecDeque内存布局

![VecDeque内存布局](/images/collections/vecdeque.png)

`VecDeque` 堆中的数据并不总是从头开始存储, 是可以环绕起来的
栈中的数据除了当前缓冲区的大小,还包括了起点,终点,用于保存数据在缓冲区中的开始和结束位置.
向队列的任何一端添加值都需要使用一个未被使用的"槽位",及途中的红色区域.如果添加时空间不够,则分配更多的内存.

### 基本使用

- deque.push_front(value) 在队列前面添加值
- deque.push_back(value) 在队列后面添加值
- deque.pop_front() 移除并返回队列前面的值，返回 `Option<T>`,队列为空时返回 `None`,跟 vec.pop() 一样
- deque.pop_back() 移除并返回队列后面的值，同样返回 `Option<T>`
- deque.front() 和 deque.back() 与 vec.first() 和 vec.last() 相似, 返回队列前端和后端元素的引用. 返回 `Option<T>`，队列为空时返回 `None`
- deque.front_mut() 和 deque.back_mut() 与 vec.first_mut() 和 vec.last_mut() 相似,返回 `Option<&mut T>`

更多方法参考: <https://doc.rust-lang.org/std/collections/vec_deque/struct.VecDeque.html>

## `LinkedList<T>`

`LinkedList<T>` 链表,也是一种存储序列值的方式,不过链表的每个值都存储在独立的堆内存中

创建`LinkedList<T>`的方法:

- `LinkedList::new()`
- `LinkedList::from([1, 2, 3, 4])`

`LinkedList`实现了 `From<[T; N]>` 和 `FromIterator<T>` 所以我们还是可以非常方便通过 `LinkedList::from([1, 2, 3, 4])` 来创建一个
`LinkedList`.或者如下方式:

```rust
use std::collections::LinkedList;

fn main() {
    let v = vec![1, 2, 3, 4];
    let list = LinkedList::from_iter(v);
    println!("{:?}", list);
}
```

### LinkedList 内存布局

![LinkedList内存布局](/images/collections/linkedlist.png)

## `BinaryHeap<T>`

`BinaryHeap` 是一个元素松散组织的集合，而且最大值始终会冒泡到队
列前端

常用的方法:

- heap.push(value) 向堆中添加值
- heap.pop() 从堆中移除并返回最大值,返回类型为 `Option<T>`,如果堆为空则返回 None
- heap.peek() 返回堆中最大值的引用,返回类型为 `Option<&T>`

```rust
use std::collections::BinaryHeap;

fn main() {
    let mut heap = BinaryHeap::from(vec![1, 10, 14, 9, 23, 21, 17]);
    assert_eq!(heap.peek(), Some(&23));
    assert_eq!(heap.pop(), Some(23));
}

```

`BinaryHeap` 并不限于存储数值。实现了 `Ord` trait 的任何类型,都可以保存在 BinaryHeap 中, 可以用在有优先级的工作任务队列中, '.pop()' 方法始终返回接下来有优先级最高的任务.

注意 `BinaryHeap` 是可迭代类型，也有自己的 `.iter()` 方法,但迭代器会以任意顺序产生堆的元素,而不是从大到小.

如果需要按照优先级顺序消费,可以通过while循环:

```rust
while let Some(item) = heap.pop() {
    println!("{}", item);
}
```

## `HashMap<K, V>`和`BTreeMap<K, V>`

Rust 提供了两种map 类型 `HashMap<K, V>`和`BTreeMap<K, V>`
`HashMap` 把键和值保存在一个散列表中,因此要求键的类型 K 实现标准的散列和相等性特型 `Hash` 和 `Eq`
`BTreeMap` 按照键的顺序存储条目,总体为树形结构,因此要求键类型 K 实现 `Ord`

创建map

- `HashMap::new()` 和 `BTreeMap::new()`
- `iter.collect()` 可用于从"键 – 值"对创建和填充新 HashMap 或 BTreeMap. iter 必须是一个 `Iterator<Item=(K, V)>`
- `HashMap::with_capacity(n)`

常用的方法:

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert("fan-tastic", 18);
    map.insert("fan", 11);
    map.insert("cc", 23);
    println!("len is {}", map.len());
    println!("is empty {}", map.is_empty());
    println!("is contains key cc {}", map.contains_key("cc"));
    if let Some(result) = map.get("fan") {
        println!("{}", result);
    }
    if let Some(result) = map.get_mut("cc") {
        *result = 2;
    }
    map.remove("cc");
    println!("{:?}", map);
    map.entry("fan-tastic").or_insert(11);
    map.clear();
}
```

map 的迭代

- 按值迭代（for (k, v) in map）产生 (K, V) 对,这样会消费map
- 按共享引用迭代（for (k, v) in &map）产生 (&K, &V) 对
- 按可修改引用迭代（for (k, v) in &mut map）产生 (&K, &mut V)对

### 补充

之前一直没有把 <https://github.com/rust-lang/rustlings> 的练习做一遍，所以最近想的是把官方的这个小练习做一遍，在做到关于HashMap的部分的时候发现 `hashmap 3`的练习挺有意思的，在搜索解决方式的时候看到了一个自己之前没有用的方法：<https://www.reddit.com/r/rust/comments/w4vf5a/idiomatic_hashmap_access_from_rustlings_exercise/>

```rust
// A structure to store the goal details of a team.
struct Team {
    goals_scored: u8,
    goals_conceded: u8,
}

fn build_scores_table(results: String) -> HashMap<String, Team> {
    // The name of the team is the key and its associated struct is the value.
    let mut scores: HashMap<String, Team> = HashMap::new();

    for r in results.lines() {
        let v: Vec<&str> = r.split(',').collect();
        let team_1_name = v[0].to_string();
        let team_1_score: u8 = v[2].parse().unwrap();
        let team_2_name = v[1].to_string();
        let team_2_score: u8 = v[3].parse().unwrap();

        scores
        .entry(team_1_name)
        .and_modify(|t| {
            t.goals_scored += team_1_score;
            t.goals_conceded += team_2_score;
        })
        .or_insert(Team {
            goals_scored: team_1_score,
            goals_conceded: team_2_score,
        });

        scores
        .entry(team_2_name)
        .and_modify(|t| {
            t.goals_scored += team_2_score;
            t.goals_conceded += team_1_score;
        })
        .or_insert(Team {
            goals_scored: team_2_score,
            goals_conceded: team_1_score,
        });
    }
    scores
}
```

`hashmap` 中的 `entry` 方法可以可以非常方便的让我们对hashmap进行处理来实现这个联系的需求

## `HashSet<T>`和`BTreeSet<T>`

`HashSet＜K＞`和`BTreeSet＜K＞`其实就是`HashMap＜K，V＞`和`BTreeMap＜K，V＞`把Value设置为空元组的特定类型，等价于`HashSet＜K，（）＞`和`BTreeSet＜K，（）＞`

这两种集合类型的特性:

- 集合中的元素应该是唯一的
- 集合中的元素应该都是可哈希的类型
- HashSet应该是无序的，BTreeSet应该是有序的

```rust
use std::collections::{BTreeSet, HashSet};

fn main() {
    let mut hnames = HashSet::new();
    hnames.insert("fan-tastic");
    hnames.insert("fan");
    hnames.insert("cc");
    println!("{}", hnames.len());
    println!("{}", hnames.contains("cc"));
    // 顺序随即
    println!("{:?}", hnames);
    let mut bnames = BTreeSet::new();
    bnames.insert("fan-tastic");
    bnames.insert("fan");
    bnames.insert("cc");
    // 顺序固定
    println!("{:?}", bnames);
}
```

## 相关链接

- <https://doc.rust-lang.org/std/vec/struct.Vec.html>
- <https://doc.rust-lang.org/std/collections/vec_deque/struct.VecDeque.html>
- <https://doc.rust-lang.org/std/collections/struct.LinkedList.html>
- <https://doc.rust-lang.org/std/collections/struct.BinaryHeap.html>
- <https://doc.rust-lang.org/std/collections/struct.HashMap.html>
- <https://doc.rust-lang.org/std/collections/struct.BTreeMap.html>
- <https://doc.rust-lang.org/std/collections/struct.HashSet.html>
- <https://www.reddit.com/r/rust/comments/w4vf5a/idiomatic_hashmap_access_from_rustlings_exercise/>
- <https://doc.rust-lang.org/std/collections/struct.HashMap.html#method.entry>
- <https://doc.rust-lang.org/std/collections/hash_map/enum.Entry.html#method.and_modify>
