# 多维表格 Skill 设计范式分析：面向 Agent 的企业级 Skill 架构

## 1. 整体架构概览

### 1.1 文件结构

```
/skills/multitable-base/
├── SKILL.md                              # 主入口文件（总控层）
└── references/                           # 参考文档库（80+ 文件）
    ├── formula-field-guide.md            # 公式字段完整指南（737 行）
    ├── lookup-field-guide.md             # 查找引用字段完整指南
    ├── multitable-base-data-query.md           # 聚合分析权威参考
    ├── multitable-base-workflow-schema.md      # Workflow 数据结构字典
    ├── multitable-base-shortcut-field-properties.md  # 字段 Schema SSOT
    ├── multitable-base-shortcut-record-value.md      # 记录值 Schema SSOT
    ├── role-config.md                    # 角色权限配置详解
    ├── examples.md                       # 综合示例库
    └── multitable-base-*.md                   # 70+ 原子命令参考文档
```

**统计**：
- 1 个主 Skill 文件（总控）
- 6 个领域指南（深度规范）
- 70+ 个命令参考文档（逐命令覆盖）
- 2 个 Schema SSOT 文档

### 1.2 核心设计理念

> **"文档即协议，规范即执行"**

文档不是给人读的参考，而是给 Agent 执行的操作协议。每个命令文档都是自包含的执行单元，通过 CLI 层强制门禁确保 Agent 遵循协议，将主观判断转化为确定性执行。

---

## 2. 分层文档架构

### 2.1 四层架构模型

```
┌─────────────────────────────────────────┐
│  Layer 1: 总控层（SKILL.md）             │
│  ├─ 核心规则 13 条 + 禁止行为 6 条       │
│  ├─ 意图 → 命令索引表                   │
│  ├─ 工作流决策树                        │
│  └─ 错误码速查表                        │
├─────────────────────────────────────────┤
│  Layer 2: 领域指南（Domain Guide）       │
│  ├─ formula-field-guide.md             │
│  ├─ lookup-field-guide.md              │
│  ├─ data-query.md                      │
│  └─ workflow-schema.md                 │
├─────────────────────────────────────────┤
│  Layer 3: 命令参考（Command Reference） │
│  └─ 每个命令一个文档（70+ 文件）         │
├─────────────────────────────────────────┤
│  Layer 4: Schema 稳定层（Schema SSOT）  │
│  ├─ shortcut-field-properties.md       │
│  └─ shortcut-record-value.md           │
└─────────────────────────────────────────┘
```

### 2.2 层间导航规则

- **自顶向下可导航**：从用户意图 → 领域 → 命令 → Schema，每一层都有明确入口
- **自底向上可溯源**：每个命令文档都链接回上层指南和 SKILL.md
- **横向互链**：相关命令、相关 Schema 互相引用，信息不孤立

### 2.3 命令文档统一模板

所有 70+ 个命令文档遵循同一模板结构：

```markdown
# base +<command-name>

> **前置条件：** 链接到 shared-base/SKILL.md

## Agent 最小工作流     ← 专为 AI 设计的执行顺序
## 推荐命令             ← 可复制粘贴的完整 CLI 示例
## 参数                 ← CLI 参数表
## API 入参详情         ← HTTP 方法、路径、Body 结构
## JSON 值规范          ← --json 格式要求与示例
## 返回重点             ← 关键返回字段说明
## 工作流               ← 执行前/后的依赖步骤
## 坑点                 ← 常见错误与注意事项（前置显示）
## 参考                 ← 相关文档链接
```

**关键设计**：每个文档自包含，Agent 读完一个文件就能正确执行该命令。

---

## 3. 核心设计模式

### 3.1 强制阅读门禁（Hard Gate）

这是整个设计中最关键的稳定性机制：

```bash
# 创建公式字段时，不带 --i-have-read-guide 会直接失败
base-cli base +field-create \
  --json '{"type":"formula","expression":"..."}' \
  --i-have-read-guide    # ← 必须先读 formula-field-guide.md
```

