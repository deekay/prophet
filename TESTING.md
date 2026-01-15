# Prophet Testing Guide

## Automated Tests

Open `test.html` in a browser to run the automated test suite.

### What's Covered
- Pure function unit tests (parsing, encoding, calculations)
- Event structure validation
- Cryptographic verification (preimage → hash)
- Payout math edge cases
- Mock wallet flow simulation

### What's NOT Covered (requires real infrastructure)
- Real Nostr relay connections
- Real NWC wallet interactions
- Real Lightning payments
- Cross-browser state isolation

---

## Manual Test Scenarios

### Scenario 1: Full Happy Path (Creator + Bettor)

**Setup:**
- Browser A: Chrome (Creator)
- Browser B: Firefox or Safari (Bettor)
- NWC wallet: Primal, Alby Hub, or similar

**Steps:**

1. **[Browser A] Create Market**
   - Go to app, click "Create a Market"
   - Paste NWC connection string from your wallet
   - Fill in question, resolution date, criteria
   - Click "Create Market"
   - ✓ Verify: Success message, nsec shown, share URL generated
   - **SAVE THE NSEC** (copy to notes)
   - Copy the share URL

2. **[Browser B] Place Bet**
   - Open the share URL (should NOT have any localStorage from Browser A)
   - ✓ Verify: Market loads, shows question, resolution date
   - Enter Lightning Address
   - Enter bet amount
   - Click "Bet YES" or "Bet NO"
   - ✓ Verify: QR code appears
   - Pay the invoice with any Lightning wallet
   - ✓ Verify: "Payment received! Recording bet..." then "Bet confirmed!"
   - ✓ Verify: Bet appears in "Recent Bets" list

3. **[Browser A] Check Creator Dashboard**
   - Click creator icon (top right)
   - Enter the nsec you saved
   - ✓ Verify: Your market appears in the list
   - ✓ Verify: Shows "Active" or "Ready to resolve" status

4. **[Browser A] Resolve Market**
   - Click on your market in the dashboard
   - Click "YES Won" or "NO Won"
   - ✓ Verify: Payout summary appears
   - Click "Pay All Winners"
   - ✓ Verify: Payments are sent (check wallet for outgoing transactions)

---

### Scenario 2: State Isolation Test

**Purpose:** Verify bettor doesn't accidentally use creator's localStorage

**Steps:**
1. [Browser A] Create a market, copy share URL
2. [Browser A] Open DevTools → Application → Local Storage
3. Note the keys: `nwc_<marketId>`, `sk_<marketId>`, `creator_nsec`
4. [Browser B] Open the share URL
5. [Browser B] Open DevTools → Application → Local Storage
6. ✓ Verify: NONE of the creator's keys exist
7. [Browser B] Place a bet
8. ✓ Verify: Only `bettor_sk_<marketId>`, `lightning_address`, `pending_bet_<marketId>` exist

---

### Scenario 3: Payment Recovery Test

**Purpose:** Verify the app recovers if user pays but closes browser

**Steps:**
1. Open a market, start placing a bet
2. When QR code appears, copy the invoice
3. **Close the browser tab** (before paying)
4. Pay the invoice from your wallet
5. Re-open the market URL
6. ✓ Expected: App should detect pending bet and auto-verify it
   - Note: This relies on `lookup_invoice` which may not work with Primal

---

### Scenario 4: Multiple Bettors Test

**Purpose:** Verify payout calculations with multiple bets

**Steps:**
1. Create a market
2. [Bettor 1] Bet 1000 sats on YES
3. [Bettor 2] Bet 2000 sats on YES
4. [Bettor 3] Bet 3000 sats on NO
5. Resolve as YES
6. ✓ Verify payout breakdown:
   - Total pool: 6000 sats
   - Fee (5%): 300 sats
   - To winners: 5700 sats
   - Bettor 1 gets: 1000/3000 × 5700 = 1900 sats
   - Bettor 2 gets: 2000/3000 × 5700 = 3800 sats

---

### Scenario 5: NWC Provider Compatibility

**Purpose:** Test different NWC wallets work

| Wallet | make_invoice | lookup_invoice | pay_invoice |
|--------|--------------|----------------|-------------|
| Alby Hub | ✓ | ✓ | ✓ |
| Primal | ✓ | ✗ (timeout) | ? |
| Zeus | ? | ? | ? |
| Coinos | ? | ? | ? |

**Steps for each wallet:**
1. Get NWC connection string
2. Create a market
3. ✓ Verify: Invoice generates (make_invoice works)
4. Place a bet and pay it
5. ✓ Verify: Payment detected (lookup_invoice works)
6. Resolve and pay winners
7. ✓ Verify: Payouts sent (pay_invoice works)

---

### Scenario 6: Edge Cases

**6a. Invoice Expiration**
1. Create market, generate invoice
2. Wait >1 hour without paying
3. ✓ Verify: App handles expired invoice gracefully

**6b. Max Bet Enforcement**
1. Create market with max bet = 1000
2. Try to bet 2000
3. ✓ Verify: Error message shown, bet prevented

**6c. Invalid Lightning Address**
1. Try to bet with address "not-an-address"
2. ✓ Verify: Error shown before invoice generation

**6d. Resolution Date Enforcement**
1. Create market with resolution date in past
2. Try to place bet
3. ✓ Verify: "Betting is closed" message shown

---

## CI/CD Integration Ideas

For actual CI/CD, consider:

1. **Playwright for E2E tests**
   - Can run multiple browser contexts (simulate creator + bettor)
   - Can intercept/mock network requests

2. **Local Nostr relay for testing**
   - Run `nostr-relay` locally
   - Point tests to localhost instead of public relays

3. **Lightning regtest**
   - Run Bitcoin Core in regtest mode
   - Run LND or CLN
   - Create test invoices with instant settlement

4. **Mock NWC server**
   - HTTP server that responds to NWC events
   - Returns predictable invoices and preimages

---

## Known Issues to Test Around

1. **Primal NWC doesn't support lookup_invoice**
   - Payments won't auto-verify
   - Manual preimage paste was removed in v42
   - Consider: Re-add manual verification as fallback?

2. **Relay connection drops**
   - App tries to reconnect, but sometimes fails silently
   - Test: Kill network, restore, verify app recovers

3. **Multiple tabs**
   - Opening same market in multiple tabs may cause issues
   - localStorage is shared, but in-memory state is not
