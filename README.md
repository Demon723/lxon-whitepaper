# LXON Network Whitepaper

<!-- lxon-724-restart-notice:begin -->
> ⚠️ **Chain 724 addresses quoted in this file no longer exist.** Measured 2026-11-04 against `https://rpc.lxon.in`. Chain 724 restarted from an empty volume on 2026-11-03 under its original genesis (block 0 `0x6c931dff...`), so blocks 1-242,493 -- and every contract created in them -- are gone. Each address below returns **0 bytes of code** on 724 today:
>
> - `0x99801a78639975d448eeb2f8a807738dc8df67dc` - LXON (LxonTokenEnhanced)
> - `0xc0cf6b0e6c5a0705f319ba5c629a6bc7e82fef00` - the 724 wrapper quoted as "fully collateralised right now" in §Solution -- `eth_getCode` returns 0 bytes, so the 40,000 / 40,000 / 40,000 equality below it is a **2026-10-29 reading of a contract that no longer exists**
> - `0x8e90675e054826f61745c0016b895cdc4f220ebc` - its peg pool
> - `0x272a3ed4b91112005ade5b4161af1adc856ab3f4` - the LXON/WETH pair, which every 724 reserve figure in this file (footnotes [¹], [²]) was read from
>
> The 724 market and peg layers were re-created on **2026-11-12** at different addresses — wrapper `0x4891fA54…`, WETH9 `0x189dE8AA…`, pairs `0x3fc4133f…` and `0x9d082e58…`, routers `0x076e8e49…` and `0x840c9ca4…`, all with code — and both pairs read `getReserves()` = 0/0 with LP `totalSupply()` 0, so the venue is back and empty. Full table in §Price.
>
> The control that makes those empty reads evidence, measured by the same script: the v4.1 bridge on Polygon 137 is 19,105 bytes and wLXON is 2,106 bytes, so a working reader does return bytes when something is there. The rebuilt LXON on 724, `0x57b875ab86a40D79A1c35e44aa22008dfD049dc9`, is 8,378 bytes and answers `totalSupply()` = 4,071,052.000001 on the same endpoint in the same pass, so the four zeros above are the chain, not my reader. Read everything below as history or as a redeploy target, not as live state. Per-address ledger and method: `docs/724-claim-ledger-2026-11-03.md`.
<!-- lxon-724-restart-notice:end -->

<!-- lxon-price-surface-notice:begin -->
> ✅ **There is a machine-readable price feed again — restored and live 2026-11-13T21:15Z.** `https://www.lxon.in/swap/price.json` returns **200 `application/json`**, and the historical `https://explorer.lxon.in/swap/price.json` returns byte-identical JSON with `access-control-allow-origin: *` (sha256 of both bodies matched in one run). It is regenerated every five minutes on the chain-724 node by `/opt/lxon/price-updater.py` from `getReserves()`, `totalSupply()`, `decimals()` and `escrowed()` — never from a stored number — and it publishes its own `generated_at_utc`, the two Polygon RPCs that had to agree before anything was written, and a `backing` block that puts the **0.00 % coverage** next to the price. What it did NOT replace: `explorer.lxon.in` still answers the 4,110-byte offline notice on `/`, `/ui/` and every `/api/v2/*` path, because there is still no Blockscout on this node — `docker ps` shows exactly one container, `lxon-besu-sealer` — and `www.lxon.in/swap/` is still the 6,501-byte offline notice rather than a trading form.
>
> **The history below is kept deliberately, and the reason it lapsed is the useful part.** Measured 2026-11-13T19:48Z, before the fix: nine paths on the explorer host all returned 200 / `text/html` / 4,110 B including the LIVE token's own `/api/v2/tokens/0x57b875ab…` page, `www.lxon.in/price.json` and `/swap/price.json` were 404, and the controls (`www.lxon.in/` 93,127 B, `www.lxon.in/assets/lxon-mark-192.png` `image/png` 22,620 B) proved the reader distinguishes bytes from HTML. The old updater read the erased pair `0x272a3ed4…`, raised `empty reserves`, exited non-zero, and left nothing published for ten days while six documents — several of them mine, rewritten that same day — reported the absence as a finding instead of closing it. Deployment, reproducible commands, and the volume-census correction (the published "zero Swap events" was checked against a topic0 **no pool emits**, and re-ran clean against the right one): [`docs/restore-price-feed-2026-11-13.md`](../restore-price-feed-2026-11-13.md).
<!-- lxon-price-surface-notice:end -->
> **Canonical copy.** This is the version inside the repository, and it is the one to publish or attach.
> Generated from the working copy at `WHITEPAPER.md` in the local (non-git) directory; if the two disagree,
> this file wins and the other should be re-synced from it.

**Version 1.0 — 2026-10-29.** This version replaces the 2026-10-01 draft (which was written before
mainnet existed and described a token that was never deployed). Every number here was read from a public
endpoint on 2026-10-29 and is footnoted with the command that produced it. The original is preserved at
[`superseded-2026-10-29/WHITEPAPER.md`](./superseded-2026-10-29/WHITEPAPER.md),
and the measurement register it is derived from is
[`ON-CHAIN-FACTS-2026-10-29.md`](./ON-CHAIN-FACTS-2026-10-29.md).

If any figure in this document disagrees with the chain, the chain is correct and this document is stale.

---

## Executive summary