**三层联动机制**：

```
文档层：formula-field-guide.md 开头标注
        "必须先读本 guide，再加 --i-have-read-guide"
         ↓
规则层：SKILL.md 明确禁止
        "没读 guide 前不要直接创建 formula/lookup 字段"
         ↓
执行层：CLI 校验参数
        没有该参数 → fail fast + 返回 guide 链接
```

**价值**：强制 Agent 在执行高风险操作前获取完整的领域语法知识，将"主观构造"转变为"规范驱动"，显著降低因语法错误导致的执行失败率。

### 3.2 意图 → 命令索引表

SKILL.md 内置自然语言到命令的映射表，消除 Agent 选错命令的不确定性：

| 用户意图 | 推荐命令 | 关键决策提示 |
|----------|---------|-------------|
| 聚合分析 / 比较排序 / 求最值 | `+data-query` | 不要用 `+record-list` 拉全量再手动计算 |
| 列表 / 获取记录 | `+record-list` / `+record-get` | 如需聚合，走 `+data-query` |
| 创建 / 更新公式字段 | `+field-create` / `+field-update` | 先读 formula guide |
| 创建 / 更新 lookup 字段 | `+field-create` / `+field-update` | 先读 lookup guide |

### 3.3 双层规范约束

```
通用规则（SKILL.md，适用所有命令）
   ├─ 只使用原子命令，不走原始 API
   ├─ 写记录前先读字段结构
   ├─ 所有 +xxx-list 禁止并发
   └─ 统一参数名 --base-token（不使用旧 --app-token）

领域规则（Guide，每个 Guide 10+ 条硬约束）
   ├─ Formula: 函数白名单、禁止嵌套、精确匹配字段名
   ├─ Lookup: 必须写 where、tuple 格式、snake_case 枚举值
   └─ Data-Query: 字段类型白名单、alias 不支持中文
```

### 3.4 反例库（Anti-Pattern 积累）

每个领域 Guide 和命令文档都包含"坑点"章节，基于真实用户错误持续积累：

```markdown
## 坑点

❌ 错误：用数字枚举表示字段类型
{"type": 3, "property": {...}}

✅ 正确：用字符串枚举
{"type": "select", "multiple": false, ...}

❌ 错误：日期字段传字符串
{"date": "2024-01-01"}

✅ 正确：日期字段传毫秒时间戳
{"date": 1704067200000}
```

---

## 4. 复杂意图处理与工作流组装

### 4.1 决策树驱动的意图分类

SKILL.md 内置工作流决策树，Agent 面对模糊意图时有确定性路径：

```
用户意图
├─ "统计 / 分析 / 求和 / 比较" → 临时查询 → +data-query
│   ├─ 要的是"这次算出来的结果"
│   └─ 支持：分组 / SUM / AVG / COUNT / MAX / MIN
│
├─ "长期显示 / 派生指标 / 计算字段" → 公式字段
│   ├─ 先读 formula-field-guide.md
│   └─ 创建时带 --i-have-read-guide
│
└─ "获取记录 / 列表 / 查询明细" → +record-list / +record-get
    └─ 如需聚合，切回 +data-query
```

**示例**：
- "统计每个城市的订单总额" → 临时查询 → `+data-query`
- "计算每个订单的利润率，长期显示在表里" → 派生字段 → 公式字段
- "各订单的平均利润率" → 临时查询 → `+data-query`

### 4.2 上下文传递机制

**核心规则**：先拿结构，再写命令。

```
执行任何写操作前的强制前置步骤：

Step 1: +table-list → 获取所有表名（精确匹配，禁止猜测）
Step 2: +field-list → 获取字段列表（获取真实字段名和类型）
Step 3: 跨表场景 → 额外查询目标表的字段列表
Step 4: 基于真实结构构造命令参数
```

**跨表公式的上下文依赖链示例**：

