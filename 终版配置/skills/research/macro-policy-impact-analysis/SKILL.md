---
name: macro-policy-impact-analysis
category: research
description: Structured framework for analyzing macroeconomic policy impact on companies from economist, historian, sociologist, and entrepreneur perspectives
---

# Macro Policy Impact Analysis Skill

## Overview
A structured framework for analyzing the impact of macroeconomic policies, diplomatic activities, and foreign investment trends on specific companies or industries from multiple disciplinary perspectives.

## Trigger Conditions
- User requests analysis of recent government policies (e.g., MIIT, MOF, NDRC) from past 2-4 weeks
- User asks for multi-perspective analysis of company development under current policy environment
- User needs strategic forecasting based on policy trends and market dynamics
- User specifically requests analysis from **Economist, Historian, Sociologist, and Entrepreneur** perspectives
- User wants to identify which enterprise types gain strong positive momentum from policy changes

## Step-by-Step Approach

### 1. **Data Collection Phase**
- **Policy Search**: Query recent (2-4 weeks) policy announcements from relevant ministries (MIIT, MOF, NDRC, etc.)
- **Diplomatic Intelligence**: Gather foreign leader visits, high-level diplomatic meetings, and bilateral agreements (2-3 months)
- **Foreign Investment**: Track MNC CEO visits, investment commitments, and supply chain relocations
- **Company-Specific**: Collect recent news, financial reports, strategic announcements, and market movements

**Search Strategy**:
- Use specific ministry names + policy keywords + timeframes
- Combine "foreign leader visit China" + "MNC CEO visit" + "investment" queries
- Cross-reference with company-specific news and financial data
- Extract key dates, policy details, investment amounts, and strategic directions

### 2. **Multi-Perspective Analysis Framework**

#### **Economist Perspective**
- Analyze policy tool combinations (fiscal + monetary coordination)
- Evaluate supply-demand side interventions
- Assess structural policy targeting and resource allocation
- Interpret foreign investment signals and supply chain implications
- Calculate multiplier effects and sectoral impacts
- **Key metrics**: GDP impact, industry cycle positioning, supply-demand dynamics

#### **Historian Perspective**
- Compare current cycle with historical precedents (e.g., 1998 Asian Financial Crisis, 2008 Global Financial Crisis)
- Trace policy evolution across reform eras
- Identify patterns in foreign investment cycles
- Draw lessons from past tech competition scenarios
- Assess structural transformation trajectories
- **Key insights**: Historical analogies, evolution stages, lessons learned

#### **Sociologist Perspective**
- Examine employment impacts and social stability mechanisms
- Analyze consumption stratification and policy targeting
- Evaluate social confidence and consumer sentiment drivers
- Assess digital inclusion and rural-urban dynamics
- Identify social identity factors affecting brand performance
- **Key focus**: Social cohesion, consumption behavior changes, demographic shifts

#### **Entrepreneur Perspective**
- Map industry opportunity windows by priority tier
- Identify key decision variables (policy certainty vs. tax incentives)
- Evaluate cash flow management and technology roadmap choices
- Assess supply chain resilience requirements
- Provide actionable short/medium/long-term recommendations
- **Key questions**: Commercial value, competitive landscape, implementation feasibility

### 3. **Synthesis and Conclusion**
- **Categorize Beneficiaries**: Tier 1 (high certainty), Tier 2 (medium-high), Tier 3 (selective), Limited beneficiaries
- **Develop Action Framework**: Short-term (0-6mo), Medium-term (6-18mo), Long-term (18-36mo)
- **Risk Assessment**: Identify policy risks, competitive risks, execution risks
- **Strategic Recommendations**: Specific actions for different stakeholder groups

### 4. **Output Structure**
- Policy background summary with key dates and details
- Four-perspective analysis with specific examples
- Categorized beneficiary analysis with rationale
- Time-phased development forecast
- Actionable recommendations for management, investors, and policymakers

## Key Insights to Capture
- Policy certainty > tax incentives as decision driver
- Compliance capability and technical barriers as competitive moats
- Supply chain resilience from "just-in-time" to "just-in-case"
- Digital transformation as investment, not cost
- ESG体系的完善 as competitive threshold

## Common Pitfalls to Avoid
- Over-reliance on single policy source
- Ignoring historical context and cycles
- Neglecting social dynamics and consumer sentiment
- Focusing only on short-term gains without long-term strategy
- Underestimating geopolitical risks in international expansion

## Example Applications
- Analyzing impact of MIIT manufacturing policies on tech companies
- Assessing foreign investment trends on supply chain companies
- Evaluating fiscal stimulus effects on consumer-facing businesses
- Forecasting company development under new industrial policies

## Adaptation Notes
- Adjust timeframes based on user requirements (e.g., 2 weeks vs. 2 months)
- Scale analysis depth based on available data quality
- Customize perspective emphasis based on company type (B2B vs. B2C, tech vs. traditional)
- Incorporate real-time data when available for dynamic updates

## Critical Additions (v1.3)
### Stock Price Calibration
- **Always verify** user-provided stock prices against real-time sources
- Calculate unrealized P/L: `(Current Price - Cost Basis) / Cost Basis * 100%`
- Note discrepancies explicitly and adjust analysis accordingly
- Use delayed quotes if real-time unavailable (mark as "延迟报价")

### Data Source Limitations
- **Future Date Searches**: Real-time data for future dates (e.g., 2026) may not exist; use historical trends + logical extrapolation
- **Bot Detection**: Major financial sites (Google, Bloomberg, CNBC) often block automated access
  - **Fallback Strategy**: Use multiple sources (Sina Finance, 10jqka, Investing.com), manual verification, or skip if blocked
  - **Workaround**: Try different domains, add delays between requests, or use mobile versions
- **Delayed Quotes**: Clearly mark if data is delayed (>15 min)

### Mandatory Output Requirement
- **Always conclude** with "强正向增益企业类型结论" (Strong Positive Gain Enterprise Types)
- Categorize beneficiaries into tiers (Tier 1: high certainty, Tier 2: medium-high, etc.)
- Provide specific rationale linking policy/news to enterprise type benefits
- Include risk等级 (risk level) for each category

### Example Workflow
1. User provides stock price (e.g., "美团 86.5 HKD") → Verify vs. real-time data
2. Search policy/news with specific timeframes → Extract key events
3. Run four-perspective analysis → Synthesize insights
4. **Identify strong positive gain enterprise types** → Conclude with actionable recommendations
5. Provide portfolio construction advice (high-elasticity + safety net)

## Version History
- v1.0: Initial framework creation based on 26 年 4 月 macro policy analysis
- v1.1: Added JD.com case study and multi-perspective integration
- v1.2: Enhanced perspective definitions with key metrics and focus areas; clarified trigger conditions for user-specific perspective requests; added explicit instruction to identify strong positive momentum enterprise types