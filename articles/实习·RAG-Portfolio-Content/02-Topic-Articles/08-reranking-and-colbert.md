---
title: "重排序、Cross-Encoder 与 ColBERT"
description: "初次检索后为什么还要重排？Cross-Encoder、ColBERT 与其他重排方式怎样在质量、延迟和存储之间取舍？"
category: "RAG"
subcategory: "Reranking"
tags:
  - "Reranking"
  - "Cross-Encoder"
  - "ColBERT"
  - "Late Interaction"
type: "learning-note"
sources: []
status: "reviewed"
---

# 重排序、Cross-Encoder 与 ColBERT

本文整理自 RAG 课程学习记录，保留其中的解释与理解。
## 为什么粗检索之后还要重排

语义检索常用 Bi-Encoder：查询和文档分别编码成向量，文档向量可以提前算好，再借助 ANN 快速找候选。这种方法速度好，但把整段压成一个向量后，某些词语间的细微关系不容易充分表达。对少量候选再做更细的比较，就是重排序的思路。

### Cross-Encoder：把问题和候选一起看

Cross-Encoder 把一个查询与一个候选文档组成一对，同时输入模型，让模型直接判断两者的相关程度。

![图示：Cross-Encoder：把问题和候选一起看](图片附件/Pasted%20image%2020260831151047.png)

基本流程是：

1. 先用较快的检索器取一批候选片段。
2. 将查询和每个候选片段分别配成一对，交给 Cross-Encoder。
3. 按模型输出的相关性分数重新排序，再取最靠前的结果。

分数的数值范围取决于模型和实现，不一定天然是 0～1。它能同时观察问题与文档的词语关系，通常适合做精排；但每个候选都要在查询到来后计算，无法把“查询—文档对”的结果全部提前存好，所以延迟和计算成本较高,因此只能对向量检索粗筛出来少量 top‑k 候选做二次打分。它也说明了检索质量与速度之间的取舍。

### ColBERT：保留词元级向量，晚一点再比较

ColBERT（Contextualized Late Interaction over BERT）介于“整段只存一个向量”和“查询与文档一起送进 Cross-Encoder”之间。它分别编码查询和文档，却为文档的多个 Token 保留细粒度向量；查询时再比较两边的 Token 向量。

![图示：ColBERT：保留词元级向量，晚一点再比较](图片附件/Pasted%20image%2020260831154912.png)

以查询有 10 个 Token、文档有 100 个 Token 为例，如果直接逐一比较，就会得到 1000 个相似度值。对**每个查询 Token**，找它与该文档中最相似的 Token，再把这些最大值相加，得到文档的匹配分数。这种计算常被称为 MaxSim。

![图示：ColBERT：保留词元级向量，晚一点再比较](图片附件/Pasted%20image%2020260831160003.png)

如果对百万级知识库的每篇文档都完整做词对词比较，计算和存储会很重。因此大规模实现会借助索引先缩小候选，再对留下的文档做更完整的晚交互打分；ColBERTv2 与 PLAID 是相关的模型及索引工作。它能保留更细的匹配信息，也要为大量 Token 向量付出存储和索引成本。实际速度与效果不能只凭架构名称断定。

三种方式可以这样记：Bi-Encoder 先独立编码、适合大范围快速召回；Cross-Encoder 成对比较、适合少量候选精排；ColBERT 把细粒度交互留到检索阶段，换取另一种质量、速度和存储的组合。可以比较三条选型路径：

- **通用的两阶段路径：**Bi-Encoder 召回 → Cross-Encoder 重排 → LLM。
- **希望保留更细匹配信息的路径：**ColBERT 召回 → 必要时再用 Cross-Encoder 重排 → LLM；额外重排是否有收益需实测。
- **轻量路径：**只用 Bi-Encoder 召回并交给 LLM，减少重排带来的延迟与费用，但要检查质量是否满足需求。


## Reranking 的两种做法

**Cross-Encoder 重排序**延续上面的流程：Bi-Encoder 先找较多候选，Cross-Encoder 再把最相关的块排在前面。

![图示：Reranking 的两种做法](图片附件/Pasted%20image%2020260901142830.png)

**LLM 评分重排序**则把查询和候选文档交给 LLM，让它依据明确标准判断相关程度并给出排序或分数。这种方式灵活，但需要在用户提问后逐条或成批调用模型，增加延迟和费用。无论哪种方法，重排序只能重排已召回的候选；如果关键文档在粗检索阶段就没进来，后面也找不回来。