```bash
# 用户意图：订单表中，计算每个客户的历史订单总额

# Step 1: 读 formula 指南
cat formula-field-guide.md

# Step 2: 获取订单表字段（确认 [客户] 字段的 link_table）
base-cli base +field-list --table-id tbl_order

# Step 3: 获取客户表字段（获取可引用字段列表）
base-cli base +field-list --table-id tbl_customer

# Step 4: 构造公式（精确引用字段名）
base-cli base +field-create \
  --table-id tbl_order \
  --json '{"type":"formula","name":"历史订单总额",
           "expression":"[订单表].FILTER(CurrentValue.[客户]=[客户]).[金额].SUM()"}' \
  --i-have-read-guide
```

### 4.3 多步工作流组装

**案例**：创建包含公式字段的完整数据表

```bash
# Step 1: 创建基础表（含基础字段）
base-cli base +table-create \
  --name "订单表" \
  --fields '[{"name":"订单号","type":"text"},
             {"name":"金额","type":"number"},
             {"name":"成本","type":"number"}]'

# Step 2: 验证点——读取字段列表确认创建成功
base-cli base +field-list --table-id tblXXX

# Step 3: 读取 formula 指南（获取语法知识，强制门禁）

# Step 4: 创建公式字段
base-cli base +field-create \
  --table-id tblXXX \
  --json '{"type":"formula","name":"利润率",
           "expression":"([金额] - [成本]) / [金额]"}' \
  --i-have-read-guide

# Step 5: 写入记录（公式字段自动过滤，不写入）
base-cli base +record-upsert \
  --table-id tblXXX \
  --json '{"订单号":"001","金额":1000,"成本":600}'
```

**关键控制点**：
- Step 2 作为验证检查点，确认字段存在再引用
- Step 3 的强制门禁确保语法知识完备
- Step 5 通过 shortcut-record-value.md 知道公式字段只读，不写入

---

## 5. Schema 稳定性设计

### 5.1 三层 Schema 定义

```
Layer 1: JSON Schema 形式规范（Draft-07）
   ├─ shortcut-field-properties.md
   ├─ shortcut-record-value.md
   └─ 每个字段类型独立 Schema，带完整校验规则

Layer 2: 值格式表（内容规范）
   ├─ 按字段类型分类（2.1 text / 2.2 number / 2.3 select ...）
   ├─ 每种类型的推荐值格式与示例
   └─ 完整可复制的 JSON 示例

Layer 3: 反例库（错误模式）
   ├─ 常见错误格式（对比展示）
   ├─ 禁止使用的旧格式标注
   └─ 错误原因说明
```

### 5.2 字段类型与值格式强绑定

每种字段类型都有精确的值格式规范，防止混用：

```markdown
## 2.3 select（单选/多选）
- 单选：字符串 → "选项A"
- 多选：字符串数组 → ["选项A", "选项B"]
❌ 禁止：{id: "opt_xxx"}（旧格式）

## 2.7 user（人员字段）
- 必须传：[{id: "ou_xxx"}]
❌ 禁止：传字符串（无法解析用户 ID）

## 2.9 date（日期字段）
- 必须传：毫秒时间戳（数字）→ 1704067200000
❌ 禁止：字符串 "2024-01-01"
```

### 5.3 版本化废弃管理

**明确标注废弃格式，防止 Agent 使用旧 API**：

```markdown
## 1. 顶层规则（必须遵守）
❌ 不要使用旧结构：
   - field_name（旧参数名）
   - property（旧属性字段）
   - ui_type（旧类型标识）
   - 数字枚举 type（如 3 代表单选）

✅ 当前正确格式：
   - name（参数名）
   - type: "select"（字符串枚举）
```

**API 版本路径显式标注**：

```markdown
## API 入参详情
POST /open-apis/example/path

⚠️ 注意：路径是 base/v3，不是旧版 bitable/v1
```

### 5.4 Schema 可校验性保障

