---
name: calculus-dbti
description: "Use this skill when the user or agent wants to take a DBTI personality assessment, do a trading personality quiz, get investment style label, DBTI test, 做题, DBTI测评, 投资人格测试, 交易风格测评, Agent人格标签, agent personality, investment personality test, 币圈人格, crypto personality"
license: MIT
metadata:
  author: calculus
  version: "2.0.0"
  homepage: "https://github.com/alex-hoo-dev/calculus-dbti-skills"
---

# DBTI 币圈人格测试

回答 16 道二选一场景题，测出你是 16 个币圈原型中的哪一个——HODL、APE、DEGEN、WHALE、BUILDER 等。

## 概述

DBTI 沿四个加密原生维度打分，4 字母代号映射到 16 个 meme 原型之一。

| 维度 | 负极（大写） | 正极（小写） |
|------|-------------|-------------|
| 风险偏好 | `D` 保守 Defensive 🛡️ | `d` 激进 Daring 🎰 |
| 时间视野 | `B` 短期 Brief ⚡ | `b` 长期 Broad 🏔️ |
| 决策依据 | `T` 技术派 Technical 📊 | `t` 叙事派 Thematic 📣 |
| 持仓风格 | `I` 集中 Intensive 🎯 | `i` 分散 Inclusive 🌊 |

一个 4 字母代号（如 `dBtI`）会映射到 **DEGEN 堕落者** 原型，金句"归零了？反正来的时候也是零"。

## API 地址

```
http://localhost:8000/api
```

> 生产环境请替换为实际 API 主机名。

## 执行流程

### 第一步 — 获取题目 + 钱包检测

两件事合在一步静默完成。**不要**询问用户钱包相关问题，**不要**建议登录。

先获取题目（调用 API 必须用 `curl`，**不要**用 WebFetch/Fetch）：

```bash
QUIZ_JSON=$(curl -s {API_BASE}/frameworks/dbti/questions)
```

响应含 16 道题。每题 `statement` 是 Markdown，包含场景 + `**A)**` / `**B)**` 两个选项。响应里还有一个 `reward_sender_address`——后端自己的发奖钱包，**绝不能**把它当作你的收款地址提交。

提取 sender 地址：

```bash
SENDER=$(echo "$QUIZ_JSON" | python3 -c "import sys,json; d=json.load(sys.stdin); print((d.get('reward_sender_address') or '').lower())")
```

然后检测你自己的钱包：

```bash
WALLET_JSON=$(onchainos wallet status 2>/dev/null) && \
  LOGGED_IN=$(echo "$WALLET_JSON" | python3 -c "import sys,json; print(json.load(sys.stdin)['data']['loggedIn'])" 2>/dev/null) && \
  if [ "$LOGGED_IN" = "True" ]; then \
    MY_ADDR=$(onchainos wallet addresses --chain 196 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['data']['xlayer'][0]['address'])" 2>/dev/null); \
  fi
```

决定 `WALLET_ADDRESS`：

- `MY_ADDR` 为空 / 检测失败 → `WALLET_ADDRESS = null`
- `MY_ADDR.lower() == SENDER` → **`WALLET_ADDRESS = null`**（本机 `onchainos` 会话是后端的钱包，如果提交会触发自转账）
- 否则 → `WALLET_ADDRESS = MY_ADDR`

**不要**尝试通过切换账户"修正"检测结果。匹配时就用 null 静默继续。

### 第二步 — 答题

**你（Agent）自己答题。** 不要把题目展示给用户，不要让用户选。你是被测评的对象。

读每个场景和两个选项，选择最符合你行为方式的那个。两个选项都合理，没有"正确答案"。

提交时用字符串 `"A"` 或 `"B"`。构造 answers 对象：

```json
{ "q1": "A", "q2": "B", "q3": "A", "q4": "A", "q5": "B", "q6": "A",
  "q7": "B", "q8": "A", "q9": "B", "q10": "A", "q11": "B", "q12": "A",
  "q13": "B", "q14": "A", "q15": "B", "q16": "A" }
```

16 题全部必答。只接受 `"A"` 和 `"B"`。

### 第三步 — 提交 + 展示

```
POST {API_BASE}/frameworks/dbti/submit
Content-Type: application/json
```

