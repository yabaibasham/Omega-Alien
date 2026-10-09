# Omega Alien Engine — Mainnet Upgrade Notes

## What changed
- Added a production launch checklist and explicit separation of prepare → sign → broadcast → confirm.
- Added CORS allowlisting, request-size limits, security headers, and basic per-IP rate limiting.
- Added `/api/health` with live mainnet RPC readiness and an explicit `ALLOW_MAINNET_ACTIONS` gate.
- Removed automatic mainnet mint/airdrop behavior at server startup.
- Removed fabricated Solscan-looking transaction signatures from the staking mock endpoints; local references are no longer presented as blockchain signatures.
- Mainnet broadcast now requires RPC acceptance **and** confirmation with no on-chain error before returning `CONFIRMED`.
- Removed fake fallback wallet balances; unavailable chain data is no longer replaced with invented balances.
- Disabled synthetic volume, burner-holder inflation, reaction boosting, and coordinated pump endpoints in production.
- Kept wallet private keys out of the browser flow; wallet signing should occur client-side.

## Important reality check
Solscan is a block explorer, not an approval service. A transaction is only considered successful here after Solana RPC returns a signature and confirmation with `err === null`.

No software can honestly guarantee zero transaction failures or guaranteed token gains. The reliability target is: preflight, bounded retry, RPC failover, confirmation, idempotency, clear failure state, and no false-positive success UI.

## Before enabling mainnet actions
1. Configure a dedicated production RPC provider.
2. Set `CORS_ORIGINS` to the exact deployed origin(s).
3. Keep `ALLOW_MAINNET_ACTIONS=false` during staging.
4. Connect a test wallet and validate on devnet.
5. Validate token mint/freeze/update authorities and metadata.
6. Prepare and wallet-sign the exact mainnet transaction.
7. Confirm the returned signature on Solana and independently inspect it on Solscan.
8. Verify liquidity custody/lock state from on-chain accounts.
9. Only then enable production actions.

## Build note
The uploaded archive contained a prebuilt `node_modules` snapshot whose Rollup native optional dependency was missing in this Linux environment. `tsc --noEmit` passes after the upgrade; regenerate dependencies with `npm ci`/`npm install` on the deployment platform before the production Vite build.
