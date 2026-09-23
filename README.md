# check-ai-credits

**Check AI provider credits and balance across 10 providers from one CLI.**
One command answers "how much credit do I have left?" for DeepSeek, DeepInfra,
Codex (ChatGPT), Grok, MiMo, Claude, Gemini, Kimi, OpenRouter and the
apikey.fun relay — with burn trend and estimated days-left.

- Single static binary (TypeScript + Bun), zero runtime dependencies
- Auto-discovers local credentials (env vars, CLI auth stores, CodexBar)
- Read-only: never refreshes or rewrites OAuth tokens, never prints secrets
- Uniform snapshot schema → JSON output ready for dashboards/cron

## Install

```bash
# from source (requires Bun >= 1.1)
bun install
bun run build          # -> dist/check-ai-credits

# or run directly
bun src/cli.ts --help
```

## Usage

```bash
check-ai-credits                 # all configured providers
check-ai-credits deepseek        # one provider
check-ai-credits --json          # machine-readable snapshots
check-ai-credits --trend         # burn rate + days-left from logged history
check-ai-credits --list          # show providers and credential detection
check-ai-credits deepinfra --models   # per-model month breakdown (DeepInfra)
```

## Provider matrix

| Provider | ID | Credential source (auto-detected) | Metric |
|---|---|---|---|
| DeepSeek official | `deepseek` | `DEEPSEEK_API_KEY` (env / `$HERMES_HOME/.env`) | wallet balance USD |
| DeepInfra | `deepinfra` | `DEEPINFRA_API_KEY` (env / `$HERMES_HOME/.env`) | prepaid pool, month spend |
| Codex (ChatGPT) | `codex` | `~/.codex/auth.json` (Codex CLI, read-only) | credits + rate-limit window |
| Grok (xAI) | `grok` | `~/.grok/auth.json` (Grok CLI, read-only) | monthly credits + prepaid |
| Xiaomi MiMo | `mimo` | `MIMO_COOKIE` env, else CodexBar fallback | CNY balance + plan |
| Claude | `claude` | CodexBar usage | window used % |
| Gemini | `gemini` | CodexBar usage | window used % |
| Kimi | `kimi` | CodexBar usage | window used % |
| OpenRouter | `openrouter` | CodexBar usage | window used % |
| apikey.fun relay | `apikeyfun` | CodexBar plugin | shared wallet balance |

Providers whose credentials are not present report `not configured` with a
login/setup URL instead of failing the whole run.

## Credential resolution (per provider, in order)

1. Process environment variable
2. `$HERMES_HOME/.env` (if you run Hermes Agent; never exported)
3. Local CLI auth store (`~/.codex/auth.json`, `~/.grok/auth.json`) — **read-only**
4. CodexBar CLI (macOS menu-bar app) for web-session providers
5. Interactive fallback: the tool prints the login URL and exits non-zero

Security rules baked in: OAuth access tokens are never refreshed or rewritten
(re-refreshing would rotate the CLI's own refresh token and log you out),
secrets are never echoed in output or `--json`, and there is no telemetry.

## JSON snapshot schema

```jsonc
{
  "provider": "deepseek",
  "source": "api",
  "timestamp": "2026-09-24T00:00:00.000Z",
  "currency": "USD",
  "balance": 12.34,
  "usedPercent": null,
  "resetsAt": null,
  "monthCost": null,
  "monthPeriod": null,
  "details": { },     // provider-specific extras
  "error": null
}
```

## Trend log

Snapshots are appended to `$CHECK_AI_CREDITS_LOG` (default
`~/.check-ai-credits/log.json`). `--trend` fits a least-squares burn rate over
a 7-day window, clips quota series before the last reset, and flags low
balances (USD wallet < 5, or < 20% remaining). Use `--no-log` to skip
appending.

## Development

```bash
bun run typecheck   # tsc --noEmit, strict
bun test            # fixture-based unit tests (no live API calls)
bun run build       # single-file binary
```

Adding a provider = one file in `src/providers/` implementing the `Provider`
interface (`detect()` + `fetch()`), one registry entry in `src/registry.ts`.

## License

MIT