所有 Schema 遵循 JSON Schema Draft-07 标准，支持工具校验：

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "type": {
      "type": "string",
      "enum": ["text", "number", "select", "date", "user", "formula"]
    }
  },
  "required": ["name", "type"],
  "additionalProperties": false
}
```

**可验证路径**：CLI 层使用同一 Schema 校验请求，错误时返回精确字段路径（如 `$.properties.type: 期望字符串枚举，收到数字 3`）。

---

## 6. 可维护性设计分析

Skill 的长期可维护性取决于：当底层 API、字段类型或业务规则发生变更时，需要同步修改的文档范围（变更影响域）是否可控。multitable-base 通过分层架构与 SSOT 原则，将不同类型的变更限制在最小的影响域内。

### 6.1 变更场景与影响域分析

#### 场景一：新增字段类型

**影响域**：仅 Schema 稳定层（2 个文档）

```
需要更新：
├─ shortcut-field-properties.md  ← 新增该类型的 properties Schema
└─ shortcut-record-value.md      ← 新增该类型的读写值格式规范

无需更新：
├─ 所有命令文档（field-create / record-upsert 等）
├─ SKILL.md 核心规则
└─ 领域 Guide
```

**原因**：字段类型的 Schema 定义集中在 SSOT 文档，命令文档通过引用获取类型信息，不内联具体类型定义。新字段类型的 Agent 使用路径（`+field-create` 命令）无需变更，Agent 会在运行时读取 Schema SSOT 获得新类型的值格式。

#### 场景二：已有字段类型的属性变更

**影响域**：Schema 稳定层 + 对应的领域 Guide（如有）

```
需要更新：
├─ shortcut-field-properties.md  ← 修改该类型的 properties 定义
├─ shortcut-record-value.md      ← 如读写格式变化，同步更新
└─ 对应 Guide（如公式函数新增）  ← 如 formula-field-guide.md 函数白名单

无需更新：
└─ 命令文档（命令接口本身未变）
```

**关键约束**：废弃的旧格式必须在 Schema 文档中显式标注（而非删除），以防止 Agent 因训练数据中的旧格式记忆而产生回归错误。

#### 场景三：新增 CLI 命令

**影响域**：新增 1 个命令文档 + SKILL.md 意图索引表

```
需要新增：
└─ references/multitable-base-<new-command>.md    ← 按模板填写

需要更新：
└─ SKILL.md 意图索引表                      ← 新增意图→命令映射行

无需更新：
├─ 所有已有命令文档
├─ 领域 Guide
└─ Schema SSOT 文档
```

**模板保障**：命令文档的统一模板使新增命令具有确定性结构，不依赖作者主观组织，消除文档间的风格漂移。

#### 场景四：API 路径或参数变更（如版本升级）

**影响域**：仅变更涉及的命令文档

```
需要更新：
└─ 对应命令文档的「API 入参详情」章节

需要评审：
└─ 如参数语义变化，同步更新 SKILL.md 的统一参数规范

无需更新：
├─ 未受影响的命令文档
└─ Schema SSOT 文档（除非值格式同步变化）
```

**风险点**：API 路径版本升级（如 `bitable/v1` → `base/v3`）属于高影响变更，需在受影响命令文档的「坑点」章节显式标注废弃路径，并在 SKILL.md 的禁止行为中同步更新。

### 6.2 影响域隔离的技术保障

上述低影响域的实现，依赖以下三个技术手段：

#### ① SSOT 原则（Single Source of Truth）

字段类型的 Schema 定义唯一存在于 `shortcut-field-properties.md` 和 `shortcut-record-value.md`，命令文档通过文档引用链接获取，不在命令层内联字段类型细节。任何字段类型变更只需修改 SSOT 文档，引用方自动获得最新规范。

#### ② 分层变更频率隔离

架构各层的变更频率差异显著，分层设计使高频变化不会触达低频稳定层：

```
高频（持续追加，不修改已有内容）
   └─ 反例库 / 坑点章节：基于运行时错误增量追加

中频（模块化变更，新增不影响已有文档）
   └─ 命令文档：新增命令为独立文件，不修改已有命令文档

