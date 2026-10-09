# Omega Alien Engine 1.5.5

## Release focus
Reliability, transaction truthfulness, secret-material containment, and production-safe simulation boundaries.

### Changes
- Mainnet transaction broadcasting is gated by `ALLOW_MAINNET_ACTIONS=true`.
- Broadcast confirmation uses the blockhash captured before send, bounded confirmation timeout, RPC failover, and a post-confirmation signature-status check.
- Optional `X-Idempotency-Key` prevents duplicate successful application-level responses during browser retries.
- Removed fabricated wallet balances and static transaction-signature fallbacks from Solana data paths.
- Removed hard-coded fallback blockhash/rent values from transaction construction.
- Server-generated wallet private keys are never returned to API clients and are no longer exportable from the UI.
- Simulated volume generation is opt-in via `ENABLE_SIMULATION=true`; production defaults to false.
- Autonomous local flywheel accounting is disabled unless simulation is explicitly enabled.
- Mainnet auto-cycle defaults to disabled.
- Fee/holder/burst paths fail closed rather than inventing balances or confirmed signatures.

## Validation
`npm run verify:release` performs dependency-free release assertions. Full dependency reinstall/build was attempted, but the supplied environment could not complete registry installation: the cached dependency tree is incomplete and network installation timed out. Run `npm ci`, then `npm run typecheck` and `npm run build` on the deployment environment before release.

## Mainnet launch steps
1. Run `npm ci`.
2. Run `npm run verify:release`.
3. Run `npm run typecheck`.
4. Run `npm run build`.
5. Keep `ALLOW_MAINNET_ACTIONS=false` during staging.
6. Exercise wallet signing and confirmation on devnet.
7. Verify real signatures independently on Solana RPC and Solscan.
8. Configure production RPC and exact CORS origins.
9. Set `ALLOW_MAINNET_ACTIONS=true` only after the preceding checks pass.

No software can guarantee zero transaction failures or consistent investment gains. 1.5.5 reports actual chain state and fails closed instead of manufacturing success.

## Final test status
- `npm run verify:release`: PASS.
- TypeScript parser pass: no TS1005/TS1109/TS1128/TS1136 syntax diagnostics after the final patch.
- Full typecheck/build: blocked only by the incomplete dependency snapshot in this runner; reinstall dependencies with `npm ci` on the deployment host.
