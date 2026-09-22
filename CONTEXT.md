# Handoff: Moon.com vs Pyth and Binance price timing

Research date: September 22, 2026.

This document preserves the context and findings of a local research conversation. It contains no API credentials. Only this document is included in this repository; referenced raw data, scripts, charts, and archives remain in the original local workspace unless separately uploaded.

## User objective and request history

1. Use [Laya-MLX](https://github.com/mizorewww/laya-mlx) to read price changes and assess whether [Moon](https://moon.com/bet) follows [Binance BTC/USDT](https://www.binance.com/en/trade/BTC_USDT?type=spot).
2. Resume the investigation and compare other platforms to understand Moon's pricing.
3. Add authenticated Pyth data using a user-supplied API credential.
4. Investigate whether external prices arrive before Moon, potentially allowing an earlier bet.
5. Explain Binance's lead relative to Pyth's approximately 140 ms lead.
6. Produce a complete handoff for another conversation.
7. Create this private repository under AbdulhakimZatar, using me@zatar.dev as the commit email.

The user allowed a browser fallback if Laya did not work. No bets, trades, deposits, or account changes were made during the investigation. Work consisted of market-data collection, local model benchmarking, and offline analysis.

## Main conclusions

**Moon's BTC feed appears to relay Pyth prices during the observed period.**

- In a 58-second historical verification, all 200 changing Moon quotes matched a preceding Pyth price exactly after truncating Pyth to five decimal places.
- Pyth arrived at the recording computer approximately 140 milliseconds before Moon's streamed feed.
- Binance price movements appeared approximately 320 milliseconds ahead of Moon in the same capture, but Binance prices matched Moon less closely.
- These are local feed-arrival relationships, not evidence that Moon would accept a bet at an outdated price.
- The recordings were short and experienced connection problems. Findings must not be presented as universal or persistent.

### Same-capture comparison

Usable window: approximately 201.18 seconds, September 22, 2026, 11:32:58.642–11:36:19.822 UTC.

| Reference | Estimated lead over Moon | Mean absolute simultaneous price gap | Five-second return correlation |
|---|---:|---:|---:|
| Pyth Pro BTC/USD | 140 ms | $0.263 | 0.9921 |
| Binance midpoint, USDT converted to USD | 320 ms | $3.364 | 0.9059 |
| Coinbase BTC/USD midpoint | 260 ms | $2.846 | 0.8900 |

Lead is the shift maximizing one-second-return correlation, not a claim that every update arrives that much earlier. After shifting Pyth by 140 ms, its mean absolute difference from Moon fell to $0.0784.

The latest explanatory answer was: Binance moved earlier, but its price was not necessarily the price Moon would show next. Neither estimated lead is a guaranteed betting window.

## Original local environment and deliverables

Workspace:

```text
/Users/azatar001/Documents/Codex/2026-09-22/https-github-com-mizorewww-laya-mlx
```

Machine: macOS, Apple M3 Pro, zsh. Intermediate work is in `work/`; user-facing deliverables are in `outputs/`. These paths describe the original machine and will not exist in a fresh clone.

Primary deliverables, relative to that workspace:

```text
outputs/PYTH_REPORT.md
outputs/pyth_comparison.png
outputs/pyth-study.zip
outputs/MULTI_PLATFORM_REPORT.md
outputs/README.md
outputs/moon-price-study.zip
outputs/replay_display_delay.py
```

Earlier reports contain follow-up notes directing readers to the newer Pyth findings. The Pyth archive predates the final offline display-delay replay; the replay is a separate local file.

## Laya-MLX testing

Repository cloned into `work/laya-mlx`.

- Commit: `0a859518634112655cb97c745dbf04f5191aaf13`.
- Isolated environment: `work/laya-venv`.
- Python 3.12.14; MLX 0.32.2.
- Model: `aac6fef/laya-multilingual-mlx`.
- Model revision: `f2b4faf51023039425946074e2cf1361d2db11d5`.
- Node v26.7.0; `ws` installed in `work/node_modules` for authenticated WebSockets.

Results:

- Compact repeated single-question inference: compiled FP16 with padding 16 and prompt cache had a median of 9.75 ms; eager execution had a median of 10.20 ms. Thirty alternating calls were made per configuration.
- A more detailed three-question summary had a median of 68.2 ms.
- Laya misinterpreted numerical correlations.
- On 30 balanced real up/down/unchanged transitions, it predicted unchanged for all 30, scoring 10/30.

Conclusion: direct numerical processing is better suited to this task. Laya was excluded from the collection and timing path.

## Moon's public feed

Public frontend inspection revealed:

```text
wss://moon.com/_api/websocketsv2
```

WebSocket subprotocol: `graphql-transport-ws`.

Initialization:

```json
{"type":"connection_init"}
```

After acknowledgement, the BTC subscription used:

```json
{
  "id": "btc",
  "type": "subscribe",
  "payload": {
    "query": "subscription SymbolQuote($symbolIds:[String!]!){symbolQuote(symbolIds:$symbolIds){p s ts b}}",
    "variables": {
      "symbolIds": ["eedad81f-c0a0-4653-9a10-4b2495bb877b"]
    }
  }
}
```

Observed fields:

- `p`: price.
- `s`: symbol.
- `ts`: minute-bucket timestamp, not a reliable per-tick generation timestamp.
- `b`: base price.

The frontend updates prices from the feed and batches rendering with `requestAnimationFrame`. Measurements concern quote receipt, not exact screen-paint time or Moon's internal bet-pricing timestamp.

Read-only asset discovery used `https://moon.com/_api/graphqlv2`. Public frontend code was cached locally in `work/moon-graphql.js`. No betting mutations were executed. See the terms section below before considering further collection or execution testing; discoverability does not itself establish permission.

## Earlier comparisons

### Initial Moon/Binance pilot

Approximately 237.4 seconds, September 22, 2026, 10:53:08.660–10:57:06.064 UTC.

- 795 Moon quotes and 5,804 Binance trades.
- Binance feed: `wss://stream.binance.com:9443/ws/btcusdt@trade`.
- Analysis used a 100 ms receipt-time grid.
- Mean Moon minus Binance: approximately -25.526 quote units.
- Five-second return correlation: 0.912.
- Ten-second return correlation: 0.950.
- Estimated Binance lead: 200–300 ms.

Limitation: this first comparison mixed nominal BTC/USD and BTC/USDT without the later FX correction.

### Six-minute multi-platform study

Common window: approximately 355.1 seconds, September 22, 2026, 11:09:25.308–11:15:20.408 UTC.

Compared Moon with Binance, Coinbase, Kraken, and OKX. USDT prices were converted using observed USDT/USD quotes. Median USDT/USD was approximately 0.99966, rather than assumed to equal $1.

| Reference | Mean absolute gap from Moon | Five-second return correlation |
|---|---:|---:|
| Binance, USD-normalized | $6.249 | 0.948 |
| Coinbase USD | $3.940 | 0.937 |
| Kraken USD | $9.064 | 0.817 |
| OKX, USD-normalized | $3.894 | 0.942 |
| Equal-venue median | $2.534 | 0.968 |

This suggested an aggregated or shared upstream source; the later Pyth fingerprint supplied substantially stronger evidence. Feed types and update cadences differed, so these are not perfectly controlled exchange-latency benchmarks.

## Pyth authentication and collection

The user supplied an authenticated request to:

```text
https://pyth.dourolabs.app/v1/fixed_rate@1000ms/history?symbol=Crypto.BTC%2FUSD&from=1781079582&to=1783671582&resolution=1D
```

It succeeded and returned 30 daily candles. Daily candles cannot establish sub-second timing.

The bearer credential is deliberately omitted. Previous runs used a hidden terminal prompt and passed it to the collector through a child-process environment variable. Credentials were not embedded in scripts, reports, URLs, charts, or recorded data. A new live run needs an appropriate credential supplied securely; do not assume one is stored locally.

Pyth symbol lookup identified BTC/USD price-feed ID `1`, exponent `-8`.

Authenticated WebSocket endpoints:

```text
wss://pyth-lazer-0.dourolabs.app/v1/stream
wss://pyth-lazer-1.dourolabs.app/v1/stream
wss://pyth-lazer-2.dourolabs.app/v1/stream
```

Successful subscription:

```json
{
  "type": "subscribe",
  "subscriptionId": 1,
  "priceFeedIds": [1],
  "properties": ["price", "exponent", "confidence", "publisherCount", "feedUpdateTimestamp"],
  "formats": [],
  "channel": "fixed_rate@200ms"
}
```

All three endpoints were subscribed. Duplicate updates were removed using timestamp and price, preserving the earliest local receipt. Price conversion was integer price mantissa multiplied by 10 to the exponent. The feed reported 26 contributing publishers.

Historical requests used `/v1/{channel}/price/range` with feed ID, start/end microsecond timestamps, a limit, and pagination where needed.

### Timing analysis

- Feeds were collected in one process using a shared monotonic receipt clock.
- Last-observed prices were aligned on a 20 ms grid.
- BTC quote-age cutoff: 3 seconds; FX quote-age cutoff: 60 seconds.
- A fine grid does not improve Pyth's underlying 200 ms sampling resolution.
- Approximately 98.84% of common-window grid points passed the freshness filter.
- Pyth's estimated lead across four segments was 140, 140, 120, and 140 ms.
- Selecting 140 ms on the first half improved second-half correlation from 0.9431 at zero lag to 0.9845 at the selected lag.

Pyth receipt-minus-generation latency had a median of approximately 119 ms and a 95th percentile of approximately 126 ms in the valid window. This separate metric depends on clock synchronization; it is not the shared-clock relative lead.

### Exact price fingerprint

Fetched 1,200 full-rate historical records covering 60 seconds, using twelve five-second requests. A larger request initially timed out.

All 200 changing Moon quotes in the tested 58-second slice matched a Pyth quote published within the preceding two seconds after truncation to five decimals. Verification used decimal arithmetic.

```text
Pyth 85926.33517496 -> Moon 85926.33517
Pyth 85926.07419782 -> Moon 85926.07419
Pyth 85927.58410801 -> Moon 85927.58410
Pyth 85927.37835333 -> Moon 85927.37835
Pyth 85930.51942822 -> Moon 85930.51942
```

This supports Pyth as Moon's source in that window. It does not prove exclusivity across all assets or times. Shared upstream infrastructure cannot be ruled out.

### Failed or limited data

- The intended five-minute capture suffered later connection gaps; only the approximately 201-second overlap was used.
- A separate live `real_time` attempt received no Moon quotes and was excluded from timing claims.
- Historical Pyth publication times compared with earlier Moon receipt times were supporting price evidence only, not a relative delivery-latency test.
- An earlier exploratory nearest-price search included future receipt times; it was not a causal betting signal.
- No accepted execution prices or server-side bet timestamps were collected.

## Offline display-delay replay

Script: `outputs/replay_display_delay.py` in the original workspace.

Method:

1. At each received Pyth update, inspect only the latest received Moon quote.
2. Select observations where the absolute gap is at least $0.50, a fixed descriptive threshold.
3. Freeze that Pyth price as the target.
4. Inspect Moon's received price after fixed delays.
5. Require Moon quotes to be no more than one second old and sufficient remaining recording coverage.

There were 211 qualifying observations.

| Delay after Pyth receipt | Moon reached or passed the frozen Pyth target |
|---|---:|
| 50 ms | 1.9% |
| 100 ms | 1.9% |
| 150 ms | 81.5% |
| 200 ms | 92.4% |
| 300 ms | 93.4% |
| 500 ms | 63.0% |

Reached/passed is evaluated at that specific delay, allowing one cent of tolerance in the initial gap's direction. It is not a cumulative first-hit statistic; later reversals can reduce the percentage.

This is not a profit simulation. Observations are correlated, the sample is short, and accepted prices, order delays, fees, and actual fills were not modeled. Subsequent data are outcomes, never signal inputs.

## Moon's rules and practical limitations

The investigation checked [Moon's Terms of Service](https://moon.com/policies/terms-of-service), updated September 15, 2026.

- Section 7.2: no guaranteed execution price, speed, or fill.
- Section 12.1: restricts automated monitoring/extraction except through expressly provided functionality.
- Section 12.2: automated betting and systematic exploitation of latency or pricing delays are prohibited without express written permission.
- Section 12.6: explicitly prohibits using faster external feeds to act on prices that have not updated.
- Section 12.10: affected bets can be voided, winnings withheld or reclaimed, and accounts restricted or closed.

A measured display delay is not proof of an executable or permitted betting advantage. The recommendation was offline analysis; actual execution testing should be arranged with Moon's permission in an authorized test environment. No execution testing was performed.

[Moon's fee explanation](https://help.moon.com/en/articles/15388257-how-do-the-fees-on-moon-work) stated a 1% opening fee on the wager and a performance fee of at least 10% of realized profit. Recheck current applicable fees before any new financial analysis. Do not directly compare a fee percentage on wager size with an unleveraged asset-price percentage without accounting for the different bases.

## Local files for continuing analysis

All paths below are relative to the original workspace, not files included in this repository.

```text
outputs/pyth_ticks.jsonl                 Successful simultaneous raw capture
outputs/pyth_comparison.json            Detailed metrics and lag curves
outputs/pyth_full_rate_history.json      Full-rate historical fingerprint data
outputs/pyth_price_fingerprint.json      Exact-match verification results
outputs/pyth_realtime_ticks.jsonl        Failed attempt; exclude from timing claims
outputs/pyth_prior_history.json          Earlier-window Pyth history
outputs/pyth_daily_example.json          User-supplied daily-history query result
outputs/multi_ticks.jsonl                Multi-exchange raw capture
outputs/multi.json                      Multi-exchange metrics
outputs/price_ticks.jsonl                Initial Moon/Binance capture
outputs/collect_pyth.mjs                 Authenticated collector
outputs/run_pyth_secure.py               Hidden credential prompt launcher
outputs/analyze_pyth.py                  Timing and price analysis
outputs/fetch_pyth_full_history.mjs      Historical retrieval
outputs/verify_pyth_prices.py            Decimal-precision fingerprint check
outputs/replay_display_delay.py          Offline display-gap replay
outputs/laya_benchmark.json              Model performance measurements
outputs/laya_direction_test.json         Direction-classification diagnostic
```

Raw capture records include `recv_ts_ms`, monotonic `recv_elapsed_ms`, `feed`, `price`, and source timestamps where available. Event records without a feed describe configuration, connections, errors, and completion. Pyth records additionally include batch timestamp, integer price mantissa, exponent, publisher count, confidence, and earliest endpoint.

Existing local reproduction commands, subject to reviewing current access permissions and using a new destination filename:

```sh
work/laya-venv/bin/python work/run_pyth_secure.py 300 outputs/pyth_new_ticks.jsonl
work/laya-venv/bin/python work/analyze_pyth.py outputs/pyth_new_comparison outputs/pyth_new_ticks.jsonl
```

The launcher prompts with terminal echo disabled and refuses to overwrite a raw capture. The analysis also references earlier local captures and history. These scripts and dependencies are not included in this context-only repository.

No capture was intentionally left running. No recurring monitor was created.

## Primary sources

- [Laya-MLX repository](https://github.com/mizorewww/laya-mlx)
- [Moon's provider explanation](https://help.moon.com/en/articles/15928641-how-does-moon-s-data-feed-work), which names DXFeed, Pyth Pro, and SEDA without disclosing precise BTC provider weights.
- [Moon Terms of Service](https://moon.com/policies/terms-of-service)
- [Moon risk disclosure](https://moon.com/policies/risk-disclosure)
- [Moon fee explanation](https://help.moon.com/en/articles/15388257-how-do-the-fees-on-moon-work)
- [Pyth subscriptions](https://docs.pyth.network/price-feeds/pro/subscribe-to-prices)
- [Pyth payload reference](https://docs.pyth.network/price-feeds/pro/payload-reference)
- [Pyth historical API](https://docs.pyth.network/price-feeds/pro/api/history)

## Suggested continuation

Ask what the user wants next: deeper explanation, longer read-only validation with appropriate access, or an offline paper-analysis tool. Do not assume authorization to place bets.

The key unresolved issue is Moon's actual accepted execution price and server-side timing. Public quote timing alone cannot resolve it. Avoid claims of guaranteed prediction, profitability, or an actionable 140/320 ms betting window.

If the next conversation cannot access the original workspace, it has this narrative context only. Request the relevant data or archive before attempting to reproduce or extend the numerical results.
