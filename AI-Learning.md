# AI Learning System

> **🤖 给接手的 AI（先读这段）**：你被指派继续给 @liryan 教 AI/LLM。开始前请：
> 1. 读本文件的 **教学偏好**（下方）、**Current Status**（学到哪）、**Next Lesson**（下一课）、**待讲清单**（欠讲的概念）。
> 2. 教学契约（务必遵守）：目标是建 mental model 不是背定义；**一课一个新概念**、严格循序渐进，别砸没铺垫的概念；提前用到的概念记进「待讲清单」只说"后面讲"；每讲完一个概念**让学生用自己的话复述→你抓错纠正→确认再前进**；学生可随时打断。**讲深度：每个零件都要讲到"它具体在算什么、语义作用是什么"的机制层（如 FFN=检测特征+联想记忆），别停在"增强表达能力"这种宽泛说法。** **环环相扣：介绍任何对象前，先明确它和已学对象的关系——①已学对象是什么 ②为什么还不够 ③新对象解决什么 ④结果流向哪里；复用的旧对象要明确说"不是新对象，只是从另一个环节再次出现"（如输出端的 token 词表）。** **课程结构（从 Module 2 起）：每个 Module 先上「概览课」自顶向下给完整骨架（各框一句话+🅿️细节后挖），再逐块下钻——详见 Roadmap 上方「教学结构」。**
> 3. 学生画像：软件工程师（工程直觉强）；线代/微积分学过但生疏 → 用**直觉+图+最小公式**，不硬推导；节奏偏慢、爱刨根问底（这是优点，顺着走）。
> 4. 产出：讲义是**自包含 HTML**（内嵌 CSS+SVG，顶部带导航条），写在 `source/ai-learning/`，`git push origin master` 由 GitHub Actions 自动部署到 **https://jtr109.com/ai-learning/**；沿用现有配色/结构（照抄任一现有讲义当模板）。每课后更新本文件的 Current Status / Lesson Log / 待讲清单。
> 5. 方法论详见 `source/ai-learning/how-to-learn.html`。
>
> ---
>
> 本文档由 AI 维护。学生 = @liryan。AI 同时担任 **Tutor（讲课/出题/纠错）** 和 **Knowledge Manager（存档/维护进度）**。
> Roadmap 低频变化；每节课后更新的是 `Current Status` / `Lesson Log`。
>
> **📍 讲义位置（2026-07 起）**：HTML 讲义现位于本仓库 `source/ai-learning/`，push `master` 后由 GitHub Actions 自动部署到 **https://jtr109.com/ai-learning/**。旧的独立仓库 `jtr109/AI-Learning` 已归档弃用。本文件（进度追踪）在仓库根，不发布到网站。

## Goal

建立系统性的 AI / LLM / AI Engineering 知识体系。

核心目标：

> **Build Mental Models, not memorize definitions.**

不是背概念，而是能回答：为什么这个技术存在？它解决什么问题？它和已有知识如何连接？它在整个体系里处于什么位置？

学习范围：Transformer · LLM · LLM Inference · AI Engineering · RAG · Agents · AI System Design

---

## Roles（本 session 合并版）

- **Copilot = Tutor + Knowledge Manager**
  - 设计每节课、讲解、建立 mental model、出诊断题、发现并纠正误区、按需补 just-in-time 数学
  - 维护本文档：`Current Status`、`Lesson Log`、`Knowledge`
  - 不随意改 `Roadmap`（那是课程设计文档，改动需学生同意）
- **Student（@liryan）**
  - 主动思考、提问、**用自己的话总结**、判断自己是否真的懂

### 学生画像（影响讲课深度）
- 软件工程师（Microsoft, Windows 365 Data Platform），工程背景强
- 数学：线代 + 微积分学过但生疏 → **需要时用直觉+图+最小必要公式快速复习，不硬上推导**
- 偏好：图 / 具体例子 > 公式；先建直觉再上细节

---

## Learning Workflow

**产出格式** — 课程内容（讲解 + 图表）以**自包含 HTML** 输出到 `lessons/NN-topic.html`（内嵌 CSS + SVG，可离线打开）；进度追踪继续用本 markdown 文档。

