---
layout:     post
title:      Workflow 之意图识别、问题分流、RAG 知识库、工具调用
subtitle:   从客服机器人到生产级 Agent Pipeline 的流程复盘
date:       2026-05-20
author:     bbq
header-img: img/post-bg-control-room.jpg
catalog: true
tags:
    - AI
    - Workflow
    - RAG
    - Agent
---

> 在这一章节里，我主要把agent初期的一些探索进行回顾。对于现在成型的agent架构，流程是人写死的，LLM 只是每个节点的执行者，不是整个流程的决策者。
>
> 注意：Workflow/Pipeline 不是落后的东西，很多生产环境反而更需要它的稳定性。

## 概览

这一阶段的workflow/Pipeline有几个典型的应用，比如客服机器人、问答系统、工单处理机器人。他的大致逻辑是这样：

```
用户输入
    ↓
【意图识别】← 判断用户想干什么
    ↓
【问题分流】← 根据意图决定去哪里处理
    ↙         ↘
【知识库问答】  【单轮工具调用】
  (RAG)          (Tool Use)
  
标准的有向无环图（DAG）
典型的 Workflow 结构
```

以一个“客户购买产品质量问题退货”场景来说明：

```
用户说："我上周买的那个蓝牙耳机质量有问题，
        想退货，但我不知道退货政策是什么，
        另外帮我查一下我的订单现在到哪了"
### 识别结果：
  意图1：complaint（投诉）     置信度 0.71
  意图2：query_policy（查政策） 置信度 0.68
  意图3：query_order（查订单）  置信度 0.82

### 路由规则（硬编码）：
  if 意图 == query_order:
      → 走订单查询流程
  elif 意图 == query_policy:
      → 走RAG知识库流程
  elif 意图 == complaint:
      → 走投诉处理流程

### RAG知识库：
假设用户单独问："退货政策是什么？"
    ↓
向量检索：在知识库里搜索"退货政策"
    ↓
找到相关文档片段：
  "本店支持7天无理由退货，
   商品需保持原包装..."
    ↓
喂给LLM生成回答

### 单轮工具调用：

查询订单状态：
    ↓
调用工具：query_order(order_id=???)

```

但是事实上，这里面的每一块都会出现问题，我会在后面的每一个章节中把这个问题串起来。



### 与Agent系统有什么差别？

首先不要觉得"不是Agent就没用"

```
你的系统更适合：
  ✅ 意图明确的场景（客服、FAQ、工单）
  ✅ 需要稳定可控的业务流程
  ✅ 不允许LLM自由发挥的场景（金融、医疗）
  ✅ 延迟要求极低的场景

真正的 Agent 更适合：
  ✅ 目标模糊、需要探索的任务
  ✅ 多步骤、需要动态决策的任务
  ✅ 允许一定错误率的场景
  ✅ 对延迟不敏感的场景
```

还是以刚刚用户想要退货的场景来对比：

```
用户："我上周买的蓝牙耳机质量有问题，
      想退货，不知道退货政策，
      帮我查一下订单到哪了"

Thought: 用户有三个需求，我逐一处理

━━━━━━━━ 处理需求1：查订单 ━━━━━━━━

Thought: 需要订单号，但用户没给，
         先查用户最近的耳机订单

Action: query_recent_orders(
          user_id="U123",
          keyword="耳机",
          time_range="最近一个月"
        )
Observation: 找到订单 ORD456，
             蓝牙耳机，购买于7天前，
             状态：已签收

━━━━━━━━ 处理需求2：查退货政策 ━━━━━━━━

Thought: 用户是质量问题退货，
         不是普通退货，要查对应政策

Action: RAG检索(
          query="质量问题退货政策",
          filter="售后类"
        )
Observation: 质量问题支持15天退货，
             需提供问题描述和照片

━━━━━━━━ 处理需求3：处理投诉 ━━━━━━━━

Thought: 用户反映质量问题，
         我应该主动帮他发起售后

Action: create_aftersale(
          order_id="ORD456",
          type="quality_issue"
        )
Observation: 售后单创建成功，
             编号 AS789

━━━━━━━━ 整合所有结果回复用户 ━━━━━━━━

"您好！我已经帮您处理好了：

 📦 订单状态：ORD456 已于7天前签收
 
 📋 退货政策：质量问题支持15天退货，
    您的订单还在退货期内 ✅
    需要您提供问题描述和照片
    
 🔧 售后进度：已为您创建售后单 AS789，
    售后人员将在24小时内联系您
    
 请问还有什么需要帮助的吗？"
```

可以看到，在这个agent流程里面，是一个“Thought ——> Action ——> Observation”反复循环的过程。**意图识别和问题分流，在 ReAct 里不再是独立的模块，而是融化在每一个 Thought 里，由 LLM 自然完成。**（严格来说，最小 Agent Loop = TAO循环 + 终止判断。）

用伪代码来表示的话，就这 10 几行逻辑，是所有 Agent 框架的核心骨架。

```
while True:
    
    # Thought：LLM 思考下一步
    thought = LLM.think(目标, 历史记录)
    
    # 终止判断
    if thought == "目标达成":
        break
    
    # Action：LLM 决定调什么工具
    action, params = LLM.decide_action(thought)
    
    # Observation：执行工具，拿到结果
    observation = tools[action](params)
    
    # 把结果加入历史，进入下一轮
    历史记录.append(thought, action, observation)

输出最终结果
```





所以我们发现其实是Agent也无非是对这里面的一些东西的“吸收”（意图识别&分流，需要交给LLM本身）和“改进”（工具调用需要变成多轮、RAG需要多轮，并且要沉淀出memory）

```
┌─────────────────────────────────────┐
│           Agent 系统                 │
│                                     │
│  ┌─────────────────────────────┐    │
│  │      LLM 规划和决策层         │    │
│  │  （取代了硬编码的分流逻辑）    │    │
│  └──────────────┬──────────────┘    │
│                 ↓                   │
│  ┌──────────────────────────────┐   │
│  │         工具箱                │   │
│  │  ┌──────┐ ┌──────┐ ┌──────┐ │   │
│  │  │ RAG  │ │工具  │ │代码  │ │   │
│  │  │知识库│ │调用  │ │执行  │ │   │
│  │  └──────┘ └──────┘ └──────┘ │   │
│  │      ↑          ↑            │   │
│  │  你学的RAG   你学的工具调用   │   │
│  └──────────────────────────────┘   │
│                                     │
│  意图识别和分流 → 融入LLM理解能力里   │
└─────────────────────────────────────┘
```

其实有了这个雏形之后，我们只要牢记**Agent Loop中的（Perceive、plan、action、memory、Observe）**就可以很明确改进点有哪些：

```
现在的系统：
  意图识别 + 问题分流 + RAG + 单轮工具调用
  
加上第一步 → 初级 Agent：
  把"单轮工具调用"升级为"多轮工具调用"
  加入 ReAct 循环（思考→行动→观察→再思考）—— 终止条件‌（如最大轮次、任务成功标志、人工干预）

加上第二步 → 中级 Agent：
  加入任务规划能力（把大目标拆成小步骤）
  加入记忆模块（跨轮次记住上下文）

加上第三步 → 高级 Agent：
  加入自我反思（评估结果质量）
  加入多 Agent 协作（复杂任务分给不同Agent）
```





## 意图识别

我们先看目前系统的问题，

```
用户说："我上周买的那个蓝牙耳机质量有问题，
        想退货，但我不知道退货政策是什么，
        另外帮我查一下我的订单现在到哪了"
        
识别结果：
  意图1：complaint（投诉）     置信度 0.71
  意图2：query_policy（查政策） 置信度 0.68
  意图3：query_order（查订单）  置信度 0.82

系统取最高置信度：query_order ← 选这个

❌ 用户实际上有三个意图：
   1. 反馈质量问题
   2. 想了解退货政策
   3. 查订单物流

但系统只能处理一个意图！
另外两个需求被直接丢弃了
```

核心运作方式

- **需求映射**：扮演 AI 世界的“总机接线员”，将人类的自然语言转化为机器可执行的标签（例如，将“帮我查下明天的天气”映射为 `查询天气` 意图）。
- **参数提取**：除了识别主动作，还会提取关键信息（如“明天”、“巴黎”），以便系统执行具体操作。

随着技术的进步，技术演化是一个过程，但是有几个原则：

1. 按业务动作划分，不按用户说法划分；意图粒度太粗/太细是实现过程中必须要注意的
2. 多级意图/隐含意图，因为用户的输入会可能会含有混合语义。



0. **规则匹配**

```
规则：
  包含"退货" OR "退款" → apply_refund
  包含"坏了" OR "质量" → quality_complaint
  包含"查" OR "订单"   → query_order

问题显而易见：
  "我不想退货了" → 误识别为 apply_refund ❌
  "这个东西真的太好用了不想退" → 也误识别 ❌
  规则越写越多，越来越脆
```



1. **Embedding 相似度匹配**

```
原理：
  每个意图准备几个示例句子
  把示例句子向量化存起来
  用户输入向量化后
  找最相似的示例句子对应的意图

适合场景：
  ✅ 意图数量较少（<50个）
  ✅ 有少量示例数据（每个意图5-10条）
  ✅ 需要快速响应

缺点：
  ❌ 示例覆盖不全时容易出错
  ❌ 语义相似但意图不同时容易混淆
```

因此，这个时候需要再上一个轻量的模型，

2. **传统分类模型**

```
原理：
  收集大量标注数据
  训练一个文本分类模型
  输入文本 → 输出意图类别

适合场景：
  ✅ 意图固定不变
  ✅ 有大量标注数据（每个意图>500条）
  ✅ 对延迟要求极高（<100ms）
  ✅ 不想依赖外部LLM API

缺点：
  ❌ 冷启动难（没数据怎么训练？）——这一点还好
  ❌ 新增意图需要重新训练 —— 这一点麻烦
  ❌ 对表达变化敏感
  
进化方法：文本分类模型（BERT时代）

原理：
  收集大量标注数据：
    "退货" → apply_refund
    "坏了" → quality_complaint
    ...
  
  训练模型学习：
    文本的语义特征
    和意图类别之间的关系
  
  本质：
    把文本映射成向量
    在向量空间里找最近的意图类别
```

随着LLM的出现，它有别于之前的BERT模型，他突出一个Large！

**在预训练阶段，已经做了大量的样本学习**，LLM 不是被教会了"意图识别"这件事，而是在学"预测下一个词"的过程中，把意图理解作为副产品学会了。

```
理解词义
  "退货" "退款" "不要了" 是相似的概念

理解句子结构
  "我想退货" 和 "退货我想办" 意思一样

理解语境
  "苹果" 在不同语境下意思不同

理解意图
  "这个东西坏了" 后面大概率跟着
  "想退货" "找售后" "要换货"
  → 模型学到了：坏了 → 投诉/退货 的关联
```

另外一部分就是Attention机制，特别是多头注意力机制，能够保证同时关注多个维度的数据，

```
处理一句话时，每个词应该"关注"哪些其他词？

例子：
"我上周买的那个蓝牙耳机质量有问题想退货"

处理"退货"这个词时：
  关注"质量有问题"  → 权重高 ✅
  关注"蓝牙耳机"    → 权重高 ✅
  关注"上周"        → 权重中 
  关注"我"          → 权重低
  关注"那个"        → 权重极低

模型通过这个权重分布
理解了"退货"和"质量问题"强相关
```

这两个能力和在一起之后，

```
用户输入：
"耳机戴了没两天耳朵都疼了"

━━━━━━ Attention 在做什么 ━━━━━━

第一步：每个词关注其他词

"耳朵疼" 关注 "耳机"
  → 建立关联：耳机 导致 耳朵疼

"没两天" 关注 "戴了"
  → 建立关联：使用时间很短

━━━━━━ 多层叠加 ━━━━━━

第一层 Attention：
  建立词与词的直接关联

第二层 Attention：
  在第一层基础上建立更高层语义
  "耳机 + 耳朵疼 + 没两天"
  → 产品使用体验差

第三层 Attention：
  更高层的语义抽象
  "产品使用体验差"
  → 用户可能在投诉

...（LLM 有几十层）

最后一层输出：
  这句话的语义向量
  在意图空间里
  最接近 quality_complaint
```

所以，<font color='red'>现阶段看，意图识别这一块再去训练一个本地的小模型价值不大（除非是特别要求低延时的问题）</font>，从成本考量的时候做无限分层套娃其实也没必要。



### 在Agent中怎么做意图识别？

在AI Agent的设计中，意图识别是自然语言理解（NLU）的核心环节，直接影响用户体验和业务目标达成。作为AI产品经理，需从**业务场景（产品问题）、技术实现（算法问题）和用户体验（流程问题）**三个维度系统设计意图识别方案

1. **明确业务需求与意图分类体系**

- 场景拆解：根据Agent的应用场景（如客服、智能家居、电商导购）梳理高频用户诉求。例如：
- 客服场景：咨询、投诉、退款、查询进度等
- 智能音箱：播放音乐、设置闹钟、控制设备
- 意图分层设计：采用树状结构（主意图→子意图→槽位），避免分类粒度混乱。例如：
  主意图：订机票
  ├─子意图：查询航班（槽位：出发地、目的地、日期）
  └─子意图：改签机票（槽位：订单号、新日期）
- 兜底策略：设计&quot;未知意图&quot;分类，结合澄清话术（如“您是想查询订单还是联系客服？”）或转人工流程。

