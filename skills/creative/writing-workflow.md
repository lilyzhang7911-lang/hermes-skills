---
name: writing-workflow
description: "写作工作流统一入口：探索→组装→编辑→去AI化四阶段。"
tags: [writing, editing, humanize, fragments, beats, article]
---

# 写作工作流

统一入口：覆盖探索、组装、编辑、去AI化四个阶段。

## 场景决策树

```
开始
├─ 有原始想法但无结构？ → 探索阶段（fragments）
├─ 有原始材料想组装成文章？ → 组装阶段（beats 或 shape）
├─ 有草稿想改进结构/清晰度？ → 编辑阶段（edit-article）
└─ 文本听起来像 AI 写的？ → 去AI化阶段（humanizer）
```

## 一、探索阶段：writing-fragments

**适用**：有想法但无结构，需要挖掘原始素材  
**核心**：纯探索，不承诺结构——承诺结构是"利用"阶段的工作

### 工作流程

1. **开始采访**：无情地采访用户关于他们想写的任何内容
2. **捕捉片段**：从用户说的第一件事开始捕捉，包括初始提示
3. **追加到文件**：片段出现时追加到单个 markdown 文件
4. **不强加结构**：不要强加阶段、大纲或文章结构

### 什么是片段

片段是任何可能存活到最终文章的文本。它必须**对作者可读**，但不需要定义术语或对冷读者可理解。标准是"这是好的写作吗？"，而不是"这是自洽的论证吗？"

片段故意异质：
- 你想在某处使用但还不知道在哪里的尖锐句子
- 有一行理由的声明
- 小插图：发生的事情、代码片段、场景、类比
- 半思考："关于 X 感觉像 Y 的东西，以后再研究"
- 引用、对话、听到的话
- 一组靠感觉挂在一起的相关观察
- 抱怨、忏悔、妙语
- **引导词**——整个作品可以悬挂的紧凑隐喻或造词（最重要的片段）

### 文件格式

```markdown
# Working title

第一个片段在这里。

可以是多段。可以包括列表、代码、引用——片段自然采取的任何形状。

---

第二个片段。

---

> 用户想保留的引用行。

对它的反应。

---

- 一组相关观察
- 靠感觉挂在一起
- 想彼此靠近
```

片段由水平规则（`\n---\n`）分隔。正文内无标题。无标签。无顺序。

### 写作节奏

- 静默追加。不要为每个片段请求许可。
- 每次写入前：从磁盘重新读取文件。用户可能已编辑、重新排序或删除片段。
- 永远不要覆盖文件；只追加（或如果用户要求，就地编辑特定片段）。
- 用户可以随时说"剪切最后一个"、"更尖锐地重写那个"、"合并那两个"。

## 二、组装阶段

有两种方法可选：

### A. writing-beats（节拍旅程）

**适用**：有原始材料，想组装成"选择你自己的冒险"风格的旅程  
**核心**：逐个节拍构建，每个节拍在落地概念后才能被后续节拍使用

#### 工作流程

1. **建立前提条件**：在开始节拍前，与用户确定读者进来时已经知道什么——从一开始就**落地**的概念。其他所有内容必须由节拍在后续节拍使用它之前落地。

2. **写 2-3 个候选起始节拍**：每个是不同的入口点。每个只能依赖已落地的概念；注明每个落地了什么新概念。在写入文章文件前向用户展示节拍。用户选择一个。预览该选择解锁了什么节拍。

3. **一旦用户选择起始节拍，只写那个节拍**：节拍可以是一句话或几段——那个节拍自然需要的任何长度。停在那里。

4. **从磁盘重新读取文章文件**。然后提供 2-3 个候选**下一节拍**——从文章现在所在的地方可以转向的不同方向。每个必须从当前落地集可达；注明每个落地了什么。

5. **循环步骤 3-5**，直到文章达到自然结束。

#### 什么是节拍

节拍是旅程中的一步。它做一件事——设置场景、落地一个点、提出问题、抛出旁白、扭转角度。然后停止，让读者停留在下一个节拍可以转向的地方。

节拍按需要大小：
- 如果动作只需要一句话（"然后三周什么都没发生。"）。
- 如果动作需要设置，一段短段落。
- 如果节拍是自包含的小插图、论证或示例，多段。

如果"节拍"需要五段和三个子标题，它不是节拍——是两个粘在一起的节拍。拆分它。

#### 落地

