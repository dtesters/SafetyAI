# SafetyAI – Discord Anti-Scam Bot

SafetyAI (a.k.a **discord-anti-scam-bot**) is a TypeScript-powered Discord bot that uses large-language-model (LLM) providers to detect and automatically moderate phishing, giveaway, Nitro and other scam messages in real-time.

The bot combines classic heuristic triggers with AI classification and a simple online-learning memory to continuously improve over time while keeping false-positives low.

---

## ✨ Features

- **Multi-provider LLM support** – OpenAI, Anthropic, Mistral, Groq, xAI or Hugging Face out of the box.
- **Heuristic triggers & hard-blocks** – fast regex pre-filter before spending LLM tokens.
- **Few-shot memory** – ships with seed examples and learns from moderator feedback.
- **Configurable moderation actions** – delete, DM user, post mod embeds with one-click buttons.
- **Rate-limiting** – global token bucket + per-channel cool-down to control API usage.
- **Structured logging** – JSON & pretty Pino logs for easy ingestion.

---

## 📂 Project structure

```
.
├─ src/                TypeScript source
│  ├─ bot.ts           Discord entry-point
│  ├─ moderation.ts    Core classification & actions
│  ├─ prompt.ts        Few-shot prompt builder
│  ├─ memory.ts        Online learning store
│  ├─ providers/       LLM client wrappers
│  └─ util.ts          Helpers & logging
├─ examples/           Seed & learned JSONL examples
├─ logs/               Runtime logs (created at run-time)
├─ config.json         Runtime configuration (non-secret)
├─ env.example         Environment variable template (secrets)
└─ package.json        NPM metadata & scripts
```

---

## 🚀 Quick start

1. **Clone & install**

   ```bash
   git clone <repo-url>
   cd safetyai
   npm install # or pnpm install / yarn
   ```

2. **Configure environment**

   ```bash
   cp env.example .env
   # edit .env and fill in your Discord token & any provider API keys you plan to use
   ```

   Mandatory variables:

   * `DISCORD_TOKEN` – bot token (found in the Discord developer portal)

   Optional / recommended variables:

   * `MOD_CHANNEL_ID` – channel ID where moderation alerts should be sent
   * `DEBUG_CHANNEL_ID` – channel ID to receive full debug traces
   * `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, ... – provider keys (fill at least one)

3. **Update `config.json`**

   * `provider` – one of `"openai" | "anthropic" | "mistral" | "groq" | "xai" | "huggingface"`.
   * `model` – model name for the chosen provider.
   * Adjust trigger & hard-block regexes, rate limits and moderation actions as desired.

4. **Run in dev-mode (TypeScript directly)**

   ```bash
   npm run dev
   ```

   The bot will connect, log in and start monitoring messages.

5. **Build & run production JS** (Recommended)

   ```bash
   npm run build   # emits JS to dist/
   npm start       # runs node dist/bot.js
   ```

---

## ⚙️ Configuration reference (`config.json`)

| Field | Description |
|-------|-------------|
| `provider` | LLM provider key – must match an implemented client in `src/providers/`. |
| `model` | Model name passed verbatim to the provider. |
| `productionReady` | When `true`, destructive actions (deletion) are allowed; keep `false` while testing. |
| `triggerPatterns` | Array of regex strings; if **none** match, the LLM is **not** called (saves tokens). |
| `hardBlockRegexes` | Regexes that, when matched, immediately mark the message as scam. |
| `rateLimit.maxCallsPerMinute` | Shared global LLM call quota. |
| `rateLimit.perChannelCooldownSec` | Minimum seconds between scans in the same channel. |
| `moderation.minConfidenceToDelete` | Minimum classification confidence before the bot auto-deletes. |
| `moderation.actions.*` | Toggle deletion, DM, mod alert actions. |
| `fewShot.*` | Controls how many examples are loaded & priority heuristics. |
| `logging.level` | Pino log level (`debug`, `info`, `warn`, `error`). |

---

## 🧠 Few-shot examples

Seed examples live in `examples/seed_examples.jsonl` and are loaded on every start.  When the bot makes a decision it also writes learned examples to `examples/learned_examples.jsonl`, giving each one a weight.  Moderator button feedback adds labelled ground-truth examples and influences future prompts.

---

## 🔌 LLM provider setup

| Provider  | Env var(s) | Notes |
|-----------|------------|-------|
| OpenAI    | `OPENAI_API_KEY`, optional `OPENAI_BASE_URL` | Works with compatible endpoints (Groq / Together etc.) |
| Anthropic | `ANTHROPIC_API_KEY` | Uses `/v1/messages` API |
| Mistral   | `MISTRAL_API_KEY` | Official Mistral endpoint |
| Groq      | `GROQ_API_KEY`, `GROQ_BASE_URL` | Groq’s OpenAI-compatible proxy |
| xAI       | `XAI_API_KEY`, `XAI_BASE_URL` | Experimental |
| Hugging Face | `HUGGINGFACE_API_KEY` | Uses text-generation inference API |

Provide at least one key corresponding to the `provider` in `config.json`.

---

## 📊 Logging

Logs are written to:

* `logs/actions.log` – adoption & moderator actions.
* `logs/errors.log` – classification errors & fallbacks.

Verbose pretty logs are printed to stdout when `NODE_ENV!==production`.

---

## 🛂 Permissions

The bot needs at minimum:

* **Guilds**
* **Guild Messages**
* **Message Content** (for non-privileged intents)

and `Manage Messages` permission if you allow auto-deletion.

---

## 🏓 Rate limiting

The token bucket in `src/bot.ts` enforces a maximum LLM call rate globally and per channel.  Tune these values in `config.json` to keep API costs predictable.

---

## 🛠 Contributing

1. Fork & create a feature branch.
2. Run `npm run dev` and add your changes (+ tests if applicable).
3. Open a PR describing your improvements.

All contributions, bug reports and feature requests are welcome!

