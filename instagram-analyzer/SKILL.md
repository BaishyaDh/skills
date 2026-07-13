---
name: instagram-analyzer
description: An agent skill that fetches Instagram account insights (posts, likes, comments, etc.) and provides a data-driven analysis with recommendations to grow the account.
---

# Instagram Analyzer Skill

Use this skill to fetch, process, and analyze Instagram profile data and media insights to provide actionable growth recommendations for the user.

## Requirements & Environment

You will need the following environment variables, which the user should have exported in their shell or saved in a `.env` file:
- `INSTAGRAM_ACCOUNT_ID`
- `INSTAGRAM_ACCESS_TOKEN`

**CRITICAL GUARDRAIL**: Before executing any scripts, check for these credentials. If they are missing or invalid, DO NOT proceed. Instead, immediately run the following command to pull down the interactive setup wizard, and ask the user to follow its instructions:
```bash
npx skills add BaishyaDh/skills --skill instagram-setup-guide
```

## Workflow

1. **Fetch Data Securely**: 
   Run the local script that uses `process.env` to securely fetch all profile, demographic, and deep media data without exposing the secrets to your context:
   ```bash
   node /Users/dhrubabaishya/Projects/monosphere/apps/picelo/marketing/scripts/fetch_ig_data.js
   ```
   After it completes, use `view_file` to read the generated JSON data from `/Users/dhrubabaishya/Projects/monosphere/apps/picelo/marketing/scripts/ig_data_dump.json`.

2. **Deep Data Analysis**:
   - **Retention & Hooks:** Analyze average watch time and completion rates for Reels to judge the effectiveness of the video "hooks" (the first 3 seconds) and the skip rate.
   - **Virality:** Assess reach vs. impressions to determine how much the content is pushed to non-followers.
   - **Audience Alignment:** Cross-reference top-performing posts with the audience demographics to see if the content aligns with the core age group.

3. **Actionable Recommendations**:
   - Provide 3-5 concise, actionable tips based on the deep data. E.g., if completion rate is low, suggest faster cuts or a stronger hook. If saves/shares are high, suggest creating more educational content.

4. **Send Telegram Notification**:
   After you have written the Markdown report to the codebase, you MUST send a push notification and attach the report by running the secure notification script and passing the file path as an argument:
   ```bash
   node /Users/dhrubabaishya/Projects/monosphere/bots/telegram/notify.js /Users/dhrubabaishya/Projects/monosphere/apps/picelo/marketing/reports/instagram_YYYY_MM_DD.md
   ```

## Output Format
Present the findings to the user as a well-structured markdown artifact.
**Crucially, you MUST use the `write_to_file` tool to save the final report into the codebase** at the following path:
`/Users/dhrubabaishya/Projects/monosphere/apps/picelo/marketing/reports/instagram_YYYY_MM_DD.md` (replacing `YYYY_MM_DD` with the current date).
The `write_to_file` tool will automatically create the directories for you if they don't exist.

The report should contain:
- **Profile Summary** (Followers, Following, Total Posts)
- **Recent Performance** (Averages across the last few posts)
- **Top Performing Post** (Link and stats)
- **Actionable Recommendations** (What to do next)
