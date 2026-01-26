# Prophet - Payment Verification Tradeoffs

## The Core Problem

We're building a **serverless** prediction market where:
- Bettors pay to place bets
- We need to **verify payments were received** before recording bets
- Creator pays out winners after resolution

Without a server, we can't run a Lightning node or query payment databases directly.

---

## Current Architecture

```
Creator                          Bettor                         Our Static App
   │                                │                                │
   │──NWC connection string────────>│                                │
   │                                │                                │
   │                                │<──────────Invoice──────────────│
   │<──────make_invoice (NWC)───────│                                │
   │                                │                                │
   │                                │────Pay invoice with wallet────>│
   │                                │                                │
   │                                │<───────Preimage (proof)────────│
   │                                │                                │
   │                                │───Submit preimage to verify───>│
   │                                │                                │
   │                                │         SHA256(preimage) == payment_hash?
   │                                │                    ✓
   │                                │<──────Bet recorded─────────────│
```

---

## Tradeoff #1: Creator Must Use NWC Wallet

### What we require
Creator must provide an NWC (Nostr Wallet Connect) connection string to create a market.

### Why
- Need `make_invoice` to generate unique invoices per bet
- Each invoice has a unique `payment_hash` we use for verification
- Without NWC, we'd need a server to generate invoices

### Wallets that support NWC
- Alby Hub ✅ (best support)
- Primal ✅ (limited - no lookup_invoice)
- Zeus v0.12+ ✅
- Coinos ✅
- LNbits ✅

### Wallets that DON'T support NWC
- Phoenix ❌
- Wallet of Satoshi ❌
- Cash App ❌
- MetaMask ❌ (even with new Lightning support)
- Most custodial wallets ❌

### Impact
~10-20% of Lightning users have NWC-compatible wallets. This limits who can CREATE markets (not who can bet).

### Alternatives considered
1. **Lightning Address only** - Can't generate unique invoices per bet, can't verify specific payments
2. **LNURL-pay** - Requires server to host callback endpoint
3. **Static invoice** - Can't distinguish between bettors, amounts, or verify who paid
4. **Require specific wallet (e.g., Lexe)** - Even more restrictive

---

## Tradeoff #2: Payment Verification via Preimage

### What we require
Bettor must provide the payment preimage after paying to prove payment.

### Why
- NWC's `lookup_invoice` is unreliable (Primal doesn't support it, others timeout)
- Preimage is cryptographic proof: `SHA256(preimage) == payment_hash`
- Can be verified client-side without any server

### How bettors get preimage

| Method | UX | Wallet Support |
|--------|-----|----------------|
| **WebLN** (automatic) | Best - click to pay, preimage returned | Alby extension, Alby Hub |
| **Bitcoin Connect** | Good - modal UI, preimage returned | NWC wallets, Alby |
| **Manual paste** | Poor - user finds preimage in wallet | Any wallet that shows it |

### Wallets that expose preimage to users
- Phoenix ✅ (in payment details)
- Zeus ✅
- Breez ✅
- Wallet of Satoshi ✅
- Alby ✅ (automatic via WebLN)
- BlueWallet ✅

### Wallets that probably DON'T expose preimage
- Cash App ❓ (unclear, likely hidden)
- MetaMask ❓ (new Lightning, unknown)
- Venmo ❌ (no Lightning)
- Most custodial wallets hide technical details

### Impact
Bettors with WebLN/Bitcoin Connect get seamless UX. Others need to find and paste preimage (friction). Some wallets won't work at all.

### Alternatives considered
1. **NWC lookup_invoice polling** - Unreliable, many wallets don't support
2. **Trust-based (no verification)** - Vulnerable to fake bets
3. **Custodial service** - Adds server dependency, custody risk
4. **HODL invoices** - Requires server
5. **Keysend with TLV metadata** - Data goes TO creator's node, still need server to read it

---

## Tradeoff #3: BOLT11 Only (No BOLT12)

### Current state
We use BOLT11 invoices via NWC's `make_invoice`.

### Why not BOLT12?
- NWC spec doesn't support BOLT12 offers yet
- `pay_invoice` method only accepts BOLT11
- BOLT12 wallet support is still limited

### What we lose
- BOLT12 offers don't expire (better for long-running markets)
- BOLT12 has built-in payer metadata
- Better privacy features

