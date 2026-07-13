---
name: telegram-agent-builder
description: Automates the setup of a native, zero-prompt Telegram AI Agent that runs entirely in the background of your IDE using your local LLM session without requiring paid APIs.
---

# Telegram Agent Builder

This skill helps a user deploy a native "Reactive Wake-Up" Telegram Bot on their local machine. This bot will securely listen to Telegram messages, wake up their active Antigravity AI session upon receiving a text, and seamlessly route the AI's reply back to Telegram—all without requiring external pay-as-you-go developer API keys (like OpenAI, Claude, or Gemini APIs).

## Execution Workflow

When invoked, strictly follow these steps sequentially:

### Phase 1: Setup & Environment Validation
1. Ask the user if they already have a Telegram Bot Token and their personal Chat ID. 
2. If they do NOT have them, provide these explicit instructions:
   - Open Telegram and search for `@BotFather`. Send `/newbot` and follow the prompts to get your **Bot Token**.
   - Search for `@userinfobot` (or similar) to find your numeric **Chat ID**.
   - Depending on their OS, tell them to add these to their terminal environment (e.g., `~/.zshrc` or `~/.bashrc` for macOS/Linux) as:
     `export TELEGRAM_TOKEN="your_token"`
     `export TELEGRAM_CHAT_ID="your_chat_id"`
3. Wait for the user to confirm they have loaded these into their environment and restarted their terminal. Verify by running a quick `env` check.

### Phase 2: Generating the Architecture
1. Once authenticated, create a directory in their workspace to house the bot (e.g., `bots/telegram`).
2. Write a `package.json` file in that directory to initialize a clean Node.js environment.
3. Generate the **Poller Script** (`bots/telegram/poller.js`). Ensure the code strictly does the following:
   - Uses native `fetch` to poll `https://api.telegram.org/bot${TELEGRAM_TOKEN}/getUpdates?timeout=30`.
   - Filters incoming messages strictly by `process.env.TELEGRAM_CHAT_ID` so strangers cannot trigger the agent.
   - Upon receiving a valid message, fires a `sendChatAction?action=typing` to Telegram.
   - Prints a highly visible string block to `stdout` containing the message text (e.g., `[NEW_TELEGRAM_MESSAGE_RECEIVED]`).
   - Acknowledges the message to Telegram by updating the `offset`.
   - Executes `process.exit(0)` immediately.
   *(Explain to the user that cleanly exiting the script is the secret sauce: it forces the background task to complete, which instantly triggers Antigravity's reactive wake-up alarm in the IDE!)*
4. Generate the **Reply Script** (`bots/telegram/reply.js`). This script must:
   - Accept the response text as a command line argument (`process.argv[2]`).
   - Use RegEx to automatically convert standard Markdown (like `**bold**`, `[text](url)`, and `` `code` ``) into Telegram-friendly HTML (e.g. `<b>bold</b>`, `<a href="url">text</a>`, `<code>code</code>`).
   - Append `parse_mode=HTML` to the Telegram API request URL.
   - URL-encode the converted text and send it to the `sendMessage` Telegram API using `fetch`.

### Phase 3: Arming the System
1. Request explicit permission from the user to whitelist the execution of `node bots/telegram/poller.js` and `node bots/telegram/reply.js` using the `ask_permission` tool so the loop can run silently without prompting them.
2. Instruct the user that you are starting the listener in the background, and they should text their bot on Telegram to begin.
3. Start the poller script using the `run_command` tool in the background (using a low `WaitMsBeforeAsync` so it detaches). 
4. When the user sends a message, the poller will exit and the system will wake you up with the message content in your context. You must process their request natively, use the `run_command` tool to execute `reply.js` with your intelligent answer, and finally re-run `poller.js` via a background command to re-arm the loop!

### Phase 4: Session Management & Restarting
Because this architecture relies on the active Antigravity session listening to the background task's exit signal, the `poller.js` script *must* be launched by the AI agent itself.
1. Explicitly instruct the user that if they close their IDE/CLI or start a new Antigravity session tomorrow, the background task will die.
2. Tell them that to restart the bot in a new session, they must explicitly tell the agent (or type in the chat): "Start my Telegram bot." 
3. Explain that the agent will then run the `poller.js` command (or `nx run bots:start`) on their behalf, re-attaching the wake-up alarm to the new session's AI brain. Emphasize that running it manually in a separate terminal will not link the exit signal to the AI agent.
