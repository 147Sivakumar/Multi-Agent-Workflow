# Multi Agent Telegram Bot (n8n)

An n8n workflow where a Telegram bot answers messages with a main AI agent. When a question needs current or researched information, the main agent hands it to a **research sub-agent** that uses Google search (SerpApi) and returns the findings.

## How it works

```
Telegram Trigger -> AI Agent -> Send a text message (Telegram)
                      |
                      +-- Chat models:  OpenAI (primary) / DeepSeek (fallback)
                      +-- Memory:       Simple Memory (per Telegram chat)
                      +-- Tool:         AI Agent Tool (research agent)
                                           |
                                           +-- Chat model: OpenAI
                                           +-- Tool: Google search in SerpApi
```

1. **Telegram Trigger** - receives a message sent to the bot.
2. **AI Agent** - the main agent. It answers simple questions directly and calls the research tool for anything needing current information (news, prices, recent events).
3. **Simple Memory** - keeps a conversation history per chat, using the key `telegram_<chat id>`.
4. **AI Agent Tool** - a sub-agent exposed as a tool. The main agent decides when to call it and passes the question in through `$fromAI('research_question', ...)`.
5. **Google search in SerpApi** - the search tool used by the research sub-agent.
6. **Send a text message** - sends the agent's final answer back to the same Telegram chat.

## Key design point: how the tool gets called

A tool node is **not** a step in the main flow. The main agent calls it mid-reasoning, and the result returns to the agent. For this to work, the AI Agent Tool needs:

- a clear **description**, so the main agent knows when to use it
- a **Text** field set to `{{ $fromAI('research_question', 'The full question or topic the user wants researched', 'string') }}`, so the agent passes the question in
- a **system message** on the main agent telling it to call the tool for researched or current information

## Tech stack

- [n8n](https://n8n.io)
- Telegram Bot API
- OpenAI chat models (`gpt-5-mini` main, `gpt-4.1-mini` research agent)
- DeepSeek (`deepseek-v4-pro`) as fallback model
- SerpApi (Google search)

## Setup

### 1. Import the workflow
In n8n: **Workflows -> Import from File** and select the exported JSON.

### 2. Create a Telegram bot
1. Open Telegram, message **@BotFather** and send `/newbot`.
2. Copy the bot token.

### 3. Connect credentials
Credentials are stored in the n8n credential store and are **not** in the exported JSON. Reconnect these after importing:

| Node | Credential |
|------|------------|
| Telegram Trigger, Send a text message | Telegram API (bot token) |
| OpenAI Chat Model, OpenAI Chat Model1 | OpenAI API key |
| DeepSeek | DeepSeek API key |
| Google search in SerpApi | SerpApi API key |

### 4. Activate and test
1. Toggle the workflow to **Active** (Telegram triggers need an active workflow).
2. Message your bot, for example: `What's the latest news on AI today?`
3. Open **Executions** in n8n. You should see **AI Agent Tool** and **Google search in SerpApi** run before **Send a text message**.

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Research tool never gets called | Check the tool has a description and the main agent has a system message telling it to use the tool. |
| Tool runs but ignores the question | Make sure the tool's Text uses `$fromAI(...)` instead of a fixed sentence. |
| Fallback model skips the tool | Test with **Enable Fallback Model** turned off to compare tool-calling behavior. |
| Search returns nothing | Check the SerpApi credential and your SerpApi quota. |
| Bot doesn't reply | Confirm the workflow is Active and the Telegram credential is valid. |

## Screenshots

Add screenshots to a `screenshots/` folder:

- `screenshots/workflow-canvas.png`
- `screenshots/telegram-chat.png`
- `screenshots/execution-log.png`

## Files

```
.
├── Multi Agent Workflow.json   # exported n8n workflow
├── README.md
└── screenshots/
```

## Notes

- Never commit API keys or bot tokens. n8n exports reference credentials by name only.
- Check the exported JSON for any secrets before pushing.
