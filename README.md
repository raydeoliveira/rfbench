# rfbench

Deterministic Smith-chart, transmission-line, IEC 60063 E-series impedance matching, Friis link-budget, Touchstone `.s1p`/`.s2p`, and 47 CFR Part 15 FCC compliance math oracle over Streamable HTTP MCP (`2025-03-26`).

## Live Service

- **Streamable HTTP MCP**: `https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp`
- **Server Card**: `https://rfbench-mcp.raymond-deoliveira.workers.dev/.well-known/mcp/server-card.json`
- **Health Check**: `https://rfbench-mcp.raymond-deoliveira.workers.dev/health`
- **Official MCP Registry**: `io.github.raydeoliveira/rfbench` (`0.1.0`)
- **Product Page**: https://www.raymonddeoliveira.com/software#rfbench

## Tools & Pricing

| Tool | Tier | Price (USD) | Capabilities |
|---|---|---|---|
| `rf_smith_transform` | Free | `$0.00` | Impedance / admittance / `Gamma` / `VSWR` conversions, return loss, mismatch loss, quality factor `Q` |
| `rf_link_budget` | Free | `$0.00` | Friis FSPL, EIRP, receiver thermal noise floor, SNR, fade margin, link closure |
| `rf_txline` | Metered | `$0.005` | Lossy complex hyperbolic transmission-line transform (`tanh(gamma * l)`), electrical length, attenuation |
| `rf_match_network` | Metered | `$0.010` | L, Pi, and T matching synthesis, IEC 60063 E-series snapping (`E12`–`E96`), exact swept `-3 dB` and `VSWR <= 2.0` bandwidth |
| `rf_touchstone_analyze` | Metered | `$0.015` | `.s1p`/`.s2p` parser (`RI`/`MA`/`DB`, `<= 204,800` B, `<= 4,096` pts), delay de-embedding, passbands, Rollett `K` / `|Delta|` stability |
| `rf_fcc_compliance` | Metered | `$0.020` | 47 CFR Part 15 (`§15.205` restricted bands, `§15.207`, `§15.209`, `§15.249`, clock harmonics up to `4,096` pts / `64` clocks) |

## Authentication & HTTP `402` Settlement

- **Free tier**: No authentication required for `rf_smith_transform` and `rf_link_budget`.
- **x402 v2 (`PAYMENT-SIGNATURE`)**: Base Mainnet (`eip155:8453`) USDC (`0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913`) per-call settlement verified before tool execution and settled only after valid tool output (`RfValidationError` inputs never charge).
- **Polar License (`X-Polar-License-Key`)**: Unlocks metered tools via https://polar.sh/raymond-de-oliveira.
- **Bearer API Key (`Authorization: Bearer <key>`)**: Organization API key authentication.

Unauthenticated metered requests return HTTP `402` with Base64 `PAYMENT-REQUIRED` and `WWW-Authenticate: Payment` headers.

## Quickstart (`curl`)

Discover metadata and tool schemas:

```bash
curl -sS https://rfbench-mcp.raymond-deoliveira.workers.dev/.well-known/mcp/server-card.json
```

Call a free Smith-chart transform:

```bash
curl -sS https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"rf_smith_transform","arguments":{"z":{"re":25,"im":50},"freq_hz":100000000}}}'
```

Inspect flat object JSON Schemas (`tools/list`):

```bash
curl -sS https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

## Documentation & Security

- [SKILL.md](./SKILL.md) — MCP schema details, exit codes, and tool conventions
- [SECURITY.md](./SECURITY.md) — Request bounds, numeric overflow guards, and vulnerability reporting
- [LICENSE](./LICENSE) — Proprietary license terms
