---
title: "周报20231228"
date: 2023-12-28T11:39:28+08:00
tags: ["weekly_review"]
categories: ["weekly_review"]
keywords: ["weekly review","weekly", "周报"]
draft: false
---

上周因为回老家处理事情没有时间写周报，所以放到这周统一进行整理，这两周自己很多事情进展比较缓慢，计划要写的东西没有进行好好整理，而对于计划中的目标还是不够坚定，这个需要再后续进行调整。

## 关于开源

这两周在阅读《大教堂与集市》，书中一些观点或者一些描述自己觉得对自己有一些影响，或者产生了一些思考的这里进行一下整理。

> 当开始建设社区的时候，你需要拿出一个像样的承诺。程序此时并不需要特别好，它可以简陋、有错、不完整，文档可以少得可怜。但它至少要做到：(a)能运行，(b)让潜在的合作开发者相信，这个软件在可预见的未来，能演变成一个非常棒的东西。

对于自己来说，自己总是想着在最开始把程序写的非常好，这样就会造成这个程序很可能写了一段时间之后，自己就不想要写了，这个在近期自己想要写的一个项目上非常明显，也导致目前的进展非常缓慢。在选中一个目标的时候，先尝试做，不要着急在最开始的时候每一个步骤都做的多么的完美，完美都是通过一点一点的完善最终才可能尽可能的更加完美。

> 人类通常会从一种位于“最佳挑战区”的任务中获得乐趣，也即它不是太容易而让人无聊，也不是太困难而无法完成。一个快乐的程序员是一个既没有被浪费也没有被压垮（由于不适当的目标或过程中充满压力与冲突）的人，乐趣预示着效率

这个自己在最近开开源的项目尝试贡献自己的代码上感受还是非常明显的，在自己计划开始慢慢更多的参与到开源社区中的时候，自己是通过先开始使用这个项目`loco`，因为`loco` 还处于一个比较早期的阶段，使用中自己会碰到一些文档的问题，或者代码上的小问题，自己先从这些开始进行修复和提交`PR`，这个过程怎么形容呢？很奇怪，虽然大部分都是一些简单的文档修改，以及小的bug的修复，但是至少在这个初期阶段，做这个事情的的乐趣要比工作中的代码更多，或许就像书中的这个描述：

> 与完全出于兴趣的工作相比，被委托的工作通常表现出较少的创造性。

### 关于技术

也正是因为在了解和使用`loco`的过程中，发现通过这种开源项目的issue 列表里通过一些有经验的开发者的回答可以学到一些开发上经验，其中这个issue 给自己的印象比较深刻：<https://github.com/loco-rs/loco/issues/245>

其实我在最开始使用的时候也碰到了这个疑问，因为确实我在之前的老代码中也是像这个提问者一样，当时也在想作者为什么这样设计了这个表结构，users表中的`pid`是什么意思，看了下面的评论之后，明白了作者的设计初衷：

- **Data Integrity:** Numeric IDs (id) are typically managed by the database and offer performance advantages, especially when it comes to indexing and joining tables. They are sequential and compact, which helps in maintaining a streamlined and efficient database operation.
- **External Flexibility:** UUIDs (pid) provide a globally unique identifier that is ideal for public-facing interfaces. They reduce the risk of enumeration attacks and are not sequential, which can be an advantage for security and distributed systems.
- **Best of Both Worlds:** By using both, we ensure that the system remains robust and efficient internally while also providing the necessary flexibility and security considerations for external interactions

## 一些感悟

一个人的性格很难改变，自己最近在思考，好像从初中开始到现在自己的性格方面没有太大的改变，那如果自己做的事情或者工作时适应自己性格的，可能自己就能够做的更好，而和自己性格特点相反的或者相违背的事情或者工作，自己可能就只能做的普普通通。

或许可以根据自己的性格特点，来调整自己学习，工作，这样自己可能更容易做出成绩，并且可以自己更加乐此不疲的坚持做，并乐于做更加深入的研究。

可以先从做这个事情的人具有什么样的性格，即使这个事情和自己的性格相违背，也可以让尽可能调整自己，不至于把事情做的太差，至少保证可以更好的完成。

同时可以思考自己的性格，来反推哪些事情更适合自己，来进行探索，在没有明确自己喜欢或者想要做什么的情况下，通过这种方式来找到自己喜欢的事情。

### 一个司机

最近打车碰到一个司机，在我快到公司的是十字路口的时候，她提前问我，“可以先结束订单么？” 因为提前结束订单，她就可以开始接下一单，而红绿灯本身也要等快一分钟，事实也确实，在她结束我的这个订单之后，还在等待红绿灯的间隙，她就接到了下一单。而一天下来，通过这种方式，她可能机会比其他司机多跑好几单，一年下来可能就是多几百单。

这个司机还是挺让自己感触的，不知道从什么开始，自己做事情可能就像普通的司机一样，一单一单的来呗，但如果也能像上面的这个司机一样，那么自己可能可以做更多的事情，读更多的书？学习的更多？工作完成的更好？等等吧。这个事情也是对自己的一个激励，或许随着年龄的增长，随着工作时间的增加，少了年轻时的冲劲，不管是工作，还是学习，还是应该保持一个年轻积极的心态。
