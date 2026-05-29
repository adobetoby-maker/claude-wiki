---
ai-first: true
type: project
date: 2026-05-28
tags: [dex, crypto, amm, uniswap, blockchain, pending]
status: archived
---

# DEX — AMM Exchange

## For future Claude
Uniswap V2 AMM fork targeting TrusteeSmartChain (TSC) and Salvorias (SAV). Phase 2 frontend complete; blocked on testnet deploy (needs wallet + faucet). Load this if the DEX project resumes.

---

## Status (as of 2026-05-10)
- Phase 1 (smart contracts): Complete, awaiting testnet deploy for live addresses
- Phase 2 (frontend): Complete — all 11 tasks done via subagent-driven dev
- Blocker: needs deployer wallet + testnet BNB to get contract addresses

## Repo
- GitHub: https://github.com/adobetoby-maker/DEX
- Local: `/Users/drive/dex-project/`
- Phase 1 worktree: `.worktrees/phase1-smart-contracts/dex-contracts/`
- Phase 2 worktree: `.worktrees/phase2-frontend/dex-frontend/`

## Key Technical Facts
- Uniswap V2 AMM fork on BNB/EVM
- `INIT_CODE_PAIR_HASH`: `be620b1a2e463c4ca0c010182689ecc39ef43f3323ae38f2bb8f75d05d150e6d`
- wagmi v2.19.5 (NOT v3 — RainbowKit v2 peer dep constraint)
- `tsconfig.json target: "ES2020"` required for BigInt literals (`0n`)
- `export const dynamic = 'force-dynamic'` in layout.tsx — prevents wagmi SSR localStorage crash

## Deploy Checklist (when resuming)
1. Choose deployer wallet, export private key
2. Get testnet BNB from bnbchain.org/en/testnet-faucet
3. Get free BSCScan API key at bscscan.com/myapikey
4. Fill `.worktrees/phase1-smart-contracts/dex-contracts/.env` with DEPLOYER_PRIVATE_KEY + BSCSCAN_API_KEY
5. Run: `forge script script/Deploy.s.sol --rpc-url https://data-seed-prebsc-1-s1.binance.org:8545 --broadcast --verify -vvvv`
6. Copy Factory + Router addresses → set as Vercel env vars → redeploy
7. Merge PR #1

## Key Files
- `lib/tokens.ts` — TOKENS array (WBNB, USDT, USDC, BTCB, ETH, XRP; SAV pending)
- `lib/contracts.ts` — ABIs + address constants
- `lib/dex.ts` — Pure AMM math (getAmountOut, getPriceImpact)
- `lib/wagmi.ts` — Chain defs; update for TSC chain ID
- `components/SwapPanel/` — swap UI
- `components/LiquidityPanel/` — add/remove liquidity
