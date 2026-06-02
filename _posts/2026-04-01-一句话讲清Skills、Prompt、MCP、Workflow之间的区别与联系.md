---
layout:     post
title:      一句话讲清 Skills、Prompt、MCP、Workflow 的区别与联系
subtitle:   从 Skill 工程到 Agent 系统分层的概念梳理
date:       2026-04-01
author:     bbq
header-img: img/post-bg-control-room.jpg
catalog: true
featured: true
featured_rank: 2
tags:
    - AI
    - Agent
    - Skills
    - MCP
---

# 一句话讲清skills、prompt、mcp、workflow之间的区别与联系（详解skills）

>最近2年AI大模型一直很火，大家都在各种定义“新名词”，也是新技术迭代过程中非常关键，分厂经典的一个技术演进就是大家对于agent一词的理解
>
>Agent=[LLM](https://zhida.zhihu.com/search?content_id=268800741&content_type=Article&match_order=1&q=LLM&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njk1NzE2MDMsInEiOiJMTE0iLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoyNjg4MDA3NDEsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.MzZ6TgBFG1CjASE9RZk6qyXKlPTOf3QQVIlrdLIOTTg&zhida_source=entity) + Tools
>
>Agent=LLM + Agent框架 + Prompts+ Tools
>
>Agent=LLM + Agent框架 + RAG + Prompts+ MCP
>
>Agent = LLM + Agent框架 + Prompts + MCP + WorkFlow
>
>**Agent = LLM + Skills + MCP**
>
>**如果把Agent比喻为一台电脑：** LLM 就是 CPU，负责核心运算。Agent 框架就是操作系统，负责调度资源、运行程序。
>
>但是换句话说，最后agent也没有具体定义，我们用到的各类大模型产品已经是具备agent能力，但是底层开发时，我们尽量把他们当作一个基础泛化大模型。

# 一、一句话总结

以一次调用入手，用户输入“我想要知道xx公司近期的运营情况和未来预测”。大模型接收自然语言任务，并结合 RAG 检索长期知识（更好的识别意图、理解上下文；所以叫**检索增强**），主要就是负责“听懂用户的需求”。 Skills 作为主要承载，把“怎么做这件事”以结构化的流程与规范固化下来，相当于我们把步骤拆开了，知道“现在第一步要干啥，第二步要干啥”（其中的精髓就是渐进式披露，先看框架，而不用先开第一步的细节），由 MCP 连接数据库、业务系统、API、文件等外部资源，是智能体的“神经与四肢”，比如“怎么去用Google搜索”、“连接上知识数据库”，**Agent 是调度与大脑，Skills 是行业经验，MCP 是执行肌肉，RAG 则是一整套持续更新的长期记忆系统。**

其实本质上，这几个东西是处在系统的不同层面的，这就好像我们经常在对比RPC调用、HTTP请求一样。







# 二、什么是skills？

- **定义**：Skill 通常指的是一个“能力单元”或“功能模块”，它可以是一个独立的任务（如智能问答、文本摘要、图片识别等），也可以是模型能力的二次封装（**从“提示词工程”到“技能工程”的范式转移**）。
- 特点
  - 具备清晰的输入输出接口。
  - 可以单独调用，也可组合到更复杂流程中。
  - 在阿里云百炼平台，Skill 还可以是由 prompt、工具调用、代码等构建的智能服务。
- **应用场景**：比如“智能写作 Skill”、“法律咨询 Skill”、“图片生成 Skill”等。

![图片](https://inews.gtimg.com/news_bt/O9GdlJMoLwFhnL-TPqD8dUSZCBZl0LuiHE_wYe5jw29jYAA/641)





对比传统的prompt engineering

| 维度       | 传统 System Prompt 模式                      | Agent Skill 模式                   |
| ---------- | -------------------------------------------- | ---------------------------------- |
| 规则载体   | 纯文本，随会话发送                           | 本地结构化文件 (.md / .py)         |
| 上下文占用 | 全量占用（无论是否用到，所有规则都在窗口内） | 按需占用（仅加载被触发的技能规则） |
| 可维护性   | 极低（修改一处需测试整体影响）               | 高（模块化独立封装）               |
| 执行能力   | 仅限于文本生成                               | 原生支持脚本执行与文件操作         |

## 2.1、你必须要知道的几个skills基础概念

### **2.1.1、标准文件目录结构**

以read_doc的skill为例，在 `~/.claude/skills/read_doc` 目录下，包含以下核心文件：

> [SKILL.md](http://SKILL.md)必须放在每个 skill 的根目录下，不可放入其子目录下，否则无法被识别。对应上图举例：不可放在 scripts 文件夹下。

```text
read_doc/
├── SKILL.md                # 核心定义与编排文件
├── forms.md							 # 表单填写指南
├──reference.md   # [Reference] 外部引用的api&&合规手册（20页） —— 可选
├──scripts/
└── archive_report.py       # [Script] 自动归档脚本
```

其中最重要的就是skill.md文件，

## 2.1.2、如何编写SKILL.md文件

> SKILL.md文件中， YAML Frontmatter 的 name 和 description 字段是必须的（示例中标红的字段），模型基于 description 来决定何时使用这个 skill，如果不填写 skill 将永远无法被调用

### **1）SKILL.md 文件组成：**

- YAML Frontmatter：需要包括 name、description 两个必须字段
- content：文件内容，skill 的使用说明

### **2）SKILL.md 文件示例**

> ✅ **验证工具**：可通过 [Skill Linter](https://example.com/skill-linter) 检查文件规范性。

#### **1. 文件头（Metadata）**

```
markdown
复制---
name: "skill-name"                # 技能唯一标识（如 content-research-writer）
version: "1.0.0"                  # 语义化版本号（SemVer）
author: "Team/Individual Name"    # 开发者或团队
description: "1-2句核心功能描述"   # 简洁说明（≤100字）
tags: ["tag1", "tag2"]            # 分类标签（如 writing, research）
dependencies:                     # 依赖项（可选）
  - "python>=3.8"
  - "api:weather"
---
```

#### **2. 核心指令（Action Guide）**

```
markdown
复制## 🎯 功能说明
- **目标**：明确技能的核心用途（如“生成技术博客初稿”）。
- **输入**：用户需提供的内容（如主题、关键词）。
- **输出**：预期结果（如Markdown格式文章）。

## 📝 使用步骤
1. **步骤1**：简要说明（如“输入主题关键词”）。
2. **步骤2**：进阶操作（如“选择文章风格”）。
3. **步骤3**：高级选项（如“启用参考文献生成”）。

> 💡 **提示**：复杂功能默认隐藏，需用户主动触发（如输入“/advanced”）。
```

#### **3. 资源引用（Resources）**

```
markdown
复制## 🛠️ 资源依赖
| 类型       | 路径/说明                  | 是否必需 |
|------------|---------------------------|----------|
| Python脚本 | `resources/research.py`   | 是       |
| API接口    | `resources/api/search.yml`| 否       |
| 模板库     | `resources/templates/`    | 是       |

## 🔗 外部工具
- **MCP协议**：`mcp://tool-name`（如 `mcp://weather-api`）。
- **知识库**：`kb://domain-data`（如 `kb://legal-terms`）。
```

#### **4. 交互逻辑（Workflow）**

```
markdown
复制## ⚙️ 执行流程
```mermaid
graph TD
    A[用户输入主题] --> B{是否需要高级功能?}
    B -->|否| C[调用基础模板]
    B -->|是| D[加载API/高级脚本]
    C --> E[生成初稿]
    D --> E
```

#### **5. 示例与用例（Examples）**

```
markdown
复制## 📌 示例
### 基础用例
**输入**：`写一篇关于AI的科普文章`  
**输出**：生成3段式科普文（含比喻解释）。

### 高级用例
**输入**：`分析AI领域最新论文并生成综述`  
**输出**：调用`research.py`，输出带参考文献的综述。
```

#### **6. 附录（Appendix）**

```
markdown
复制## 📚 附录
- **常见问题**：  
  Q: 如何启用学术模式？  
  A: 在输入中添加 `--academic` 参数。
- **版本历史**：  
  v1.0.0: 初始版本（基础写作功能）。  
  v1.0.1: 新增社交媒体分析API。
```

## 2.1.3、claude框架下，skills的加载逻辑

可以加载哪些目录下的 skill

- 项目目录下的： .claude/skills、.cursor/skills 、.catpaw/skills、 .codex/skills、.agents/skills文件夹
- 用户个人目录下的： ～/.claude/skills、 ～/.cursor/skills、 ～/.catpaw/skills、 ～/.codex/skills、～/.agents/skills 文件夹

## 2.1.4、快速体验

推荐：openskills 开源工具

使用教程：https://zhuanlan.zhihu.com/p/1979612552804713312

开源地址：https://github.com/numman-ali/openskills




# 三、skills的厉害之处

**Skills 是能力单元**，可以嵌入 Workflow，也可用 Prompt 构建。

## 3.1、渐进式披露的魅力

- skill 采用渐进式读取的方式，模型决定调用 skill 时先读取skill.md的内容
- 再根据skill.md中的描述按需读取其他文件或者脚本

![image-20260317203028859](/Users/mtldf/Library/Application Support/typora-user-images/image-20260317203028859.png)

L1 — description（触发层）

- 触发关键词
- 使用/跳过条件
- 双引号单行格式规则

L2 — SKILL.md 正文（工作流层）

- 工作流阶段结构（编号 + 入口/出口条件）
- 原则速查表
- L3 引用索引

L3 — references/（细节层）

- `scripts/` — 可执行脚本
- `references/` — 详细文档
- `assets/` — 输出模板

我们都知道，所谓的AI agent，都只不过是把更多的信息和在一起，给到模型，但是这里面有个问题是，都给了模型之后，可能你只是想问一个简单问题“今天天气怎么样？”，但是模型会回答一大堆不需要的信息（lost in middle问题），并且消耗大量的tokens。

skills就是解决tokens消耗量大的问题，并且更重要的是，很多复杂任务（如处理表格、批量重命名、自动化绘图），光靠提示词是搞不定的。**Skills 允许加入代码附件，极大地扩展了 AI 的能力边界**。举个例子，本来让文本大模型给你出海报，他只会告诉你每一步需要做什么，但是结合一些绘图py脚本，甚至是远程api，一个“海报制作skill”就能直接给你出一个图出来。

<font color='red'>**那如何解决tokens消耗大呢？**</font>

skills会进行按需加载（**效果**：复杂任务中可降低 **40%~60%** 的上下文占用。）：

```
第一级 - 元数据（始终加载，~100 token）
  └── YAML 里的 name + description
      → AI 靠这个判断"要不要触发这个 Skill"

第二级 - SKILL.md 主体（触发后加载，建议 < 500行）
  └── 具体执行指令、工作流、注意事项

第三级 - 附属资源（按需加载，无限容量）
  └── scripts/、reference.md、examples.md
      → 脚本可直接执行，无需加载进上下文
```

举个例子，用户输入：“帮我写一份关于区块链的行业报告。”

- **路由阶段**：Claude Code 扫描本地所有 Skill 的 `description`，匹配到 `industry-research`
- **加载阶段**：系统读取 `SKILL.md` 的正文。发现是需要“区块链”相关的技术，并且要提前配置一些查询工具信息，指明了要进行自动引用学术论文（scrpts/cite.py）、数据可视化（图表生成）（scrpts/chart.py）、查询区块链相关的新闻等方式，并且要按照‘templates/ ’目录下模板进行生成；
- **动态引用**：模型解析到指令中要求阅读 `reference.md`，此时才将这份文档加载进上下文，然后去调用具体的工具（非mcp方式的调用工具不需要上下文）（这里面其他的方法就不需要调用了，做到按需查找和加载工具）
- **推理与生成**：模型结合论文记录与热点新闻，提取数据指标生成图表信息，输出行业报告。



**合理使用渐进式披露**

```
SKILL.md 主体 < 500 行
    ↓ 复杂内容拆出去
reference.md（API 文档）
examples.md（完整样例）
scripts/（可执行脚本）
```







# PS：如何产生生产力？

这是我在接触AI之后，反复问自己的一个问题。AI确实有很多很棒的能力，但是也有很多关键问题待解决，

1、**ai跟先前低代码平台没有啥本质的区别**，都是拿以前的经验来套现在的模型？区别的是ai能看得懂用户的文字，低代码平台极其难用 ——我觉得后面软件开发会类似于现在的服装行业，工业流水线可以做出一堆相似的东西，确实也满足了用户的需求，同时保留定制的权利，后面会有明显的界限，能被ai处理的项目，不能被ai处理的项目

2、想办法减少上下文、减少无效输入，本质上都是在省 token。——如果 token 不要钱，所有东西写在一个 prompt 里，完全没问题

3、当前的ai和之前的人的工作模式没有任何区别，ai代替了人，并不代表着ai比人便宜，没有更大的生产力出现，也没有把人从繁杂的琐事中解脱；——我时常觉得我们是不是真的需要这么多ai，来代替人力，现在算力太贵，ai如果比人力性价比低，为什么还要裁员哇，裁的又不都是做ai的，很多是别的条线

4、包括现在的openclaw，也只是完成了模仿人类工作，但是他比一个专业人员强多少呢，可信多少呢？没人可以回答。机器回答错了，你可以容忍（哦，机器还不行，还要一些时间），人回答错了，你却要喷他，有点讽刺～

5、我曾经看到了一个给人赋能的场景，「产品思维+架构设计+vibe coding 」，这种模式，就是现在新型的生产力。但是实际过程中发现，这里面得有人呀！vibe coding之后，留下的屎山代码，会非常快地沦为企业的技术债务。

6、一人公司很新颖，但是并不是简单的。AI把创业变成了一个新的游戏，旧游戏比的是谁能做出来，新的游戏比的是谁值得被信任，谁能被找到，谁真正懂行，但大多数人还拿着旧的游戏策略在新的游戏里拼命，他们疯狂学AI工具，学prompt技巧，然后追某个新模型的发布，这个东西有用，但他们不是你的护城河，因为你学的会，别人也学的会，所以你真正投入需要投入时间的呢，是那些AI做不到的事，深入到一个你有积累的行业呢，然后建立自己的受众，然后跟你的用户之间建立真实的关系，而这些东西很慢，很不性感，不像一周上线一个AI产品那么有传播力，但是摄影术发明快200年了，没有人记得当年画的最像的那个画家，但是人人都记得莫奈。



这个问题会持续，直到我找到“答案”～



## 1、如何让快速地搭建自己的skills

我提供了两个方案，如果你对一个行业还没有认知，可以先从方案1直接开始搭建，如果你对一个行业又了认知并且知道如何获取关键信息，请采用方案2.

方案1: 

让 Claude Code 或其他 AI 编程工具迅速知道怎么创建一个规范的 skill，最便捷的方法就是**把官方 skills 仓库克隆到本地，然后让它先阅读**。这样无论它最开始懂不懂创建，都会通过官方仓库的 skill-creator 这个 Skill 快速学会。



方案2: 

个人实践过程中skill写作方式为：

1. 先和Agent对话，一步步引导它查某个case。在成功查到结论后，让它自己总结一个skill（总结）
2. 总结完成后，打开新的会话窗口，在不给任何提示的情况下，让它自主去查case。如果它出现错误的步骤或结论，告诉他正确的步骤和结论，并要求它修正上一步总结的skill（修正）
3. 循环步骤2，直到在不给任何提示的情况下，Agent给到正确的结论

**其它经验：**每次用Agent新总结出多个skills，需要让Agent重构一次现有的skills结构与内容，避免某个skill记录了大量探索过程中的无价值信息，同时避免单个skill文件过大导致没利用Skills“渐进式披露”的优势（摘要，保持可用&整合的平衡）

**1. Description 是灵魂**

```
# ❌ 太简单，AI 不知道何时触发
description: 查询A股金价变更信息

# ✅ 清晰说明 WHAT + WHEN
description: 诊断 金价 变更原因的工具。当用户提供A股并询问"今日金价多少"、"今日金价变动来源是什么"等问题时使用。自动同花顺、国际金价、金价新闻等。
```

**2. 核心流程要写清楚步骤（如果有分支逻辑也需要写清楚）**

```
## 核心流程

### Step 1：获取基础信息
调用同花顺MCP，获取股价信息（含股票代码值）

### Step 2：分页查热点信息
每页 100 条，查询是否有热点信息（访问量大的贴）

### Step 3：输出报告
报告的格式是xx
```



# 参考：

1. https://zhuanlan.zhihu.com/p/1993017122910119259
2. 一文带你看懂，火爆全网的Skills到底是个啥: https://news.qq.com/rain/a/20260113A01QIK00?suid=&media_id=
3. Claude悄悄更新了Skills生成器，这绝对是一次史诗级升级：https://zhuanlan.zhihu.com/p/2015069616045573047
4. 2026年AI应用技术栈：深度剖析Agent Skill“渐进式披露”架构！企业如何利用Agent Skill，为通用大模型配备精准的“岗位SOP”？：https://zhuanlan.zhihu.com/p/1992907559875654043
5. 
