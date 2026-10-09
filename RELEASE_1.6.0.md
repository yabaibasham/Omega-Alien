# Omega Alien Engine 1.6.0 — public release

One rule for this release: **nothing on screen counts as money unless it came from the chain.**

## Two sites, one server

| URL | Who | What |
|---|---|---|
| `/` | Everyone | Landing, contract address (or pre-launch notice + countdown), how to buy, live chain stats, treasury |
| `/#/operator` | You | The IDX console, behind `OPERATOR_KEY` |

## What is live and real

- **Token stats** are read server-side every 30 s once `OMEGA_MINT` is set. Bonding-curve progress and the SOL left to graduation come from the pump.fun curve account itself. Price, market cap, volume and liquidity come from DexScreener. Holder count is a direct Solana RPC count, refreshed every 5 minutes.
- **Treasury balance** is read from Solana every 60 s, pre-launch included.
- **Graduation planner** uses the real constant-product curve. It replaces the old made-up "2% per SOL". Checked against pump.fun's known numbers: a fresh curve needs 85.005 SOL and starts at 2.8e-8 SOL.
- **Copy** says graduation goes to **PumpSwap** (true since March 2025), not Raydium.
- **Real chain actions kept, now behind the operator key:** mint-all, SPL token creation, broadcasting transactions your wallet signed, Jupiter quotes, devnet airdrop. Mainnet signing still needs `ALLOW_MAINNET_ACTIONS=true`.

## Switched off, and why

Nothing was deleted; every route answers with the reason.

| Feature | What it actually did |
|---|---|
| Auto-cycle | Faked a buyback every 30 s: pushed bonding %, price +0.8%, market cap, and wrote `5kRaydiumPump…` signatures. The public site would have shown $OMEGA climbing on its own. |
| Seeded state | Started with a fake RARINU token (152K mcap, 62.4% bonding, 428 holders), 8.92 SOL "injected", fake ledger and syndicate wallets, and a $30.8M pool list. |
| Staking | Wrote stakes to a JSON file and returned made-up signatures. No tokens moved. |
| Pool create / LP / burn / revoke / bulk-send / metadata | Updated an in-memory list and reported success with `local-…` IDs. Now **501, not on-chain**, marked "not live" in the sidebar. |
| Volume bot, holders maker, reaction booster, syndicate cycle | They manufacture demand. 1.5.5 had already disabled them server-side; now they're also out of the UI. |
| Earning engine (opportunities, autopilot, agents, x402 demo, BLANK balance) | Produced SOL with `Math.random()` that never existed. |
| Launch page contract address `6uUUpKjR7…` | Not a real mint. Removed. |
| Copy | "GAINS", the "148% APY" badge, "$BASHAM" ticker, Raydium graduation, and a fake 14,820 whitelist with a countdown that restarted on every load. |

## Bugs fixed

- **`npm start` crashed on boot.** The CommonJS production bundle called `fileURLToPath(import.meta.url)`, so this app could never have been deployed. Fixed and guarded.
- The port is now read from `PORT`; it was hardcoded to 3000, which Render and Railway can't use.
- Live mode on the landing showed the simulated price under the "On-chain price" label whenever chain data was missing.
- Failed swaps were shown as "✅ Order Dispatched … Signature generated."
- The operator console tripped its own rate limit (default raised to 300/min).

## Security

- An `OPERATOR_KEY` is required on every non-public API route. If it isn't set, those routes stay locked (fail closed).
- Lockout after 10 distinct wrong keys per IP for 15 minutes. A stale key repeated by an open tab doesn't count.
- The key is kept in `sessionStorage` only.

## Verification (run in this build)

- `npm ci`: clean install
- `npm run verify:release`: 17/17 pass, including 9 new release guards
- `npm run typecheck`: 0 errors
- `npm run build`, then `npm start`: boots and serves
- Route policy probed: public 200, no key 401, wrong key 401, retired 410, not-on-chain 501, lockout 429
- No drift: public state identical after 35 s
- Browser: public desktop and 390 px mobile with no overflow; operator gate rejects wrong keys and unlocks with the right one; 0 JS errors

## Not verified here, check before launch

The build machine had no Solana or DexScreener access, so **live chain reads were not exercised**. On your deployed server:

1. Set `OMEGA_MINT` to any existing pump.fun coin and confirm the stats match its pump.fun page.
2. Holder counts need an RPC that allows `getProgramAccounts` (Helius and similar). On the public RPC that field shows "—".
3. pump.fun's USDC-paired curves (added May 2026) aren't decoded. Launch on a SOL curve.
4. Token Hub still lists a local registry (`.data/persisted_tokens.json`), so it is badged "not live".