**每节课前** — 根据 Roadmap + Current Status 输出 `Today's Lesson`（Topic / Objectives / Prerequisites / Expected Outcome / Warnings）。

**课中** — 讲 What / Why / How，配具体例子和图，随时抓误区。

**课后** — 学生用自己的话按下面格式总结，Copilot 检查逻辑、纠错、更新 Current Status：
```
# Lesson Summary
## My Understanding      （用自己的话）
## Key Mental Models     （核心理解）
## Remaining Questions   （未解决）
## Misconceptions Fixed  （错 -> 对）
```

### Learning Principles
1. Mental Model > Definition
2. Understanding > Speed
3. Math is learned Just-in-Time
4. 新知识必须接到已有知识上
5. 缺前置知识就暂停补齐
6. 图和例子优先于死记
7. 每个概念都要回答：解决什么问题 / 为何存在 / 如何工作

### 教学结构：自顶向下 + 逐块下钻（螺旋式）⭐ 从 Module 2 起采用
> 学生反馈：Module 1 总复习那种"先给完整链路骨架、再逐块往里挖"的顺序最合理。据此，每个 Module 采用两遍螺旋：
> 1. **第一遍·概览课（自顶向下骨架）**：先给这个 Module 的完整链路当"一串黑盒"，每个框只一句话（干什么/在全局地图哪个位置），细节全标「🅿️ 后面挖」。这一遍在抽象层仍是"一个概念一个框"，不深入。
> 2. **第二遍·逐块下钻**：一个框一个框展开，框内部仍按下面的颗粒度原则自底向上严谨讲——但此刻学生已知道它在整图的位置与作用（动机前置）。
>
> 好处：自顶向下给动机+地图，自底向上给严谨。限制：概览必须克制（框=一句话+🅿️，不展开）；纯数学微工具（点积/softmax 类）不进骨架，仍在下钻到需要处 just-in-time 补。

### 教学颗粒度原则 (Granularity)
- **一课一个新概念**：每个讲义只引入一个主要新概念；数学小工具（点积、softmax…）拆成独立微课，在需要它的前一刻讲（just-in-time）。
- **严格循序渐进**：一次解释里不砸一堆没铺垫过的概念。用到的东西必须"要么已教、要么明确标注为🅿️预告"。
- **预告即登记**：提到但暂不展开的概念 → 记入「待讲清单」，只说"后面讲"。
- **概念分层**：区分"算法 / 对算法的解读 / 下游概念"三层，不混谈（例：点积 → 相似度 → 注意力权重）。
- **先确认再前进**：每个概念确认掌握后再进下一个；鼓励随时打断提问。
- **机制层深度**：每个零件讲到"它具体在算什么、语义作用是什么"（如 FFN=检测特征+联想记忆），别停在"增强表达能力"这种宽泛说法。

---

## Roadmap

### Module 0 — Foundations
建立 LLM 基础 mental model。
- Token · Embedding · Vector（向量空间直觉）· Linear Algebra Basics（按需）

### Module 1 — Transformer ✅ 完结
理解 Transformer 为什么能工作。（自底向上讲的，总复习 `overview-module1-review.html` 补了自顶向下地图）
- 微课链：softmax → 朴素注意力 → Q/K/V → ÷√d 缩放 → Multi-head → 残差 → LayerNorm → FFN
- **Outcome**：能解释 Transformer 如何根据已有 token 加工出 hidden state（预测下一个 token 的输出层留给 Module 2）。

### Module 2 — Large Language Models（下一个）
**先上「Module 2 概览课」**（自顶向下：从 hidden state 到吐出下一个词的完整骨架），再逐块下钻：
- 输出层/unembedding · Next-Token Prediction · Context Window · Sampling · Temperature · Top-k · Top-p · Tokenization 细节 · BPE
- 衔接：Module 1 停在"最后一层 hidden state"，Module 2 从这里把它变成下一个词。

### Module 3 — LLM Inference Engineering
- Prefill · Decode · KV Cache · FlashAttention · Continuous Batching · Speculative Decoding · vLLM

