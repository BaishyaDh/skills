---
name: instagram-setup-guide
description: Interactive wizard to guide users through setting up Meta Developer credentials for Instagram Graph API access.
---

# Instagram API Setup Wizard

This skill acts as an interactive setup guide to help users create a Meta Developer app, get the required permissions, and acquire a long-lived access token for Instagram analytics.

## Execution Flow

When you run this skill, you must execute the following steps interactively with the user:

### Step 1: Greeting & Prerequisites
1. Say: "Welcome to the Instagram Analytics Setup Wizard! To fetch live data from your Instagram account, we need to set up a Meta Developer App. Let's do this step by step."
2. Ask the user if they already have an Instagram Professional Account linked to a Facebook Page. If not, guide them to do so first.

### Step 2: Meta App Creation
1. Instruct the user to go to [developers.facebook.com](https://developers.facebook.com/), create a **Business** app, and add the **Instagram Graph API** product.
2. Wait for them to confirm this is done.

### Step 3: Token Generation
1. Instruct the user to go to the **Graph API Explorer** tool.
2. Tell them to select their App, select "Get User Access Token", and add the following permissions:
   - `instagram_basic`
   - `instagram_manage_insights`
   - `pages_show_list`
   - `pages_read_engagement`
3. Have them generate the token and get their Instagram Account ID.
4. Wait for them to confirm they have the Token and Account ID.

### Step 4: Saving Credentials
1. Ask the user: "Would you like me to save these credentials locally for this project only (e.g., in a `.env` file), or globally?"
2. Based on their answer, securely ask for the Token and Account ID and save them to the desired location (e.g., `.env` as `INSTAGRAM_ACCESS_TOKEN` and `INSTAGRAM_ACCOUNT_ID`).

### Step 5: Handoff & Agent Creation
1. Once credentials are saved, inform the user that the setup is complete.
2. Ask them: "Would you like me to automatically generate a dedicated 'Instagram Analyst' agent (`agent.md`) that leverages the core analytics skill for you?"
   - If yes, create an `agent.md` configured to run `instagram-analyst`.
3. Finally, automatically run the following command in the terminal to pull down the actual analytics engine:
   ```bash
   npx skills add BaishyaDh/skills --skill instagram-analyst
   ```
4. Tell the user they can now run the analyst skill anytime!
