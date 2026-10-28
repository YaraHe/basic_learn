---
title: "Prompt、少样本示例与上下文管理"
description: "检索资料、对话历史和少样本示例怎样进入 Prompt？上下文窗口有限时，如何保留回答真正需要的信息？"
category: "RAG"
subcategory: "Prompting and Context"
tags:
  - "Prompt Engineering"
  - "Few-shot Learning"
  - "Context Window"
  - "Context Management"
type: "learning-note"
sources: []
status: "reviewed"
---

# Prompt、少样本示例与上下文管理

本文整理自 RAG 课程学习记录，保留其中的解释与理解。

## Prompt engineering: building your augmented prompt

Prompt 通常包含系统要求、用户问题和检索到的资料。要让模型知道哪些文本是任务要求、哪些是参考文档，以及回答时如何引用。下面先看消息里的角色和内容，再看系统要求如何规定回答方式，最后看历史对话、检索文档与当前问题如何一起进入 Prompt。图中的问答只用来说明消息如何组织。

![图示：把检索内容放进 Prompt](图片附件/Pasted%20image%2020260902134911.png)

前一张解释 `system`、`user`、`assistant` 等消息角色；接下来两张展示系统要求可以怎样规定语气、回答边界与格式。

![图示：把检索内容放进 Prompt](图片附件/Pasted%20image%2020260902135059.png)

![图示：把检索内容放进 Prompt](图片附件/Pasted%20image%2020260902135141.png)

第四张进一步把“只依据检索资料回答、引用来源”等要求写进系统提示。最后的模板图则把系统要求、历史对话、检索文档和当前问题放到一起；其中的示例答案只是排版演示，需要依据实际检索证据核对。

![图示：把检索内容放进 Prompt](图片附件/Pasted%20image%2020260902135323.png)

![图示：把检索内容放进 Prompt](图片附件/Pasted%20image%2020260902140219.png)

### In-Context Learning：用示例说明希望怎样回答

在 Prompt 里放“问题—回答”示例，可以告诉模型期待的结构和语气。一个示例称为 one-shot，多个示例称为 few-shot。示例是上下文提示，不会像微调那样更新模型参数。

如果希望每次都更换示例，可以这样做——Retrieval‑Augmented Few‑Shot 检索增强少样本。以经过筛选的客服对话作为示例，可以按三步理解：

1. 先把经过筛选的历史成功对话建立索引，供后续找示例。
2. 用户提出新问题时，同时检索两路内容：一路找知识库里的事实资料，另一路找与当前问题相近的历史问答。
3. 把两路结果放进 Prompt，让模型既看到回答所需的资料，也看到合适的回答形式。

两类内容在 Prompt 里应明确区分：历史对话用于示范结构和语气，事实仍要以当前有权限、有效的知识库资料为准；还要留意对话隐私和质量。

### 引导分步回答与草稿

复杂问题可以要求模型先列出处理步骤，再给出结论。`<scratchpad>` 标签可以表示一个**显式输出的草稿区域**，例如：

```text
<scratchpad>
Option 1: Could be X because...
Option 2: Might be Y if...
Check which option is supported by the retrieved material.
</scratchpad>
```

如果应用设计成让模型**显式输出**这段草稿，程序就可以读取和记录它；若平台不提供模型的内部推理过程，就不能假定程序能取得“隐藏思考”。草稿会消耗 Token，也可能包含不宜展示或保存的内容，因此应按实际需要决定是否使用。Chain of Thought（链式思考）在这里可理解为要求模型按步骤处理问题，而不是保证它一定推理正确。

### 管理上下文窗口

检索片段、历史对话和示例都要占窗口。可以先问：某种提示技巧是否真的改善回答？若没有收益，就不必长期占用上下文。

单轮问题重点是只放必要资料。多轮对话可以保留最近消息、压缩较早内容为摘要，或移除不再有用的中间草稿。在 RAG 场景中，应优先放与当前问题相关的 chunk；复杂任务也可以考虑更长上下文模型与更简洁的系统指令。裁剪的同时要防止丢掉用户仍在引用的前文。

![图示：管理上下文窗口](图片附件/Pasted%20image%2020260902145906.png)