### Module 4 — Training
- Loss · Gradient Descent · Backprop · Pretraining · SFT · RLHF · DPO
- 原则：理解流程与核心思想，不追求成为 ML researcher。

### Module 5 — AI Engineering
- Prompt Engineering · Embedding Models · RAG · MCP · Agents · Memory · Evaluation · AI System Architecture
- **Outcome**：能设计一个完整 AI 应用系统。

### Module 6 — Advanced（按兴趣）
- MoE · LoRA · PEFT · Diffusion · Multimodal · Reasoning Models · Long Context · Mamba · SSM · AI Coding Agents · Latest Research

---

## Current Status

**Current Module:** Module 2 — Large Language Models（进行中）
**Current Lesson:** Greedy 与 Sampling（`15-sampling-basics.html`，进行中）

### Completed
- **Token** — 基本掌握（诊断 5 题中 Q2/Q3/Q4 正确）
  - ✅ token ≠ 单词（subword，如 `liked → like + d`）
  - ✅ 不同模型用不同 tokenizer，切分结果可不同
  - ✅ 同一 tokenizer 对同一文本是确定性的
  - ✅ 误区 A 已补：tokenizer 输出是整数序列，不是矩阵；"变矩阵"发生在 embedding 层
  - ✅ 误区 B 已补：transformer 吃的是向量，不是整数 id
- **Embedding** — ✅ 通过，mental model 完整正确
  - ✅ token id = 行号/座位号（index），无语义，可任意分配但需前后一致
  - ✅ embedding = 查表取多维向量，按序叠成 `[N × d_model]` 矩阵
  - ✅ 意义存在于向量在高维空间的"位置"；相近 token 位置近，方向也有含义
  - ✅ 区分两种训练：① tokenizer(BPE) 建词表并冻结；② 模型训练只改向量数值，index 不迁移
- **Vector / 点积** — ✅ 通过（讨论中证明掌握，`lessons/03-vector.html`）
  - ✅ 点积 = 对应位置相乘求和 = 衡量两向量"对齐程度" = 相似度
  - ✅ 点积大 ⟹ 该关注（attention 里）；但"点积大 ⟺ 方向最像"❌（长度会掺入，纯方向用余弦）
  - ✅ 深挖：长度不是绝对重要性、是模型可用的自由度；"重要"是上下文相关
- **softmax** — ✅ 通过（`lessons/04-softmax.html`，3 题全对）
  - ✅ 两步：指数化（e^x，全变正、放大大的）+ 归一化（÷总和 → 和为 1）
  - ✅ 性质：和为1/全非负、单调、soft（非硬 argmax）；独大分数→接近 one-hot
  - ✅ 为何用 e^x：朴素"÷总和"遇负数会崩（负权重/÷0/顺序反）；e^x 先变正
- **朴素注意力** — ✅ 通过（`lessons/05-naive-attention.html`，3 题全对）
  - ✅ 三步：点积打分 → softmax 成权重 → 加权求和混合上下文
  - ✅ 核心直觉：注意力=按相似度对上下文做加权平均；新向量=邻居向量的加权平均
  - ✅ 洞察：xᵢ·xᵢ=|xᵢ|² 恒正且方向一致 → 朴素版天生最关注自己（后面要松绑的动机）
- **Query / Key / Value** — ✅ 通过（`lessons/06-qkv.html`，3 题全对）
  - ✅ 三个学出来的矩阵 W_Q/W_K/W_V 把 x 投影成三个角色向量：q（我找谁）/ k（谁找我）/ v（交出的内容）
  - ✅ 修朴素版：q·k 打分不再对称、不再最爱自己；混合用 v 而非 x
  - ✅ 深挖：q/k/v 是 x 的平级投影，差异源于用途；不预存 q/k/v（x 随上下文/层变化）
  - 🔧 纠正用词：W_Q/W_K/W_V 是矩阵(变换)，不是向量；q=x·W 是"向量经矩阵变换"
