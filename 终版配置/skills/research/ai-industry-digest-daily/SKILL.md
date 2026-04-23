---
name: ai-industry-digest-daily
title: AI Industry Daily Digest Production Pipeline
description: End-to-end workflow for generating comprehensive AI industry news digests with multi-perspective analysis and investment recommendations
---

# AI Industry Daily Digest Production Pipeline

## When to Use
- Daily AI industry news aggregation and analysis (24-hour window)
- Multi-perspective analysis (economist/historian/entrepreneur)
- Investment recommendation generation with risk assessment
- Automated cron job execution for scheduled delivery

## Key Constraints
- **Date replacement**: All `YYYY 年 MM 月 DD 日` and `YYYY-MM-DD` must be replaced with execution date
- **Timeliness**: Only use news from past 24 hours; exclude older content
- **Verification**: Each news item requires ≥2 independent sources
- **Objectivity**: Avoid subjective claims; base all conclusions on data
- **Actionability**: Investment recommendations must be specific and executable

## Data Collection Strategy

### 1. Search Query Categories (10 total)
**Chinese queries (5):**
1. AI 人工智能 企业新闻 [date] 最新 头条 热度
2. 人工智能 大模型 融资 发布 [date] 实时
3. 国内 AI 公司 新闻 [date] 排行榜 36 氪 晚点
4. AI 芯片 机器人 监管 [date] 财经
5. OpenAI 百度 阿里 腾讯 AI [date] 动态

**English queries (5):**
1. AI news today [YYYY-MM-DD] top trending tech companies
2. artificial intelligence breaking news [YYYY-MM-DD] Bloomberg Reuters
3. AI startup funding acquisition [YYYY-MM-DD] TechCrunch
4. OpenAI Google Microsoft AI [YYYY-MM-DD] latest
5. AI regulation policy [YYYY-MM-DD] government

### 2. Priority Sources
- **Tier 1**: 36氪, 晚点 LatePost, TechCrunch, Reuters, Bloomberg
- **Tier 2**: 澎湃新闻, CNBC, Yahoo Finance
- **Vertical**: AI News, The AI Track, Hugging Face Blog

### 3. Extraction Fields
- Model release dates/parameters/benchmarks
- Regulatory milestones (dates, requirements, enforcement)
- Funding/valuation figures (cross-verify)
- Hardware/software ecosystem updates
- Contradictions to flag (e.g., valuation discrepancies)

## Analysis Framework

### 1. Heat Index Formula
```
Heat Index = (Views×0.3) + (Shares×0.3) + (Social Discussion×0.2) + (Authority Sources×0.2)
```

### 2. Filtering Criteria
- **Quantity**: ≥5 domestic, ≥5 international
- **Coverage**: Large models/robots/chips/apps/regulation (2 each)
- **Deduplication**: Keep highest-heat version of same event

### 3. Multi-Perspective Analysis
**Economist:**
- GDP impact analysis
- Industry cycle assessment
- Supply-demand trends

**Historian:**
- Historical analogies (past AI waves)
- Evolutionary stage positioning
- Lessons learned

**Entrepreneur:**
- Business value assessment
- Competitive landscape
-落地 feasibility

### 4. Investment Framework
**Short-term (1-3 months):**
- Event-driven factors
- Sentiment impact
- Catalyst identification

**Medium-term (3-12 months):**
- Earnings realization path
- Market share changes
- Technology commercialization

**Long-term (1-3 years):**
-格局 reshaping opportunities
- Technology paradigm shifts
- Ecosystem building potential

## Output Structure (Strict Format)

```
⚠️ Data Quality Warning:
- [Reason 1]
- [Reason 2]

## News Summary (10 items, heat-ordered)
【News 1】Title
- Source: [Name]
- Heat Index: [Value]
- Core Content: [100-word summary]
- Related Companies: [Names]

## Multi-Dimensional Analysis
【Economist】
- [Analysis content]

【Historian】
- [Analysis content]

【Entrepreneur】
- [Analysis content]

## Investment Ranking & Actionable Recommendations
【Highest Certainty】
1. [Company/Asset] - [Reason]

【Highest Elasticity】
1. [Company/Asset] - [Reason]

【High Risk, High Return】
1. [Company/Asset] - [Reason]

## Positioning Recommendations
【Aggressive】[XX%] - [Logic]
【Balanced】[XX%] - [Logic]
【Conservative】[XX%] - [Logic]

## Key Catalyst Calendar
- [Date]: [Event] - [Impact]
- [Date]: [Event] - [Impact]

## Risk Alerts & Mitigation
【Primary Risks】
1. [Risk Type] - [Mitigation]

【Secondary Risks】
1. [Risk Type] - [Mitigation]
```

## Error Handling & Fallback

### Scenario A: 0 search results
- **Action**: Expand to 48-hour window; use 7-day hot news
- **Output**: "[SILENT]" if no valid data

### Scenario B: <5 valid news items
- **Action**: Lower verification requirement (≥1 source); label "unverified"
- **Output**: Full report with data quality warning

### Scenario C: Search timeout/API error
- **Action**: Retry 2× with 30s intervals; use cached data if available
- **Output**: "[SILENT]" if data unavailable

### Scenario D: Model response truncation
- **Action**: Generate in steps (news first, analysis second)
- **Output**: Prioritize core conclusions; note "partial output"

## Quality Checklist
- [ ] Date correctly replaced
- [ ] 10 searches executed (5 Chinese + 5 English)
- [ ] ≥10 valid news items
- [ ] Each news item cross-verified (≥2 sources)
- [ ] Heat-index sorted
- [ ] 5 domains covered (large models/robots/chips/apps/regulation)
- [ ] 3-perspective analysis complete
- [ ] All 6 output sections included
- [ ] Data quality checked; warnings added if needed

## Notes
- Use web_extract for deep content extraction after initial web_search
- Cross-verify valuation/funding figures across multiple sources
- Flag contradictions (e.g., valuation discrepancies) for user review
- Prioritize concrete data points: release dates, parameters, benchmarks, figures
- Focus on verified, recent (current month) information
- Use Chinese queries for domestic ecosystem, English for global coverage