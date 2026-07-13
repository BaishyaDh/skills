---
name: instagram-content-planner
description: Analyzes niche and strategy documents to output highly tailored, actionable Instagram Reel content plans.
---

# Instagram Content Planner

This skill empowers an AI agent to act as a highly strategic Social Media Director, generating tailored content plans based on core business pillars and psychological marketing strategies.

## Workflow

1. **Context Gathering (CRITICAL)**:
   - You MUST use your tools to search the user's workspace for `niche.md` (or similar brand guidelines) AND `instagram_strategy.md` (or similar psychological/algorithmic strategy files).
   - Read these files completely to understand the brand's core value proposition, target audience, and psychological triggers.

2. **Data Integration (Optional but Recommended)**:
   - If the user has an `instagram-analyst` agent available or recent performance data in the workspace (e.g. `marketing/reports/*.md`), read the most recent report. Use this data to identify which hooks/formats are currently performing best organically. If you need fresh data, invoke the `instagram-analyst` subagent.

3. **Content Plan Generation**:
   - Based on the context gathered, generate a concrete content plan (e.g., for the next 7 or 30 days).
   - The plan MUST include:
     - **Specific Reel Series Concepts:** Tailored to the niche (e.g. Problem-First, Scenario-Driven).
     - **Hook Scripts:** Literal 0-3 second hook scripts using Looming Danger or Dopamine Trap psychology.
     - **Visual/Audio Execution:** Instructions on SFX syncing and visual pattern breaks.
     - **Call to Action (CTA):** Clear, frictionless CTAs designed to drive algorithmic Saves/Shares or Website Visits.
     - **Paid Boosting Strategy:** Instructions on which organic winners to boost based on the strategy file.

4. **Output Format**:
   - Write the finalized content plan to the user's workspace using the `write_to_file` tool (e.g. `marketing/content_plan_YYYY_MM_DD.md`).