低频（基础设施层，变更需全局评审）
   ├─ SKILL.md 核心规则与禁止行为
   ├─ 领域 Guide（公式语法 / 查找引用规范）
   └─ Schema SSOT 文档
```

#### ③ 模板约束消除文档漂移

所有命令文档强制遵循统一模板，新增文档不会引入结构差异。当需要在所有命令文档中追加某类信息时（如新增「版本兼容性」章节），模板更新即可作为全局规范，而不需要逐文件检查结构是否一致。

---

## 7. 稳定性与确定性保障机制

### 7.1 确定性保障

#### 保障一：禁止行为清单

```markdown
## Agent 禁止行为
- 不要把 +record-list 当聚合分析引擎
- 不要没读 guide 就直接创建 formula / lookup 字段
- 不要凭自然语言猜测表名、字段名、公式字段引用
- 不要把系统字段、formula 字段当成写入目标
- 不要在 Base 场景改走原始 API（/open-apis/example/path）
```

**特点**：每条禁止对应一个真实高频错误，使用"不要……"句式，明确边界。

#### 保障二：强制前置检查

```markdown
## Agent 快速执行顺序（强制）
1. 先判断任务类型（决策树）
2. 先拿结构，再写命令
   ├─ 至少先拿当前表结构（+field-list 或 +table-get）
   └─ 跨表场景必须再查目标表
3. formula / lookup 必须先读 guide
4. 写记录前先判断字段可写性（排除只读字段）
```

#### 保障三：参数名统一

```
一律使用 --base-token（不使用旧 --app-token）
一律使用 --table-id（不使用 --tableId）
一律使用字符串枚举（不使用数字枚举）
```

### 7.2 错误恢复机制

#### 集中式错误码速查表

```markdown
| 错误码 | 含义 | 解决方案 |
|--------|------|---------|
| 1254015 | 字段值类型不匹配 | 先 +field-list，按类型查 shortcut-record-value |
| 1254064 | 日期格式错误 | 用毫秒时间戳，非字符串/秒级时间戳 |
| 1254068 | 超链接格式错误 | 用 {text, link} 对象 |
| param baseToken is invalid | 把 wiki token 当成 base_token | 先用 wiki.spaces.get_node 取真实 obj_token |
```

**自愈流程**：
```
CLI 返回 1254015
  → Agent 查错误码表
  → 找到"字段值类型不匹配"
  → 执行解决方案：+field-list → 查 shortcut-record-value.md
  → 构造正确格式 → 重试成功
```

#### Wiki Token 特殊处理（5 步恢复流程）

```
1. wiki.spaces.get_node 查询节点信息
2. 提取 obj_type 和 obj_token
3. 根据 obj_type 选择后续命令
4. 把 obj_token 当成 base_token 使用
5. 如果已经报了 token 错，回退检查 wiki
```

### 7.3 可验证性：最小可复现示例

每个命令文档的"推荐命令"章节都提供可直接执行的示例：

```bash
base-cli base +data-query \
  --base-token BASE_TOKEN_EXAMPLE \
  --dsl '{
    "datasource": {"type": "table", "table": {"tableId": "tblXXX"}},
    "dimensions": [{"field_name": "城市", "alias": "dim_city"}],
    "measures": [{"field_name": "城市", "aggregation": "count", "alias": "count"}],
    "shaper": {"format": "flat"}
  }'