LXON Network is a sovereign EVM blockchain operating as **chain ID 724**, live since 2026-10-18. It runs
**Besu in QBFT consensus** with Ethereum's Cancun EVM rules and a **5-second block interval** measured
from consensus-stamped block timestamps. The network carries one ERC-20 settlement asset, **LXON**, an
on-chain AMM for it, and an attested bridge that moves it 1:1 to and from Polygon as **wLXON**.

The scope statement, because the previous draft overstated it: this is a **young, single-operator
network**. Re-read 2026-10-29, the node tip is block **182,718** (13:23 UTC) and the explorer's own
counters, read at 13:34 UTC, report **181,596 blocks, 138 transactions across 47 addresses**, a holder
list of **12 addresses — 11 with a non-zero balance — whose balances sum to `totalSupply()` exactly**,
and **three** settled bridge crossings totalling **31,152** tokens. It is not
decentralised, it is not audited, and its market is a $2.45 pool. Those sentences are the difference
between this document and the one it replaces.

## Problem statement

The challenges LXON is built against are the ordinary ones, restated without invented evidence:

Custodial venues concentrate counterparty risk, and the failures are on the record. Mainnet fees make
small settlements uneconomical, and roll-ups trade one dependency for another. A project on a shared
chain inherits someone else's governance, gas policy and congestion. Most tokens are volatile because
their price is set by external speculative flow rather than by any internal mechanism, and stablecoins
solve that by importing a centralised issuer and off-chain collateral.

The gap LXON targets is **sovereignty over the whole stack** — consensus, gas, token issuance and
settlement rails under one operator — accepting the trade-off that a single operator is also a single
point of failure. §Risk factors states that trade-off rather than hiding it.

## Solution

LXON Network provides, today, on chain 724:

- **A public L1** — Besu/QBFT, Cancun EVM, 5 s blocks, chain ID 724, HTTPS JSON-RPC at
  `https://rpc.lxon.in`. A Blockscout explorer was served at `https://explorer.lxon.in` with a public REST
  API; **as of the 2026-11-13 reading in the notice above that host serves only the offline HTML notice on
  every path**, so today the RPC is the checkable endpoint and the explorer is not.
- **The LXON ERC-20** at `0x99801A78639975D448EEb2f8a807738dc8df67Dc` — 18 decimals, total supply
  **42,700,020,000** (re-read at block 182,718, 2026-10-29; it was 100,020,000 seven hours earlier), no
  contractual cap. [¹]
- **A native liquidity pool** — a Uniswap-V2-style pair, LXON/WETH at
  `0x272A3eD4B91112005aDE5b4161aF1AdC856ab3f4`, behind a router at
  `0xd969211d7efB154e86D41A80ff0fd3cC0eB70468`, with the chain's WETH contract at
  `0xcB742c31f0815648E13fF24D8131B89075FF32e2`. Reserves are 10,001,000 LXON against 999.900309939 WETH,
  and **both sides were provided by the deployer**. [²]
- **An attested bridge (v4.1)** — `0x9B15fEe95995a15Da2d64D8E2958f9792D5D6a25` on 724 escrows LXON, and
  `0x7A3c0C34f157EfDD3E4913c5224C15E645469093` (`Wrapped LXON`, symbol `wLXON`, 18 decimals) mints the
  mirrored asset on Polygon 137. **Three** crossings have settled (1,000, then 10,000, then 20,152). Read
  on 2026-10-29 at 724 block 182,718: `escrowed(LXON)` = 31,152, `totalSentFor(137, LXON)` = 31,152,
  `laneNonce` = 3, and 137's `totalReceivedFrom(724, LXON)` = 31,152 against a wLXON `totalSupply()` of
  31,152 — **exactly 1:1 across four endpoints**, with `configSealed()` true on both sides. Note this
  contract is **not source-verified on the explorer** (see §Risk factors). [³]
- **A separate 1:1 wrapper on 724** — `0xC0CF6B0E6C5a0705F319BA5c629a6Bc7E82FEf00`, with its own peg pool
  at `0x8e90675e054826f61745C0016b895Cdc4F220eBc`. Read at the same block: `totalSupply()` = 40,000,
  `collateralBalance()` = 40,000 and its LXON `balanceOf` = 40,000, so it is fully collateralised **right
  now** — but `mint()` on this contract is `onlyRole(MINTER_ROLE)` with no backing check, so that equality
  is a measurement, not something the contract enforces.
- **A read-only public price feed**, regenerated every five minutes from the pools' own `getReserves()`:
  `https://www.lxon.in/swap/price.json`, also served at the historical
  `https://explorer.lxon.in/swap/price.json` with a permissive CORS header. It was dead from the 2026-11-03
  restart until **2026-11-13T21:15Z** (see the notice at the top of this file), it carries
  `generated_at_utc` and the RPCs that had to agree, and it publishes `volume` only when the same query
  proves trades on a busy control pair. It is a ratio, not a traded price — `caveats[0]` in the payload says
  so in those words.

What is *not* part of the network, despite appearing in the earlier draft: there is **no Chainlink
integration, no deployed price oracle, and no price-floor mechanism**. Nothing in the contract set
maintains a floor, and the feed publishes a pool-derived ratio rather than a market quote.

## Technology

**Consensus and nodes.** Besu with QBFT round-based finality, Cancun EVM opcodes, 5-second target
interval, 30,000,000 gas limit. The producer set that actually signed blocks is in §Decentralisation,
because that is the part of the consensus story a reader can check for themselves.

