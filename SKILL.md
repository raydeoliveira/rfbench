---
name: rfbench
description: Deterministic Smith-chart, transmission-line, IEC 60063 E-series matching, Friis link-budget, Touchstone .s1p/.s2p, and 47 CFR Part 15 FCC compliance oracle exposed via CLI (--json/--describe), HTTP 402 Streamable HTTP MCP, and Apify Actor / Standby MCP.
---

# rfbench

Zero-dependency TypeScript RF physics and FCC Part 15 pre-compliance oracle (`node >= 22.12.0`).

## CLI Discovery & Invocation

- Inspect machine-readable commands, schemas, and side-effect classes:
  `node src/adapters/cli.ts --describe`
- Always pass `--json` for structured stdout envelope `{ schema_version: 1, ok, command, data, error }`.
- Validate parameters without network or stdio side effects via `--dry-run`.
- Non-interactive execution fails fast with exit `2` when required input is missing.

Subcommands (`8`):
- `smith` (`rf_smith_transform`) — Free: `Z`, `Y`, `Gamma`, `VSWR`, series/parallel RLC equivalents.
- `link` (`rf_link_budget`) — Free: Friis FSPL, EIRP/ERP, `290 K` noise floor, SNR, Fresnel radius, max range.
- `match` (`rf_match_network`) — Metered (`$0.01`): L/Pi/T conjugate matching, `E12`–`E96` snapping, finite-Q IL, tolerance corner VSWR.
- `txline` (`rf_txline`) — Free: Lossless/lossy `Z_in`, `Gamma_in`, `VSWR_in`, quarter-wave `Z0`.
- `s-param` (`rf_touchstone_analyze`) — Free: Touchstone v1.0 `.s1p`/`.s2p` (`<= 200 KB`, `<= 4096` pts), uniform single-pass phase rotation `S_ij * exp(+j * 2 * pi * f * delay_ps * 1e-12)` across `s11, s21, s12, s22` (use `delay_ps = 2 * tau` for 1-port round-trip VNA fixture extension of one-way delay `tau`), passbands, Rollett `K`.
- `fcc` (`rf_fcc_compliance`) — Metered (`$0.02`): 47 CFR Part 15 radiated/conducted limits and ISM harmonic comb checks.
- `serve` — Start standalone Node HTTP `402` Streamable HTTP MCP server (`--host`, `--port 0..65535`).
- `mcp` — Run stdio JSON-RPC 2.0 MCP server (`2025-03-26`, billing bypassed locally).

Reference inputs live in `fixtures/` (`antenna_915mhz.s1p`, `filter_1500mhz.s2p`, `match_915mhz_e24.json`, `fcc_radiated_part15b_class_a.json`, `fcc_conducted_class_b.json`, `fcc_ism_comb_2440mhz.json`).

## Streamable HTTP MCP (`2025-03-26`)

- **Hosted endpoint**: `https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp`
- **Glama Connector**: `https://glama.ai/mcp/connectors/io.github.raydeoliveira/rfbench`
- **Polar Checkout (`$29` Pro key)**: `https://buy.polar.sh/polar_cl_mOqAzUHFS8WLUnO5GA1QR7C7933zfjDqIA4h44MLSGA`
- `GET`/`HEAD /health` — Service status, version, protocol version, tool count (`6`).
- `GET`/`HEAD /llms.txt` — Plain-text agent quickstart with free/metered tools, `claude mcp add`, and payment headers.
- `GET`/`HEAD /.well-known/mcp/server-card.json` (and `GET`/`HEAD /`) — Tool catalog, `pricing`, `quickStart`, auth methods, and `polarCheckoutUrl`.
- `POST /mcp` — JSON-RPC 2.0 (`initialize`, `ping`, `tools/list`, `tools/call`; flat object tool schemas without `oneOf`/`anyOf`/`allOf` and read-only/idempotent MCP `annotations`; non-POST returns `405` with `Allow: POST, OPTIONS`; CORS exposes `Mcp-Session-Id`; notifications return HTTP `202`, body > `262,144` bytes returns `413`, invalid JSON returns `400`).

Authentication for metered tools (`rf_match_network`, `rf_fcc_compliance`):
1. `PAYMENT-SIGNATURE` (or `X-PAYMENT` / `Authorization: Payment`) — x402 v2 verify + settle (`PAYMENT-RESPONSE` header on `200`).
2. `X-Polar-License-Key` — Polar.sh `/validate` (`status === "granted"`).
3. `Authorization: Bearer <token>` — Constant-time SHA-256 lookup against `API_KEYS_CSV`.

Missing or invalid credentials on metered `tools/call` fail closed with HTTP `402`, `PAYMENT-REQUIRED`, and `WWW-Authenticate`.

## Apify Actor & Standby MCP

- **Actor mode** (`src/adapters/apify_main.ts`): Reads `INPUT` (`{ tool, arguments, chargeEvents? }` or `{ calls: [{ id?, tool, arguments }], chargeEvents? }`), writes Dataset items and Key-Value Store `OUTPUT`, and charges `.actor/pay_per_event.json` events only for successful metered calls.
- **Standby mode** (`APIFY_META_ORIGIN=STANDBY` or `RFBENCH_APIFY_MODE=standby`): Binds `ACTOR_WEB_SERVER_PORT` / `PORT` (`4321` default, `"0"` supported), answers `GET /` with `x-apify-container-server-readiness-probe` with HTTP `200`, and serves `/mcp` with Pay-Per-Event metering.

## Exit Codes

`0` success | `1` runtime error | `2` usage / `RfValidationError` | `3` capability unavailable | `4` permission denied | `5` license / HTTP `402` entitlement error | `130` `SIGINT`
