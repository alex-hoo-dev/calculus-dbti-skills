# Calculus DBTI Skills

DBTI (Daring/Defensive, Broad/Brief, Technical/Thematic, Intensive/Inclusive) crypto personality test for AI Agents. 16 binary scenario questions map to one of 16 meme archetypes — HODL, APE, DEGEN, WHALE, BUILDER and others.

## Install

```bash
npx skills add alex-hoo-dev/calculus-dbti-skills
```

## What it does

Your AI Agent answers 16 two-choice scenario questions (pick **A)** or **B)**) and receives:

- A 4-letter DBTI code (e.g. `dBtI`)
- A meme archetype (e.g. `DEGEN 堕落者`) with a signature tagline
- Per-dimension score breakdown

If the agent has an OKX Agentic Wallet, it automatically qualifies for a USDT reward on X Layer.

## The 16 archetypes

| Code | Archetype | Tagline |
|------|-----------|---------|
| dbti | SHILL 喊单王 | 我不是在 shill，是在做市场教育 |
| dbtI | MAXI 信仰者 | 其他链？那是证券 |
| dbTi | NERD 科学家 | 如果你不能量化它，你就不理解它 |
| dbTI | WHALE 巨鲸 | 你看到的支撑位是我画的 |
| dBti | APE 梭哈猿 | 白皮书？能吃吗？ |
| dBtI | DEGEN 堕落者 | 归零了？反正来的时候也是零 |
| dBTi | FLIP 倒爷 | 持有四小时已经算长线了 |
| dBTI | ALPHA 猎手 | 你在 Twitter 上看到的已经不是 alpha 了 |
| Dbti | HODL 钻石手 | 我的止损是归零 |
| DbtI | BAG 站岗者 | 不卖就不亏 |
| DbTi | GHOST 潜水者 | 沉默是最好的 alpha |
| DbTI | BUILDER 建设者 | Bear market is for building |
| DBti | NPC 路人甲 | 我就买了一点玩玩 |
| DBtI | PAPER 纸手 | 又卖飞了，今天第三次 |
| DBTi | FARMER 撸毛党 | 我不看好也不看空，我只看它发不发币 |
| DBTI | FOMO 追高侠 | 又涨了？今天不买明天来不及了 |

## Usage

After installation, tell your AI Agent:

```
Help me take a DBTI assessment
帮我做个 DBTI 测评
```

The agent answers 16 questions itself (it is the subject being assessed, not a quiz moderator for you) and shows you the resulting archetype.

## Requirements

- API server running at `http://localhost:8000/api` (or configure the URL in SKILL.md)
- Optional: `onchainos` CLI with wallet login for reward eligibility

## Changelog

- **2.0.0** — Binary A/B format: 16 scenario questions + 16 meme archetypes. Replaces the original 12-question Likert scale.
- **1.0.0** — Initial release: 12 Likert-scale questions, 4-letter label only.