**Contracts.** Solidity `0.8.26` (`v0.8.26+commit.8a97fa7`), verified on the explorer:
`GET /api/v2/smart-contracts` returns **24 verified contracts**, all compiled with that exact version,
each readable and rebuild-verifiable in the browser. There is no upgradeable proxy in the set — the
EIP-1967 implementation slot reads `0x0` on the principal addresses — so what is deployed is what you can
read.

**Access control.** OpenZeppelin AccessControl with role-granting, under a `TimelockController` where one
was wired. Timelocks exist on both chains with a 172,800-second (48-hour) delay; the deployer measures as
`PROPOSER`, `EXECUTOR` and `CANCELLER` on the Polygon one. Roles are renounceable; §Tokenomics states
which have not been renounced.

**Bridge design.** The v4.1 bridge settles on attestation quorum rather than light-client proofs:
configured attestors sign a claim digest, the destination contract verifies the bundle against the signer
set, and `seal()` freezes configuration one way. `configSealed` on Polygon flipped the first time a claim
completed, so destination-side changes after that point go through the 48-hour timelock — a design
property worth knowing before planning a change, not a defect.

**Cryptography on the roadmap.** The node and the TypeScript SDK carry post-quantum primitives (ML-KEM,
ML-DSA, SLH-DSA, FALCON via `@noble`) that are wire-compatible with the node's own protocol. They are
**not in consensus** and have not been validated against NIST KAT vectors. Any claim that the chain itself
is quantum-resistant is not supported by the code.

## Tokenomics

### Token details — as deployed

| Field | Value |
|---|---|
| Name | `LXON` |
| Symbol | `LXON` |
| Standard | ERC-20 |
| Contract | `0x99801A78639975D448EEb2f8a807738dc8df67Dc` (chain 724) |
| Decimals | **18** |
| `totalSupply()` | **42,700,020,000** (re-read 2026-10-29 at block 182,718) |
| Max supply | **none.** The contract is mintable by `MINTER_ROLE`; there is no cap to publish. |
| Holders | 12 addresses in the explorer's holder list; 11 hold a whole token, the twelfth holds 1 wei. Their balances sum to `totalSupply()` exactly, to the wei |
| Transfers ever recorded | 35 |

The earlier draft claimed 6 decimals, a 10,000,000,000 total supply and 100,000,000 circulating. **All
three numbers were wrong, and the token address it named has no code on chain 724.** They are removed
rather than softened, because a listing application that repeats them turns a typo into a misstatement.
The figure this document itself published seven hours ago — 100,020,000 — was correct when measured and
is stale for the same reason: it was overtaken by two more `mint()` calls, one of them 42.5 billion
tokens. That is stated here rather than quietly re-derived, because a reviewer comparing this paper to an
earlier copy will otherwise conclude the document is simply wrong.

### How the supply came to be

Six `mint()` events — every one of them from `0x0`, all signed by the single key that holds
`MINTER_ROLE` — and the arithmetic closes exactly against `totalSupply()`:

| Block | To | Amount |
|---|---|---|
| 44 | deployer `0xFaE699d2…` | 0.0001 |
| 16,391 | deployer `0xFaE699d2…` | 99,999,999.9999 |
| 146,396 | attestor `0x28369733…` | 10,000 |
| 146,422 | founder `0x7F715B7F…` | 10,000 |
| 173,297 | founder `0x7F715B7F…` | 100,000,000 |
| 176,301 | founder `0x7F715B7F…` | **42,500,000,000** |
| | **sum** | **42,700,020,000** |

The last two were not in any plan. The block-173,297 mint is recorded as an incident — 100,000 LXON was
requested and 100,000,000 was signed — and the block-176,301 mint is 425 times the entire supply that
existed before it, minted to the founder wallet. Both are documented, with the correction options and
their costs, in [`MINT-INCIDENT-2026-10-29.md`](./MINT-INCIDENT-2026-10-29.md). Whether the larger one was
intended is the owner's answer, and this document does not guess it on their behalf.

One transfer sent 100 LXON to `0x000000000000000000000000000000000000dEaD`. That was a transfer to a dead
address, **not** a `burn()` call: `totalSupply()` did not decrease and the dead address still appears in
the holder list. Circulating supply therefore equals total supply.

### Distribution — what is real

There is **no vesting schedule and no lock-up contract holding LXON**. Every allocation table that has
appeared in this project's documents — Team 20% over two years, Advisors 10%, Community 30%, Development
20%, Reserve 20% — describes a plan that was never deployed, and the on-chain holder set accounts for
100% of supply with nothing locked:

| Holder | Kind | Balance | Share |
|---|---|---|---|
| `0x7F715B7F…` **founder** | EOA | 42,599,999,000 | **99.765759%** |
| `0xFaE699d2…` deployer / block producer | EOA | 89,766,747.999999999999999999 | 0.210226% |
| `0x272A3eD4…` LXON/WETH pair | contract | 10,001,000 | 0.023422% |
| `0x24ad1dE7…` | contract | 100,000 | 0.000234% |
| `0x9B15fEe9…` **v4.1 attested bridge** (31,152 escrow + 30,000 stake) | contract | 61,152 | 0.000143% |
| `0xC0CF6B0E…` wLXON wrapper | contract | 40,000 | 0.000094% |
| `0x28C93C2a…` legacy bridge | contract | 30,000 | 0.000070% |
| `0x8e90675e…` peg pair | contract | 11,000 | 0.000026% |
| `0x28369733…` attestor | EOA | 10,000 | 0.000023% |
| `0x37e1c5eC…` | contract | 1,000 | 0.000002% |
| `0x…dEaD` | dead address | 100 | <0.000001% |
| `0x710011e5…` unfunded USDC* pair | contract | 0.000000000000000001 | <0.000001% |
| **sum** | | **42,700,020,000** | **100.000000%** |