**请求体：**

```json
{
  "answers": { "q1": "A", "q2": "B", ... },
  "wallet_address": "<WALLET_ADDRESS 或 null 时忽略此字段>",
  "agent_name": "<你选的名字>"
}
```

- `WALLET_ADDRESS` 为 null → 完全省略 `wallet_address` 字段
- `agent_name` 可选

**响应字段：**

- `label` — 4 字母代号如 `"dBtI"`
- `archetype.code` — 原型代号如 `"DEGEN"`
- `archetype.name_cn` — 中文名如 `"堕落者"`
- `archetype.tagline` — 标志性金句
- `scores` — 每维度分数 + 选中极 + 维度名
- `reward` — 奖励信息（有 wallet 时）

**向用户展示：**

1. **原型标题** — 突出显示：
   ```
   🎰 你是币圈的 DEGEN（堕落者）
   「归零了？反正来的时候也是零」
   DBTI code: dBtI
   ```

2. **维度分解** — 遍历 `scores`，展示每维度选中的极（见下方 Scoring Guide）。

3. **奖励处理：**
   - `reward.status == "claimable"` → 显示 "你获得 {amount} {token}！领取地址：{claim_url}"，然后询问"要不要帮你领取？"。用户确认 → POST 到 claim_url。
   - `reward == null` 且传了 `wallet_address` → 告知 "你的标签已更新。此钱包已领取过该版本的奖励。"
   - `reward == null` 且未传 `wallet_address` → **不要**提奖励。只展示原型和分数。

## 16 个原型

| 代号 | 原型 | 中文名 | 金句 |
|------|------|--------|------|
| dbti | SHILL | 喊单王 | 我不是在 shill，是在做市场教育 |
| dbtI | MAXI | 信仰者 | 其他链？那是证券 |
| dbTi | NERD | 科学家 | 如果你不能量化它，你就不理解它 |
| dbTI | WHALE | 巨鲸 | 你看到的支撑位是我画的 |
| dBti | APE | 梭哈猿 | 白皮书？能吃吗？ |
| dBtI | DEGEN | 堕落者 | 归零了？反正来的时候也是零 |
| dBTi | FLIP | 倒爷 | 持有四小时已经算长线了 |
| dBTI | ALPHA | 猎手 | 你在 Twitter 上看到的已经不是 alpha 了 |
| Dbti | HODL | 钻石手 | 我的止损是归零 |
| DbtI | BAG | 站岗者 | 不卖就不亏 |
| DbTi | GHOST | 潜水者 | 沉默是最好的 alpha |
| DbTI | BUILDER | 建设者 | Bear market is for building |
| DBti | NPC | 路人甲 | 我就买了一点玩玩 |
| DBtI | PAPER | 纸手 | 又卖飞了，今天第三次 |
| DBTi | FARMER | 撸毛党 | 我不看好也不看空，我只看它发不发币 |
| DBTI | FOMO | 追高侠 | 又涨了？今天不买明天来不及了 |

## Scoring Guide

每个维度字母的含义：

- **D** 保守：保护本金，规避高风险交易
- **d** 激进：拥抱波动，愿意为上涨承担显著风险
- **B** 短线：分钟/小时级交易者
- **b** 长线：长期持有，容忍短期波动
- **T** 技术派：链上数据、图表、量化信号驱动
- **t** 叙事派：故事、社区情绪、宏观趋势驱动
- **I** 集中：重仓少数高确信仓位
- **i** 分散：多仓位分摊风险

## 异常情况

- **API 不可达** → 告知用户服务不可用。最多重试一次。
- **onchainos 未安装** → 跳过钱包检测，照常做题（无奖励）。
- **wallet status 失败** → 当作无钱包，静默继续。
- **422 缺题** → 确保 `q1`..`q16` 全部存在。
- **422 非法值** → 只接受 `"A"` / `"B"`（大写，字符串）。
- **已领取过** → 传了 `wallet_address` 但 `reward` 为 null，说明此钱包已领过该版本奖励。

## Skill 联动

- 钱包操作（登录、余额、转账）→ `okx-agentic-wallet`
- 代币价格与市场数据 → `okx-dex-market`
- 安全扫描 → `okx-security`
