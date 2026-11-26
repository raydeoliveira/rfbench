# rfbench

Deterministic Smith-chart, transmission-line, IEC 60063 E-series impedance matching, link-budget, Touchstone `.s1p`/`.s2p`, and 47 CFR Part 15 FCC compliance math oracle in zero-dependency TypeScript (`node >= 22.12.0`, Cloudflare Workers, and Apify Actor / Standby MCP).

## Quick Start

Calculate the standing-wave ratio of a 25 + j50 Ω load on a 50 Ω line at 100 MHz. This free hosted call needs only curl.

```bash
curl -sS https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp \
  -H 'Content-Type: application/json' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"rf_smith_transform","arguments":{"z":{"re":25,"im":50},"freq_hz":100000000}}}'
```

The response gives `vswr` of `4.265564437074637` and `return_loss_db` of `4.149733479708181` under `result.structuredContent`. To keep the example handy, star [RFBench on GitHub](https://github.com/raydeoliveira/rfbench).

### Find the 2:1 VSWR band in a Touchstone sweep

Paste a one-port VNA sweep as `.s1p` text. This anonymous call uses three synthetic samples on a 50 Ω reference.

```bash
curl -q -fsS --max-time 15 https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp \
  -H 'Content-Type: application/json' \
  --data-binary '{"jsonrpc":"2.0","id":2,"method":"tools/call","params":{"name":"rf_touchstone_analyze","arguments":{"touchstone_text":"# MHz S RI R 50\n900 0.5 0\n910 0 0\n920 0.5 0","vswr_threshold":2}}}'
```

VSWR is 3 at 900 MHz, 1 at 910 MHz and 3 at 920 MHz. RFBench linearly interpolates these sampled VSWR values to estimate the 2:1 crossings. The band in `result.structuredContent.passbands` runs from 905 to 915 MHz (`bandwidth_hz` = `10000000`).

## What It Does / What It Does Not Do

| Surface / Tool | Tier / Price | What It Does |
|---|---|---|
| `rf_smith_transform` / `rfbench smith` | Free (`$0`) | Exact `Z`, `Y`, `Gamma` (rectangular/polar), `VSWR + phase`, series/parallel RLC equivalents, and passive-region guards (`Re(Z) >= 0`, `\|Gamma\| <= 1` unless `allow_active: true`) |
| `rf_link_budget` / `rfbench link` | Free (`$0`) | Friis FSPL, EIRP/ERP, `290 K` thermal noise floor, SNR, binding link margin, 1st Fresnel midpoint radius, and max range |
| `rf_txline` / `rfbench txline` | Free (`$0`) | Lossless and lossy `Z_in`, `Gamma_in`, `VSWR_in`, wavelength, dissipative/transducer loss, and quarter-wave `Z0` |
| `rf_match_network` / `rfbench match` | Metered (`$0.01`) | Analytical L, Pi, and T matching with `Q_min` guards, `2^N` bracketing `E12`/`E24`/`E48`/`E96` snapping, finite-Q insertion loss, and `2^N` tolerance corner VSWR |
| `rf_touchstone_analyze` / `rfbench s-param` | Free (`$0`) | IEEE Touchstone v1.0 `.s1p`/`.s2p` (`HZ`/`KHZ`/`MHZ`/`GHZ`, `RI`/`MA`/`DB`), uniform single-pass phase rotation `S_ij * exp(+j * 2 * pi * f * delay_ps * 1e-12)` across `s11, s21, s12, s22` (use `delay_ps = 2 * tau` for 1-port round-trip VNA fixture extension of one-way delay `tau`), VSWR passband, insertion loss, reverse isolation, group delay, and Rollett `K`/`\|Delta\|` stability |
| `rf_fcc_compliance` / `rfbench fcc` | Metered (`$0.02`) | 47 CFR §15.107/15.109/15.207/15.209 radiated and conducted limits (`20*log10` >=30 MHz, `40*log10` <30 MHz), plus §15.247 ISM harmonic comb and §15.205 restricted-band collision checks |
| HTTP `402` MCP & stdio MCP | MCP `2025-03-26` | Live Worker at `https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp`: `GET`/`HEAD /health`, `GET`/`HEAD /.well-known/mcp/server-card.json`, `POST /mcp` (flat schemas, read-only/idempotent `annotations`, `405` + `Allow` on non-POST, Free, SHA-256 Bearer, Polar `/validate`, deferred x402 v2 `/verify`+`/settle`), plus `rfbench mcp` over stdin/stdout |
| Apify Actor & Standby MCP | PPE / Standby | Single (`{ tool, arguments }`) and batch (`{ calls }`) Dataset/`OUTPUT` runs plus Standby HTTP `/mcp` with readiness probe and Pay-Per-Event charging |

**What It Does Not Do**:
- No physical VNA or spectrum-analyzer USB/serial drivers (`$0` Day 1 stateless compute only).
- No full-wave EM/FDTD field simulation.
- No remote Touchstone URL fetching (inline payloads only; `<= 204,800` bytes and `<= 4096` frequency points).
- No Touchstone v2 `[...]` directives (Touchstone v1.0 only).

## Install & Uninstall

Use the hosted endpoint with curl or an MCP client. The public GitHub repository contains documentation and MCP metadata. The local CLI, fixtures and tests require a separate source checkout.

The source checkout requires Node.js `>= 22.12.0` (verified on `v25.9.0`; `0` runtime and `0` dev npm dependencies, `267,022` bytes / `260.8 KiB` source across `29` TypeScript files and `5,781` lines in `src/`, `312 KB` on disk via `du -sk`, `768 KB` project total):

```bash
node --test tests/*.test.ts
node --test e2e/*.test.ts
```

Measured runs on `node v25.9.0` (macOS):
- Unit & integration (`node --test tests/*.test.ts`): `82` tests across `23` suites in `9` test files (`82` pass, `0` fail, `0` skipped).
- End-to-end (`node --test e2e/*.test.ts`): `135` tests across `11` test files (`135` pass, `0` fail, `0` skipped).
- Total: `217` tests (`217` pass, `0` fail).
- Representative CLI latency (median of 5 runs on `node v25.9.0`): `388.5 ms` (`smith --z 25+j50 --freq-hz 100e6 --json`), `415.3 ms` (`s-param --touchstone fixtures/antenna_915mhz.s1p --json`).

Hosted & container deployment:
- **Live Cloudflare Worker**: Deployed at `https://rfbench-mcp.raymond-deoliveira.workers.dev` (MCP endpoint `POST /mcp`, agent summary `GET`/`HEAD /llms.txt`, discovery card `GET`/`HEAD /.well-known/mcp/server-card.json`, health `GET`/`HEAD /health`, telemetry in `METER_KV`).
- [Official MCP Registry listing](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.raydeoliveira%2Frfbench/versions/latest)
- **Glama Connector**: https://glama.ai/mcp/connectors/io.github.raydeoliveira/rfbench
- **Polar Pro License (`$29` perpetual, up to 3 devices)**: https://buy.polar.sh/polar_cl_mOqAzUHFS8WLUnO5GA1QR7C7933zfjDqIA4h44MLSGA
- **Apify Actor / Standby**: Verified `234 MB` `node:22-alpine` container (`FROM node:22-alpine`, zero `npm install`, non-root `USER node`, requires Node `>= 22.18` native type-stripping) across batch Actor and HTTP Standby modes.

To uninstall locally, remove the directory (`rm -rf rfbench`).

## CLI and MCP examples

### 1. Local CLI (source checkout)

Run Smith-chart conversion and inspect structured `--json` output:

```bash
node src/adapters/cli.ts smith --z 25+j50 --freq-hz 100e6 --json
```

Observed output:

```json
{"schema_version":1,"ok":true,"command":"smith","data":{"z0":50,"allow_active":false,"is_passive":true,"z_norm":{"re":0.5,"im":1},"y_norm":{"re":0.4,"im":-0.8},"z":{"re":25,"im":50},"y":{"re":0.008,"im":-0.016},"gamma_rect":{"re":0.07692307692307689,"im":0.6153846153846153},"gamma_polar":{"mag":0.6201736729460422,"deg":82.87498365109822},"vswr":4.265564437074637,"return_loss_db":4.149733479708181,"mismatch_loss_db":2.1085336531489314,"reflected_power_ratio":0.3846153846153845,"loaded_q":2,"equivalent_circuit":{"freq_hz":100000000,"omega_rad_s":628318530.7179586,"series":{"r_s":25,"x_s":50,"l_s_h":7.957747154594767e-8,"c_s_f":null},"parallel":{"r_p":125,"x_p":62.5,"g_p":0.008,"b_p":-0.016,"l_p_h":9.94718394324346e-8,"c_p_f":null}}},"error":null}
```

Run included reference fixtures directly:

```bash
node src/adapters/cli.ts s-param --touchstone fixtures/antenna_915mhz.s1p --json
node src/adapters/cli.ts s-param --touchstone fixtures/filter_1500mhz.s2p --json
node src/adapters/cli.ts match --input fixtures/match_915mhz_e24.json --json
node src/adapters/cli.ts fcc --input fixtures/fcc_radiated_part15b_class_a.json --json
node src/adapters/cli.ts fcc --input fixtures/fcc_conducted_class_b.json --json
node src/adapters/cli.ts fcc --input fixtures/fcc_ism_comb_2440mhz.json --json
```

### 2. HTTP `402` Streamable HTTP MCP Server

From a source checkout, dry-run and start the local HTTP `402` MCP server:

```bash
node src/adapters/cli.ts serve --dry-run --json
API_KEYS_CSV="demo-key" node src/adapters/cli.ts serve --port 0
```

Unauthenticated metered `POST /mcp` calls fail closed with HTTP `402` plus Base64 `PAYMENT-REQUIRED` and `WWW-Authenticate` headers (linking to live Polar checkout `https://buy.polar.sh/polar_cl_mOqAzUHFS8WLUnO5GA1QR7C7933zfjDqIA4h44MLSGA` and x402 v2 requirements); authenticated calls (`Authorization: Bearer demo-key`, `X-Polar-License-Key`, or x402 v2 `PAYMENT-SIGNATURE`) return HTTP `200` with JSON-RPC `result.structuredContent`.

Connect Claude Code (free tier out of the box, or pass `-H "X-Polar-License-Key: ..."`):

```bash
claude mcp add --transport http rfbench https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp
```

For a client that accepts `mcpServers` JSON, configure the free hosted tools with the server URL.

```json
{
  "mcpServers": {
    "rfbench": {
      "url": "https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp"
    }
  }
}
```

### 3. Apify Actor & Standby MCP (source checkout)

Run offline Actor mode against `APIFY_LOCAL_STORAGE_DIR`:

```bash
APIFY_LOCAL_STORAGE_DIR=./storage node src/adapters/apify_main.ts
```

Run Standby MCP mode (reads `ACTOR_WEB_SERVER_PORT` / `PORT`, preserves `0` for ephemeral bind, responds `200` to `GET /` with `x-apify-container-server-readiness-probe`):

```bash
APIFY_META_ORIGIN=STANDBY ACTOR_WEB_SERVER_PORT=0 node src/adapters/apify_main.ts
```

## Safety & Compute Bounds

- **Stateless `$0` hardware compute**: Zero OS driver installation and zero hardware state mutation.
- **Fail-closed input & numeric bounds**: Non-finite values, non-positive `z0`/`freq_hz`, active impedances without `allow_active: true`, unknown JSON fields (`ERR_UNKNOWN_FIELD`), prototype-chain keys (`__proto__`, `constructor`, `prototype`), and extreme-scale numeric overflows (`ERR_NUMERIC_OVERFLOW`) fail deterministically via `RfValidationError`.
- **JSON-safe singularities**: Singular values (`VSWR` at `|Gamma| >= 1`, open-circuit `Z`, short-circuit `Y`) serialize as `null`, with `-0` normalized to `+0`.
- **Bounded remote surfaces & exact matching sweeps**: HTTP `POST /mcp` enforces a `262,144`-byte request limit (`413` on overflow, `400` on invalid JSON, `405` + `Allow: POST, OPTIONS` on unsupported methods), Touchstone caps input at `204,800` bytes and `4,096` frequency points, FCC Part 15 caps input at `4,096` points and `64` clock frequencies (`ERR_TOO_MANY_POINTS` / `ERR_TOO_MANY_CLOCKS`), and matching reports swept `-3 dB` transducer gain `bandwidth_3db_hz` and `vswr_2_bandwidth_hz` directly from ladder ABCD analysis.
- **Fail-closed `402`, deferred x402 settlement & Apify PPE billing**: Authorization (`/verify` for x402, `/validate` for Polar with full 64-hex SHA-256 cache keys, or SHA-256 Bearer) runs before tool argument validation on metered HTTP MCP calls (`402` with `PAYMENT-REQUIRED` and `WWW-Authenticate` on missing/invalid credentials), while x402 `/settle` executes only after tool execution succeeds so `RfValidationError` inputs never charge the caller. In Apify Actor mode, metered calls with `chargeEvents` enabled (default `true`) fail closed with `ERR_PAYMENT_REQUIRED` and withhold metered output data if `chargeEvent` rejects or throws (pass `chargeEvents: false` for unpaid local runs); free and failed calls never emit Apify Pay-Per-Event charges. In Apify Standby (`APIFY_IS_AT_HOME=1`), internal `apify_api_*` / runtime tokens are ignored unless an explicit external credential or `API_KEYS_CSV` is present; unauthenticated metered requests fail closed (`402`/`500`) on PPE failure, while explicit external auth headers authenticate via the HTTP `402` gate without double-charging PPE.
- **Clean shutdown**: `serve` and Standby `apify_main.ts` close sockets and flush pending metering tasks on `SIGINT` (`130`) and `SIGTERM` (`0`).

## Exit Codes & CLI Reference

| Code | Meaning |
|---|---|
| `0` | Success |
| `1` | Runtime error |
| `2` | Usage or `RfValidationError` input error (including closed non-TTY stdin) |
| `3` | Capability unavailable |
| `4` | Permission denied |
| `5` | License or HTTP `402` payment entitlement error |
| `130` | `SIGINT` interrupted |

In a source checkout, inspect CLI flags and schemas with `--help` or `--describe`. Reference inputs live under `fixtures/`.

- `fixtures/antenna_915mhz.s1p`
- `fixtures/filter_1500mhz.s2p`
- `fixtures/match_915mhz_e24.json`
- `fixtures/fcc_radiated_part15b_class_a.json`
- `fixtures/fcc_conducted_class_b.json`
- `fixtures/fcc_ism_comb_2440mhz.json`

## License & Support

Proprietary commercial software. Copyright (c) 2026 Raymond de Oliveira. All rights reserved. See `LICENSE` and `SECURITY.md`.