2. **数据驱动的模型构建**

- 数据采集与标注：
  - 通过用户历史对话、搜索日志等获取真实语料。
  - 标注时需注意同义表达覆盖（如“帮我订票”和“买张去北京的机票”）。
- 技术选型方案：
  - 规则引擎（正则表达式、关键词）：冷启动阶段/高确定性场景（如命令词）
  - 深度学习（BERT、TextCNN）：复杂语义场景
  - 大模型微调（Few-shot Learning）：长尾意图识别
  - 多模型融合：规则兜底+模型预测，例如先用规则处理高频意图，剩余流量走模型。

3. **用户体验闭环设计**
- 容错机制：
   - 置信度阈值设置（如低于0.7时触发澄清）
   - 上下文继承（用户说“换一个时间”时继承前文航班查询意图）

- 效果评估指标：
   - 技术指标：准确率、召回率、F1值
   - 业务指标：任务完成率、转人工率、单次对话解决率（FCR）
   - 用户感知：用户主动纠正次数、满意度调研

- 持续迭代闭环：
   - 建立bad case分析流程，将误识别样本反馈至标注池
   - 监控意图分布变化（如新增促销活动可能引发未覆盖的咨询意图）




其实现在LLM的能力已经挺不错的了，现在要做的就是工程来做保驾护航。

#### Prompt 设计是核心

```
一个完整的意图识别 Prompt：

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
系统提示：
你是一个意图识别专家。
根据用户输入，从以下意图列表中识别用户意图。

意图列表：
- query_order_status：用户想查询订单的当前状态、物流信息
- apply_refund：用户想申请退款或退货
- quality_complaint：用户反馈商品质量有问题
- product_inquiry：用户咨询商品信息、规格、价格
- chitchat：闲聊，与业务无关
- unclear：用户表达不清楚，无法判断意图

规则：
1. 可以识别多个意图，按置信度排序
2. 置信度用0-1之间的小数表示
3. 如果无法判断，返回unclear
4. 只返回JSON，不要解释

返回格式：
{
  "intents": [
    {"intent": "意图名", "confidence": 0.95},
    {"intent": "意图名", "confidence": 0.60}
  ],
  "entities": {
    "order_id": "ORD123",
    "product": "耳机"
  }
}

用户输入：{user_input}
```

伪代码实现：

```
import json
from openai import OpenAI

client = OpenAI()

# 意图列表配置（单独维护，方便修改）
INTENT_LIST = [
    {
        "name": "query_order_status",
        "description": "用户想查询订单状态、物流信息"
    },
    {
        "name": "apply_refund", 
        "description": "用户想申请退款或退货"
    },
    {
        "name": "quality_complaint",
        "description": "用户反馈商品质量有问题"
    },
    {
        "name": "chitchat",
        "description": "闲聊，与业务无关"
    },
    {
        "name": "unclear",
        "description": "用户表达不清楚"
    }
]

def build_intent_prompt(user_input: str) -> str:
    intent_desc = "\n".join([
        f"- {i['name']}：{i['description']}" 
        for i in INTENT_LIST
    ])
    
    return f"""
{% raw %}
你是意图识别专家，从以下意图中识别用户意图：

{intent_desc}

用户输入：{user_input}

返回JSON格式：
{{
  "intents": [
    {{"intent": "意图名", "confidence": 0.95}}
  ],
  "entities": {{}}
}}
只返回JSON，不要其他内容。
{% endraw %}
"""

def recognize_intent(user_input: str) -> dict:
    response = client.chat.completions.create(
        model="gpt-4",
        messages=[
            {"role": "user", "content": build_intent_prompt(user_input)}
        ],
        temperature=0  # 设为0，让结果更稳定
    )
    
    result = json.loads(response.choices[0].message.content)
    return result

# 测试
test_inputs = [
    "我上周买的耳机坏了想退货",
    "订单ORD123现在到哪了",
    "你好啊",
    "那个...我想..."
]

for text in test_inputs:
    result = recognize_intent(text)
    print(f"输入：{text}")
    print(f"结果：{result}")
    print("---")
```

##### 工程上必须处理的问题

##### 问题1：置信度阈值（这里要确定好，一般设置在0.75之上）

```
def handle_intent_result(result: dict) -> str:
    intents = result["intents"]
    top_intent = intents[0]
    
    # 置信度太低，反问用户
    if top_intent["confidence"] < 0.6:
        return "clarify"  → 触发反问流程
    
    # 多个意图置信度都很高，需要并行处理
    high_conf_intents = [
        i for i in intents 
        if i["confidence"] > 0.75
    ]
    if len(high_conf_intents) > 1:
        return "multi"    → 触发多意图处理
    
    # 正常情况
    return top_intent["intent"]
```

##### 问题2:要做好反问流程（对话状态、槽位信息）

反问的核心难点不是不是"怎么问"的问题，而是"问完之后怎么办"的问题。

```
用户回答了反问之后：
  这个回答 和 之前的上下文 怎么关联？
  系统怎么知道用户在回答哪个问题？
  怎么把补充信息和原始意图合并？
```

关键点：**"用 Dialogue State（对话状态）记住问了什么、缺什么、已知什么"，用户的每一个回答都是在填充状态里的空缺，上下文关联靠的是 pending_slot 这个锚点。**

```
每一轮对话，系统都维护一个"状态"（Dialogue State）：

{
  "current_intent": "apply_refund",    // 当前意图
  "missing_slots": ["order_id"],       // 还缺什么信息
  "filled_slots": {                    // 已经有什么信息
    "product": "耳机"
  },
  "pending_question": "请问您的订单号是多少？",  // 当前在问什么
  "dialogue_history": [...]            // 完整对话历史
}
```

用户的每一个回答，都是在填充这个状态里缺失的槽位（Slot）。即便是面临两种不同的场景：

1. 用户的意图很清晰，但是不知道下一步的信息；比如，用户说“我要退货”，系统的槽位信息应该是“miss_slots = orderid”
2. 用户意图不清晰，需要进一步确认信息；比如，用户说“那个东西有问题”，系统槽位应该是“pending_slot = "ntent_clarification”，用户回复“退货”，再更新“current_intent=apply_refund”



##### 问题2.2 反问的结果要融合到之前的上下文中（ **不要把Agent 的工具化和状态管理混为一谈**）

伪代码实现：

```python
from langchain.tools import tool
from langchain.memory import ConversationBufferMemory
from dataclasses import dataclass, field
from typing import Optional
import json

# ━━━━━━ State 依然需要 ━━━━━━

@dataclass
class DialogueState:
    current_intent: Optional[str] = None
    confidence: float = 0.0
    filled_slots: dict = field(default_factory=dict)
    missing_slots: list = field(default_factory=list)
    pending_slot: Optional[str] = None
    pending_question: Optional[str] = None

# 全局状态（实际项目里按 session_id 隔离）
state = DialogueState()

# ━━━━━━ @tool 里要读写 State ━━━━━━

@tool
def recognize_intent(user_input: str) -> str:
    """
    识别用户输入的意图和实体。
    每次收到用户新消息时调用。
    如果当前有 pending_slot，说明用户在回答上一个问题，
    应该调用 fill_slot 而不是这个工具。
    """
    # 把当前state信息带入，帮助LLM理解上下文
    context = f"""
{% raw %}
    当前对话状态：
    - 已识别意图：{state.current_intent or '无'}
    - 已收集信息：{json.dumps(state.filled_slots, ensure_ascii=False)}
    - 等待回答：{state.pending_question or '无'}
    
    用户输入：{user_input}
    
    识别意图，返回JSON：
    {{
      "intents": [{{"intent": "xxx", "confidence": 0.95}}],
      "entities": {{"key": "value"}}
    }}
{% endraw %}
    """
    
    result = call_llm(context)
    
    # 更新 state
    top_intent = result["intents"][0]
    if top_intent["confidence"] > 0.6:
        state.current_intent = top_intent["intent"]
        state.confidence = top_intent["confidence"]
    
    # 把识别到的实体填入槽位
    for key, value in result.get("entities", {}).items():
        state.filled_slots[key] = value
    
    return json.dumps(result, ensure_ascii=False)


@tool
def fill_slot(user_answer: str) -> str:
    """
    当系统已经问了用户一个问题（pending_slot不为空），
    用户回答后调用这个工具填充槽位信息。
    不要在没有 pending_slot 的情况下调用。
    
    Args:
        user_answer: 用户对上一个问题的回答
    """
    if not state.pending_slot:
        return "当前没有等待填充的槽位"
    
    # 用 LLM 从回答中提取槽位值
    # 关键：把 pending_question 带进去，告诉LLM在问什么
    prompt = f"""
{% raw %}
    系统问了用户："{state.pending_question}"
    用户回答了："{user_answer}"
    
    请从回答中提取"{state.pending_slot}"的值。
    返回JSON：{{"value": "提取的值", "success": true/false}}
{% endraw %}
    """
    
    result = call_llm(prompt)
    
    if result["success"]:
        # 填入槽位
        slot_name = state.pending_slot
        state.filled_slots[slot_name] = result["value"]
        
        # 清除等待状态
        state.pending_slot = None
        state.pending_question = None
        
        return f"已获取{slot_name}：{result['value']}"
    else:
        return f"未能从回答中获取{state.pending_slot}，需要重新询问"


@tool
def check_slots() -> str:
    """
    检查当前意图所需的槽位是否都已填充。
    在意图明确之后调用，判断是否可以执行业务逻辑。
    """
    if not state.current_intent:
        return "意图未确定，无法检查槽位"
    
    required_slots = INTENT_SLOTS[state.current_intent]["required"]
    
    missing = [
        slot for slot in required_slots
        if slot not in state.filled_slots
    ]
    
    state.missing_slots = missing
    
    if missing:
        return json.dumps({
            "complete": False,
            "missing": missing,
            "filled": state.filled_slots
        }, ensure_ascii=False)
    else:
        return json.dumps({
            "complete": True,
            "filled": state.filled_slots
        }, ensure_ascii=False)


@tool
def ask_for_slot(slot_name: str) -> str:
    """
    向用户询问缺失的槽位信息。
    在 check_slots 返回 complete=False 之后调用。
    调用后系统进入等待状态，下一轮用户回答要用 fill_slot 处理。
    
    Args:
        slot_name: 需要询问的槽位名称
    """
    questions = {
        "order_id": "请问您的订单号是多少？",
        "reason": "请问退货原因是什么？",
        "problem_description": "请描述一下具体是什么问题？"
    }
    
    question = questions.get(slot_name, f"请提供{slot_name}的信息")
    
    # 关键：记录等待状态
    state.pending_slot = slot_name
    state.pending_question = question
    
    return question


@tool
def execute_intent() -> str:
    """
    执行当前意图的业务逻辑。
    只在 check_slots 返回 complete=True 之后调用。
    """
    intent = state.current_intent
    slots = state.filled_slots
    
    if intent == "apply_refund":
        result = create_refund_request(slots["order_id"])
    elif intent == "query_order_status":
        result = query_order(slots["order_id"])
    elif intent == "quality_complaint":
        result = create_complaint(
            slots["order_id"],
            slots["problem_description"]
        )
    
    # 执行完清空状态，准备下一轮
    state.__init__()
    
    return result
    
    
    
## 接入Agent Loop之后
# ━━━━━━ 1. 初始化 LLM ━━━━━━

llm = ChatOpenAI(
    model="gpt-4",
    temperature=0  # 意图识别场景要稳定，设0
)

# ━━━━━━ 2. 注册工具 ━━━━━━

tools = [
    recognize_intent,
    fill_slot,
    check_slots,
    ask_for_slot,
    execute_intent
]

# ━━━━━━ 3. 定义 Prompt ━━━━━━

prompt = PromptTemplate.from_template("""
你是一个智能客服助手。

你有以下工具可以使用：
{tools}

工具调用规则：
1. 收到用户新消息：
   - 如果 pending_slot 不为空 → 调用 fill_slot
   - 如果 pending_slot 为空   → 调用 recognize_intent

2. 意图识别后：
   → 调用 check_slots 检查信息是否完整

3. check_slots 返回 complete=false：
   → 调用 ask_for_slot 询问缺失信息

4. check_slots 返回 complete=true：
   → 调用 execute_intent 执行业务

对话历史：
{chat_history}

用户输入：{input}

思考过程：
{agent_scratchpad}
""")

# ━━━━━━ 4. 初始化记忆 ━━━━━━

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# ━━━━━━ 5. 创建 Agent ━━━━━━

agent = create_react_agent(
    llm=llm,
    tools=tools,
    prompt=prompt
)

# ━━━━━━ 6. 创建执行器 ━━━━━━

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,           # 自动维护对话历史
    verbose=True,            # 打印 ReAct 过程，调试用
    max_iterations=10,       # 最多循环10次，防止死循环
    handle_parsing_errors=True  # 解析出错时不崩溃
)
```



这里有几个设计原则：

1. State 和 @tool 职责分离   State  → 记住"现在在哪"  @tool  → 定义"能做什么"  两者缺一不可 
2. 工具的 docstring 要说清楚"什么时候用" 
   1. 错误：只说工具做什么  "填充槽位信息"    
   2. 正确：说清楚调用时机  "当系统已经问了用户问题（pending_slot不为空），   用户回答后调用这个工具"   ，LLM 靠 docstring 决定调不调这个工具
3. State 信息要注入到工具里   LLM 看不到 state 对象  必须在工具的输入或输出里  把关键状态信息暴露给 LLM