- **缩放点积 ÷√d** — ✅ 通过（`lessons/07-scaled-dot-product.html`，3 题全对）
  - ✅ 维度 d 大 → 点积和的波动≈√d → 分数天然大（非更相关）
  - ✅ 分数大 → softmax 落 e^x 陡峭区 → 一家独大 + 梯度饱和难训练
  - ✅ ÷√d 把落点拉回平缓区（几何：横向撑开曲线）；不改排序、只缩小分差
  - ✅ 完整单头注意力公式闭环：softmax(QKᵀ/√d)·V
- **Multi-head Attention** — ✅ 通过（`lessons/08-multihead.html`，3 题全对）
  - ✅ 动机：单头只表达一种关系 → 多头并行各看一种（指代/句法/邻接…）
  - ✅ 维度流：8 头各 [N×64]，concat 回 [N×512]，再过 W_O(512×512)
  - ✅ 8 组独立 (W_Q,W_K,W_V)（≠ Q/K/V 是三个头）；头数 vs 角色数是两个轴
  - ✅ 分化机制：随机初始化(起点不同) + 各头不同梯度(确定但不同) + 降误差压力奖励分工
  - 深挖沉淀：补充 A（X·W=Q 形状/矩阵=变换）、补充 B（梯度直觉，完整版 Module 4）
  - 🔧 纠正：N=词数 vs d_model=向量维度；单头 d_k 自由(常=d_model)，多头才 =d_model/h；梯度非随机(确定)、随机的是初始值