每个**概念**必须在节拍依赖它之前**落地**：读者要么进来时就知道，要么在早期节拍中遇到它。依赖未落地概念的节拍会失去读者——这是旅程不能做的唯一动作。单位是概念，不是它的词：即使没有术语，节拍也可以依赖读者缺乏的想法。当概念有名称——**术语**——落地它意味着同时落地想法和术语。

概念通过两种方式之一落地：
- **前提条件**——在第一个节拍之前落地。读者带来它。开始时固定。
- **引入**——节拍建立它，从那时起它为每个后续节拍落地。

保持一个运行列表，记录到目前为止落地的内容，每次节拍落地时更新它。

### B. writing-shape（逐段塑造）

**适用**：有原始材料堆，想逐段塑造成文章  
**核心**：提交到结构并挖掘材料堆来填充它

#### 工作流程

1. **读取材料堆**。完整读取输入文件。形成对它的内容的感觉。

2. **建立前提条件**。与用户确定读者进来时知道什么——从一开始就**落地**的概念。其他所有内容必须由块在后续块依赖它之前落地。

3. **起草 2-3 个候选开头**。每个开头应该暗示文章的不同论点或角度。展示所有。强制用户选择或组合混合。选择的开头定义了文章其余部分必须做什么。

4. **逐段生长**。开头落地后，问"给定这个开头，读者接下来需要听到什么？"从材料堆中提取材料来回答。下一个块只能依赖已落地的概念，并在落地时落地新概念。争论下一个块采取的形式——段落、列表、表格、标注、引用、代码块。每个格式选择应该是深思熟虑和可辩护的。

5. **边走边追加到文章文件**。不要批处理。立即写入每个同意的段落或块，让用户看到文章成形。

6. **循环步骤 4**，直到文章完成。用户决定何时完成。

#### 对话感觉

这是采访的倒置。在构思中，问题是"你实际注意到什么？"在这里是"这篇文章实际论证什么，读者需要按什么顺序听到它？"推回。拒绝让弱过渡滑过。如果段落没有赢得它的位置，剪切它。

具体动作：
- "这段为读者做了什么前一段没做的？"
- "如果我剪切这个，什么会坏？"
- "这是散文，还是应该是列表？为什么是散文？"
- "这句话做两个工作——拆分它或选一个。"
- "开头承诺了 X。我们漂移到 Y。要么重新线程化它，要么改变开头。"

#### 从堆中提取

将原始材料视为采石场，不是脚本。提取片段，重新工作它以适应周围段落，然后放置它。片段可以跨多段拆分，与另一个合并，或改述。堆的工作是被挖掘；文章的工作是读起来像一个声音。

如果堆缺少文章需要的东西，明确命名差距："我们需要一个例子，堆里没有——现在给我一个，否则我们剪切这个部分。"

## 三、编辑阶段：edit-article

**适用**：有草稿，想改进结构、清晰度、收紧散文

### 工作流程

1. **按标题分节**。考虑你想在这些部分中做的要点。

考虑信息是有向无环图，信息片段可以依赖其他信息片段。确保部分及其内容的顺序尊重这些依赖。

与用户确认部分。

2. **对每个部分**：

2a. **重写部分**以改进清晰度、连贯性和流畅性。每段最多 240 字符。

2b. **检查依赖**：这段依赖的概念在前面落地了吗？

2c. **检查过渡**：从前一段到这一段的过渡自然吗？

2d. **检查节奏**：这段的长度和结构适合它的内容吗？

3. **整体审查**：

3a. **开头是否承诺了 X，文章是否交付了 X？**

3b. **是否有冗余部分可以合并？**

3c. **结尾是否有力，还是拖沓？**

## 四、去AI化阶段：humanizer

**适用**：文本听起来像 AI 写的，需要移除 AI 模式并添加真实声音  
**触发**：用户说 "humanize"、"de-AI"、"de-slop"、"un-ChatGPT"

### 核心洞察

LLM 使用统计算法猜测接下来应该是什么。结果倾向于最统计上可能的完成，这就是下面的模式如何被烘焙进去的。

### 工作流程

1. **识别 AI 模式**。扫描下面列出的 34 个模式。
2. **重写有问题的部分**。用自然替代替换 AI 主义。
3. **保留意义**。保持核心信息完整。
4. **保持声音**。匹配预期语调（正式、随意、技术等）。如果提供了声音样本，特别匹配它。
5. **添加灵魂**。移除坏模式只是一半工作；重写还需要真实个性。见"个性和灵魂"部分。
6. **做最终反AI通过**。问自己："下面什么使它如此明显是 AI 生成的？"用任何剩余提示简要回答，然后再修改一次。

### 34 个 AI 模式

#### 内容模式（1-6）