`@tool` 做了三件事：（这里我不展开，在后面的tool scheme中会涉及）

```
markdown
复制1. 把函数包装成 LLM 可以调用的工具

2. 把函数的：
   - 名称
   - docstring（描述）
   - 参数类型
   自动转成 Tool Schema

3. 注册进工具箱
   LLM 可以自己决定要不要调用它
```





##### 问题3：意图识别结果异常兜底

```
def recognize_intent_safe(user_input: str) -> dict:
    try:
        result = recognize_intent(user_input)
        
        # 校验返回格式
        assert "intents" in result
        assert len(result["intents"]) > 0
        
        return result
        
    except json.JSONDecodeError:
        # LLM 没有返回合法JSON
        return {
            "intents": [{"intent": "unclear", "confidence": 1.0}],
            "entities": {}
        }
    except Exception as e:
        # 其他异常，走兜底
        return {
            "intents": [{"intent": "unclear", "confidence": 1.0}],
            "entities": {}
        }
```

##### 问题4:评估意图识别的质量

```
准备测试集：
  收集100条真实用户输入（注意，一定要真实并且随机）
  人工标注正确意图
  
评估指标：

准确率（最重要）：
  预测正确的数量 / 总数量
  目标：> 85%

各意图的召回率：
  某意图被正确识别的比例
  如果某个意图召回率很低
  → 说明这个意图的描述不够清晰
  → 或者需要加 Few-shot 示例

混淆矩阵：
  看哪两个意图经常被混淆
  → 说明这两个意图定义有重叠
  → 需要重新设计意图体系
```



## 问题分流

我们先看目前系统的问题，

```
意图 = query_order
    ↓
路由规则（硬编码）：
  if 意图 == query_order:
      → 走订单查询流程
  elif 意图 == query_policy:
      → 走RAG知识库流程
  elif 意图 == complaint:
      → 走投诉处理流程
      
❌ 分流逻辑是写死的
   现实中用户的问题千变万化：

   "我的耳机坏了想退货" 
   → 这是 complaint？还是 refund_request？
   
   "退货要多久能到账"
   → 这是 query_policy？还是 query_refund_status？

   边界模糊的问题，分流经常出错
   而且每出现一种新情况，就要改代码
```

既然规则很不灵活，我们现在就直接介绍LLM下的问题分流吧。其实意图识别只是帮我们把用户的问题简化了，最后还是要靠流程来进行路由。

```
场景判断：

意图固定 + 逻辑简单
    → 硬编码路由

意图复杂 + 边界模糊
    → LLM 动态路由

需要快速响应 + 路由类别少
    → 向量路由

生产环境推荐：
    硬编码 + LLM 兜底
    ↓
    先走硬编码路由表
    匹配不到 → 走 LLM 动态路由
    两层保障，既快又准
```

这里很多人说得太复杂了，完全没必要，比如下面这个：

```
维度1：按意图分（最常见）
意图 = query_policy    → 走 RAG 知识库
意图 = apply_refund    → 走工具调用
意图 = chitchat        → 走闲聊模块
意图 = unclear         → 走反问流程
维度2：按问题复杂度分
简单问题（单步能解决）：
  "退货政策是什么" → 直接查知识库，一步出结果

复杂问题（多步才能解决）：
  "帮我分析最近的退货原因并生成报告"
  → 需要查数据 + 分析 + 生成
  → 走 Agent 多步处理流程
  
维度3：按数据来源分
需要静态知识    → RAG 知识库
需要实时数据    → 工具调用（API/数据库）
需要推理计算    → LLM 直接处理
需要人工介入    → 转人工客服
```



硬编码路由 ——> 向量路由 ——> LLM 动态路由.

#### LLM动态路由

我们主要讨论LLM动态路由，这里面有很多比较难的工程问题：

```
表面上的难题：          真正难的：

多意图分流              ← 难，但有解
意图和路由不是一对一    ← 难，而且容易忽视
分流失败的兜底          ← 不难，但容易做错
上下文影响分流          ← 很难，大多数系统没做
分流结果的置信度        ← 难，而且影响整个链路

四个难题的本质：

多意图分流          → 不是选一个，是处理所有
同一意图不同路由    → 路由不只看意图，还看状态
上下文影响分流      → 同一句话，上下文不同结果不同
分流置信度          → 不确定时不要硬分，先确认
```

但是核心就一个点，<font color='red'>五个难题能合并解决，工程技巧是：用 DecisionContext 统一收集所有决策信息，用 unified_route 一次性做综合决策，把"分散的五个判断"变成"一次完整的决策"。</font>(有点类似于plan模式)

本质上是把"信息收集"和"决策"分离，信息收集尽量并行，决策尽量一次完成，这样既减少了 LLM 调用次数，又避免了信息孤岛导致的决策偏差。

#### 1.统一决策上下文（Unified Decision Context）

> 核心思想：**把分流决策需要的所有信息，统一收集成一个结构，一次性交给 LLM 做决策。**

```
@dataclass
class DecisionContext:
    """
    分流决策所需的全部信息
    一次性收集，一次性决策
    """
    
    # ━━━━ 来自意图识别 ━━━━
    intents: list           # 所有意图及置信度
    entities: dict          # 提取的实体
    
    # ━━━━ 来自对话状态 ━━━━
    filled_slots: dict      # 已有信息
    missing_slots: list     # 缺少信息
    pending_task: dict      # 未完成任务
    attempt_count: int      # 失败次数
    
    # ━━━━ 来自对话历史 ━━━━
    last_route: str         # 上一次路由
    last_intent: str        # 上一次意图
    recent_history: list    # 最近3轮对话
    
    # ━━━━ 来自业务规则 ━━━━
    business_rules: dict    # 业务规则检查结果
    user_profile: dict      # 用户画像
    
    # ━━━━ 来自情感分析 ━━━━
    emotion: str            # 用户情绪
    emotion_trend: str      # 情绪趋势（平稳/恶化）
```

收集DecisionContext：

```
def build_decision_context(
    user_input: str,
    state: DialogueState,
    history: list
) -> DecisionContext:
    """
    在调用 unified_route 之前
    统一收集所有决策所需信息
    """
    
    # 并行收集，减少延迟
    import asyncio
    
    async def collect_all():
        
        # 同时执行这些收集任务
        intent_task = asyncio.create_task(
            recognize_intent_async(user_input, history)
        )
        emotion_task = asyncio.create_task(
            detect_emotion_async(user_input, history)
        )
        rules_task = asyncio.create_task(
            check_business_rules_async(state.filled_slots)
        )
        
        # 等待全部完成
        intent_result, emotion_result, rules_result = \
            await asyncio.gather(
                intent_task,
                emotion_task,
                rules_task
            )
        
        return intent_result, emotion_result, rules_result
    
    intent_result, emotion, rules = asyncio.run(collect_all())
    
    # 组装 DecisionContext
    return DecisionContext(
        # 意图信息
        intents=intent_result["intents"],
        entities=intent_result["entities"],
        
        # 状态信息
        filled_slots=state.filled_slots,
        missing_slots=state.missing_slots,
        pending_task=state.pending_task,
        attempt_count=state.attempt_count,
        
        # 历史信息
        last_route=state.last_route,
        last_intent=state.last_intent,
        recent_history=history[-3:],
        
        # 业务规则
        business_rules=rules,
        user_profile=get_user_profile(state.user_id),
        
        # 情感信息
        emotion=emotion["current"],
        emotion_trend=emotion["trend"]
    )
```

#### 3. 一个加速方案：决策缓存（适用于重复问题较多的场景）

```
# 相似的决策上下文，缓存结果
# 避免重复调用 LLM 做相同的决策

decision_cache = {}

def get_cache_key(context: DecisionContext) -> str:
    """
    把关键决策因素做成缓存key
    不需要完全一样，关键因素一样就能复用
    """
    key_factors = {
        "top_intent": context.intents[0]["intent"],
        "confidence_level": (
            "high" if context.intents[0]["confidence"] > 0.85
            else "medium" if context.intents[0]["confidence"] > 0.6
            else "low"
        ),
        "has_slots": bool(context.filled_slots),
        "emotion": context.emotion,
        "attempt_count": min(context.attempt_count, 3)
    }
    
    return hashlib.md5(
        json.dumps(key_factors, sort_keys=True).encode()
    ).hexdigest()


def unified_route_with_cache(
    user_input: str,
    context: DecisionContext
) -> str:
    
    cache_key = get_cache_key(context)
    
    # 命中缓存，直接返回
    if cache_key in decision_cache:
        return decision_cache[cache_key]
    
    # 未命中，调用 LLM 决策
    result = unified_route(user_input, context)
    
    # 存入缓存
    decision_cache[cache_key] = result
    
    return result
```



#### 2. 一个路由解决五个问题（并行执行、兜底、上下文影响、置信度计算、多意图拆解）

```
@tool
def unified_route(
    user_input: str,
    context: DecisionContext
) -> str:
    """
    统一分流决策工具。
    整合所有信息，一次性解决：
    - 多意图分流
    - 意图路由不一对一
    - 分流失败兜底
    - 上下文影响
    - 置信度处理
    
    Args:
        user_input: 用户当前输入
        context: 统一决策上下文（包含所有信息）
    """
    
    prompt = f"""
    你是分流决策专家，根据以下完整信息做出路由决策。

    ━━━━ 当前输入 ━━━━
    用户说：{user_input}
    
    ━━━━ 意图识别结果 ━━━━
    识别到的意图：{context.intents}
    提取的实体：{context.entities}
    
    ━━━━ 当前状态 ━━━━
    已有信息：{context.filled_slots}
    缺少信息：{context.missing_slots}
    未完成任务：{context.pending_task}
    失败次数：{context.attempt_count}
    
    ━━━━ 对话历史 ━━━━
    上一次路由：{context.last_route}
    上一次意图：{context.last_intent}
    最近对话：{context.recent_history}
    
    ━━━━ 业务规则 ━━━━
    {context.business_rules}
    
    ━━━━ 用户状态 ━━━━
    情绪：{context.emotion}
    情绪趋势：{context.emotion_trend}
    用户画像：{context.user_profile}
    
    ━━━━ 可选路由 ━━━━
    - rag：查知识库
    - tool：调用工具执行
    - clarify：反问用户
    - direct：LLM直接回答
    - human：转人工
    - fallback：降级处理
    - parallel：并行处理多个意图
    - sequential：串行处理多个意图
    
    ━━━━ 决策规则 ━━━━
    1. 最高置信度 < 0.6 且 失败次数 < 2 → clarify
    2. 最高置信度 < 0.6 且 失败次数 >= 2 → fallback
    3. 多个意图置信度 > 0.75 → parallel 或 sequential
    4. 情绪恶化 或 失败次数 >= 3 → human
    5. 有未完成任务 且 当前输入是延续 → 继承上下文
    6. 信息不完整 → clarify
    7. 信息完整 → tool 或 rag
    
{% raw %}
    返回JSON：
    {{
      "route": "主路由",
      "strategy": "处理策略",
      "sub_routes": [],
      "inherit_context": true/false,
      "confidence": 0.95,
      "reason": "决策原因",
      "next_action": "具体下一步"
    }}
{% endraw %}
    """
    
    return call_llm(prompt)
```



所以，可以发现，**Agent 里的"分流"，本质上是 System Prompt 里的规则 + LLM 的理解能力，不是一段路由代码。**（而且这里工程可做工的地方已经相对较少了。）





## RAG知识库

我们先看目前系统的问题，

```
假设用户单独问："退货政策是什么？"
    ↓
向量检索：在知识库里搜索"退货政策"
    ↓
找到相关文档片段：
  "本店支持7天无理由退货，
   商品需保持原包装..."
    ↓
喂给LLM生成回答

❌ 问题1：检索到的内容可能不够精准

用户问的是"质量问题退货"
但检索到的是"普通退货政策"
两者政策不同！（质量问题可能支持15天）

RAG 检索的是"语义相似"
不一定是"用户真正需要的"

❌ 问题2：知识库内容过期了怎么办？

政策3个月前更新了
但知识库没有同步
LLM 用旧政策回答用户
→ 给出错误信息！

❌ 问题3：知识库没有这个信息怎么办？

用户问了一个知识库里没有的问题
RAG 检索不到相关内容
系统要么胡说，要么说"我不知道"
没有任何补救措施
```

这三个问题的本质是：

```
问题1：检索不准    → 找到了，但找错了
问题2：内容过期    → 找到了，但内容是错的
问题3：检索为空    → 根本没找到

三个问题的根源不同：
  问题1 → 检索策略的问题
  问题2 → 知识库管理的问题
  问题3 → 知识库覆盖的问题
```



##### 检索不准的优化1——多路召回 + Rerank + 元数据过滤

```
多路召回：

用户问题
    ↓
┌───────────────────────────────┐
│  路1：向量检索（语义相似）      │ → 5条结果
│  路2：BM25关键词检索           │ → 5条结果
│  路3：元数据过滤检索           │ → 5条结果
└───────────────────────────────┘
    ↓
合并去重（15条 → 10条不重复）
    ↓
Rerank精排（10条 → Top3最相关）
    ↓
喂给LLM生成回答
```

为什么现在“多路召回”/“混合检索”很流行？因为在实际业务中，你会发现：

- **全文检索**搜“工号 12345”很准，但搜“心情不好想请假”很差。
- **向量检索**搜“情绪”很准，但搜“工号”经常乱匹配。



