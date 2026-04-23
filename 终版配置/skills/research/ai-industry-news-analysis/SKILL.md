---
name: ai-industry-news-analysis
category: research
description: Framework for daily AI industry news analysis with multi-perspective investment evaluation
---

# AI Industry News Daily Deep Analysis Skill

## Overview
A structured framework for analyzing AI industry news from multiple disciplinary perspectives (Economist, Historian, Entrepreneur) to provide short/medium/long-term investment recommendations with actionable operational guidance.

## Trigger Conditions
- User requests daily AI industry news analysis
- User needs investment recommendations based on AI sector developments
- User wants multi-perspective analysis of AI company news
- User requires structured output with investment sorting and position recommendations

## Step-by-Step Approach

### 1. **Data Collection Phase** (Strict Time Control)

#### **Search Queries** (Must execute all combinations)
**Chinese Search (5 queries):**
- "AI 人工智能 企业新闻 YYYY年MM月DD日 最新 头条 热度"
- "人工智能 大模型 融资 发布 YYYY年MM月DD日 实时"
- "国内 AI 公司 新闻 YYYY年MM月DD日 排行榜 36 氪 晚点"
- "AI 芯片 机器人 监管 YYYY年MM月DD日 财经"
- "OpenAI 百度 阿里 腾讯 AI YYYY年MM月DD日 动态"

**English Search (5 queries):**
- "AI news today YYYY-MM-DD top trending tech companies"
- "artificial intelligence breaking news YYYY-MM-DD Bloomberg Reuters"
- "AI startup funding acquisition YYYY-MM-DD TechCrunch"
- "OpenAI Google Microsoft AI YYYY-MM-DD latest"
- "AI regulation policy YYYY-MM-DD government"

*(Note: Replace YYYY-MM-DD with current date)*

#### **Authority Source Priority**
- **Tier 1**: 36Kr, LatePost, TechCrunch, Reuters, Bloomberg
- **Tier 2**: The Paper, CNBC, Yahoo Finance
- **Vertical**: AI News, The AI Track, Hugging Face Blog

#### **Time Filtering**
- Time range: Past 24 hours only
- Exclude: >24h old news, pure technical papers, outdated content
- Verification: Each news item must have ≥2 independent sources

### 2. **Heat Index Calculation & Filtering**

**Formula:**
```
Heat Index = (Reads × 0.3) + (Shares × 0.3) + (Social Discussion × 0.2) + (Authority Source × 0.2)
```

**Filtering Rules:**
- Domestic: ≥5 news items
- International: ≥5 news items
- Domain coverage: Large models/Robotics/Chips/Applications/Regulation (2 items each)
- Deduplication: Keep highest heat version for same event

### 3. **Multi-Perspective Deep Analysis**

#### **Economist Perspective**
- GDP impact analysis
- Industry cycle positioning
- Supply-demand dynamics
- Investment multiplier effects
- Market structure changes

#### **Historian Perspective**
- Historical analogies (e.g., 1995-2000 Internet bubble, 2010 mobile internet)
- Evolution stage identification
- Lessons from past tech cycles
- Pattern recognition in funding waves
- Regulatory evolution comparison

#### **Entrepreneur Perspective**
- Commercial value assessment
- Competitive landscape mapping
- Implementation feasibility analysis
- Business model viability
- Go-to-market strategy evaluation

### 4. **Investment Value Extraction**

#### **Short-term (1-3 months)**
- Event-driven catalysts
- Sentiment factors
- Immediate triggers (earnings, product launches, regulatory changes)

#### **Medium-term (3-12 months)**
- Performance realization
- Market share dynamics
- Technology commercialization progress
- Revenue recognition patterns

#### **Long-term (1-3 years)**
- Industry structure reshaping
- Technology paradigm shifts
- Ecosystem building
- Moat development

### 5. **Output Structure**

**Required Sections:**
1. **News Summary** (10 items, sorted by heat index)
2. **Multi-perspective Comprehensive Analysis** (Short/Medium/Long-term)
3. **Investment Sorting & Operational Recommendations**
   - High certainty plays
   - High elasticity plays
   - High-risk high-reward plays
4. **Position Recommendations**
   - Aggressive (80-100% position)
   - Balanced (60-80% position)
   - Conservative (40-60% position)
5. **Key Catalyst Calendar** (Timeline of upcoming events)
6. **Risk Warnings & Response Strategies**

### 6. **Quality Control**