1. **过度强调重要性、遗产和更广泛趋势**
   - 警惕词：stands/serves as, is a testament/reminder, vital/significant/crucial/pivotal role, underscores/highlights, reflects broader, symbolizing, contributing to, setting the stage for, marking/shaping, represents/marks a shift, key turning point, evolving landscape, focal point, indelible mark, deeply rooted

2. **过度强调 notable 和媒体报道**
   - 警惕词：independent coverage, local/regional/national media outlets, written by a leading expert, active social media presence

3. **以 -ing 结尾的肤浅分析**
   - 警惕词：highlighting/underscoring/emphasizing..., ensuring..., reflecting/symbolizing..., contributing to..., cultivating/fostering..., encompassing..., showcasing...

4. **促销和广告式语言**
   - 警惕词：boasts a, vibrant, rich (figurative), profound, enhancing its, showcasing, exemplifies, commitment to, natural beauty, nestled, in the heart of, groundbreaking (figurative), renowned, breathtaking, must-visit, stunning

5. **模糊归因和鼬鼠词**
   - 警惕词：Industry reports, Observers have cited, Experts argue, Some critics argue, several sources/publications

6. **大纲式"挑战和未来前景"部分**
   - 警惕词：Despite its... faces several challenges..., Despite these challenges, Challenges and Legacy, Future Outlook

#### 语言和语法模式（7-13）

7. **过度使用的"AI 词汇"词**
   - 高频 AI 词：Actually, additionally, align with, crucial, delve, emphasizing, enduring, enhance, fostering, garner, highlight (verb), interplay, intricate/intricacies, key (adjective), landscape (abstract noun), pivotal, showcase, tapestry (abstract noun), testament, underscore (verb), valuable, vibrant
   - 营销和博客陈词滥调：at the end of the day, when it comes to, in a world where, moving forward, circle back, deep dive, game-changer, double down, take a step back, on the same page, make no mistake, it turns out, let me be clear, navigate (for challenges), lean into, unpack (before analysis), straightforward

8. **避免"is"/"are"（系动词回避）**
   - 警惕词：serves as/stands as/marks/represents [a], boasts/features/offers [a]

9. **否定并列和尾部否定**
   - "Not only...but..."或"It's not just about..., it's..."过度使用
   - 尾部否定片段如"no guessing"或"no wasted motion"

10. **三法则过度使用**
    - LLM 强制将想法分成三组以显得全面

11. **优雅变体（同义词循环）**
    - AI 有重复惩罚代码导致过度同义词替换

12. **虚假范围**
    - "from X to Y"结构，其中 X 和 Y 不在有意义的尺度上

13. **被动语态和无主语片段**
    - LLM 经常隐藏动作或完全删除主语

#### 风格模式（14-19）

14. **破折号过度使用**
    - LLM 使用破折号（—）比人类多，模仿"有力"的销售写作

15. **粗体过度使用**
    - AI 聊天机器人机械地用粗体强调短语

16. **内联标题垂直列表**
    - AI 输出列表，项目以粗体标题后跟冒号开始

17. **标题中的标题大小写**
    - AI 聊天机器人在标题中大写所有主要词

18. **表情符号**
    - AI 聊天机器人经常用表情符号装饰标题或项目符号

19. **卷曲引号**
    - ChatGPT 使用卷曲引号（"..."）而不是直引号（"..."）

#### 沟通模式（20-22）

20. **协作沟通工件**
    - 警惕词：I hope this helps, Of course!, Certainly!, You're absolutely right!, Would you like..., let me know, here is a...

21. **知识截止免责声明**
    - 警惕词：as of [date], Up to my last training update, While specific details are limited/scarce..., based on available information...

22. **谄媚/奴性语调**
    - 过度积极、讨好人的语言

#### 填充和回避（23-29）

23. **填充短语**
    - "In order to achieve this goal" → "To achieve this"
    - "Due to the fact that it was raining" → "Because it was raining"
    - "At this point in time" → "Now"

24. **过度回避**
    - "It could potentially possibly be argued that the policy might have some effect on outcomes."
    → "The policy may affect outcomes."

25. **通用积极结论**
    - "The future looks bright for the company. Exciting times lie ahead..."

26. **连字符词对过度使用**
    - 警惕词：third-party, cross-functional, client-facing, data-driven, decision-making, well-known, high-quality, real-time, long-term, end-to-end

27. **说服权威修辞**
    - 警惕词：The real question is, at its core, in reality, what really matters, fundamentally, the deeper issue, the heart of the matter