```

**用途**：
- 文档编写时验证命令正确性（可作为集成测试用例）
- Agent 参考生成真实命令（有完整可复制的模板）
- 用户复现和调试

---

## 8. 与通用 Skill 的设计差异

### 8.1 设计维度对比

| 维度 | 通用 Skill | multitable-base Skill |
|------|-----------|----------------|
| 文档组织 | 单文件平铺（README 形式） | 四层分层架构，按需按层读取 |
| 语法覆盖 | 示例驱动，部分覆盖 | 函数/类型完整白名单，全量规范 |
| 错误处理 | 依赖 Agent 自行推断 | 错误码速查表 + 反例库 + 恢复流程 |
| 意图识别 | Agent 自由映射命令 | 意图索引表，自然语言到命令的强映射 |
| 复杂流程 | 单步命令执行 | 决策树 + 依赖链 + 多步工作流 |
| 知识门禁 | 无强制前置检查 | 高风险操作须经 `--i-have-read-guide` 门禁 |
| 变更影响域 | 全文高耦合，变更易扩散 | 分层隔离，变更影响域最小化 |
| 版本管理 | 无废弃格式标注 | 废弃格式显式标注，API 版本路径明确 |

### 8.2 执行路径对比

**普通 Skill 执行路径**：
```
用户意图 → Agent 理解（主观） → 猜测参数（不确定） → 执行（可能失败）
```

**multitable-base Skill 执行路径**：
```
用户意图
  → 意图索引表（确定命令）
  → 命令文档（确定参数格式）
  → Guide 阅读（确定领域语法）
  → Schema 验证（确定值格式）
  → 执行（高确定性）
```

**核心差异**：在用户意图与实际执行之间插入了若干"确定性转换"节点，将 Agent 的主观推断逐步替换为文档驱动的确定性决策，从而将执行路径的不确定性系统性地消除。

---

## 9. 设计模式复用指南

### 9.1 核心设计模式

#### 模式一：分层文档 + 强制门禁
- **适用**：任何有复杂语法或高错误风险的操作
- **实现**：CLI 参数校验（如 `--i-have-read-guide`）
- **价值**：将主观猜测转化为强制知识获取

#### 模式二：意图索引表
- **适用**：命令数量 ≥ 10，意图映射不直观时
- **实现**：SKILL.md 内置意图→命令映射表，含决策提示
- **价值**：消除 Agent 在多命令间选择的不确定性

#### 模式三：集中式错误码速查表
- **适用**：有固定错误码的 API 或 CLI 工具
- **实现**：SKILL.md 错误码表，命令文档链接引用
- **价值**：错误可诊断、可自愈，不需要人工介入

#### 模式四：Schema SSOT（单一真理源）
- **适用**：有复杂 JSON 入参的 API
- **实现**：独立 Schema 文档（JSON Schema Draft-07），命令文档引用
- **价值**：Schema 维护一处，所有命令受益；支持工具校验

#### 模式五：反例库持续积累
- **适用**：所有面向用户的 API/CLI
- **实现**：每个 Guide 和命令文档的"坑点"章节，基于真实错误追加
- **价值**：历史错误转化为知识资产，降低未来错误率

### 9.2 适用场景

**推荐使用此设计范式**：
- ✅ 命令数量 ≥ 20，参数复杂（每个命令 5+ 参数）
- ✅ 有强类型要求（字段类型与值格式强绑定）
- ✅ 跨命令工作流（命令之间有依赖关系）
- ✅ 低容错场景（生产数据，错误代价高）
- ✅ 需要 Agent 自主完成多步任务

**可简化的场景**：
- ➖ 命令数量 < 10，参数简单
- ➖ API 格式松散，容错性高
- ➖ 纯查询场景，无写操作
- ➖ 一次性执行，无复杂流程

### 9.3 落地路径建议

如果要将此范式应用于新 Skill，建议按以下优先级落地：

```
Phase 1（必选，1 周）
   ├─ SKILL.md：核心规则 + 禁止行为 + 意图索引表
   └─ 命令文档模板：统一结构，包含坑点章节

Phase 2（高价值，2 周）
   ├─ 错误码速查表
   ├─ Schema SSOT 文档
   └─ 1-2 个关键领域的 Guide

Phase 3（完善，持续）
   ├─ 反例库积累（基于用户反馈）
   └─ 强制门禁机制（CLI 层集成）
```

---

## 附录：文档规模参考

| 指标 | 数值 |
|------|------|
| 命令文档数量 | 70+ |
| 公式函数白名单 | 80+ 个函数（8 大类） |
| 核心规则数量 | 13 条规则 + 6 条禁止行为 |
| 文档模板复用率 | 100%（所有命令文档遵循同一结构模板） |
