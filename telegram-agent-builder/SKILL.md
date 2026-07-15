---
name: telegram-agent-builder
description: Automates the setup of a native, zero-prompt Telegram AI Agent that runs entirely in the background of your IDE using your local LLM session without requiring paid APIs.
---

# Telegram Agent Builder

This skill helps a user deploy a native "Reactive Wake-Up" Telegram Bot on their local machine. This bot will securely listen to Telegram messages, wake up their active Antigravity/Claude AI session upon receiving a text, and seamlessly route the AI's reply back to Telegram—all without requiring external pay-as-you-go developer API keys (like OpenAI, Claude, or Gemini APIs).

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
3. Generate the **Master Poller Script** (`bots/telegram/poller.js`). Ensure the code strictly does the following:
   - Uses native `fetch` to poll `https://api.telegram.org/bot${TELEGRAM_TOKEN}/getUpdates?timeout=30`.
   - **Handles Callback Queries (2FA Queue):** If the update contains a `callback_query` (e.g., an "Approve" button tap from an MCP server), it must parse the ID and action, write it to a local JSON file queue (e.g., `/tmp/telegram_2fa.json`), instantly acknowledge the query via the API to stop the button's loading spinner, and then `continue` the polling loop.
   - **Handles Text Messages (Reactive Wake-Up):** If the update contains a text message, filters it strictly by `process.env.TELEGRAM_CHAT_ID` so strangers cannot trigger the agent.
   - Upon receiving a valid text message, fires a `sendChatAction?action=typing` to Telegram.
   - Prints a highly visible string block to `stdout` containing the message text (e.g., `[NEW_TELEGRAM_MESSAGE_RECEIVED]`).
   - Acknowledges the message to Telegram by updating the `offset`.
   - Executes `process.exit(0)` immediately.
   *(Explain to the user that this split architecture is the secret sauce: cleanly exiting the script on a text message forces the background task to complete, triggering the AI agent's reactive wake-up alarm in the IDE, while seamlessly bypassing the Telegraf polling lock conflict for 2FA MCP tools by proxying button taps through a local file queue!)*
4. Generate the **Reply Script** (`bots/telegram/reply.js`). This script must:
   - Accept the response text as a command line argument (`process.argv[2]`).
   - Use RegEx to parse literal `\n` strings into actual newlines (since bash arguments pass them as literal characters).
   - Use RegEx to automatically convert standard Markdown (like `**bold**`, `[text](url)`, and `` `code` ``) into Telegram-friendly HTML (e.g. `<b>bold</b>`, `<a href="url">text</a>`, `<code>code</code>`).
   - Append `parse_mode=HTML` to the Telegram API request URL.
   - URL-encode the converted text and send it to the `sendMessage` Telegram API using `fetch`.
5. Generate the **2FA MCP Server** (`apps/telegram-mcp/src/index.ts`). Ensure the code strictly does the following:
   - Uses `@modelcontextprotocol/sdk` to set up a standard StdioServerTransport.
   - Exposes an `execute_command` tool.
   - When executed with a dangerous command, fires a Telegram message containing an inline keyboard with "Approve" and "Reject" buttons.
   - Uses a `setInterval` loop to read `/tmp/telegram_2fa.json` every second, checking if the `Master Poller` registered an approval/rejection for the callback ID.
   - **Self-Healing Fallback:** Before firing the 2FA request, it must use the `net` module to ping the local port `49999` to check if the Master Poller is currently alive. If the port is dead, it must spin up its own temporary fallback polling loop to catch the button tap, gracefully shutting it down once resolved.
   - Upon finding the action, deletes the entry from the JSON queue, edits the Telegram message, and executes the bash command safely.
   - **Crucially:** Instruct the agent to natively mount this MCP server to the user's AI environment. Do not hardcode specific commands (like editing `mcp_config.json` or running `claude mcp`); simply declare the requirement ("Mount this new MCP server to the current AI platform") and trust the agent to use its own native tools and knowledge to configure it securely.

### Phase 3: Arming the System
1. Request explicit permission from the user to whitelist the execution of `node bots/telegram/poller.js` and `node bots/telegram/reply.js` using the `ask_permission` tool so the loop can run silently without prompting them.
2. Instruct the user that you are starting the listener in the background, and they should text their bot on Telegram to begin.
3. Start the poller script using the `run_command` tool in the background (using a low `WaitMsBeforeAsync` so it detaches). 
4. When the user sends a message, the poller will exit and the system will wake you up with the message content in your context. You must process their request natively, use the `run_command` tool to execute `reply.js` with your intelligent answer, and finally re-run `poller.js` via a background command to re-arm the loop!

### Phase 4: Session Management & Restarting
Because this architecture relies on the active AI session listening to the background task's exit signal, the `poller.js` script *must* be launched by the AI agent itself.
1. Explicitly instruct the user that if they close their IDE/CLI or start a new AI session tomorrow, the background task will die.
2. Tell them that to restart the bot in a new session, they must explicitly tell the agent (or type in the chat): "Start my Telegram bot." 
3. Explain that the agent will then run the `poller.js` command (or `nx run bots:start`) on their behalf, re-attaching the wake-up alarm to the new session's AI brain. Emphasize that running it manually in a separate terminal will not link the exit signal to the AI agent.
4. **CRITICAL WARNING - The Cached Process Trap:** If you (the AI) are ever instructed to update the source code of the `telegram-mcp` server itself, remember that AI CLI environments spawn MCP servers as persistent background child processes! Even if you successfully compile the new code, the CLI session will still route commands to the stale process! You must manually find and kill the stale node process (`ps aux | grep dist/index.js`) so the CLI is forced to boot up the fresh code!