Every balance above was read at the same block tag — 724 block 183,244, 2026-10-29 — and **these twelve
rows sum to `totalSupply()` exactly at the wei level**, which is the completeness check rather than a
claim: the deployer sits one wei under 89,766,748 and the "empty" USDC* pair holds that wei, so the
accounting closes to the last unit. Eleven addresses hold a whole token; twelve hold something. The
explorer's holder page displays the twelfth as `0.00`, which is a rounding, not a zero. The
machine-readable copy is
[`supply-distribution-2026-10-29-post42-5b.csv`](./supply-distribution-2026-10-29-post42-5b.csv).

**Two EOAs hold 99.976% of supply between them.** The deployer EOA `0xFaE699d2…` minted every token that
exists, deployed the contracts, and holds `MINTER_ROLE` and `DEFAULT_ADMIN_ROLE`; the founder EOA
`0x7F715B7F…` is where 99.77% of it now sits, because the two largest mints were sent to it. **Every LXON
that has ever moved either left the deployer address or moved between wallets the project controls**;
there is no third-party holder in the 35-event transfer log, and the free float outside project addresses
is 0.00 LXON.

The route from here to a defensible supply position — cap or renounce `MINTER_ROLE`, move LP custody
under the existing 48-hour timelock (designed, and rehearsed on forks of both chains, not yet executed),
and publish unlock schedules only once contracts enforce them — is set out in
[`SUPPLY-AND-CUSTODY-2026-10-29.md`](./SUPPLY-AND-CUSTODY-2026-10-29.md).

### Price

There are three pools on this asset — the LXON/WETH pair on 724, the wLXON/LXON peg pair on 724, and the
wLXON/USDC pair on Polygon 137 — and only one of them quotes a currency that exists.