**Verification Checklist:**
- [ ] News authenticity verified (cross-source validation)
- [ ] Data-driven analysis (no subjective speculation)
- [ ] Differentiated viewpoints highlighted
- [ ] Actionable recommendations provided
- [ ] Time-sensitive information marked with timestamps
- [ ] Source attribution for all claims

## Common Pitfalls to Avoid

1. **Source Bias**: Relying on single-source information
2. **Recency Bias**: Overweighting most recent news
3. **Hype Cycle**: Getting caught in technology hype without fundamental analysis
4. **Confirmation Bias**: Only seeking information that supports preconceived views
5. **Lack of Context**: Failing to connect news to broader industry trends
6. **Over-complication**: Creating analysis that's too complex for actionable insights

## Adaptation Notes

- **Timeframe Adjustment**: Modify search window based on user needs (24h, 48h, weekly)
- **Domain Focus**: Customize for specific AI subsectors (LLMs, robotics, chips, applications)
- **Geographic Scope**: Adjust for China-only, US-only, or global coverage
- **Depth Level**: Scale analysis complexity based on user expertise level
- **Update Frequency**: Can be adapted for daily, weekly, or monthly reports

## Example Output Template

```
# AI Industry News Daily Deep Analysis - [Date]

## 1. Top 10 News Items (by Heat Index)
1. [News Title] - Heat: [Score] - Sources: [List]
2. ...

## 2. Multi-Perspective Analysis
### Economist View
- GDP impact: [Analysis]
- Industry cycle: [Positioning]
- Supply-demand: [Dynamics]

### Historian View
- Historical analogy: [Comparison]
- Evolution stage: [Current phase]
- Lessons learned: [Key insights]

### Entrepreneur View
- Commercial value: [Assessment]
- Competitive landscape: [Mapping]
- Implementation: [Feasibility]

## 3. Investment Recommendations
### High Certainty (Core Position)
- [Company/sector]: [Rationale] - Position: [X%]

### High Elasticity (Satellite Position)
- [Company/sector]: [Rationale] - Position: [X%]

### High Risk/Reward (Speculative)
- [Company/sector]: [Rationale] - Position: [X%]

## 4. Position Strategy
- Aggressive: [Allocation breakdown]
- Balanced: [Allocation breakdown]
- Conservative: [Allocation breakdown]

## 5. Catalyst Calendar
| Date | Event | Impact |
|------|-------|--------|
| ... | ... | ... |

## 6. Risk Warnings
- [Risk type]: [Description] - [Mitigation strategy]
```

## Version History
- v1.0: Initial framework based on April 18, 2026 AI news analysis
- v1.1: Added heat index formula and authority source prioritization
- v1.2: Enhanced multi-perspective analysis with specific metrics and focus areas
- v1.3: Updated Apr 19, 2026 with source strategy for empty search results, GitHub/HuggingFace integration, and heat index fallback calculations

## Technical Notes

**Automated Scheduling Considerations:**
- Cronjob tool may have type validation bugs (str vs int comparisons)
- Alternative: Use system-level cron or task scheduler
- Manual execution recommended until tooling is fixed
- Consider external webhook triggers for real-time updates

**Data Quality & Source Strategy (Updated Apr 19, 2026):**
- Primary sources (official blogs, GitHub repos, press releases) preferred over aggregated news
- When Tier 1 sources (36Kr, LatePost, TechCrunch, Bloomberg, Reuters) return empty results:
  - Expand to Tier 2 sources (Sohu, NetEase, Sohu, Xinlang, Xueqiu, IT之家, 经济观察网)
  - Use GitHub/HuggingFace for open-source model releases (ABot-M0, M2.7)
  - Cross-reference with academic venues (CVPR, NeurIPS) for research breakthroughs
  - Flag unverified items with "[未交叉验证]" tag
- Search query optimization:
  - Include specific date formats (YYYY年MM月DD日, YYYY-MM-DD)
  - Combine company names with key events (e.g., "高德 ABot-World 开源 2026年4月19日")
  - Use English-Chinese hybrid queries for international coverage
  - Add benchmark names for technical validation (Libero-Plus, WorldArena, SWE-Pro)
- Heat index calculation:
  - When quantitative metrics unavailable: use source authority weighting (Tier 1: 1.0, Tier 2: 0.7, Tier 3: 0.5)
  - Apply domain coverage penalty: items not covering required categories receive 0.8× multiplier
  - Deduplication: keep highest authority-weighted version for same event

**Data Quality Assurance:**
- Always verify news from ≥2 independent sources
- Cross-reference with financial data and company announcements
- Use authoritative sources as primary references
- Flag unverified or speculative information clearly