- **残差连接 Residual** — ✅ 通过（`lessons/09-residual.html`，3 题基本正确）
  - ✅ 核心：output = x + Layer(x)；Layer 只算 delta(偏移量)，不算绝对值
  - ✅ 三好处：① 梯度高速路(捷径 skip connection 导数=1，1+f' 连乘不塌→不消失) ② 易学恒等(f≈0 即原样通过，加层最差不变差) ③ 逐层微调(向量 += delta)
  - ✅ 维度对齐：x+Layer(x) 要求同形状 [N×d_model] → d_model 恒定的又一理由
  - 深挖沉淀(FAQ)：x+f(x) 与 g(x) 表达能力相同，残差赢在优化(梯度不消失+inductive bias 锚在 x)，非表达能力；g(x)=普通堆叠层
  - 🔧 Q2 修正：梯度不消失的直接原因是"捷径给梯度留了导数=1 的直通路"，不是"带了起点信息"(那是好处②)
  - 📌 名词：跳连/捷径 = skip connection（shortcut）
- **LayerNorm** — ✅ 通过（`lessons/10-layernorm.html`，3 题全对）
  - ✅ 对每个词向量单独：减均值除标准差 → 均值0标准差1，再 γ·x̂+β（γ/β 学出来，初始 γ=1 β=0）
  - ✅ 作用：防数值漂移（忽大忽小/单维拉爆）→ 梯度稳、可用更大学习率
  - ✅ "Layer"=在一个词向量内部统计，不跨词/不跨样本；和缩放点积 ÷√d 同精神
  - ✅ 与残差搭档 = Add & Norm：x = LayerNorm(x + 子层(x))
  - 深挖沉淀(FAQ)：控的是数值分布(非仅长度)+居中；γ/β 可控不失控(参数被梯度监督+下层再归一化，非被动漂移)；"均值0标准差1"vs"γ/β学"是两个时间点
- **FFN（前馈网络）** — ✅ 通过（`lessons/11-ffn.html`，3 题全对）
  - ✅ 结构：W₁ 放大(512→2048) → 激活(ReLU/GELU 非线性) → W₂ 缩回(2048→512)
  - ✅ 非线性是灵魂：无激活则两矩阵相乘还是线性、白叠；激活引入折线拐点→能逼近复杂函数
  - ✅ 逐词独立(position-wise)，不看别的词；所有位置共用同一套 W₁/W₂
  - ✅ 分工：注意力=交流(开会)、FFN=计算/提炼(会后复盘)；一个 Block = 交流→提炼
  - ✅ 参数大户：d_ff=4×d_model → FFN 约占 2/3 参数；知识多编码于此
  - 深挖沉淀(FAQ)：激活=门控(非0/1二元化)；FFN=提炼(非淘汰)；注意力更新的是表示非W参数
  - 📌 名词：激活函数=activation function；两种weight(模型权重 vs 注意力权重)已入补充D
- **输出层 / Unembedding** — ✅ 通过（`14-output-unembedding.html`，3 题全对）
  - ✅ h_last [1×d_model] × W_U [d_model×vocab_size] → logits [1×vocab_size]
  - ✅ W_U 每列是一个候选 token 的匹配方向；hidden state 是当前语境需求，不是某个 token embedding 的复制品
  - ✅ logits 可正可负；softmax → 0~1 正小数且总和1；采样后才得到整数 token id
  - ✅ 词表不是新对象：输入 token→id、输出 id→token 复用同一 vocabulary
  - ✅ W_U 可独立训练，也常用 weight tying：W_U=Eᵀ；方向相反但 embedding/unembedding 非严格逆运算

### Open Questions
- （无）

### Next Lesson
- 采样：greedy vs sampling；temperature / top-k / top-p 如何改变候选分布

---

## 待讲清单 (Parking Lot)
> 已在讲解中提到、但承诺"后面再展开"的概念。讲到时勾掉。

**数学补充讲义**：`lessons/A-matrix-as-transform.html`（矩阵=变换 vs 向量=料，Q/K/V 打分不对称的数学根源）
**总览/复习**：`lessons/overview-attention-pipeline.html`（注意力四步全链：q·k→÷√d→softmax→加权和 v；每步对应哪一课）

| 概念 | 何处提到 | 计划何时讲 |
|---|---|---|
| softmax（把分数变成和为 1 的权重） | lesson 03/04 | ✅ 已讲（`lessons/04-softmax.html`） |
| ÷√d 缩放（scaled dot-product） | lesson 04 公式 | ✅ 已讲（`lessons/07-scaled-dot-product.html`） |
| Multi-head Attention（多头） | lesson 04 | ✅ 已讲（`08-multihead.html`） |
| W_O 注意力输出投影 | 概念问答 | ✅ 已讲（Multi-head 08 中） |
| 梯度 / 反向传播 / 梯度下降 | 多头分化讨论 | 🅿️ 直觉预告已给（`B-gradient-intuition.html`）；完整版 Module 4 |
| FFN（前馈网络） | roadmap | ✅ 已讲（`11-ffn.html`） |
| 残差连接 Residual | roadmap | ✅ 已讲（`09-residual.html`） |
| LayerNorm | roadmap | ✅ 已讲（`10-layernorm.html`） |
| 位置编码 / RoPE（lesson 04 提到"位置信息"） | lesson 04 | Module 1 |
| 输出层 / unembedding（映射回词表） | 概念问答 | ✅ 已讲（`14-output-unembedding.html`） |
| 多层堆叠（Stacking N Transformer Blocks / depth） | 多次提及（W 每层一套、深层 x 非固定） | Module 1 收尾（先学完单个 block 的零件） |
| KV Cache（缓存已算出的 k/v） | qkv 预存讨论 | Module 3 |
| 因果掩码 causal mask（每个位置只能看前面的词） | M2 概览讨论（最后位置预测） | Module 2 下钻（注意力/训练相关处） |
| chat template + 指令微调(SFT/RLHF)（一问一答怎么实现） | M2 概览讨论 | Module 4（训练）/ Module 5 |

---

## Lesson Log

### 2026-07-20 — Token（诊断）
- 形式：5 题诊断，测 mental model 而非记忆
- 结果：Q2/Q3/Q4 ✅；Q1（把"变矩阵"错记到 tokenizer 头上）、Q5（以为整数直接喂 transformer）需修
- 结论：不重学 Token，直接进 Embedding，两个缺口在 Embedding 课自然补上

### 2026-07-20 — Embedding（✅ 完成）
- 首次采用 HTML 输出：`lessons/02-embedding.html`（含 SVG 查找表图 + 向量空间直觉图 + 两段沉淀问答）
- 覆盖：Why（整数不能直接喂 NN）/ How（token id = 行号，查表取向量）/ 意义来自训练学出的向量位置
- 深入问答沉淀进讲义：① id 编号任意但需一致；② 两种训练，index 不迁移、只改向量
- 学生总结完整正确，判定通过
- 追加 FAQ：tokenizer 锁死/重训、重训三档（预训练 / 词表迁移 / 蒸馏）

### 2026-07-22 — Vector / 点积（✅ 完成）
- `lessons/03-vector.html`：点积=相似度、方向 vs 长度、通往 attention 的桥
- 深挖讨论：点积大≠方向最像（长度反例 [3,4] vs [1,1]）；长度非绝对重要性、是模型自由度
- 未正式答检查题，但讨论中已充分证明掌握，判定通过

### 2026-07-22 — Self-Attention（旧 lesson 04 作废，改为微课链）
- 反馈：原 lesson 04 一次塞入 Q/K/V+softmax+√d+多头，违背"一课一概念"
- lesson 03 也把 点积/相似度/注意力权重 混谈 → 已补"本课只需掌握 + 🅿️后面讲"标注，明确概念层次链
- 重构为微课链：softmax → 朴素注意力 → Q/K/V → scaled score → Multi-head
- 新增「教学颗粒度原则」到文档

### 2026-07-22 — softmax（✅ 完成）
- `lessons/04-softmax.html`：分数→(指数化+归一化)→和为1的权重；性质；为什么用 e^x
- 加 y=e^x 曲线图（永远>0 + 越右越陡）；FAQ 补"为何负数不能直接÷总和"、"为何要和为1"
- 3 题全对，判定通过

### 2026-07-23 — 朴素注意力（✅ 完成）
- `lessons/05-naive-attention.html`：只用点积+softmax+加权和，建立"按相似度混合上下文"核心直觉，不引入 Q/K/V
- §④ 留悬念：单一向量兼任 比较/被比较/提供内容 三角色 → 下一课 Q/K/V 拆开
- 3 题全对；学生自己推出 xᵢ·xᵢ=|xᵢ|²、空间上向相关邻居偏移
- 判定通过

### 2026-07-23 — Query/Key/Value（✅ 完成）
- `lessons/06-qkv.html`：从朴素版两毛病（角色纠缠、对称最爱自己）引出 Q/K/V 三投影
- FAQ 沉淀：q/k/v vs x 关系（平级投影，用途决定差异，k=广告词/v=货）；为何不预存 q/k/v
- 待讲清单加：多层堆叠、KV Cache
- 3 题全对；纠正"矩阵 vs 向量"用词（W 是变换矩阵）
- 判定通过

### 2026-07-23 — 缩放点积 ÷√d（✅ 完成）
- `lessons/07-scaled-dot-product.html`：维度高→点积和的std≈√d→分数偏大→softmax饱和成one-hot；÷√d 拉回温和范围
- 承接 softmax"独大"性质 + Q/K/V 打分；完整公式 softmax(QKᵀ/√d)V 拼齐
- 加"e^x 落点"图（陡峭区20 vs 平缓区2.5，横向撑开直觉）
- 附加产出：数学补充 A（矩阵=变换）、总览讲义（注意力全链）、index 目录页 + 各课导航条
- 3 题全对，判定通过

### 下次预告 — E. Multi-head Attention
- 学生已提前踩到门口：512维拆成若干64维head、算完拼回512；W_O 整合
- 动机：单头只有一种"关注视角"，多头并行捕捉不同关系

### 2026-08-04 — Module 2 概览 + 自回归/因果掩码（完成）
- 采用自顶向下：先 12 概览（生成链路骨架 + 自回归循环），再 13 下钻（每位置预测下一词 + 因果掩码）
- 12 修 SVG bug（text 元素内不能放 HTML 标签）；改用常驻本地克隆 repos\jtr109.github.io（不再反复 clone）
- FAQ 沉淀：最后位置预测下一词、因果掩码、聊天=拼接续写、模型无状态、稳定性→KV Cache（Module 3 伏笔）
- 学生自己推出「模型无状态 + 前面位置可缓存」，理解很深
