# Prophet - Decentralized Prediction Markets

🔮 A decentralized prediction market platform using Nostr for data and Lightning for payments.

---

## Executive Summary

### What We're Building

A static website (no backend) where anyone can create a binary prediction market, share it via URL, collect bets in Bitcoin via Lightning, and pay out winners - all without a central server, user accounts, or regulatory entity. Data lives on Nostr relays; payments flow wallet-to-wallet.

### Key Tradeoffs

| We chose... | Instead of... | Because... |
|-------------|---------------|------------|
| **Parimutuel model** | AMM/liquidity pool | Zero creator risk; simpler math |
| **Creator holds funds** | Smart contract escrow | No backend needed; acceptable for MVP trust model |
| **NWC for creators** | Any wallet | Enables invoice generation + automated payouts |
| **Any wallet for bettors** | NWC required | Lower friction; they just pay invoices |
| **Paste nsec** | External signers | Simpler MVP; better signers can come later |
| **Kind 1 events** | Custom NIP | Use existing infrastructure; formalize later if popular |
| **5% fee hardcoded** | Configurable fee | One less decision; simplify MVP |
| **2 relays** | 3+ relays | Simpler; Damus + Primal are reliable enough |

### MVP Limitations We're Accepting

| Limitation | Risk | Mitigation |
|------------|------|------------|
| Creator is trusted (holds funds, resolves) | Creator could steal or lie | Small amounts, known friends, reputation at stake |
| No dispute resolution | Wrong resolution has no recourse | Accept for MVP; add later |
| Creator wallet must be online | Bettors can't bet if wallet offline | Use always-on wallets (Alby Hub, Primal) |
| Unresolved markets lose funds | Creator disappears | Accept for MVP; timeout mechanism later |
| Nsec paste is insecure | Key could be compromised | Client-side only; small amounts; add signers later |
| No market discovery | Hard to find markets | URL sharing via social media; discovery later |

### What MVP Successfully Demonstrates

1. **Decentralized prediction markets work** - No server, no company, no custody
2. **Nostr as data layer** - Markets, bets, resolutions all stored on public relays
3. **Lightning as payment layer** - Real Bitcoin, instant settlement, global access
4. **Creator-as-operator model** - Anyone can run a market for their audience
5. **Parimutuel mechanics** - Fair, transparent, zero creator risk
6. **Shareable via URL** - Tweet a link, anyone can bet

### What's NOT in MVP

- Multi-outcome markets (only Yes/No)
- Automated dispute resolution
- DLC-based trustless escrow
- Market discovery/listing
- Mobile app
- Reputation system
- External signer support

---

## Vision

Create a prediction market platform that combines:
- **Polymarket/Kalshi's** real-money stakes
- **Manifold Markets'** user-generated, niche markets
- **No central server** - purely client-side JavaScript
- **Bitcoin/Lightning** for payments (avoiding USD regulatory constraints)
- **Nostr** for data persistence (invisible to users)

The result: Anyone can create a prediction market, share it via URL, and settle it with real Bitcoin - without requiring a company to operate servers, hold funds, or navigate regulations.

---

## Core Principles

### Confirmed Decisions

1. **Static site only** - No backend server. Pure client-side JavaScript. Could be hosted on GitHub Pages, IPFS, or anywhere static files can be served.

2. **Nostr as invisible infrastructure** - Users never need to know they're using Nostr. Identities (nsec/npub) are generated on-the-fly. The term "Nostr" only appears in FAQ/technical docs.

3. **Lightning only** - No on-chain Bitcoin. Simplifies UX and reduces confirmation times.

4. **Creator needs NWC wallet; bettors can use any wallet** - Creators must use a Nostr Wallet Connect (NWC) compatible wallet. This enables unique invoice generation and automated payouts. Bettors just need any Lightning wallet that can pay an invoice.

5. **Binary markets only (MVP)** - Yes/No outcomes. Multi-outcome markets can come later.