### Impact
Invoices expire (typically 1 hour). Bettors must pay promptly after generating invoice.

---

## Tradeoff #4: Creator Pays Winners Manually

### Current flow
1. Creator resolves market (signs resolution event)
2. App shows list of winners + amounts
3. Creator clicks "Pay All Winners"
4. NWC `pay_invoice` sends to each Lightning Address

### Why this way
- No server to automate payouts
- Creator's wallet has the funds
- NWC can initiate payments

### Limitations
- Creator must be online to pay out
- If creator disappears, winners don't get paid
- No escrow or trustless payout mechanism

### Alternatives considered
1. **Smart contract escrow** - Not possible on Lightning (no scripting)
2. **Federated escrow (Fedimint)** - Adds complexity, requires federation
3. **HODL invoices** - Requires server to release
4. **Discreet Log Contracts (DLCs)** - Complex, poor UX, limited wallet support

---

## Tradeoff #5: Trust in Creator

### What bettors trust
- Creator will resolve honestly
- Creator will actually pay winners
- Creator won't disappear with funds

### Why we can't avoid this
- No on-chain escrow on Lightning
- No trustless oracle system integrated
- Serverless = no custodial middleman

### Mitigations we have
- All bets recorded on Nostr (public audit trail)
- Preimage proves each payment was made
- Creator's pubkey tied to market (reputation)

### What would fix this
- DLCs (complex)
- Fedimint integration (requires federation)
- Server-based escrow (defeats serverless goal)
- On-chain Bitcoin with timelocks (slow, expensive)

---

## Summary: Who Can Use This?

### Creators (more restricted)
Must have NWC-compatible wallet:
- ✅ Alby Hub
- ✅ Primal
- ✅ Zeus v0.12+
- ✅ Coinos
- ✅ LNbits
- ❌ Phoenix, WoS, Cash App, MetaMask, etc.

### Bettors (less restricted)

| Wallet | Works? | UX |
|--------|--------|-----|
| Alby (extension) | ✅ | Best (WebLN auto) |
| Alby Hub (NWC) | ✅ | Great (Bitcoin Connect) |
| Zeus (NWC) | ✅ | Great (Bitcoin Connect) |
| Phoenix | ✅ | Manual preimage paste |
| Breez | ✅ | Manual preimage paste |
| Wallet of Satoshi | ✅ | Manual preimage paste |
| BlueWallet | ✅ | Manual preimage paste |
| Cash App | ❓ | Unknown if preimage visible |
| MetaMask | ❓ | Unknown (new Lightning) |
| Phantom | ❌ | No Lightning |
| Venmo | ❌ | No Lightning/Bitcoin |

---

## Questions for Lightning Expert

1. **Is there a better way to verify payments without a server?**
   - We're using preimage verification. Is there a simpler method?

2. **Can HTLC TLV data help us?**
   - Expert mentioned 500 bytes in onion TLV. But data goes to receiver's node - how do we read it without server?

3. **Will NWC add BOLT12 support?**
   - Would help with non-expiring offers

4. **Is there a trustless payout mechanism we're missing?**
   - Current: trust creator to pay. Any Lightning-native escrow?

5. **MetaMask/Cash App Lightning implementation**
   - Do they expose preimage? Support any programmable interface?

6. **Bitcoin Connect vs alternatives**
   - Is this the best approach for broad wallet compatibility?

---

## What We're Building (Current Plan)

1. **Creators**: Require NWC wallet (Alby Hub recommended)
2. **Bettors**:
   - Primary: Bitcoin Connect (works with NWC wallets, returns preimage automatically)
   - Secondary: WebLN (Alby extension)
   - Fallback: Manual preimage paste
3. **Verification**: Preimage + SHA256 check (cryptographic, client-side)
4. **Trust model**: Bettors trust creator to resolve honestly and pay out
5. **Data storage**: Nostr relays (decentralized, public)

---

## Recent Improvements (v79)

| Problem | Solution |
|---------|----------|
| **Payout interruption** | Progress saved after each payment. Creator can resume if browser closes. |
| **Secret key friction** | NIP-07 support - login with Alby/nos2x extension, no nsec paste needed. |
| **Relay reliability** | Added fallback relays (nos.lol, relay.nostr.band) beyond Damus and Primal. |

See DESIGN.md "Future Roadmap" section for planned improvements.
