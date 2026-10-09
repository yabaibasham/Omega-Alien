# Deploying Omega Alien 1.6.0 (Windows 11)

The site is a Node server: it serves the pages and does the chain reads. Vercel's serverless model won't keep its 30-second refresher running, so this guide uses **Render** (Railway works the same way).

## 1. Get it running on your laptop (10 minutes)

1. Unzip to **`C:\omega-alien`**, not to OneDrive. OneDrive locks `node_modules` and breaks `npm` with EBUSY/EPERM errors.
2. Install **Node.js 22 LTS** from nodejs.org.
3. In PowerShell:
   ```powershell
   cd C:\omega-alien
   npm ci
   npm run verify:release
   npm run build
   copy .env.example .env
   notepad .env
   ```
4. In `.env`, set `OPERATOR_KEY` to a long random string and save. For local runs, also set `CORS_ORIGINS=http://localhost:3000`.
5. Start the server and open both sites:
   ```powershell
   npm start
   ```
   - Public site: http://localhost:3000
   - Console: http://localhost:3000/#/operator (paste your `OPERATOR_KEY`)

## 2. Put the code on GitHub (private)

Use GitHub Desktop: File → Add local repository → `C:\omega-alien` → Publish repository, with **Keep this code private** ticked. `.env` is already ignored by `.gitignore`, so your keys stay on your machine.

## 3. Deploy on Render

1. render.com → **New → Blueprint** → pick the repo. It reads `render.yaml`.
2. Fill in the empty variables:
   - `SOLANA_MAINNET_RPC`: a Helius (or similar) mainnet URL. The free public RPC rate-limits holder counts.
   - `CORS_ORIGINS`: your site URL, e.g. `https://omega-alien.onrender.com`.
   - Leave `OMEGA_MINT` empty for now.
   - Optionally set `OMEGA_LAUNCH_TIME` (e.g. `2026-10-24T19:00:00+11:00`) to show a countdown.
3. Render generates `OPERATOR_KEY` for you. Copy it from **Environment** and keep it in a password manager.
4. Deploy. When it's done, `/api/health` should return `ok: true`.
5. Custom domain: **Settings → Custom Domains**, add the CNAME at your registrar, then add the domain to `CORS_ORIGINS`.

The `starter` plan costs money because free instances sleep, and a sleeping launch site loses buyers in the first minutes.

## 4. Launch day

1. Create the coin on **pump.fun** with your own Phantom wallet, as a SOL-paired curve. You'll need SOL for the creation fee and any dev buy; a wallet holding about 0.001 SOL isn't enough.
2. Copy the mint address from the pump.fun page.
3. In Render → Environment, set `OMEGA_MINT` and save. Render restarts the service.
4. Check the live site:
   - The contract address appears with Copy, pump.fun, DexScreener and Solscan links.
   - "Live on-chain" shows bonding-curve %, SOL to graduation, price and market cap within 30 seconds.
   - The Solscan link opens the right token.
5. Post the address from the official X account, linking to the site.

## 5. Keep it honest after launch

- Fund buybacks only from money that actually arrived, for example **PumpSwap creator fees** after graduation. Do them from your wallet; each one has a Solscan signature you can share.
- Keep `ALLOW_MAINNET_ACTIONS=false` unless you're deliberately using a server-signed action. Turn it off again afterwards.
- `ENABLE_SIMULATION` stays `false` in production.