6. **Parimutuel model** - Bettors fund the pool, losers pay winners, creator takes a fee. Creator stakes nothing and has zero risk.

7. **Creator as oracle** - The market creator resolves the market. Their reputation is at stake.

8. **Small amounts initially** - MVP targets small bets (e.g., 100-1000 sats per bet). Proves concept with minimal risk.

9. **Trust model for MVP** - Accept that the creator holds funds and resolves markets. Cryptographic escrow (DLCs) can come later.

10. **URL-based sharing** - Creator gets a shareable URL to post on Twitter/social media.

---

## How It Works (Conceptual Flow)

### Creating a Market

1. User visits static site
2. **Connects NWC wallet** (pastes NWC connection string or scans QR)
3. Fills out market form:
   - Question (e.g., "Will X happen by Y date?")
   - Resolution date
   - Resolution criteria (what Yes and No mean)
   - Max bet size (optional)
4. Site generates a Nostr keypair (nsec/npub) for the market
5. User is shown the nsec and told to save it securely (needed to resolve later)
6. Market is published as a Nostr event to relays
7. User gets shareable URL (e.g., `https://site.com/m/{event-id}`)

**Creator stakes nothing.** Fee is 5%. Their wallet receives bets and holds the pool.

### Placing a Bet

1. Someone clicks the shared URL
2. Sees market details: question, odds, current positions, resolution date
3. Chooses outcome (Yes/No) and amount
4. Provides their Lightning Address (for receiving winnings)
5. **Site generates unique invoice via NWC** (calls `make_invoice` on creator's wallet)
6. Bettor pays invoice (scans QR or copy/paste) using any Lightning wallet
7. **Site verifies payment via NWC** (calls `lookup_invoice`)
8. Bet is recorded as Nostr event

### Resolution & Payout

1. After resolution date, creator returns to site
2. **Reconnects NWC wallet** (or still connected)
3. Authenticates with their nsec (proves they're the market creator)
4. Selects winning outcome
5. Resolution is published as Nostr event
6. Site calculates payouts owed to each winner
7. **Creator triggers payouts via NWC** (site calls `pay_invoice` for each winner)

---

## Architecture

### Data Layer: Nostr

All market data stored as Nostr events:
- **Market definition** - Question, outcomes, dates, creator pubkey, Lightning Address
- **Positions/bets** - Who bet, which outcome, how much, their payout address
- **Resolution** - Which outcome won, timestamp

Using existing Nostr event kinds with structured JSON content (no custom NIPs needed for MVP).

Data is replicated across multiple Nostr relays for redundancy.

### Payment Layer: Lightning

Payments flow wallet-to-wallet, similar to Nostr zaps:
- Site never touches funds
- Creator's wallet receives bets
- Creator's wallet pays out winners

The site is purely a **coordinator/UI** - it displays information and records intents on Nostr, but all value transfer happens directly between Lightning wallets.

### Client: Static JavaScript

- Vanilla JS or lightweight framework
- Connects to Nostr relays via WebSocket
- Connects to creator's wallet via NWC for invoice generation
- Generates QR codes for payment
- Reconstructs market state from Nostr events
- Could be a single HTML file with embedded JS

### NWC-Compatible Wallets (for Creators)

Creators must use one of these wallets:

**Self-Custodial:**
| Wallet | Platform | Notes |
|--------|----------|-------|
| Alby Hub | Web, Mobile | Self-custodial Lightning node. Flagship NWC wallet. |
| Zeus | Mobile | Multi-wallet app, can run embedded node |
| Electrum | Desktop, Mobile | Established wallet, added NWC support |
| Flash Wallet | Mobile | Built on Breez SDK |
| Minibits | Mobile | Ecash wallet, performance-focused |
| Cashu.me | Web (PWA) | Ecash-based, browser wallet |

**Custodial:**
| Wallet | Platform | Notes |
|--------|----------|-------|
| Primal Wallet | Mobile, Web | Built into Primal Nostr client. Popular. |
| Coinos | Web | Free custodial web wallet |
| LNbits | Web, Self-hosted | Extensible Lightning tools, NWC plugin |

**Note:** Creator's wallet must be online/reachable when bets are placed (for invoice generation) and when payouts are triggered.

---

## Market Mechanics

### Parimutuel Betting

All bets go into a pool. Creator takes a fee. Winners split the rest.

**How it works:**
```
Pool composition:
  - 6,000 sats bet on Yes
  - 4,000 sats bet on No
  - Total pool: 10,000 sats

Creator fee: 5% of total = 500 sats
Pool for winners: 9,500 sats

If Yes wins:
  - Yes bettors split 9,500 sats
  - Payout per Yes sat: 9,500 / 6,000 = 1.58x
  - Alice bet 1,000 on Yes → gets 1,583 sats
  - Creator gets 500 sats

If No wins:
  - No bettors split 9,500 sats
  - Payout per No sat: 9,500 / 4,000 = 2.375x
  - Bob bet 1,000 on No → gets 2,375 sats
  - Creator gets 500 sats

Note: Fee is capped so winners always get at least 1.0x back.
See "Creator Fee" section for edge cases.
```

### Implied Odds

Unlike AMMs, there's no real-time "price." Instead, odds are implied by the bet distribution:

```
6,000 on Yes, 4,000 on No

Implied probability:
  - Yes: 60%
  - No: 40%

Current potential payout (before more bets):
  - Yes wins: ~1.58x (after fee)
  - No wins: ~2.375x (after fee)
```

**Odds update as bets come in.** Final odds aren't known until betting closes.

### Creator Risk: None

- Creator stakes nothing
- Creator's wallet holds the pool (bettors' funds)
- Creator takes X% fee from pool
- Creator pays winners from pool
- Creator always profits (fee), never loses

### Creator Fee

**Hardcoded at 5% for MVP.**

#### Guiding Principles

1. **Creator earns 5% on total action** - The creator's job is to drive volume and liquidity on both sides. They should be rewarded for total market activity, not just for people losing.

2. **Winners never lose money** - No matter the market composition, winners must receive at least 1.0x their bet back. Winning should always feel good.

3. **Fee is capped by losing pool** - To satisfy both principles, the fee is calculated as 5% of total pool but capped at the losers' pool size. This ensures winners always get at least their stake back.

#### Fee Calculation

```
fee_target = total_pool × 5%
actual_fee = min(fee_target, losing_pool)
winners_receive = total_pool - actual_fee
```

#### Examples

| YES Pool | NO Pool | Winner | Fee Target | Actual Fee | Winner Payout |
|----------|---------|--------|------------|------------|---------------|
| 1,000 | 1,000 | YES | 100 | 100 | 1.90x |
| 1,000 | 500 | YES | 75 | 75 | 1.43x |
| 1,000 | 100 | YES | 55 | 55 | 1.05x |
| 1,000 | 50 | YES | 52 | 50 (capped) | 1.00x |
| 1,000 | 0 | YES | 50 | 0 (capped) | 1.00x |

#### Incentive Alignment

- **Balanced markets = full 5% fee** - Creator is motivated to attract bets on both sides
- **One-sided markets = reduced fee** - Creator still gets something, but less
- **Completely one-sided = 0 fee** - This is a failed market anyway (no real prediction)
- **Winners always happy** - Even in edge cases, winners never lose money

This model aligns creator incentives with market health: the best markets have action on both sides, and those are exactly the markets where creators earn the most.

Configurable fees can come later.

### Position Limits

Optional: Creator can set max bet size when creating market. No minimum.

---

## Payment Verification

### The Solution: NWC + Unique Invoices

By requiring creators to use NWC-compatible wallets, we solve payment verification:

1. **Bettor wants to bet 500 sats on Yes**
2. **Site calls `make_invoice`** via NWC to creator's wallet
3. **Creator's wallet generates unique invoice** for exactly 500 sats, with bet metadata
4. **Bettor pays invoice** using any Lightning wallet (QR code or copy/paste)
5. **Site calls `lookup_invoice`** via NWC to verify payment
6. **Result:** Cryptographic proof of exact amount for this specific bet

### What This Provides

| Verification Need | How NWC Solves It |
|-------------------|-------------------|
| Payment happened | `lookup_invoice` confirms paid status |
| Correct amount | Amount is baked into the invoice |
| Linked to this bet | Invoice includes bet ID in memo/metadata |
| No trust required | Invoice + payment = cryptographic proof |

### Payout Flow

1. Market resolves, site calculates winnings
2. Creator clicks "Pay all winners"
3. Site calls `pay_invoice` via NWC for each winner's Lightning Address
4. Automated batch payouts

### Trade-off

- **Requires:** Creator uses NWC wallet, wallet online during betting
- **Enables:** Trustless verification, automated payouts
- **Bettors:** Still use any Lightning wallet (just paying invoices)

---

## Relay Strategy

### Relays

Use two well-established, high-availability relays:

| Relay | Operator | Why |
|-------|----------|-----|
| `wss://relay.damus.io` | Damus | Largest iOS Nostr client, well-funded, professional ops |
| `wss://relay.primal.net` | Primal | Major client with web + mobile, backed by serious team |

These are "too big to fail" - they have strong incentives to maintain uptime and retain data.

### Write Strategy

When publishing any event (market, bet, resolution):
1. Publish to BOTH relays
2. Consider success if at least 1 confirms
3. Retry failed relay in background

### Read Strategy

When loading a market:
1. Query both relays in parallel
2. Merge results (one relay may have more bets than other)
3. Deduplicate by event ID

### Client-Side Backup

Store in browser localStorage:
- Market definition (when created)
- User's own bets
- Resolution event

This protects against relay data loss. User can re-publish from local backup if needed.

### Future Enhancements

- Let creator specify additional relays
- Dedicated prediction market relay with guaranteed retention
- Paid relay option (nostr.wine) for better reliability

---

## Bettor UX

### No Account Required

Bettors don't need:
- Nostr identity
- Account/signup
- Specific wallet or extension

They just need:
- Any Lightning wallet (to pay invoice)
- A Lightning Address (to receive winnings)

### Step-by-Step Flow

```
1. LAND ON MARKET
   URL: https://site.com/m/{market-id}

   Shows:
   - Question: "Will X happen by Y date?"
   - Resolution date: Jan 30, 2025
   - Current pool: 5,000 sats
   - Implied odds:
     - Yes: 60% (3,000 sats) → pays 1.58x
     - No: 40% (2,000 sats) → pays 2.38x
   - Creator fee: 5%

2. CHOOSE SIDE
   - Click "Bet Yes" or "Bet No"
   - Enter amount: [500] sats
   - Enter Lightning Address: [you@walletofsatoshi.com]
   - Click "Get Invoice"

3. PAY INVOICE
   - Site generates invoice via NWC
   - Shows QR code
   - Shows copyable invoice string (for paste into wallet)
   - Bettor pays using any Lightning wallet

4. CONFIRMATION
   - Site detects payment (NWC lookup)
   - Shows: "Bet confirmed! 500 sats on Yes"
   - Bet recorded on Nostr
   - Shows updated odds

5. DONE
   - Bettor can bookmark or close
   - If they win, payout sent automatically to their Lightning Address
   - No need to return (unless they want to check status)
```

### What Bettors See

| Screen | Information Shown |
|--------|-------------------|
| Market page | Question, date, pool size, current odds for each side |
| Bet form | Amount input, Lightning Address input |
| Payment | QR code, invoice string, "waiting for payment..." |
| Confirmation | Bet recorded, new odds, potential payout |

### Edge Cases

- **Bettor has no Lightning Address**: They need one. Most wallets provide these now. Could link to easy options (WoS, Alby).
- **Payment fails/times out**: Invoice expires, they can try again.
- **Duplicate bets**: Same person can bet multiple times (same or different sides).

---

## Creator Auth

### MVP: Paste nsec

When creator wants to resolve (or manage) their market:

1. Creator returns to market URL
2. Clicks "Resolve Market" (only visible after resolution date)
3. Pastes their nsec (saved when they created the market)
4. Site derives pubkey from nsec, confirms it matches market creator
5. Creator selects winning outcome
6. Site signs resolution event with nsec, publishes to Nostr
7. Creator triggers payouts via NWC

**Security note:** Pasting nsec into a website is not ideal. The nsec is only used client-side (never sent to a server), but users must trust the site code. Acceptable for MVP with small amounts.

### Future: External Signers

Better approaches for later:

| Signer | How it works |
|--------|--------------|
| **NIP-07 extensions** | Alby, nos2x - browser extension signs events |
| **Primal Signer** | New tool from Primal team |
| **Amber (Android)** | Mobile signer app |
| **nsecBunker** | Remote signing service |

These allow signing without exposing the nsec to the website.

**Migration path:** When we add signer support, creators can still use nsec paste as fallback, or connect a signer for better security.

---

## Nostr Event Structure

### Overview

Use standard Nostr kind 1 events with structured JSON in the content field. Our protocol is defined by the `type` field in the JSON.

**Event types (MVP):**
1. `market` - Creates a new prediction market
2. `bet` - Records a confirmed bet
3. `resolution` - Declares the winning outcome

Payout events skipped for MVP (can add later for audit trail).

### Event 1: Market Creation

Published by the creator when setting up a new market.

```json
{
  "kind": 1,
  "pubkey": "<creator's market pubkey>",
  "created_at": 1704067200,
  "tags": [
    ["t", "prediction-market"],
    ["t", "pm-market"]
  ],
  "content": "{\"type\":\"market\", ... }"
}
```

**Content JSON:**
```json
{
  "type": "market",
  "question": "Will Bitcoin reach $100k by March 2025?",
  "resolution_date": "2025-03-01T00:00:00Z",
  "resolution_criteria": "Yes if BTC exceeds $100k USD on CoinGecko before resolution date. No otherwise.",
  "max_bet_sats": 10000
}
```

**Fields:**
| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always "market" |
| `question` | Yes | The prediction question |
| `resolution_date` | Yes | ISO 8601 datetime when market resolves |
| `resolution_criteria` | Yes | Clear description of what Yes/No mean |
| `max_bet_sats` | No | Maximum bet size (optional) |

**Notes:**
- Fee is hardcoded at 5% (not in event)
- Market ID = Nostr event ID
- Betting closes at resolution_date

### Event 2: Bet

Published by the **creator** after verifying payment via NWC.

```json
{
  "kind": 1,
  "pubkey": "<creator's market pubkey>",
  "created_at": 1704153600,
  "tags": [
    ["t", "prediction-market"],
    ["t", "pm-bet"],
    ["e", "<market_event_id>"]
  ],
  "content": "{\"type\":\"bet\", ... }"
}
```

**Content JSON:**
```json
{
  "type": "bet",
  "market_id": "<market_event_id>",
  "outcome": "yes",
  "amount_sats": 500,
  "payout_address": "bettor@walletofsatoshi.com",
  "payment_hash": "<lightning invoice payment hash>"
}
```

**Fields:**
| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always "bet" |
| `market_id` | Yes | Event ID of the market |
| `outcome` | Yes | "yes" or "no" |
| `amount_sats` | Yes | Bet amount in satoshis |
| `payout_address` | Yes | Bettor's Lightning Address |
| `payment_hash` | Yes | Hash from paid invoice |

### Event 3: Resolution

Published by the creator after the resolution date.

```json
{
  "kind": 1,
  "pubkey": "<creator's market pubkey>",
  "created_at": 1709251200,
  "tags": [
    ["t", "prediction-market"],
    ["t", "pm-resolution"],
    ["e", "<market_event_id>"]
  ],
  "content": "{\"type\":\"resolution\", ... }"
}
```

**Content JSON:**
```json
{
  "type": "resolution",
  "market_id": "<market_event_id>",
  "winning_outcome": "yes"
}
```

**Fields:**
| Field | Required | Description |
|-------|----------|-------------|
| `type` | Yes | Always "resolution" |
| `market_id` | Yes | Event ID of the market |
| `winning_outcome` | Yes | "yes" or "no" |

### Tags Summary

| Tag | Used in | Purpose |
|-----|---------|---------|
| `["t", "prediction-market"]` | All | Discover all prediction market events |
| `["t", "pm-market"]` | Market | Find markets |
| `["t", "pm-bet"]` | Bet | Find bets |
| `["t", "pm-resolution"]` | Resolution | Find resolutions |
| `["e", "<market_id>"]` | Bet, Resolution | Reference to market |

### Querying Events

```
Find all markets:     {"kinds": [1], "#t": ["pm-market"]}
Find bets for market: {"kinds": [1], "#t": ["pm-bet"], "#e": ["<market_id>"]}
Find resolution:      {"kinds": [1], "#t": ["pm-resolution"], "#e": ["<market_id>"]}
```

### Security Notes

- Creator signs all events (bettors don't need Nostr keys)
- Payment hash adds accountability (can verify invoice was paid)
- Resolution is trusted (creator's reputation at stake)

### Future Enhancements

- Payout events for audit trail
- Version field for protocol evolution
- Dedicated NIP with custom event kinds

---

## Comparable Projects & Patterns

### For Architecture Inspiration

- **Damus, Primal, Snort** - Nostr clients with Lightning integration (zaps). Prove that wallet-agnostic Lightning can work with Nostr.
- **Bisq** - Decentralized Bitcoin exchange. Solved similar escrow/trust problems.
- **Uniswap** - AMM mechanics for pricing.

### For Prediction Market Mechanics

- **Polymarket** - Centralized order book, professional market making.
- **Kalshi** - Regulated, event contracts.
- **Manifold Markets** - User-generated markets, but play money due to regulations.
- **Augur** - Ethereum-based, solved oracle problem with staking.

### For Bitcoin-Native Prediction Markets

- **DLCs (Discreet Log Contracts)** - Trustless escrow with oracle attestation. Complex but could be a future enhancement.
- **Outcome.observer, DLC.Link** - Oracle services for DLCs.

---

## Open Questions

### Market Mechanics

1. ~~What are the exact AMM formulas?~~ **RESOLVED** - Using parimutuel model instead. See Market Mechanics section.

2. ~~How do position limits and creator exposure work?~~ **RESOLVED** - Creator has zero exposure. Optional position limits. See Market Mechanics section.

3. ~~What if a market is never resolved?~~ **RESOLVED** - Accept risk for MVP. Trusted friends context. Future: timeout + auto-refund.

### UX Flow

4. ~~What does the bettor experience look like step-by-step?~~ **RESOLVED** - See Bettor UX section.

5. ~~How does the creator claim/prove ownership to resolve?~~ **RESOLVED** - Paste nsec for MVP. See Creator Auth section.

### Technical

6. ~~What Nostr event structure for markets?~~ **RESOLVED** - See Nostr Event Structure section.

7. ~~Which Nostr relays?~~ **RESOLVED** - See Relay Strategy section

---

## Trade-offs We're Accepting

### Trust in Creator

The creator holds funds and decides outcomes. For MVP with friends, this is acceptable. Later, we could add:
- Reputation systems
- Multi-sig or DLC escrow
- Dispute mechanisms

### Creator Must Be Online

Creator's NWC wallet must be online/reachable:
- When bets are placed (to generate invoices)
- When payouts are triggered (to send payments)

For always-on wallets (Alby Hub, Primal, Coinos), this isn't an issue. For mobile wallets, creator needs app open.

### Creator Wallet Requirement

Creators must use an NWC-compatible wallet. This limits who can create markets but enables:
- Cryptographic payment verification
- Automated payouts
- Better UX overall

Bettors can still use any Lightning wallet.

### No Dispute Resolution

If creator resolves incorrectly or disappears, bettors have no recourse. Accept this for MVP.

### Regulatory Uncertainty

"No server" architecture provides some protection but isn't guaranteed to satisfy all regulators. Using Bitcoin instead of USD helps. Operating internationally helps. But this isn't legal advice.

---

## MVP Scope

### In Scope

- [ ] Static site (single page or minimal pages)
- [ ] Create binary (Yes/No) market
- [ ] Store market on Nostr
- [ ] Generate shareable URL
- [ ] Display market details
- [ ] Record bets on Nostr
- [ ] Display current positions/odds
- [ ] Resolve market (creator only)
- [ ] Display resolution and who won
- [ ] Calculate payouts owed

### Out of Scope (Future)

- Multi-outcome markets
- Automated payment verification
- Automated payouts
- DLC escrow
- Dispute resolution
- Reputation system
- Market discovery/listing
- Mobile app
- Custom Nostr relays

---

## Technical Research Notes

### MDK (MoneyDevKit)

- Lightning checkout SDK for Next.js
- Requires server components (API routes)
- **Not suitable** for static site

### Lexe

- Self-custodial Lightning wallet with always-on node
- Sidecar SDK provides REST API for programmatic payments
- Could work if creator runs local binary
- **Interesting for future** but adds complexity

### NWC (Nostr Wallet Connect) - CHOSEN APPROACH

- Protocol for connecting Lightning wallets to apps via Nostr
- Key methods: `make_invoice`, `lookup_invoice`, `pay_invoice`, `get_balance`
- Requires NWC-compatible wallet (see list above)
- **Solves:** payment verification, automated payouts
- **Trade-off:** Creators must use NWC wallet; bettors unaffected

### LNURL-pay Comments

- LNURL-pay supports `commentAllowed` parameter
- Payer can include text (up to ~255 chars) with payment
- **Potential for unique codes** but not all wallets support/display this

### NIP-57 Zap Receipts

- Nostr events (kind 9735) recording Lightning payments
- Include amount, recipient, optional preimage
- Spec explicitly states: "not a proof of payment" - trust-based
- **Could leverage for bet recording** but requires zap-enabled address

---

## Appendix: Payment Flow Comparison

How similar apps handle the "receive payment" problem:

| App | How they receive payments | Applicable to us? |
|-----|--------------------------|-------------------|
| **Damus/Primal** | User has Lightning Address in profile; payer pays directly | Yes - creator provides address |
| **Stacker News** | Custodial wallet for each user | No - requires server |
| **Fountain** | Users connect external wallet | Maybe - but adds friction |
| **Geyser Fund** | Project owner provides Lightning Address | Yes - same as our model |

---

## Document History

- **2025-01-11**: Initial design document created from planning session
- **2025-01-11**: Added NWC requirement for creators; solved payment verification
- **2025-01-11**: Added relay strategy (Damus, Primal, nos.lol)
- **2025-01-11**: Switched from AMM to parimutuel model; creator stakes nothing, zero risk
- **2025-01-11**: Added bettor UX flow; Lightning Address required upfront for payouts
- **2025-01-11**: Creator auth via nsec paste for MVP; external signers later
- **2025-01-11**: Defined Nostr event structure (market, bet, resolution, payout)
- **2025-01-11**: Added executive summary with tradeoffs and MVP scope
- **2025-01-11**: Simplified: removed payout events, hardcoded 5% fee, 2 relays, fewer fields

---

## Next Steps

All open questions resolved. Ready to build.

1. Set up static site project (HTML + JS)
2. Implement Nostr connection (relay websockets)
3. Implement NWC connection (for creator)
4. Build market creation flow
5. Build betting flow
6. Build resolution + payout flow
7. Test with small amounts among friends
