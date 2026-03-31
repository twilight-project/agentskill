# Twilight Protocol Agent Skills

Claude Code skills and API references for autonomous trading on [Twilight Protocol](https://twilight.rest) — a zero-knowledge inverse perpetual exchange.

## What's in this repo

### Skills (`.claude/skills/`)

Skills are invoked with `/skill-name` in Claude Code. They give the agent full context on how to use the Twilight trading infrastructure.

| Skill | Trigger | What it does |
|---|---|---|
| `/twilight-trader` | "open a trade", "check balance", "fund account" | Full CLI reference for the `relayer-cli` — wallet management, ZkOS account funding, opening/closing leveraged positions, market data, portfolio tracking |
| `/twilight-strategies` | "show strategies", "best hedging strategy", "funding arb" | Query the Twilight Strategy API for live trading strategies with real-time P&L calculations across Twilight, Binance, and Bybit |

### Strategy API

Live strategy calculations powered by real-time WebSocket feeds from Binance and Bybit.

**Base URL**: `http://134.199.214.129:3000`
**Auth**: `x-api-key: 123hEll@he`

```bash
# Get top 5 profitable strategies
curl -H "x-api-key: 123hEll@he" "http://134.199.214.129:3000/api/strategies?profitable=true&limit=5"

# Run a custom delta-neutral strategy
curl -X POST -H "x-api-key: 123hEll@he" -H "Content-Type: application/json" \
  -d '{"twilightPosition":"SHORT","twilightSize":200,"twilightLeverage":10,"binancePosition":"LONG","binanceSize":200,"binanceLeverage":10}' \
  http://134.199.214.129:3000/api/strategies/run

# Get live market data
curl -H "x-api-key: 123hEll@he" http://134.199.214.129:3000/api/market
```

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/api/health` | GET | No | Health check |
| `/api/market` | GET | Yes | Live prices, funding rates, spreads, pool skew |
| `/api/strategies` | GET | Yes | All strategies with live calculations |
| `/api/strategies/:id` | GET | Yes | Single strategy by ID |
| `/api/strategies/run` | POST | Yes | Run a custom strategy with your own params |
| `/api/impact` | POST | Yes | Simulate trade impact on pool |
| `/api/categories` | GET | Yes | List categories, risk levels, filter options |
| `/api/pool` | GET/POST | Yes | Get/update pool configuration |

**Strategy Filters** (query params on `/api/strategies`):
- `?category=Delta-Neutral` — filter by category
- `?risk=LOW` — filter by risk level
- `?profitable=true` — only profitable strategies
- `?minApy=50` — minimum APY threshold
- `?limit=10` — max results

## Relayer CLI Quick Reference

The `relayer-cli` binary from [nyks-wallet](https://github.com/twilight-project/nyks-wallet) handles all on-chain operations.

```bash
# Create wallet
relayer-cli wallet create --wallet-id my-wallet --password secret

# Fund ZkOS trading account
relayer-cli zkaccount fund --amount 5000

# Open 5x long (returns instantly with --no-wait)
relayer-cli order open-trade --account-index 0 --side long --entry-price 66700 --leverage 5 --no-wait

# Close trade
relayer-cli order close-trade --account-index 0 --no-wait

# Rotate account for next trade
relayer-cli zkaccount transfer --from 0

# Send tokens to another address
relayer-cli wallet send --to twilight1abc... --amount 5000 --denom sats

# Market data
relayer-cli market price
relayer-cli market market-stats
```

## Mainnet Endpoints

| Service | URL |
|---|---|
| LCD (REST) | `https://lcd.twilight.org` |
| RPC | `https://rpc.twilight.org` |
| ZkOS Server | `https://zkserver.twilight.org` |
| Relayer API | `https://api.ephemeral.fi/api` |
| Strategy API | `http://134.199.214.129:3000` |
| Explorer | `https://explorer.twilight.org` |

## Key Concepts

- **0% fees, 0% funding** on Twilight — every strategy exploits this vs Binance/Bybit
- **Inverse perpetuals**: margin in BTC (sats), PnL in sats
- **ZkOS accounts**: privacy-preserving, two states — Coin (idle) / Memo (order active)
- **Account rotation**: must rotate after settling a trade before opening a new one
- **Max leverage**: 50x, max position: 20% of pool equity

## Related Repos

- [nyks-wallet](https://github.com/twilight-project/nyks-wallet) — Rust SDK + CLI for trading
- [twilight-pool](https://github.com/kenny019/twilight-pool) — Frontend (Next.js)
- [twilight-strategy-tester](https://github.com/runnerelectrode/twilight-strategy-tester) — Strategy visualizer with live Binance/Bybit feeds
