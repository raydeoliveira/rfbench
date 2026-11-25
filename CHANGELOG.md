# Changelog

## 0.1.2
- Promoted `rf_touchstone_analyze` (`rfbench s-param`) from `$0.015` metered to Free (`$0`, zero auth, zero Apify PPE charge) so RF and NanoVNA users get full Touchstone `.s1p`/`.s2p` VNA analysis, de-embedding, passbands, and Rollett `K` stability free (`4` free / `2` metered tools: `rf_match_network`, `rf_fcc_compliance`).
- Exposed `GET`/`HEAD /llms.txt` and enriched `/.well-known/mcp/server-card.json` (`llmsTxt`, `documentationUrl`, structured `pricing`, `quickStart`) for self-discovering AI clients.

## 0.1.1
- Promoted `rf_txline` (`rfbench txline`) from `$0.005` metered to Free (`$0`, zero auth, zero Apify PPE charge) so RF engineers and AI agents get full Smith-chart transforms, Friis link budgets, and lossless/lossy transmission-line math free out of the box (`3` free / `3` metered tools).

## 0.1.0
- Implemented deterministic RF physics & FCC Part 15 math core (`smith_txline.ts`, `link_budget.ts`, `matching.ts`, `touchstone.ts`, `fcc_limits.ts`) with exact swept `-3 dB` matching bandwidth, FCC `4,096`-point / `64`-clock DoS bounds, extreme-scale overflow/subnormal scaling, and fail-closed `ERR_NUMERIC_OVERFLOW` guards.
- Added dual Human + AI-Agent CLI (`smith`, `link`, `match`, `txline`, `s-param`, `fcc`, `serve`, `mcp`) with `--json`, `--describe`, `--dry-run`, `statSync().isFile()` input checks, and `realpathSync` symlink entrypoint resolution.
- Added pay-per-call Streamable HTTP MCP (`2025-03-26`) deployed live at `https://rfbench-mcp.raymond-deoliveira.workers.dev/mcp` with flat object tool schemas, MCP `annotations` (`title`, `readOnlyHint: true`, `destructiveHint: false`, `idempotentHint: true`, `openWorldHint: false`), `GET`/`HEAD` `/health` & `server-card.json` (with `polarCheckoutUrl`), `405` + `Allow: POST, OPTIONS` on unsupported methods, `Mcp-Session-Id` CORS headers, fail-closed HTTP `402` pointing to live Polar checkout `https://buy.polar.sh/polar_cl_mOqAzUHFS8WLUnO5GA1QR7C7933zfjDqIA4h44MLSGA`, full 64-hex SHA-256 Bearer/KV cache keys, Polar `/validate` with `60 s`–`300 s` KV TTL clamping, deferred x402 v2 `/verify` -> execute -> `/settle` billing, and `glama.json` (`src/index.ts`, `src/adapters/http_server.ts`, `wrangler.jsonc`, `server.json`).
- Added Apify Actor batch/single runner, fail-closed Pay-Per-Event billing (`ERR_PAYMENT_REQUIRED` on rejected PPE charges), APIv2 + local storage adapter, Standby HTTP `/mcp` entrypoint (ignoring internal `apify_api_*` / runtime tokens during unauthenticated Standby PPE fallback), `.actor/*` schemas, and verified `234 MB` `node:22-alpine` `Dockerfile`.
- Added `6` reference fixtures under `fixtures/`, `SKILL.md`, `82` unit/integration tests across `9` test files (`23` suites), and `135` E2E tests across `11` files (`217` tests total).