28. **路标和公告**
    - 警惕词：Let's dive in, let's explore, let's break this down, here's what you need to know, now let's look at, without further ado

29. **碎片化标题**
    - 标题后跟一行段落，只是简单地重述标题

#### 风格、节奏和修辞模式（30-34）

30. **强制隐喻和比喻过度写作**
    - 原始但牵强的隐喻、混合隐喻、比喻替代

31. **戏剧性碎片和有力的踢脚线**
    - 两三个词的无主语句子用于戏剧效果
    - 每段或每部分结尾的可引用"mic-drop"行

32. **立即回答的修辞问题**
    - "What if...?", "The question is...", "Ever wondered...?"
    - 问题后立即跟自己的答案

33. **句子开头习惯**
    - 警惕词：So..., Look,, 习惯性句子开头 And/But, "I think"/"I believe" 陈述事实时, 副词开头（Interestingly, Importantly, Notably, Crucially, Essentially, Ultimately）

34. **安慰踢脚线**
    - 警惕词：And that's okay., And that's fine., There's nothing wrong with that., no shame in..., you're not alone, it's completely normal

### 个性和灵魂

避免 AI 模式只是一半工作。无菌、无声音的写作同样明显。好写作背后有人。

#### 无灵魂写作的迹象（即使技术上"干净"）：
- 每句话长度和结构相同
- 没有观点，只是中性报告
- 没有承认不确定性或复杂感受
- 没有第一人称视角（当适当时）
- 没有幽默、没有边缘、没有个性
- 读起来像维基百科文章或新闻稿

#### 如何添加声音：

**有观点**。报告事实，然后对它们做出反应。"我真的不知道该怎么看待这个"比中性列出优缺点更人性化。

**变化节奏**。短而有力的句子。然后较长的句子慢慢到达目的地。混合它。

**承认复杂性**。真实人类有复杂感受。"这令人印象深刻但也有点令人不安"比"这令人印象深刻"好。

**适当时使用"I"**。第一人称读起来诚实，适合大多数散文。"我不断回到..."或"这让我..."表示真实的人在思考。

**让一些混乱进来**。完美结构感觉像算法。离题、旁白和半成形的想法是人类的。

**对感受具体**。不是"这令人担忧"，而是"代理在凌晨 3 点 churn 而没人在看，这有点令人不安。"

### 声音校准（可选）

如果用户提供了写作样本（他们自己以前的写作），在重写前分析它：

1. **先读样本**。注意：
   - 句子长度模式（短而有力？长而流畅？混合？）
   - 词选择水平（随意？学术？介于两者之间？）
   - 他们如何开始段落（直接跳入？先设置上下文？）
   - 标点习惯（很多破折号？括号旁白？分号？）
   - 任何重复短语或口头习惯
   - 他们如何处理过渡（显式连接器？只是开始下一点？）

2. **在重写中匹配他们的声音**。移除 AI 模式只是一半；也交换样本中的模式。如果他们写短句子，不要产生长句子。如果他们使用"stuff"和"things"，不要升级到"elements"和"components"。

3. **当没有提供样本时**，回退到默认行为（来自"个性和灵魂"部分的自然、变化、有观点的声音）。

### 输出格式

提供：
1. 草稿重写
2. "下面什么使它如此明显是 AI 生成的？"（简要要点）
3. 最终重写
4. 所做更改的简要总结（可选，如果有帮助）

## 五、常见陷阱汇总

### 探索阶段
- **强加结构**：不要强加阶段、大纲或文章结构
- **不捕捉**：从用户说的第一件事开始捕捉
- **覆盖文件**：永远不要覆盖；只追加

### 组装阶段
- **未落地概念**：依赖未落地概念的节拍会失去读者
- **批处理**：不要批处理；立即写入每个同意的段落或块
- **不重新读取**：每次写入前从磁盘重新读取文件

### 编辑阶段
- **不检查依赖**：确保部分及其内容的顺序尊重依赖
- **不检查过渡**：从前一段到这一段的过渡自然吗？
- **不检查节奏**：这段的长度和结构适合它的内容吗？

### 去AI化阶段
- **只移除不添加**：移除坏模式只是一半工作；重写还需要真实个性
- **不做最终通过**：问自己"什么使它如此明显是 AI 生成的？"然后再修改一次
- **不匹配声音**：如果提供了声音样本，特别匹配它

## 六、参考资源

- writing-fragments：探索阶段
- writing-beats：组装阶段（节拍旅程）
- writing-shape：组装阶段（逐段塑造）
- edit-article：编辑阶段
- humanizer：去AI化阶段（34 个模式）