这里我再多说一点技术相关的吧，因为最近Hermes Agent很火，他的记忆系统就是用的FTS5 全文引擎‌（*Full-Text Search 5*），本质上也是一个检索能力，FTS5 ‌**原生支持**‌ `bm25()` 函数用于排序。

###### BM25（best match 25）

BM25的核心公式：

$\text{score}(D, Q) = \sum_{i=1}^{n} \text{IDF}(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot \left(1 - b + b \cdot \frac{|D|}{\text{avgdl}}\right)}$

TF-IDF（词频-逆文档频率，Term Frequency–Inverse Document Frequency）是一种经典的统计加权技术，用于评估某个词对单篇文章或整个语料库的重要程度。它能过滤掉常见词（如“的”、“是”），并提取出真正能代表文章主题的关键词。

它由两部分相乘组成： [[1](https://medium.com/willhanchen/自然語言處理-概念篇-探索tf-idf-關於詞的統計與索引隱含著什麼奧秘呢-6e0079151d0a)]

- **词频 (Term Frequency, TF)**：衡量一个词在单篇文章中出现的频率。计算公式为：
  \($TF = \frac{\text{该词在文档中出现的次数}}{\text{文档的总词数}}$\)
  （出现次数越多，TF越高。）
- **逆文档频率 (Inverse Document Frequency, IDF)**：衡量一个词的普遍重要性。如果一个词在越少的文档中出现，说明它的类别区分能力越强，IDF值越大。计算公式大致为：
  \($IDF = \log(\frac{\text{语料库的总文档数}}{\text{包含该词的文档数} + 1})$\) [[1](https://zh.wikipedia.org/zh-tw/Tf-idf), [2](https://www.alibabacloud.com/help/tc/pai/user-guide/tf-idf)]

**核心思想**：如果一个词在一篇文章中出现频率很高，但在其他文章中出现频率很低，则该词具有极强的区分能力，其TF-IDF得分会很高。

这里BM25相对于TF-IDF有两个改进，一个是词频越高，分数无限涨；另一个是长文档天然占便宜。这些都反映在他的公式里面：

- 比如，先忽略长度部分，并且b=0（惩罚力度），词频计算为$\frac{f(q, D) \cdot (k_1 + 1)}{f(q, D) + k_1}$，当f无限大的时候，整体趋向于k+1。而在TF里面则无限大

- 再看长度部分，当b=0.75（控制长度惩罚力度（0~1，默认0.75））时，$k_1 \cdot \left(1 - b + b \cdot \frac{|D|}{\text{avgdl}}\right)$,`avgdl`为语料库平均文档长度，`|D|`为当前文档长度,**avgdl 就是 TF-IDF 没有的东西**，它让每篇文档的长度跟整个语料库的平均水平比较，而不是只跟自己比。

  - ```
    假设 avgdl = 200词，k1 = 1.2，b = 0.75
    
    文档A（200词，等于平均）：
      长度因子 = 1 - 0.75 + 0.75 × (200/200)
               = 0.25 + 0.75 = 1.0   ← 不惩罚，不奖励
    
    文档B（400词，是平均的2倍）：
      长度因子 = 1 - 0.75 + 0.75 × (400/200)
               = 0.25 + 1.5  = 1.75  ← 分母变大，得分被压低 ↓
    
    文档C（100词，是平均的一半）：
      长度因子 = 1 - 0.75 + 0.75 × (100/200)
               = 0.25 + 0.375 = 0.625 ← 分母变小，得分被提高 ↑
    ```

    

所以,最后的公式效果就变成了，$\text{score} = IDF \times \frac{\overbrace{f(q, D) \cdot (k_1 + 1)}^{\text{天花板上限}}}{f(q, D) + \underbrace{k_1 \cdot \left(1 - b + b \cdot \frac{|D|}{\text{avgdl}}\right)}_{\text{长度归一化}}}$

```
TF-IDF 的问题：TF 线性增长，词频越高分数无上限
BM25 的改进：TF 饱和曲线 + 文档长度惩罚

词频贡献曲线对比：
TF-IDF:  ████████████████████ → 线性无限增长
BM25:    ████████▓▓▒▒░░░░░░░░ → 趋于饱和上限

举例：
查询："猫"

文档A："猫今天吃了鱼"              → 出现1次 → 得分 1
文档B："猫猫猫猫猫猫猫猫猫猫..."   → 出现100次 → 得分 100

TF-IDF 认为文档B 相关性是 A 的100倍 ← 明显不合理


查询："猫"

TF-IDF：文档3得分 = 10/50 = 0.2，远高于文档1  ← 不合理！
BM25：  文档3的词频贡献趋于饱和，得分不会暴涨  ← 更合理 ✅
```



所以，**BM25 = 在一堆文档里，考虑"词出现多少次、文档多长、这个词有多稀有"三个因素，给每篇文档打分排序的算法。**

因此，

```
✅ BM25 擅长的：
   • 关键词精确匹配
   • 文档量大时依然高效
   • 不需要GPU，轻量部署
   • 可解释性强（能知道为什么这篇文档得分高）

❌ BM25 不擅长的：
   • "买车" 搜不到含"购买汽车"的文档（不懂同义词）
   • 理解句子语义和上下文
   • 多语言混合检索
```





伪代码：

```python
import jieba
from rank_bm25 import BM25Okapi

# 中文文档
documents = [
    "Python是机器学习最常用的编程语言",
    "深度学习是机器学习的一个重要分支",
    "自然语言处理需要大量的文本数据",
]

# 用 jieba 分词
tokenized_docs = [list(jieba.cut(doc)) for doc in documents]
# 分词结果示例：['Python', '是', '机器', '学习', '最', '常用', '的', ...]

bm25 = BM25Okapi(tokenized_docs)

# 查询
query = "机器学习Python"
tokenized_query = list(jieba.cut(query))

scores = bm25.get_scores(tokenized_query)
top_docs = bm25.get_top_n(tokenized_query, documents, n=2)

print("Top结果：")
for doc in top_docs:
    print(" -", doc)
```

BM25 在 RAG（检索增强生成）中常用于**召回候选文档**，再交给大模型回答。

```
from langchain_community.retrievers import BM25Retriever
from langchain_core.documents import Document

# 准备文档
docs = [
    Document(page_content="BM25是一种文本检索算法"),
    Document(page_content="Python广泛用于机器学习开发"),
    Document(page_content="深度学习需要大量GPU计算资源"),
    Document(page_content="自然语言处理是AI的重要方向"),
]

# 创建检索器，k=2 表示返回最相关的2个文档
retriever = BM25Retriever.from_documents(docs, k=2)

# 检索
results = retriever.invoke("机器学习算法")

for r in results:
    print(r.page_content)
# 输出：
# BM25是一种文本检索算法
# Python广泛用于机器学习开发
```



###### Rerank——检索增强生成（RAG）中的知识精排

检索增强生成（RAG）中的知识精排

- **痛点**：传统的向量检索（Embedding）擅长语义模糊匹配，但常出现“语义相似但无关痛痒”的问题。
- **Rerank的作用**：利用交叉编码器（Cross-Encoder）模型（如 阿里云百炼 提供的 `qwen3-rerank` 或开源的 `bge-rerank`）对召回的文档进行细粒度语义分析，筛选出真正能回答用户提问的片段。
- **效果**：大幅减少大模型的幻觉（Hallucination），提升回答的准确率
- 使用场景：
  - 首当其冲的就是，数据检索这一块本身，大的知识库需要快速定位到数据
  - 还有一个点其实是，**提升工具调用的准确率**，当Agent拥有几十甚至上百个API和工具时，大模型单次很难精准挑选出最合适的工具。
  - 在调用记忆模块时，**提升memory检索效率**，重新评估过去的对话历史、用户偏好或历史行动，将与当前Query最相关的记忆（如几天前未完成的任务状态）排在最前面。

工程上可以考虑：

```
文档量小、实时性要求高
        ↓
    Cross-Encoder（BGE-Reranker）

多路检索结果需要融合
        ↓
    RRF 先融合，再 Cross-Encoder 精排

对精度要求极高，允许慢
        ↓
    LLM Rerank（Listwise）


生产环境标准链路：
  BM25召回 + 向量召回
       ↓ RRF融合
  Cross-Encoder精排
       ↓
   返回Top5给LLM
```





###### RRF（倒排融合）

最简单的 Rerank，**不需要任何模型**，直接把多路召回的排名融合。

RRF 公式：score = Σ 1/(k + 排名)   k=60（常数）

```
场景：BM25 和向量检索各返回了一个排序列表，怎么合并？

BM25 结果：      向量检索结果：
第1名：文档A     第1名：文档C
第2名：文档B     第2名：文档A
第3名：文档C     第3名：文档D

RRF 公式：score = Σ 1/(k + 排名)   k=60（常数）

文档A：1/(60+1) + 1/(60+2) = 0.0164 + 0.0161 = 0.0325
文档C：1/(60+3) + 1/(60+1) = 0.0159 + 0.0164 = 0.0323
文档B：1/(60+2) + 0         = 0.0161
文档D：0        + 1/(60+3)  = 0.0159

最终排序：A > C > B > D
```



###### Cross-Encoder（主流方案）

这里的核心就是把“查询” + “文档”合并在一起，让模型直接计算相关性。**精度高的代价是必须实时计算，无法预存向量。**

```
┌─────────────────────────────────────────┐
│              Cross-Encoder              │
│                                         │
│  输入层：                               │
│  [CLS] 查询 [SEP] 文档 [SEP]            │
│                                         │
│  编码层：                               │
│  Transformer（BERT/RoBERTa）            │
│  → 每一层都能看到查询和文档的交互        │
│                                         │
│  输出层：                               │
│  取 [CLS] 向量 → 线性层 → 一个分数      │
└─────────────────────────────────────────┘

拼在一起（Cross-Encoder）：
  [CLS] 如何 学习 机器学习 [SEP] 机器学习 入门 路线 [SEP]
                      ↓
              Transformer 每一层
              "如何"可以注意到"入门"
              "学习"可以注意到"路线"
                      ↓
              语义交互发生在编码过程中  ← 这是关键
```



不过调用的时候倒是不需要“手搓”，直接用Qwen3-Reranker即可	

```python
from FlagEmbedding import FlagReranker

# 加载 BGE Reranker
reranker = FlagReranker("BAAI/bge-reranker-large")

query = "机器学习怎么入门"
candidates = [
    "机器学习完整路线图从入门到精通",
    "深度学习框架PyTorch使用教程",
    "Python基础语法入门",
]

# 打分
pairs = [[query, doc] for doc in candidates]
scores = reranker.compute_score(pairs)

# 排序
ranked = sorted(zip(scores, candidates), reverse=True)
for score, doc in ranked:
    print(f"{score:.3f} | {doc}")
```



###### LLM 重排序（不推荐，太慢且太贵）

```
# 把所有候选文档一起给 LLM，让它直接排序
prompt = """
查询：{query}

以下是候选文档列表：
[1] {doc1}
[2] {doc2}
[3] {doc3}

请按相关性从高到低输出文档编号，例如：[2,1,3]
"""

# 一次调用搞定，但文档多时 context 太长
```





当然，现在基础的encoder模型已经自制了rerank能力，比如：

```python
from sentence_transformers import CrossEncoder

# 加载 rerank 模型
model = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

query = "如何学习机器学习"

# BM25 召回的候选文档
candidates = [
    "机器学习入门需要学习Python和数学基础",
    "如何高效学习，番茄工作法介绍",        # 不相关
    "机器学习完整学习路线，从入门到精通",
    "深度学习和机器学习的区别",
]

# Rerank：模型同时看 query 和每篇文档，给出相关性分数
pairs = [[query, doc] for doc in candidates]
scores = model.predict(pairs)

# 按分数排序
ranked = sorted(zip(scores, candidates), reverse=True)

for score, doc in ranked:
    print(f"{score:.3f} | {doc}")
```

现在一些索引引擎在召回的时候已经会自带倒排的能力了，比如Elasticsearch，官方已经原生内置了 RRF（Reciprocal Rank Fusion，将 ES 的全文检索列表和向量搜索列表融合排序），你不需要自己写 Python 逻辑去算分，直接在查询请求中配置 `rank` 参数即可。



###### 元数据过滤

建库时，给每个文档片段加元数据标签。核心工作不是写过滤代码，而是设计好标签体系和自动打标签流程。这个类似于一个数据清洗过程，主要应对几个场景：

```
✅ 适合的场景：
  文档有明确的业务分类
  文档数量 > 1000 条
  用户问题类型相对固定
  对检索精准度要求高

⚠️ 需要注意的成本：
  初期：设计标签体系（1-2天）
  初期：批量打标签（LLM成本）
  持续：新文档入库时打标签
  持续：标签体系随业务变化维护

❌ 不适合的场景：
  文档数量很少（< 100条）
  → 直接全量检索就够了，不需要过滤
  
  文档类型非常混杂
  → 标签体系难以设计
```

**建议先跑通基础RAG，发现检索不准的规律后，再有针对性地加元数据过滤，不要一开始就过度设计。**

```python
from langchain.vectorstores import Chroma

# 文档示例
documents = [
    {
        "content": "质量问题支持15天退换货...",
        "metadata": {
            "category": "售后政策",
            "sub_category": "质量问题",  # ← 关键标签
            "version": "2024-01",
            "effective_date": "2024-01-01"
        }
    },
    {
        "content": "7天无理由退货政策...",
        "metadata": {
            "category": "售后政策",
            "sub_category": "无理由退货",  # ← 不同标签
            "version": "2024-01",
            "effective_date": "2024-01-01"
        }
    }
]

@tool
def smart_search(
    user_input: str,
    intent: str
) -> str:
    """
    完整的智能检索流程：
    推断过滤条件 → 过滤检索 → 验证结果
    
    Args:
        user_input: 用户问题
        intent: 用户意图
    """
    
    # ━━━━ 第一步：推断过滤条件 ━━━━
    filter_result = json.loads(
        infer_metadata_filter(user_input, intent)
    )
    
    filters = filter_result["filters"]
    filter_confidence = filter_result["confidence"]
    
    # ━━━━ 第二步：带过滤条件检索 ━━━━
    if filter_confidence >= 0.7:
        # 过滤条件可信，先用过滤检索
        results = search_with_metadata_filter(
            query=user_input,
            filters=filters,
            k=3
        )
        
        # 过滤后没找到，去掉过滤条件重试
        if not results:
            results = search_with_metadata_filter(
                query=user_input,
                filters=None,
                k=3
            )
    else:
        # 过滤条件不可信，直接全量检索
        results = search_with_metadata_filter(
            query=user_input,
            filters=None,
            k=3
        )
    
    if not results:
        return json.dumps({"found": False})
    
    # ━━━━ 第三步：验证结果相关性 ━━━━
    # 防止检索到的内容和问题完全不相关
    relevance_check = verify_relevance(
        query=user_input,
        results=results
    )
    
    if not relevance_check["is_relevant"]:
        return json.dumps({
            "found": False,
            "reason": "检索结果与问题不相关"
        })
    
    return json.dumps({
        "found": True,
        "results": [doc.page_content for doc in results],
        "metadata": [doc.metadata for doc in results],
        "filters_used": filters
    })
```







##### 检索不准的优化2——Query改写

这个也不是Agent后才出现的新鲜词，Query Rewriting是搜索系统、推荐系统以及RAG（检索增强生成）系统中的核心环节。其主要工作是通过一系列操作，将用户输入的原始查询转换为语义更丰富、更符合底层检索器和数据库理解的形式，从而大幅提升检索的准确率与召回率。

在agent系统中，其实就是把**用户模糊、口语化、多意图、残缺的原始问题**，换成**结构化、可检索、可执行、适配工具**的标准化查询，用来**提升任务成功率、降低幻觉、适配工具调用的**核心预处理模块。

改写目标：

- **提取核心关键词**：去掉无关话语
- **补充隐含限定词**：加上业务类别、时间范围等
- **生成多个查询版本**：从不同角度覆盖知识库

框架中的实现方式，

**LangChain**

- 可以在 `Retriever` 前加一个 `QueryTransformer`（自定义函数）
- 改写后的查询传给多个 Retriever（向量 + BM25）

**LlamaIndex**

- 提供 `QueryTransform` 接口，可以在检索前改写查询
- 支持链式改写（多次变换）

**LangGraph**

- 把改写作为一个节点，检索节点的前置步骤（几乎没用）



所以，<font color='red' size=5>千万千万不要把query rewriting当作是prompt里面做几个结构化的限制，他是一整套工程设计。</font>

主流实现方案：

1. Prompt 模板改写 设计固定指令模板，引导模型做补全、拆分、关键词提取，实现轻量级改写，适合简单Agent —— 这个是最常见的，但是效果无法做到很好（通用改写模板在专业领域召回效果下降，并且对高频问题的改写结果进行prompt缓存（Redis / SQLite））

   ```
   可直接用于 Agent 的 Query Rewriting Prompt 模板
   
   下面分通用版、检索增强版、多意图拆分版、NL2SQL 专用版，复制即可接入 Agent 框架。
   
{% raw %}
   一、通用查询改写模板（最常用）
   任务：对用户原始问题进行查询改写，用于后续检索与工具调用。
   
   改写规则：
   1. 补全省略的指代、上下文信息，语句完整清晰；
   2. 去除口语化、冗余、情绪化词汇，保留核心问题；
   3. 不增加额外假设，不篡改用户真实意图；
   4. 输出精简、关键词突出的标准化查询；
   5. 若问题模糊，生成1~2条更具体的备选查询。
   
   原始问题：{{user_query}}
   改写后查询：
   二、RAG 检索专用改写模板（向量库召回优化）
   你是检索查询优化器，目标是提升向量知识库召回精度。
   
   优化要求：
   1. 提炼核心实体、属性、约束条件；
   2. 精简句子，减少修饰词，优先保留名词、专业术语；
   3. 模糊问题拆分为多条精准子查询；
   4. 禁止泛泛表述，输出适合向量检索的短句。
   
   用户问题：{{user_query}}
   改写后的检索查询列表：
   三、多意图拆分模板（Agent 工具调用必备）
   任务：识别问题中的多个独立意图，拆分为可单独调用工具的子查询。
   
   拆分规则：
   1. 一个子查询只包含一个核心诉求；
   2. 子查询必须完整、可独立执行；
   3. 不合并不同领域、不同工具类型的问题；
   4. 按逻辑顺序输出编号列表。
   
   原始问题：{{user_query}}
   拆分后的子查询列表：
   四、带上下文记忆的改写模板（多轮对话 Agent）
   结合历史对话上下文，改写当前用户提问，消除指代歧义。
   
   历史对话：
   {{chat_history}}
   
   当前问题：{{user_query}}
   
   改写要求：
   1. 用实体全称替代“它、这个、那个、上文”等指代；
   2. 融合上下文信息，使问题可独立理解；
   3. 保持原意不变，语句简洁。
   
   改写结果：
   五、NL2SQL 前置改写模板（数据库查询）
   将用户自然语言改写为更适合生成 SQL 的结构化查询。
   
   改写规则：
   1. 明确查询对象、字段、筛选条件、排序要求；
   2. 区分统计、查询、对比等动作；
   3. 输出清晰的查询描述，便于后续转写SQL。
   
   用户问题：{{user_query}}
   结构化查询描述：
   六、极简版（适合轻量 Agent、低算力场景）
   请把下面问题改写为更清晰、适合工具调用的一句话，不要多余解释：
   {{user_query}}
{% endraw %}
   如果你需要，我可以再给一套英文版本 Prompt，或者基于这些模板写一个可直接运行的 Python 调用示例。
   ```

   

2. 微调专用改写模型 基于BERT、LLaMA等微调Query Rewrite模型，速度更快、效果更稳定，用于高性能Agent系统 —— 这个是最易搭配的（但是需要有训练数据、日志挖掘（用户原始Query + 成功调用的改写Query））

   ```
   模型选择
   轻量级：BERT、DistilBERT（适合低延迟场景）
   中文场景：Chinese-BERT-wwm、MacBERT
   大语言模型：LLaMA-2、Qwen-1.5、Mistral
   训练方法
   Seq2Seq 微调（T5、BART、mT5）
   Prefix-tuning / LoRA（低成本微调）
   损失函数：交叉熵（Cross-Entropy Loss）
   部署优化
   ONNX Runtime / TensorRT 加速推理
   模型量化（INT8 / INT4）
   ```

   

3. 检索增强式改写（RAG+Rewrite） 先粗检索获取少量上下文，再结合上下文二次改写，大幅提升专业领域查询质量 —— 这个是最工程化的（需要结合业务实体库、）

   ```
   第一阶段：粗检索（向量检索：FAISS、Milvus、Weaviate、关键词检索：BM25（Elasticsearch、Lucene）） + 简单改写（快速）
   第二阶段：结合上下文进行精细化改写 ——上下文长度控制：只保留最相关的 3~5 条（避免模型分心）
   
   工程优化
   检索结果缓存
   Embedding 模型领域化（如法律领域用 LegalBERT）
   ```

   

4. 强化学习优化 以检索召回率、任务完成率为奖励，迭代优化改写策略，适配复杂工业级Agent——这个是最难，但是也最attractive的；（但是数据采集和训练成本高，研发周期长）

   ```
   核心流程：
   用户输入 → 改写模型 → 工具调用 → 任务结果 → 奖励计算 → 策略更新
   
   工程挑战
   奖励延迟（Delayed Reward）
   数据稀缺（需要大量交互日志）
   训练不稳定（PPO 超参数敏感）
   
   奖励函数设计
   检索类奖励：改写后召回的相关文档数量 / 排名提升
   任务完成率奖励：最终任务是否成功
   用户反馈奖励：显式评分 / 隐式行为（点击、停留时间）
   
   ```

   





场景的困难场景：

1. 超长问题存在冗余、重复信息，改写过滤噪声，保留核心诉求，避免模型输入超限、注意力分散 
2. 结合历史对话进行查询改写，实现跨轮次理解，让Agent具备记忆一致性
3. 主动Agent需要预判用户潜在需求，查询改写可基于原始问题拓展衍生查询，提前获取相关信息，实现主动应答。
4. 多意图拆分粒度不合理：拆分过多导致调用成本飙升



> 美团搜索团队分享过一套[查询改写技术的探索与实践](https://tech.meituan.com/2022/02/17/exploration-and-practice-of-query-rewriting-in-meituan-search.html)。



##### 事实上，知识库最后也会变成一个@Tool —— **RAG 工具内部完成检索+生成**

先建立知识库：

```
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.document_loaders import TextLoader, DirectoryLoader

def build_knowledge_base(docs_path: str) -> Chroma:
    """
    离线阶段：建立知识库
    只需要运行一次
    """
    
    # ━━━━━━ 1. 加载文档 ━━━━━━
    loader = DirectoryLoader(
        docs_path,
        glob="**/*.txt"  # 支持其他格式：*.pdf, *.md
    )
    documents = loader.load()
    
    # ━━━━━━ 2. 切片 ━━━━━━
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=500,       # 每片500字
        chunk_overlap=50,     # 相邻片段重叠50字
                              # 防止关键信息被切断
    )
    chunks = splitter.split_documents(documents)
    
    # ━━━━━━ 3. 向量化 + 存储 ━━━━━━
    embeddings = OpenAIEmbeddings()
    
    vectorstore = Chroma.from_documents(
        documents=chunks,
        embedding=embeddings,
        persist_directory="./knowledge_base"  # 本地持久化
    )
    
    print(f"知识库建立完成，共 {len(chunks)} 个片段")
    return vectorstore


def load_knowledge_base() -> Chroma:
    """
    在线阶段：加载已有知识库
    每次启动服务时调用
    """
    embeddings = OpenAIEmbeddings()
    
    vectorstore = Chroma(
        persist_directory="./knowledge_base",
        embedding_function=embeddings
    )
    
    return vectorstore
```

使用混合检索（向量检索、BM25检索）：

```
from langchain.retrievers import BM25Retriever, EnsembleRetriever

@tool
def search_knowledge(query: str) -> str:
    """
    在知识库中搜索相关信息。
    使用语义检索 + 关键词检索双路召回，效果更准确。
    
    适合调用的情况：
    - 用户询问政策、规则、说明类问题
    """
    
    # ━━━━━━ 双路检索 ━━━━━━
    
    # 路1：向量检索（语义相似）
    vector_retriever = vectorstore.as_retriever(
        search_kwargs={"k": 5}
    )
    
    # 路2：BM25 关键词检索
    bm25_retriever = BM25Retriever.from_documents(
        all_documents,  # 全量文档
        k=5
    )
    
    # 合并两路结果
    ensemble_retriever = EnsembleRetriever(
        retrievers=[vector_retriever, bm25_retriever],
        weights=[0.6, 0.4]  # 语义检索权重更高
    )
    
    docs = ensemble_retriever.get_relevant_documents(query)
    
    if not docs:
        return "知识库中未找到相关信息"
    
    # 整理结果喂给 LLM
    context = "\n\n---\n\n".join([
        doc.page_content for doc in docs[:3]  # 取最相关的3条
    ])
    
    prompt = f"""
    根据以下内容回答问题，只使用提供的信息：
    
    {context}
    
    问题：{query}
    """
    
    return call_llm(prompt)
```

更新工具箱，接入agent：

```
# ━━━━━━ 完整工具箱 ━━━━━━

tools = [
    recognize_intent,     # 理解用户意图
    fill_slot,            # 填充槽位
    check_slots,          # 检查槽位完整性
    ask_for_slot,         # 反问用户
    search_knowledge,     # 查询知识库  ← 新加入
    execute_intent        # 执行业务逻辑
]

# ━━━━━━ 更新 Prompt ━━━━━━

prompt = PromptTemplate.from_template("""
你是一个智能客服助手。

你有以下工具：
{tools}

处理逻辑：
1. 收到用户消息
   - pending_slot 不为空 → fill_slot
   - pending_slot 为空   → recognize_intent

2. 意图识别后判断类型：
   - 知识咨询类（政策/规则/说明）→ search_knowledge
   - 业务操作类（退货/查单）     → check_slots → execute_intent
   - 意图不明                   → ask_for_slot 澄清

3. 槽位检查：
   - complete=false → ask_for_slot
   - complete=true  → execute_intent

对话历史：{chat_history}
用户输入：{input}
{agent_scratchpad}
""")

# Agent 和 AgentExecutor 不变
agent = create_react_agent(llm, tools, prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    memory=memory,
    verbose=True,
    max_iterations=10,
    handle_parsing_errors=True
)
```



##### 一整轮的退货场景React

```
用户：退货政策是什么？

> Entering new AgentExecutor chain...

Thought: 用户在问政策，是知识咨询类问题
         直接调用 search_knowledge
Action: search_knowledge
Action Input: {"query": "退货政策"}
Observation: 根据知识库：
             本店支持7天无理由退货
             质量问题支持15天退换货
             退款将在3-5个工作日内到账

> Finished chain.

客服：本店支持7天无理由退货，质量问题可申请
     15天退换货，退款3-5个工作日内到账。

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

用户：我的耳机坏了想退货

> Entering new AgentExecutor chain...

Thought: 用户要退货，先识别意图
Action: recognize_intent
Action Input: {"user_input": "我的耳机坏了想退货"}
Observation: {
  "intents": [
    {"intent": "quality_complaint", "confidence": 0.89},
    {"intent": "apply_refund", "confidence": 0.85}
  ],
  "entities": {"product": "耳机"}
}

Thought: 有质量问题，先查一下质量退货政策
Action: search_knowledge
Action Input: {"query": "质量问题退货政策"}
Observation: 质量问题支持15天退换货，
             需提供问题描述和照片

Thought: 了解了政策，现在处理退货
         检查槽位
Action: check_slots
Action Input: {}
Observation: {"complete": false, "missing": ["order_id", "problem_description"]}

Thought: 缺少订单号，先问
Action: ask_for_slot
Action Input: {"slot_name": "order_id"}
Observation: 请问您的订单号是多少？

> Finished chain.

客服：请问您的订单号是多少？
```

这里 Agent 自己决定：先查政策，再处理退货。这是 Workflow 做不到的。





## 单轮工具调用——多轮工具调用

我们先看目前系统的问题，

```
查询订单状态：
    ↓
调用工具：query_order(order_id=???)


❌ 问题1：order_id 从哪来？

用户说"我上周买的耳机"
没有提供订单号！

单轮工具调用：
  参数不够 → 直接报错 或 随机猜一个
  没有能力反问用户："请问您的订单号是？"
  没有能力自己去查用户的历史订单

❌ 问题2：调用结果不符合预期怎么办？

query_order 返回：
  "该订单已签收，签收时间：3天前"

但用户说质量有问题想退货
这个结果和用户的真实需求之间
还差很多步骤：
  - 需要发起退货申请
  - 需要联系售后
  - 需要生成退货单

单轮调用结束了，后续没人管
```

这些问题，其实两个方面：
1、单轮调用工具信息不足，不足以完成任务，LLM的交互不是一个one-shot的操作，可能是有few-shot操作

2、单轮调用后信息不对，这时候说明几个问题，一个可能是tool的设计不好，出现调用的tool不对，或者是tool的返回值不明确

为了解决，需要我们做这么几个事情：

1. tool scheme设计
2. 工具实现
3. agent集成

### LLM 怎么知道自己有哪些工具可以用？Tool Schema 的设计

LLM 完全靠 Schema 决定：要不要调这个工具、怎么调

```
# 一个完整的 Tool Schema 长这样

{
  "name": "query_order",
  
  "description": """
    查询订单的当前状态和物流信息。
    
    适合调用的情况：
    - 用户询问订单到哪了
    - 用户想知道订单状态
    - 用户提供了订单号想查询
    
    不适合调用的情况：
    - 用户没有提供订单号（先用ask_for_slot获取）
    - 用户询问退货政策（用search_knowledge）
  """,
  
  "parameters": {
    "type": "object",
    "properties": {
      "order_id": {
        "type": "string",
        "description": "订单ID，格式为ORD+数字，例如ORD123"
      },
      "user_id": {
        "type": "string", 
        "description": "用户ID，可选，有助于验证订单归属"
      }
    },
    "required": ["order_id"]
  }
}
```

Schema 写得好不好，直接影响调用准确率！目前看下来有几个比较重要的点，①清晰明确的功能描述；②什么时候被调用；③和其他工具的边界（什么时候不能被调用）

**工具返回值设计原则：**

1. 永远返回结构化数据（JSON）：LLM 需要解析结果，自然语言不好解析 
2. 成功和失败都要有明确标识：success: true/false 
3. 失败要说明原因和类型（error_type）：帮助 LLM 决定下一步
4. 返回足够的上下文：不只是结果，还有 LLM 生成回复需要的信息



工具多了之后，其实就变成了一个“工具箱”。工具箱的完整设计：

```
# ━━━━━━ 工具箱按功能分组 ━━━━━━

# 信息收集工具
collection_tools = [
    recognize_intent,
    fill_slot,
    check_slots,
    ask_for_slot,
    infer_metadata_filter
]

# 知识检索工具
knowledge_tools = [
    smart_search,
    search_with_freshness_check
]

# 业务查询工具（只读）
query_tools = [
    query_order,
    query_user_orders,
    query_logistics,
    query_product_info
]

# 业务执行工具（写入）
action_tools = [
    create_refund,
    cancel_order,
    modify_address,
    create_complaint
]

# 路由工具
routing_tools = [
    unified_route,
    transfer_to_human
]

# 全部工具
all_tools = (
    collection_tools +
    knowledge_tools +
    query_tools +
    action_tools +
    routing_tools
)
```

System Prompt 告诉 LLM 怎么用工具：

```python
ystem_prompt = """
你是智能客服助手，通过工具处理用户请求。

━━━━ 工具使用原则 ━━━━

1. 信息收集优先
   没有足够信息前，不要调用业务工具
   先用 ask_for_slot 收集必要信息

2. 查询后再执行
   执行写入操作前，先查询确认信息正确
   例：创建退款前，先 query_order 确认订单存在

3. 失败要处理
   工具返回 success=false 时
   根据 error_type 决定下一步
   不要直接告诉用户"操作失败"

4. 写入操作要谨慎
   create/cancel/modify 类工具
   确认用户意图明确后再调用
   不要因为理解偏差误操作

━━━━ 典型处理流程 ━━━━

查询类：
  recognize_intent → check_slots → query_xxx → 回复

操作类：
  recognize_intent → check_slots → query_xxx（确认）
  → create/cancel_xxx → 回复

知识类：
  recognize_intent → infer_metadata_filter
  → smart_search → 回复
"""
```

总结来看，在agent系统中，一个tool scheme包含这些要点：

```
Tool Schema  → 写清楚"什么时候用、怎么用"
              LLM 靠这个决定调不调

工具实现     → 参数校验 + 异常处理 + 结构化返回
              每种失败都要有 error_type

Agent集成    → 工具箱分组 + System Prompt 说明原则
              让 LLM 知道工具之间的协作关系
```

但是如果只停留到这个层面，肯定是不够的。在工程里面，tool调用引申出来的问题是，LLM的循环控制（ReAct）、流式执行(Streaming，也就是结果处理)、错误恢复机制 (Retry，也就是异常机制)、上下文管理 (Memory)。

```
这四个问题在代码层面的位置：

用户输入
    ↓
┌─────────────────────────────────────┐
│  ReAct Loop                         │
│                                     │
│  while True:                        │
│      thought, action = LLM(prompt)  │ ← Memory 在这里喂历史
│      ↓                              │
│      result = execute(action)       │ ← Retry 在这里保障
│      ↓                              │
│      yield result                   │ ← Streaming 在这里输出
│      ↓                              │
│      memory.add(action, result)     │ ← Memory 在这里记录
│      ↓                              │
│      if should_stop(): break        │ ← Loop 在这里控制
└─────────────────────────────────────┘
    ↓
最终输出
```



LangChain 的 **AgentExecutor** 帮你封装了大部分，但理解底层原理，才能在它不够用的时候知道怎么扩展。



<font color='red' size='5'>当前wiki里面，我不会对LLM的控制做非常系统化的解释，后续会单独出一个wiki。此wiki适合面向初级agent工程师。因为本质上，下面LLM相关的控制更深的技术是去如何用已有的框架去实现。</font>



### LLM的循环控制（ReAct）——调用完了谁来控制下一步？

**LangChain（AgentExecutor）**

- AgentExecutor是一个典型的 while-loop 调度器：
  1. 构建 Prompt（包含历史 Thought/Action/Observation）
  2. 调用 LLM → 解析成 `AgentAction` 或 `AgentFinish`
  3. 如果是 `AgentAction` → 执行工具 → 得到 Observation
  4. Observation 加入 memory → 回到第 1 步
  5. 如果是 `AgentFinish` → 结束循环

- 循环控制点
  1. `max_iterations`：默认 15
  2. `early_stopping_method`：到达最大迭代时，直接返回 LLM 最后一次输出
- 特点
  - 纯线性循环，没有显式状态机
  - 终止条件是**LLM 输出 finish** 或 **步数上限**

**LangGraph（StateGraph）**

- LangGraph 把循环抽象成**有向状态图**：
  - 节点：`think`（LLM 决策）、`tool`（执行工具）、`end`（终止）
  - 边：条件函数（根据 state 决定下一个节点）
- 调用完工具
  - 工具节点执行完 → 状态更新 → 条件边函数判断 → 跳回 `think` 节点或进入 `end`
- 特点
  - 循环是显式图结构，不是硬编码 while
  - 终止条件就是到达 `END` 节点
  - 支持分支、并行、回退

**Claude Code** 的底层只有约 1.6% 是大模型 AI 决策逻辑，其余 98.4% 都是处理流式处理、上下文压缩、工具路由和安全检查的确定性基础设施。

核心循环：整个流程由一个名为 `queryLoop` 的异步生成器（`async generator`）驱动。它通过简单的 `while` 循环不断执行以下四个步骤： [[1](https://github.com/sanbuphy/learn-coding-agent), [2](https://zhuanlan.zhihu.com/p/2037839635645208435), [3](https://github.com/VILA-Lab/Dive-into-Claude-Code/blob/main/docs/architecture.md)]

1. **组装上下文**：提取当前的对话历史、系统提示词（System Prompt）和工具集定义，将其打包。
2. **调用模型**：将整合好的消息体发送给 Claude API，大模型通过推理生成回复，决定下一步动作。
3. **请求工具**：大模型若决定调用工具（例如 `bash` 运行命令、`read` 读取文件），会返回一个带有 `tool_use_id` 和参数的请求块。
4. **执行动作与回流**：系统执行对应的真实环境操作（受限于严格的权限系统），将执行结果包裹为 `tool_result` 再次附加到消息历史中，并回传给模型。 [[1](https://github.com/shareAI-lab/learn-claude-code/blob/main/docs/zh/s01-the-agent-loop.md), [2](https://deepseek.csdn.net/6a05ae09662f9a54cb7483dc.html), [3](https://github.com/shareAI-lab/learn-claude-code/blob/main/docs/en/s01-the-agent-loop.md)]

整个循环会一直持续，直到模型的返回结果中不再包含工具调用（即 `stop_reason != "tool_use"`）。





“下一步控制”聊完了，后面就是什么时候结束了

#### ReAct Loop 的终止条件怎么设计？（或者说中断设计）

**通用终止条件**

1. 显式完成信号
   - LLM 输出 `AgentFinish`（LangChain）
   - LLM 输出 Final Answer / 非工具调用块（Claude/Gemini）
2. 步数上限
   - `max_iterations`（LangChain 默认 15）
3. 死循环检测
   - 最近 N 步 Action + Input 完全相同
4. 错误次数上限
   - 连续工具调用失败 ≥ N 次
5. 业务完成条件
   - 任务目标达成（例如所有必需槽位已填充并确认）





Claude为了防止 AI 陷入死循环或因为上下文超出限制崩溃，循环内部包含极强的状态控制（**核心是QueryEngine设计**）：

- **流式处理**：通过 `for await (const turn of queryLoop)` 流式拉取每一轮的状态，让用户和系统能实时看到思考和执行过程。
- **多层压缩（Compression）**：在每一轮请求之前，系统会执行多层压缩机制，丢弃冗余信息，确保在长任务中不会超出模型的上下文窗口（Context Window）。
- **恢复与规避（Circuit Breakers）**：循环内设有限制，不仅处理正常调用，还会处理 Token 超出、用户手动中断或 API 错误。系统能够“记住”失败原因并重试，甚至在出现无限重试迹象时主动跳出循环。



### LLM的流式执行(Streaming，也就是结果处理)——调用结果怎么实时给用户看？

**LangChain**

- 使用CallbackHandler：
  - `on_llm_new_token`：每个新 token 到达时触发
  - `on_tool_start` / `on_tool_end`：工具执行前后触发
  - `on_agent_finish`：最终回答
- 解析逻辑由开发者在 Callback 中实现（事件回调需要自己来实现）

**LangGraph**

- 状态图中每个节点可以流式输出
- 工具节点执行完 → Observation 注入 → 回到 LLM 节点继续生成



#### Streaming 的 token 流怎么解析？

**Streaming** 的目标：

- 在 LLM 输出 token 流的过程中，边解析边展示给用户
- 在工具调用阶段，也实时告诉用户正在做什么
- 支持中途插入 Observation（工具结果）再继续生成



Claude Code 的流式执行与 Token 解析主要依赖以下机制（**核心是StreamingToolExecutor、AsyncGenerator机制**）：

**1、流式执行底层架构（基于 SSE）**

Claude Code 与 Anthropic API 的通信并非传统的单次 HTTP 请求-响应，而是基于 **SSE（Server-Sent Events）** 建立的持久长连接。 [[1](https://zhuanlan.zhihu.com/p/2023055888286589550)]

1. **持续推送**：当向 API 发送请求（带上 `stream: true`）后，模型一旦开始推理，服务端就会通过长连接将生成的 Token 或事件源源不断地推送到客户端，无需等待整体生成完毕。
2. **异步生成器（Async Generator）**：在底层代码架构上，Claude Code 的控制流是建立在 `async function*` 之上的。每次接收到 Token，生成器会通过 `yield` 向消费端推送事件，形成“惰性求值”和天然的限流保护，避免内存爆炸。 [[1](https://claudeyy.com/zh/blog/claude-api-streaming-batch-enterprise-guide-2026), [2](https://zhuanlan.zhihu.com/p/2023055888286589550)]

**2、 Streaming Token 流的解析机制**

在流式输出中，由于数据是一小块一小块（Delta）到达的，直接反序列化（如 `JSON.parse`）通常会因为 JSON 字符串不完整而报错。Claude Code 解析 Token 流的核心策略如下：

1. 事件分流（Event Routing）：SSE 会推送不同类型的事件（例如 `content_block_start`, `content_block_delta`, `content_block_stop` 等）。解析引擎会先判断事件类型： [[1](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming), [2](https://deepseek.csdn.net/6a0abcfb662f9a54cb7557a5.html)]

- **如果是思考过程**：近期模型会输出 `<thinking>` 标签或特殊的 `thinking` 块，引擎会将其作为“内部思考/推理”实时展示给界面。
- **如果是代码/文本生成**：当监听到 `content_block_delta` 时，提取其中的 `delta.text` 并实时追加到缓冲区。 [[1](https://platform.claude.com/docs/zh-TW/build-with-claude/extended-thinking), [2](https://deepseek.csdn.net/6a0abcfb662f9a54cb7557a5.html)]
- 增量解析与状态还原

对于需要结构化输出的场景（比如让模型输出 Tool Use，即工具调用），系统会维护一个累积缓冲区（Accumulator）。 [[1](https://github.com/lintsinghua/claude-code-book/blob/main/第四部分-工程实践篇/13-流式架构与性能优化.md)]

- 每接收到一个增量（Delta），便追加到缓冲区末尾。
- **不完整 JSON 解析**：系统会尝试对这个动态增长的字符串进行部分解析。依靠 Pydantic 这类支持部分 JSON 解析的库，或者通过自定义的健壮解析器（处理丢失的引号、括号），提取出当前的 Tool 参数，实现实时显示与初步执行。 [[1](https://platform.claude.com/docs/zh-CN/build-with-claude/streaming), [2](https://github.com/lintsinghua/claude-code-book/blob/main/第四部分-工程实践篇/13-流式架构与性能优化.md)]
- 提前推测与预执行（Speculative Execution）

这是 Claude Code 提升 Agent 效率的关键创新。 [[1](https://zhuanlan.zhihu.com/p/2023055888286589550)]

- 模型在生成工具调用（例如文件读取、代码编辑的 JSON）时，系统不需要等到整个工具块生成完毕才开始处理。
- 当增量解析器捕捉到完整的 tool name 和部分参数时，**预执行（StreamingToolExecutor）** 机制就会启动。系统会在沙盒化的虚拟文件系统（Overlay FS）中预执行这些操作，节省大量等待时间





### LLM的错误恢复机制 (Retry，也就是异常机制)——调用失败了怎么办？——中断的思想

> 为什么需要中断，事件的优先级会随着时间而调整；——所以优先级的排序是提前做好的。 事件本身优先级的排序，执行中任务的排序；
>
> 当这两个出现冲突的时候，一旦收到中断，任务要先做备份；发中断之前（event），要把cpu变成元数据的状态。中断恢复，
>
> 中断元数据执行过程中，只做优先级的处理，不做任何业务的处理，中断的操作可以忽略不计；
>
> 中断完之后，中断恢复后，需要保证上下文一致，刚刚的数据已经变成了元数据的状态
>
> 在这一节，如果仔细思考这里的逻辑，会发现，LLM的错误恢复就是在做一次中断。

首先看看agent中有哪些失败？

- LLM 生成的参数不合法
- 工具返回业务拒绝
- 网络超时/限流
- 资源不存在
- 权限不足

**关键问题不是所有错误都应该重试，要搞清楚哪些失败场景是需要重试的。**有些错误重试 100 次也没用（比如业务规则拒绝），有些错误必须马上重试（比如参数格式错），有些错误需要等一段时间后重试（比如限流）。所以说，整体上来看，其实也就三类：

1. **立即重试（Immediate Retry）**即可恢复
   1. 输出参数格式错误（JSON 不合法）
   2. 缺少必填参数
   3. 处理方案：LLM根据报错自动修正
2. **需延迟重试（Backoff Retry）**
   1. 服务不可用
   2. 服务限流
   3. 处理方案：**指数退避**（Exponential Backoff，第一次等1s，第二次等2s，第三次等4s）
3. **不可重试**
   1. 权限不足
   2. 资源不存在
   3. 处理方案：收集错误信息给LLM决策换工具、反馈用户调整



现阶段，有几个非常常见的思路：

**LangChain**

- 提供 `handle_parsing_errors` 参数，可以让 LLM 在 JSON 解析失败时自动重试
- 但对业务错误没有内置分类，需要开发者自己实现

**LangGraph**

- 可以在图节点中显式区分错误类型，错误节点决定走向：
  - 可重试 → 回到工具节点
  - 不可重试 → 跳到结束节点或解释节点



Claude Code 的错误恢复机制（异常与重试处理）并非单纯依赖简单的 `try-catch`，而是通过**多层次的容错架构**来保障代理系统（Agent）在出错时能继续工作、自我修正甚至人工回滚。

1. 工具调用与失败重试 (Tool & API Retry)
   1. **Error反馈**：执行失败后，错误信息（带有 `is_error: true` 标识）会被直接发送回给大语言模型 (LLM)。
   2. **LLM自主决定**：模型在看到具体的错误栈或报错信息后，可以自主决定下一步行动
2. 代码回滚与状态恢复：`/rewind` 指令
   1. 如果 Claude 进行了错误的代码修改，你可以使用其特有的 **/rewind** 命令（或 `/checkpoint`）来像“时间机器”一样撤销失误。（Claude Code 在进行每次代码编辑（Write/Edit）前，都会在本地隐式地对受影响的文件进行快照。使用 `/rewind` 会调出交互式界面，允许你选择回滚到代码和对话的某一个特定节点。）
3. 上下文截断与压缩：Auto-compact
   1. **自动压缩**：当上下文达到窗口极限时，Claude Code 不会报错崩溃，而是触发内置的 **Autocompact** 机制，在后台静默地将历史对话进行提炼和总结，以腾出 Token 供新操作使用。
4. 会话持久化 (Session Resumption)
   1. **本地日志恢复**：Claude Code 会将每一次会话、工具调用及结果自动以 JSONL 格式保存在本地机器上（例如 `~/.claude/projects/` 下）。
   2. **无缝接续**：如果会话中断，你只需在原目录下输入 `claude --resume`（或启动时使用 `--continue`），即可唤起之前的会话选择器，恢复完整的上下文历史并继续工作。
5. 应对极端情况：模型降级 (Model Fallback)
   1. 在 API 请求量过大或主力模型（如 Claude 3.5 Sonnet 或 Opus）遇到速率限制 (Rate Limit) 及容量限制时，系统具备降级策略，可自动切换至备用模型或排空队列等候，尽量保证代理任务不会完全失败。



#### Retry 的错误分类怎么做？

> 遇到问题，解决问题。这种思路是好的，但是最好是跳出来看看为什么会发生这种问题，能不能通过架构优化避免这种情况。
>
> 记住，工程师在架构设计是，一定是要保持悲观的，做好最坏的打算。

上面我们介绍了常见的错误处理方案，工程上要有一份 **error_type → 策略** 的映射表，这个表最好和业务团队共同维护。这里我进一步（只进一步哈，更多的细节我后续出wiki分析），把 **错误分类 + Streaming 状态机** 结合起来，做一个**生产级的流式执行 + Retry 闭环**，实现一个 **边流式解析 LLM 输出，边执行工具，且能在执行失败时自动分类错误并重试** 的 Agent 执行引擎。

```
┌──────────────────────────────────────────┐
│ StreamingAgentEngine                      │
│  ┌────────────────────────────────────┐ │
│  │ StreamingParser (状态机解析器)       │ │
│  │  - 解析 token 流                    │ │
│  │  - 识别阶段 (Thought/Action...)     │ │
│  │  - 触发工具执行事件                  │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │ RetryHandler (错误恢复器)            │ │
│  │  - 错误分类 (error_type → 策略)      │ │
│  │  - Immediate/Backoff/No Retry       │ │
│  │  - LLM 自我纠错 (Self-Correction)   │ │
│  └────────────────────────────────────┘ │
│  ┌────────────────────────────────────┐ │
│  │ Memory (上下文管理)                  │ │
│  │  - 保存 Thought/Action/Observation │ │
│  │  - 供下轮 LLM 调用使用               │ │
│  └────────────────────────────────────┘ │
└──────────────────────────────────────────┘
```

**1）StreamingParser（状态机解析器）**

```python
python
复制from enum import Enum

class ParseState(Enum):
    THINKING = "thinking"
    ACTION = "action"
    ACTION_INPUT = "action_input"
    ANSWERING = "answering"

class StreamingParser:
    def __init__(self):
        self.state = ParseState.THINKING
        self.buffer = ""
        self.current_action = ""
        self.current_input = ""

    def feed_token(self, token: str):
        self.buffer += token

        if "Action:" in self.buffer and self.state == ParseState.THINKING:
            thought = self.buffer.split("Action:")[0].replace("Thought:", "").strip()
            self.buffer = self.buffer.split("Action:")[1]
            self.state = ParseState.ACTION
            return {"type": "thought", "content": thought}

        elif "Action Input:" in self.buffer and self.state == ParseState.ACTION:
            self.current_action = self.buffer.split("Action Input:")[0].strip()
            self.buffer = self.buffer.split("Action Input:")[1]
            self.state = ParseState.ACTION_INPUT
            return {"type": "action", "action": self.current_action}

        elif "Observation:" in self.buffer and self.state == ParseState.ACTION_INPUT:
            self.current_input = self.buffer.split("Observation:")[0].strip()
            self.buffer = self.buffer.split("Observation:")[1]
            self.state = ParseState.THINKING
            return {
                "type": "execute_tool",
                "action": self.current_action,
                "action_input": self.current_input
            }

        elif "Final Answer:" in self.buffer:
            answer = self.buffer.split("Final Answer:")[1].strip()
            self.state = ParseState.ANSWERING
            return {"type": "final_answer", "content": answer}

        return None
```



**2）RetryHandler（错误恢复器）**

```python
python
复制from enum import Enum
import asyncio
import json

class RetryStrategy(Enum):
    NO_RETRY = "no_retry"
    IMMEDIATE_RETRY = "immediate_retry"
    BACKOFF_RETRY = "backoff_retry"
    LLM_SELF_CORRECT = "llm_self_correct"

ERROR_STRATEGY = {
    "invalid_params": RetryStrategy.LLM_SELF_CORRECT,
    "missing_params": RetryStrategy.LLM_SELF_CORRECT,
    "timeout": RetryStrategy.BACKOFF_RETRY,
    "rate_limit": RetryStrategy.BACKOFF_RETRY,
    "service_unavailable": RetryStrategy.BACKOFF_RETRY,
    "business_rule_violation": RetryStrategy.NO_RETRY,
    "permission_denied": RetryStrategy.NO_RETRY,
    "not_found": RetryStrategy.NO_RETRY,
}

class RetryHandler:
    def __init__(self, llm_correct_func):
        self.llm_correct_func = llm_correct_func

    async def execute_with_retry(self, tool_func, action_input, action_name, max_retries=3):
        attempt = 0
        last_error = None

        while attempt < max_retries:
            attempt += 1
            try:
                result = await asyncio.to_thread(tool_func, **action_input)

                if isinstance(result, dict) and not result.get("success", True):
                    error_type = result.get("error_type", "unknown")
                    strategy = ERROR_STRATEGY.get(error_type, RetryStrategy.NO_RETRY)

                    if strategy == RetryStrategy.NO_RETRY:
                        return {"success": False, "result": result}

                    elif strategy == RetryStrategy.LLM_SELF_CORRECT:
                        action_input = await self.llm_correct_func(action_name, action_input, result["error"])
                        continue

                    elif strategy == RetryStrategy.BACKOFF_RETRY:
                        await asyncio.sleep(2 ** attempt)
                        continue

                return {"success": True, "result": result}

            except Exception as e:
                last_error = e
                await asyncio.sleep(2 ** attempt)

        return {"success": False, "error": str(last_error), "exhausted": True}
```

**3）StreamingAgentEngine（流式执行引擎）**

```python
python
复制class StreamingAgentEngine:
    def __init__(self, llm_stream_func, tools, retry_handler):
        self.llm_stream_func = llm_stream_func
        self.tools = {t.__name__: t for t in tools}
        self.parser = StreamingParser()
        self.retry_handler = retry_handler

    async def run(self, user_input):
        async for token in self.llm_stream_func(user_input):
            event = self.parser.feed_token(token)
            if event:
                if event["type"] == "thought":
                    yield f"💭 {event['content']}"

                elif event["type"] == "action":
                    yield f"⚙️ 正在执行工具: {event['action']}"

                elif event["type"] == "execute_tool":
                    tool_func = self.tools.get(event["action"])
                    if not tool_func:
                        yield f"❌ 工具 {event['action']} 不存在"
                        continue

                    result = await self.retry_handler.execute_with_retry(
                        tool_func, json.loads(event["action_input"]), event["action"]
                    )

                    if not result["success"]:
                        yield f"⚠️ 工具执行失败: {result}"
                    else:
                        yield f"📋 工具结果: {result['result']}"

                elif event["type"] == "final_answer":
                    yield f"✅ {event['content']}"
```

这里面有好几个比较重要的点：

1. **状态机解析 + Retry 解耦**（<font color='red'>这里我们用到了单一职责 (SRP)</font>）

   - StreamingParser 只负责解析 token 流，不关心错误处理

   - RetryHandler 只关心错误分类和重试策略，不关心 token 流

   - 两者通过事件接口（`event["type"]`）解耦，方便单独测试和替换

     ```
     # StreamingAgentEngine：收到 execute_tool 事件才调用 RetryHandler
     elif event["type"] == "execute_tool":
         tool_func = self.tools.get(event["action"])
         result = await self.retry_handler.execute_with_retry(
             tool_func, json.loads(event["action_input"]), event["action"]
         )
     ```

     

2. **错误分类表驱动**（<font color='red'>这里是关键，一个是维护了映射表，开放封闭原则（OCP），另一个是我们用到了一个非常好的设计模式——策略模式</font>）

   - `ERROR_STRATEGY` 是一张映射表，新增错误类型时只改表，不动主逻辑
   - 策略模式（统一叫错误分类的接口，最终操作时由LLM去实现具体的逻辑）：Immediate / Backoff / No Retry / LLM Self-Correct

3. **LLM 自我纠错闭环**

   - 参数错误时，不是直接失败，而是把错误信息反馈给 LLM 让它修正参数；当错误类型是 `invalid_params` 或 `missing_params`，直接调用 LLM 生成修正后的参数。

     ```
     elif strategy == RetryStrategy.LLM_SELF_CORRECT:
         # 调用 llm_correct_func 让 LLM 修正参数
         action_input = await self.llm_correct_func(action_name, action_input, result["error"])
         continue
       
     # llm_correct_func 示例
     async def llm_self_correct(action_name, wrong_input, error_msg):
         prompt = f"""
         工具 {action_name} 调用失败：
         错误参数：{wrong_input}
         错误信息：{error_msg}
         请修正参数，只返回 JSON。
         """
         corrected = await call_llm_async(prompt)
         try:
             return json.loads(corrected)
         except:
             return wrong_input
     ```

     

   - 修正后的参数直接进入下一次工具调用，形成闭环

4. **Observation 注入**

   - 工具结果（成功或失败）都会作为 Observation 注入到 LLM 的上下文

     ```
     elif event["type"] == "execute_tool":
         ...
         result = await self.retry_handler.execute_with_retry(...)
         if not result["success"]:
             yield f"⚠️ 工具执行失败: {result}"
             observation = result  # 失败结果
         else:
             yield f"📋 工具结果: {result['result']}"
             observation = result['result']  # 成功结果
     
         # 注入到 Memory（这里省略了 Memory 类实现）
         self.memory.add(
             action=event["action"],
             action_input=event["action_input"],
             observation=observation
         )
     ```

   - 让 LLM 能根据错误调整下一步计划



> 事实上，强如claude code，也有一整套错误重试的干预方案机制，https://code.claude.com/docs/zh-CN/errors，并不是完全让大模型自由发挥









### LLM的上下文管理 (Memory)——调用历史怎么记住？

了解memory机制，最重要的一个点就是人和LLM的交互入口只有一个——prompt。Memory 到底在解决什么问题？并不是各种类型的问题都适合用memory。是因为agent没有历史信息？应该说在大模型模型参数固定（训练结束后就不变），上下文窗口有限的情况下，大模型没有办法把昨天学到的东西，**以一种稳定、可更新、可追溯的方式利用起来。**



```
一个常见的prompt = 
System prompt（明确 Claude 的角色、能力范围、执行规则） 
+ history（当前文件列表、用户之前的请求和 Claude 的响应摘要（memory关键信息）、当前任务状态（例如：已修改哪些文件、测试结果）） 
+ Tool Schema（tool_use、tool_descriptions）
+ User Input（用户当前的请求）
```

Memory 压缩的目标是：**保留对决策有用的信息，丢掉冗余细节**，同时不破坏 LLM 的推理链。



参考codex的memory记忆机制，history中一般包含了**该session前面的执行步骤（当前上下文） + 用户长期事实（user.md） + 近期对话摘要（关键信息&代办事项） + 历史案例（历史对话/方案总结）**。但是注意，这是一个独立的记忆系统，并不会把所有的信息都装载到上下文，在装载之前也只会把和本次query关联度大的部分检索出来，然后拼接到context。（不是无脑全塞进去）



memory 是一整套分层的系统架构，而向量数据库只是一种数据检索方案，两者在不同的层面。**不同类型的记忆，需要不同的存储和记忆策略。当前上下文和长期事实用结构化储存，近期脉络对话摘要历史案例要向量化。**

向量数据库是为了解决模糊的，复杂的，难以穷举的问题（比如，之前聊过什么话题？之前聊的python 问题？这时候就需要去检索），但是大多数agent 场景，对memory 的要求是精准，可控，可更新（用户偏好是什么，用户年龄是多少，这是事实，不能用语义来模糊搜索）。什么时候需要向量数据库？当你的查询语义是模糊的，当你的问题是开放式增长的，memory 内容是非结构化的（比如，客服agent 需要从历史几万条工单中找类似的）。

工程上，除了关注存储以外，还要知道什么时候应该去触发压缩，什么时候把摘要持久化。

1. **分层存储**
   - Facts 永久保留（订单号、用户 ID、意图等）
   - Recent 保留最近 N 轮完整记录
   - Summary 存更早的历史摘要
2. **压缩触发条件**
   - 按 token 数量触发，而不是按轮次
   - 避免因为长 Observation 提前溢出
3. **摘要要可验证**
   - 压缩时保留关键实体（订单号、金额等）
   - 避免 LLM 压缩时丢掉关键信息
4. **结合业务重要度**
   - 对关键步骤（确认订单号、确认退货原因）打标签，永不丢弃



而这一切都围绕这四个维度去build：

1. 一个是用户模型（用户习惯、决策模式） ——user
2. 任务模型（任务执行到哪一步，执行状态）——project
3. 世界模型（操作环境，系统边界，数据新鲜度。大量"个性化错误"本质上不是没记住你，而是没记住你所在的环境已经变了） ——reference
4. 自我模型（试过什么，失败总结）——feedback





> 所以用向量数据库来做memory之前先看看数据库做出来是干啥，做检索的必要性。不然未来就会变成技术债务！比如现在的境外改造，之前设计多语言和主语言的同步，用了5层提报模型，做了mafka 转发，数据同步队列（sync queue ），但是现在做改造发现这里太多层之后，数据一致性保证太差！核心就是没想明白这两个字段的定位。
>
> 选对框架而不是证明你会用复杂的工具。向量数据库是为了解决模糊的，复杂的，难以穷举的问题（比如，之前聊过什么话题？之前聊的python 问题？这时候就需要去检索），但是大多数agent 场景，对memory 的要求是精准，可控，可更新（用户偏好是什么，用户年龄是多少，这是事实，不能用语义来模糊搜索）。
> 那什么时候需要向量数据库？当你的查询语义是模糊的，当你的问题是开放式增长的，memory 内容是非结构化的（比如，客服agent 需要从历史几万条工单中找类似的）。



#### Memory 的上下文压缩策略？

**LangChain**

- `ConversationBufferMemory`：简单滑动窗口
- `ConversationSummaryMemory`：LLM 自动摘要
- `ConversationBufferWindowMemory`：滑动窗口 + 限制窗口大小
- `CombinedMemory`：组合多种 Memory

**LangGraph**

- 可以把 Memory 压缩作为一个节点，在每轮对话后执行
- 支持 Facts、Summary、Recent 分层



Claude Code 通过会话记忆（Session Memory）和动态上下文压缩机制来管理调用历史。

Claude Code 拥有互补的**双重记忆系统**（内置于对话中，作为上下文加载）： [[1](https://code.claude.com/docs/zh-TW/memory), [2](https://code.claude.com/docs/zh-CN/memory)]

1. **短期会话记忆 (Session Memory)**
   - **内容**：自动捕获当前对话中的所有操作、提问、代码修改和工具输出（如 Bash、Read、Grep）。
   - **机制**：在每次新对话开始时加载。它会维护一个连续的历史记录，但并非简单的文本堆砌，系统会保留 API 的不变性（如修复工具调用和思考块的分离）。
2. **长期持久化记忆**
   - **类型**：分为四类约束性特征 —— `user`（用户偏好）、`feedback`（反馈）、`project`（项目状态）和 `reference`（参考资料）。这些记忆不仅限于单次对话，可跨会话积累。
   - **全局配置（`CLAUDE.md`）**：项目根目录下的 `CLAUDE.md` 扮演关键角色，记录架构规范、技术栈和约束条件。每次启动都会作为长期记忆加载。

当上下文数据量庞大（如达到特定阈值，约占上下文限制的 90% 以上）时，Claude Code 不会粗暴截断，而是启动**四层递进压缩体系**：

**第一层：微压缩 (Micro Compact) - 纯规则处理**

- **策略**：自动清理旧工具调用（如历史 `Read`、`Bash` 等的输出结果），不调用大模型。
- **动作**：用简短的占位符（如 `[Old tool result content cleared]`）替换过期内容，延迟极低（\(<1\) ms），能最大化保留语义。

**第二层：自动压缩 (Auto Compact) - 会话摘要**

- **策略**：当接近模型上下文上限时触发（例如 Opus 200K 模型会在 167K 左右触发）。
- **动作**：优先尝试提取“会话记忆摘要”，利用现有记忆生成简短总结，无需额外大模型调用。

**第三层：传统压缩 (Full Compact) - 结构化重构**

- **策略**：当单次对话过长且基础压缩不够时触发。
- **动作**：通过 Fork Agent 生成结构化的会话摘要，包含：主要请求、关键概念、涉及文件、错误修复及挂起任务等，同时排除冗余附件以减轻 token 负担。

**第四层：超长上下文窗口 (1M Token) 支持**

- **策略**：利用 Claude 最新模型（如 Opus 4.6/4.7 或 Sonnet 4.6）的 1M token 窗口特性。
- **动作**：通过原生支持极长文本，极大降低需要压缩的频率，保证 AI 可以记住整个项目的细节，无缝处理长时间的开发任务。





一个简单的memory伪代码：

```
class Memory:
    def __init__(self, max_tokens=6000, keep_recent=5):
        self.max_tokens = max_tokens
        self.keep_recent = keep_recent
        self.recent_turns = []  # 最近完整记录
        self.summary = ""       # 历史摘要
        self.facts = {}         # 永久事实

    def add_fact(self, key, value):
        self.facts[key] = value

    def add_turn(self, action, action_input, observation):
        turn = {
            "action": action,
            "action_input": action_input,
            "observation": observation,
            "tokens": self._estimate_tokens(action + str(action_input) + observation)
        }
        self.recent_turns.append(turn)
        self._compress_if_needed()

    def _compress_if_needed(self):
        total_tokens = sum(t["tokens"] for t in self.recent_turns)
        if total_tokens <= self.max_tokens:
            return

        # 超出限制：保留最近 N 轮，压缩其余
        to_compress = self.recent_turns[:-self.keep_recent]
        self.recent_turns = self.recent_turns[-self.keep_recent:]

        if to_compress:
            history_text = "\n".join([
                f"Action: {t['action']}, Obs: {t['observation'][:100]}"
                for t in to_compress
            ])
            new_summary = call_llm(f"""
            将以下历史压缩为摘要（150字以内），
            保留关键事实、任务进展、重要数据：
            {history_text}
            """)
            self.summary = (self.summary + "\n" + new_summary).strip()

    def build_prompt(self, user_input):
        parts = []
        if self.facts:
            facts_text = "\n".join([f"{k}: {v}" for k, v in self.facts.items()])
            parts.append(f"[已知事实]\n{facts_text}")
        if self.summary:
            parts.append(f"[历史摘要]\n{self.summary}")
        if self.recent_turns:
            turns_text = "\n".join([
                f"Action: {t['action']}\nInput: {t['action_input']}\nObs: {t['observation']}"
                for t in self.recent_turns
            ])
            parts.append(f"[最近记录]\n{turns_text}")
        parts.append(f"[当前输入]\n{user_input}")
        return "\n\n".join(parts)
```





# 总结

最后总结一下，本质是在说一件事：Workflow 的四个模块（意图识别、分流、RAG、工具调用）在Agent 里不是被替换了，而是被重新组织了——意图和分流融入 LLM 理解，RAG和工具变成 @tool，整个流程由 ReAct Loop 驱动。

```
演进路径：
单轮 Workflow → 加 ReAct Loop → 加规划+Memory → 加反思+多Agent
```

Agent 对 Workflow 做了两件事：吸收（意图识别和分流交还给 LLM 理解能力）+ 改进（工具调用变多轮、RAG 变多轮、沉淀 Memory）。

Agent Loop 的骨架本质就是10 几行 TAO 循环，其余 98% 都是确定性基础设施。