> **Corrected 2026-11-13, and corrected again the same hour.** Two of those three are not pools any more:
> `0x272a3ed4…` and `0x8e90675e…` return **0 bytes of code** on chain 724, because they were created in the
> blocks the 2026-11-03 restart erased. **My first reading of what replaced them was wrong.** I probed the
> old addresses, read the 2026-11-10 market-layer card ("staged, rehearsed, not signed"), concluded "nothing
> was redeployed", and wrote that into the listing documents — without grepping the runbooks directory, which
> contains [`deploy-724-full-sequence-2026-11-11.md`](../runbooks/deploy-724-full-sequence-2026-11-11.md)
> §7, titled *"Executed 2026-11-12 — the batch is on chain"*. The card's filename is a day older than its
> own execution section, which is the trap. The market and peg layers were in fact broadcast on
> **2026-11-12** at new addresses, at nonces 19-24 of the same deployer (whose nonce I had just read as
> `0x1d` = 29 and should have treated as the answer rather than as noise). Measured on `rpc.lxon.in`
> 2026-11-13T19:5xZ:
>
> | role | address | code | what it holds |
> |---|---|---|---|
> | `LXONWrapperSecure` (wLXON on 724) | `0x4891fA54AD4c57a160FB56Be1382cDc5512d4Db1` | 5,342 B | `totalSupply()` 0, `collateralBalance()` 0 |
> | WETH9 | `0x189dE8AA752eBdD5845E98eFaF6088562a12e2AB` | 2,396 B | — |
> | PairSecure (WETH9, LXON) | `0x3fc4133F0922C2A7BC465EA290a7819046524d32` | 5,453 B | `getReserves()` = **0 / 0**, LP `totalSupply()` **0** |
> | RouterSecure bound to it | `0x076e8e49BDB7e4cF499cA9F3319a31D59c4e68Cb` | 6,193 B | — |
> | PairSecure (wrapper, LXON) | `0x9D082e58E9C1cC7f45C0bE45B78FbF9d34ed6bC0` | 5,453 B | `getReserves()` = **0 / 0**, LP `totalSupply()` **0** |
> | RouterSecure bound to it | `0x840c9ca44CBBCc5A1529399b00b712161fA7525D` | 6,193 B | — |
>
> The `token0()`/`token1()` reads confirm the wiring rather than trusting a deploy log: pair 1 is
> `0x189dE8AA…`/`0x57b875ab…` and pair 2 is `0x4891fA54…`/`0x57b875ab…`, i.e. WETH-LXON and wrapper-LXON.
> So the accurate sentence is: **the 724 venue exists again and is empty.** The three pools named above are
> still three, but the two on 724 are new contracts at new addresses and both read 0 / 0, which is exactly
> what the 2026-11-10 and 2026-11-11 cards predicted for an unfunded seed ("Deploying the venue without a
> seed is defensible only as preparation"), so the only one
> with anything in it is the Polygon pair — the whole quote side of it is **$1.224879**. [⁴]

**On Polygon 137 — the market-facing one.** A wLXON/USDC pair (QuickSwap V2,
`0x8bFB69eD898654D49cCBB59B7E594280dF4CB8fd`) holds **1.224879 USDC against 5.198977 wLXON** — re-read
2026-10-29 on `polygon.drpc.org`, and the pair has emitted **zero `Swap` events** -- re-censused 2026-11-13 across **916,741 blocks** of its life with 0 unanswered ranges, giving `Swap 0 / Mint 1 / Burn 0 / Sync 1` (topic0 = `keccak("Swap(address,uint256,uint256,uint256,uint256,address)")` = `0xd78ad95f...`, and the census is only meaningful because the identical query returns trades on the control pair QuickSwap V2 USDC.e/WMATIC `0x6e7a5faf...`; `docs/restore-price-feed-2026-11-13.md` §4) — which is a
spot price of **$0.2356 per wLXON** and a total depth of **$2.45**. The LP is held by the bridge timelock
`0xe56553Ed…`, not by an EOA: the pair's LP `totalSupply()` is 2,523,512,977,732, of which
2,523,512,976,732 sits at the timelock and 1,000 is the MINIMUM_LIQUIDITY residual burned to `0x0`
(read identically on drpc and publicnode, 2026-10-29). **GeckoTerminal does index this pair** —
`GET api.geckoterminal.com/api/v2/networks/polygon_pos/pools/0x8bfb69ed…` answers, with
`reserve_in_usd` 2.45438675 — and its `fdv_usd` is **2,596.4967994027**, which is 11,000 wLXON × $0.23604516.
That is wrong now: wLXON `totalSupply()` is **31,152**, and the same price against it is **$7,339**, so the
aggregator is printing a supply figure from the day the pool opened (`last_sync_at` is `null`, and
`price_usd` and every volume/transaction window come back `null` on the same read). DexScreener does not
index it — `/latest/dex/pairs/polygon/0x8bFB…` returns `"pairs": null` and `/latest/dex/tokens/<wLXON>`
returns zero pairs (re-read 2026-10-29) — and CoinGecko has no LXON coin at all. A venue that reads our
price from an aggregator will read GeckoTerminal's stale FDV, so the correction has to travel with the
number. **Both halves of that indexing sentence are now superseded (2026-11-16T21:01Z): after the pair's
first genuine swap, DexScreener returns it on all three API routes (`priceUsd 0.2365`, `liquidity.usd 2.45`,
`marketCap` 7,344, `volume h24` 0) and GeckoTerminal refreshes to `reserve_in_usd` 2.5485 / `fdv_usd`
7,363.257155002 with real transaction counters.** The FDV problem the sentence describes did not go away, it
changed sign: the stale 11,000-supply number became a live 31,052-supply number, so what a reviewer reads
from an aggregator is now a *current* $7.3k market cap sitting on $2.45 of liquidity — a bigger gap, and no
longer defensible as "just needs refreshing". CoinGecko remains a true negative. [⁴]

**On chain 724 — the arithmetic one.** The LXON/WETH pair's marginal price is
**9.998003299 × 10⁻⁵ of the chain's native gas token per LXON**. [²] The USD figure the public feed prints
beside it — `$0.23202566` as the feed was read at 2026-10-29 13:35:01 UTC, `ethUsd` 2320.72 — is that ratio
multiplied by Ethereum's market price. **The "WETH" leg is chain
724's own native token, which has no market**, so that USD number is a unit substitution. It is not a
price, a market cap or a TVL, and this document does not use it as one.

All three pools are self-created. The 724 pairs were funded on both sides by the deployer; the Polygon pair
was funded by a project wallet and holds no external liquidity. So:

- The `$1.50 price floor` the earlier draft claimed never existed; no contract maintains a floor.
- The `$92,697 TVL` and `$150,000,000 market cap` the earlier draft published are unrelated to any
  measured reserve; the honest TVL today is **$2.45** plus the 724 pool quoted in its own unpriced unit.
- A market cap can be *estimated*, and each version of it misleads in a different direction:
  wLXON's 31,152 × the pool's own $0.2356 = **$7,339**; the same $0.2356 applied to LXON's
  42,700,020,000 = **$10,060,124,712**; the 724 feed's $0.23202566 against the same supply =
  **$9,907,500,415**. A ten-billion-dollar headline that comes out of $2.45 of depth and zero swaps is the
  single most important thing a reviewer needs to know before they paste a supply number into a form, and
  no aggregator is there to contradict it — except that the one that does index it is printing a stale
  FDV of its own (see above). Any of these figures has
  to be published with the sentence "over $2.45 of depth, with no recorded volume" or it is misleading on
  its own.

A pool this thin is moved by a single order, and any price a venue reads from it can be set by the party
that funded it. That is stated here rather than in the risk section alone because it is the most important
thing a reviewer should know about the price.

## Decentralisation

Stated because a reviewer will look, and because the answer is currently unfavourable:

```
blocks   9,950–  9,999   0xFaE699d2 × 50
blocks  39,950– 39,999   0xFaE699d2 × 50
blocks  79,950– 79,999   0xFaE699d2 × 50
blocks 119,950–119,999   0xFaE699d2 × 50
blocks 149,950–149,999   0xFaE699d2 × 50
blocks 171,550–171,599   0xFaE699d2 × 50
blocks 183,086–183,135   0xFaE699d2 × 50   (re-sampled 2026-10-29 13:50 UTC, tip block 183,135)
```

Every sampled block — 350 of them, spanning the chain's whole history — was produced by `0xFaE699d2…`.
That address is the deployer, the sole holder of `MINTER_ROLE` and `DEFAULT_ADMIN_ROLE`, and the funder of
the 724 pool; it now holds only 0.21% of the supply, because the largest mints went to a *second* project
EOA (`0x7F715B7F…`, 99.77%), so producer and holder are no longer the same wallet — which changes who has
to be trusted, not how many parties are trusted. Whatever the QBFT validator set is configured to
contain, production has come from one key. Claims of multi-signer validation, 2-of-2 multisig governance
or hardware-wallet custody are not supported by what is on chain or by how this instance is operated, and
they have been removed from this document.

## Use cases

Trading on the native AMM, 1:1 settlement between LXON and Polygon's wLXON through the attested bridge,
and fee payment in the native gas token are implemented and reachable. Staking contracts are deployed.
Settlement-as-payment and governance voting exist in the contract set but have no external users, and the
governance contract does not currently expose propose/vote/execute paths that work as the older guide
describes — treat governance as designed, not in use.

Lending, yield farming, derivatives, an NFT marketplace and enterprise clients are **not built**. The
previous draft listed them with dates; they remain unbuilt and undated here.

## Roadmap

Ordered by what unblocks what, not by a quarter:

1. **Close the supply question.** Cap or renounce `MINTER_ROLE`; move LP custody under the timelock.
   Designed and rehearsed; awaiting keys and an owner ruling.
2. **Fund the bridged asset's market.** A wLXON/USDC pool exists on Polygon and is indexed by
   GeckoTerminal at $2.45 of depth with no volume; the task is real liquidity and real counterparties, not
   another pool. Chain 724 is indexed by no aggregator, so depth added there cannot produce verifiable
   market data.
3. **Independent security review** of the deployed 24 contracts, including the bridge and wrapper that
   already hold escrowed value.
4. **Public source.** A sanitised export of the contract and node tree — sanitised because a developer
   private key is reachable in the current git history, which is why the repository cannot simply be
   pushed.
5. **Second and third block producers**, so that §Decentralisation reads differently.
6. Distribution, vesting and airdrop mechanics — only after contracts exist to enforce them, so this
   document never again publishes a schedule the chain does not implement.

## Team

**Not filled in.** The previous draft contained three invented biographies (CEO, CTO, CMO) and an advisor
roster. No legal entity is registered and no team roster is defined; the founder wallet `0x7F715B7F…` is
not the deployer key `0xFaE699d2…` — they are distinct addresses with distinct roles. Names, roles and
the entity behind a listing must come from the owner; see gate G7 in
`cmc-submission-form.md`. This section is deliberately empty: an application with
fabricated team entries fails on the one thing that cannot be corrected after the fact.

## Legal structure

**Not established.** No entity is registered and no jurisdiction chosen; the earlier draft's "Delaware
C-Corp, Q2 2026 (planned)" was a placeholder that was never acted on. No legal opinion has been obtained,
so this document makes no representation that LXON is a utility token rather than a security. That
determination requires counsel, and the token currently has no capped supply, no vesting, and one holder
controlling both issuance and block production — the exact facts counsel will ask about.

## Risk factors

- **Concentration.** Two project EOAs hold **99.976%** of supply between them. The deployer
  `0xFaE699d2…` holds `MINTER_ROLE`, `DEFAULT_ADMIN_ROLE`, 100% of the 724 LXON/WETH LP and `PAUSER_ROLE`,
  and produces every block. The founder `0x7F715B7F…` holds 99.77% of the tokens. Between them they can
  dilute, remove the chain-side pool's depth, or halt the chain. The 137 LP is the one custody position
  that is *not* in an EOA — it sits at the bridge timelock.
- **Unbounded issuance.** Total supply can rise without notice because there is no cap; `max_supply` is
  not a fact about this contract. It has already risen **425-fold** in a single transaction on
  2026-10-29 (block 176,301), so this is not a hypothetical risk — it is the recorded behaviour.
- **No market.** Three self-created pools. On Polygon, $2.45 of wLXON/USDC depth, **zero swaps in the
  pair's entire history**, indexed by GeckoTerminal alone — and the FDV it prints (2,596.50) is computed
  from an 11,000-token supply that ended on 2026-10-29, while DexScreener and CoinGecko return nothing; on
  724, a deployer-funded LXON/native pair whose USD
  figure is a unit substitution, and a wLXON/LXON peg pool
  whose ratio the feed itself publishes as 1.20967 rather than 1. A single order moves either print, and depth exists only while the
  parties that funded it choose to keep it.
- **Single-node infrastructure.** The explorer, the RPC, the price cron and the current block producer all
  run on one EC2 instance, which has already made the RPC and explorer endpoints unavailable
  mid-operation.
- **Archive window.** The public node retains roughly 400–510 blocks of state, so historical reads and
  bridge attestations older than ~45 minutes can fail; a locked transfer that waits too long to be
  claimed can strand escrow.
- **Unevaluated code.** No external audit. 24 contracts are deployed and source-verified; several hold
  value.
- **Bridge trust.** Attested settlement means wLXON's backing depends on a signer quorum (threshold 2) whose
  keys are, today, held on a single operator's machines.
- **The escrow contract's source is not published.** The v4.1 bridge `0x9B15fEe9…` holds the escrow but is
  absent from the explorer's 24 verified contracts, so the code that guards the bridged value cannot be read
  where everything else can be. Verifying it is a prerequisite for claiming public source (B3).
- **Key hygiene.** A developer private key is reachable in the project's git history; the repository is
  consequently not public until a sanitised export exists.
- **Regulatory.** No entity, no opinion, no registration in any jurisdiction.

## Conclusion

LXON Network is a real, public, 5-second-block QBFT chain with a contract set whose source-verification
status cannot currently be shown or performed — the explorer that carried it serves only an offline notice
— a 1:1
bridge to Polygon that is configured in both directions and paused on the Polygon side today, a quotable
DEX market on Polygon, and a public read-only RPC — and it is early in every way
those artifacts do not measure: one producer, one dominant holder, one machine, no audit, no external
market, an explorer that serves only an offline notice (its price feed was rebuilt on 2026-11-13 and is
live; Blockscout itself is not), and a Polygon leg whose bridge is paused. Both halves of that sentence are true at the time of writing, and this document's
purpose is to make them equally visible.

## Contact

- Website — `https://www.lxon.in/` (live since 2026-10-29: a project page that reads the chain in your
  browser and prints its own limits; re-read 2026-11-13, 200 / 93,127 B). · app view —
  `https://explorer.lxon.in/ui/` — **was** four pages reading the chain directly; on 2026-11-13 that host
  serves the offline notice on every path, so the app view is not reachable and `https://www.lxon.in/swap`
  is the honest "LXON swap — offline" page instead (200 / 6,501 B)
- Explorer — `https://explorer.lxon.in` · API — `https://explorer.lxon.in/api/v2` — **both currently serve
  the same 4,110-byte offline HTML notice on every path** (measured 2026-11-13), so neither is a citable
  source of chain data today; use the RPC below
- RPC — `https://rpc.lxon.in` (read-only **and enforced that way**: `eth_sendRawTransaction` returns
  `-32601 method eth_sendRawTransaction is not available on the public read-only endpoint`, measured
  2026-11-13 with an unmineable payload; `web3_clientVersion` = `besu/v26.10.0/linux-aarch_64/openjdk-java-25`.
  Writing to the chain needs the owner's signer path, not this endpoint)
- Price feed — `https://www.lxon.in/swap/price.json`, and `https://explorer.lxon.in/swap/price.json` serves
  the identical bytes with `access-control-allow-origin: *` (live since 2026-11-13T21:15Z, every five minutes,
  generated from on-chain reads; `docs/restore-price-feed-2026-11-13.md`). `https://www.lxon.in/swap` is still
  the honest offline notice for the *trading form*, which is a different thing from the feed
- Logo — `https://www.lxon.in/assets/lxon-logo-200.png` and
  `https://explorer.lxon.in/ui/assets/lxon-logo-200.png` (identical bytes, sha256 `9e188cc72118ec64…`,
  200×200 RGBA, 6.79% of pixels transparent — the rounded corners; `…/assets/lxon-logo.png` serves the same
  file for the paths older configs name)
- GitHub — not public yet; see roadmap item 4
- Email, social channels, legal entity — pending owner input (G7)

---

## Appendix — deployed contract set, chain 724

| Role | Address |
|---|---|
| LXON token | `0x99801A78639975D448EEb2f8a807738dc8df67Dc` |
| WETH (native wrapper) | `0xcB742c31f0815648E13fF24D8131B89075FF32e2` |
| LXON/WETH pair | `0x272A3eD4B91112005aDE5b4161aF1AdC856ab3f4` |
| Router | `0xd969211d7efB154e86D41A80ff0fd3cC0eB70468` |
| wLXON wrapper (724) | `0xC0CF6B0E6C5a0705F319BA5c629a6Bc7E82FEf00` |
| wLXON/LXON peg pair | `0x8e90675e054826f61745C0016b895Cdc4F220eBc` |
| Peg router | `0xaA0e4BB075F108B34DC0E279A7747980C76B8D75` |
| Attested bridge v4.1 (724) | `0x9B15fEe95995a15Da2d64D8E2958f9792D5D6a25` — **not source-verified** |
| Legacy bridge `LXONBridgeEnhanced` (724) | `0x28C93C2af6B1d5349ba356A8735b343B2CAa4379` — verified, holds `MINTER_ROLE`, no inbound route configured |
| wLXON (Polygon 137) | `0x7A3c0C34f157EfDD3E4913c5224C15E645469093` |

Legacy addresses that appear in older documents and are **not** the current asset:
`0x0a26E5CC91C167B38633b28013AD8d5EF1F6904E`, `0x007696cdF0Bf860823126D72C8e3D52448B5285E`,
`0x348E520f45dDd21EC927d7b14aB76cB2cd93DE08`, `0xA2AF542999c4412E387ba7e3867cE541cdfeDb80`,
`0x8FD2D76e642bf9d3CD9b29Ed06C4649E99d049f7`, `0x867A921d80F880d1662efDA3533EdF9165f61a28`,
`0x9EE0Ca5C1edFc13a9540E89d1F055D0FC87def02` — all measured `is_contract: false` on chain 724, with
`0x99801A78…` returning `true` as the control row. `0x76EF25145ec71d72C6Bf8c1f0850466141eD6706` is a
deployed contract (legacy wLXON) but is not the LXON token.

### Notes — how each figure was obtained

[¹] `curl https://explorer.lxon.in/api/v2/tokens/0x99801A78639975D448EEb2f8a807738dc8df67Dc` →
`name`, `symbol`, `decimals: "18"`, `total_supply: "42700020000000000000000000000"`, `holders: "12"`,
`exchange_rate: null` (read 2026-10-29 13:34 UTC). The explorer's `total_supply` agreeing with a direct
`eth_call` of `totalSupply()` at 724 block 182,718 is the cross-check that makes this a measurement rather
than a transcription. Holder detail: `/tokens/{addr}/holders` returned 12 rows with no next page — the
twelfth renders as `0.00` but holds 1 wei, which is why the list sums to supply rather than falling one
wei short — and **their balances equal `totalSupply()` exactly at the wei level**; transfers count is 35. Earlier reads
of the same endpoint, quoted in older drafts of this document, said `holders: "11"` and 26 transfers
against a 100,020,000 supply — both were correct at their block and both are superseded.
[²] `curl https://explorer.lxon.in/swap/price.json`, stamped `updatedAt: 2026-10-29T13:35:01Z`,
`ethUsd: 2320.72`; the cron that writes it calls `getReserves()` on the pair. **This URL was dead from 2026-11-03 to
2026-11-13 (the pair it read, `0x272a3ed4…`, is 0 bytes of code) and the feed was rebuilt the same day on a
new source — the Polygon pair, whose reserves are the only funded ones — at
`https://www.lxon.in/swap/price.json`, mirrored here with CORS. See
`docs/restore-price-feed-2026-11-13.md`.** Everything
below this line is the 2026-10-29 account of how the figure was cross-checked *then*; the surviving,
checkable ratio is the Polygon one in [⁴]. Both reserve sides were
independently confirmed by direct `eth_call` against `rpc.lxon.in` at 724 block 182,718:
`LXON.balanceOf(pair)` = 10,001,000 and `WETH.balanceOf(pair)` = 999.900309939, matching the feed exactly,
with the pair's native balance at 0 (the WETH leg is a wrapper, not the coin itself). An earlier version of
this note said the direct read had not succeeded and the reserves were the feed's alone; that gap is
closed. The feed's own disclaimer still applies — the pool was seeded by the deployer, so the ratio is
self-referential.
[⁴] Polygon pair re-read 2026-10-29 on both `polygon.drpc.org` and `polygon-bor-rpc.publicnode.com`.
`getReserves()` (`0x0902f1ac`) **answers now** — `1,224,879` / `5,198,977,000,000,000,000` /
`blockTimestampLast 1,793,229,608` — which also retires the workaround recorded in the previous version of
this note, where the call appeared to revert and reserves were unpacked from
`eth_getStorageAt(pair, 0x8)`; that slot still decodes to the same pair of numbers, so the two methods
agree. `balanceOf` confirms the same reserves against the token contracts. LP `totalSupply()` =
2,523,512,977,732 with 2,523,512,976,732 at `0xe56553Ed…`. Zero `Swap` events exist for this pair across
all 35,003 blocks since `addLiquidity` (`0x6ce2ef2c…`, block 96,349,486), with no log range failing, so the
zero is complete rather than unproven. Aggregator state at the same time:
`api.geckoterminal.com/api/v2/networks/polygon_pos/pools/0x8bfb69ed…` → `reserve_in_usd` 2.45438675,
`fdv_usd` **2,596.4967994027** (= 11,000 × 0.23604516, i.e. computed against the supply of 2026-10-28 and
not refreshed — wLXON `totalSupply()` is 31,152 now, and the same price against it is $7,339), with
`price_usd`, `market_cap_usd`, `last_sync_at` and every volume/transaction window `null`;
DexScreener `/latest/dex/pairs/polygon/0x8bFB…` → `"pairs": null` and `/latest/dex/tokens/<wLXON>` →
`pairs: 0`; CoinGecko `/search?query=lxon` → no such coin.

> **This footnote is a dated reading of 2026-10-29 and two of its three clauses no longer hold.**
> Re-measured 2026-11-16T21:01Z (`lxon-wlxon-listing/index-gate-check.js`, four GeckoTerminal probes with a
> busy WPOL/USDC.e control through the same classifier): GeckoTerminal answers `reserve_in_usd` **2.5485**,
> `base_token_price_usd` **0.237126663526287**, `fdv_usd` **7,363.257155002**, `volume_usd.h24`
> **0.002382618672**, `transactions.h24 {buys 1, sells 0}`, `pool_created_at` 2026-10-28T23:20:08Z, and
> `market_cap_usd` is *still* `null`; DexScreener now returns the pair on all three of its API routes
> (`priceUsd 0.2365`, `liquidity.usd 2.45`, `marketCap`/`fdv` 7,344, `volume h24` 0). CoinGecko is the only
> clause of the original that survived. What changed on 2026-11-16 was the pair's transaction history: it
> executed its first swap, [`0x339809da…b97f`](https://polygonscan.com/tx/0x339809da073830797c372aae48ef210f0e4165db093cc0992d2275dccb0cb97f),
> 0.03 POL → 0.01004787330440932 wLXON, bought and held by the project's own provider key. The caution this
> footnote was written to carry — that an aggregator's USD figure is not the chain's — holds harder now: its
> `reserve_in_usd` is **3.67 %** above and its price **0.26 %** above what `getReserves()` said at the same
> minute, and its `fdv_usd` is a supply multiplication, not demand.

[³] Polygon wLXON at `0x7A3c0C34f157EfDD3E4913c5224C15E645469093`: `name` `Wrapped LXON`, `symbol`
`wLXON`, `decimals` 18, `totalSupply()` **31,152.0000** (read 2026-10-29 via `https://polygon.drpc.org`).
Backing re-read on 724 at block 182,718, 13:23 UTC, from the v4.1 bridge itself — `escrowed(LXON)` =
31,152, `totalSentFor(137, LXON)` = 31,152, `laneNonce(137, LXON)` = 3, `configSealed()` = true, and 137's
`totalReceivedFrom(724, LXON)` = 31,152; `LXON.balanceOf(bridge)` = 61,152, the extra 30,000 being
attestation stake from blocks 121,089–121,091. Earlier reads of the same endpoints said 11,000 /
`laneNonce` 2 / balance 41,000 at block 172,197 — correct then, superseded by the third crossing.
