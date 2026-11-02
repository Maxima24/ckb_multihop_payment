# Paying across Fiber without a direct channel

This is my submission for the CKBoost Fiber multi-hop campaign.

The challenge: run two Fiber nodes in my browser, Sender A and Receiver C,
and get a payment from A to C through Bottle, a public node, even though A
and C never open a channel with each other. Then send money back the other
way and see what that does to the liquidity on both channels.

I used the hosted demo at https://fiber.world/docs/build/multi-hop-invoice,
on Testnet only. It took me two attempts, because my first Receiver C got
stuck while opening its channel. I explain that below.

**Short on time? The four required proofs:**

1. Quest 1, payment completed: [quest1/01-outcome-payment-completed.png](quest1/01-outcome-payment-completed.png)
2. Quest 2, liquidity before the return: [quest2/01-liquidity-before-return.png](quest2/01-liquidity-before-return.png)
3. Quest 2, liquidity restored: [quest2/04-liquidity-restored-with-hash.png](quest2/04-liquidity-restored-with-hash.png)
4. Quest 2, liquidity after the return: [quest2/05-liquidity-after-return.png](quest2/05-liquidity-after-return.png)

Everything else is below, in the order it happened.

---

## 1. Starting the nodes

The tutorial creates two separate identities in the browser, one for A and
one for C. You click "Prepare nodes" and they start one after the other.

![Nodes not started yet](process/01-prepare-nodes.png)

![Sender A starting](process/02-starting-nodes.png)

![A running, C still starting](process/03-node-a-running.png)

## 2. Funding Sender A

Each node needs its own Testnet CKB to open a channel. A started at 0, so I
copied its address over to the Nervos faucet.

![A's balance at zero](process/04-fund-a-zero-balance.png)

The faucet first loaded showing a balance of 0.00 CKB, and for a second I
thought it was empty. It just hadn't finished loading.

![Faucet still loading](process/05-faucet-loading.png)

I pasted A's address and claimed 10,000 CKB.

![A's address in the faucet](process/06-faucet-a-address.png)

![Claim success for A](process/07-faucet-a-claim-success.png)

The claim sat on "Pending" for a moment, then went through.

![A's claim pending](process/08-faucet-a-pending.png)

![A's claim processed](process/09-faucet-a-processed.png)

Back in the tutorial, A showed 10,000 CKB and the "Open channel" button
unlocked.

![A funded](process/10-node-a-funded.png)

## 3. Funding Receiver C

Same routine for C, which has its own separate address.

![C's balance at zero](process/11-fund-c-zero-balance.png)

![Copying C's address](process/12-c-address-copied.png)

![C's address in the faucet](process/13-faucet-c-address.png)

![Claim success for C](process/14-faucet-c-claim-success.png)

![C's claim pending](process/15-faucet-c-pending.png)

![C's claim processed](process/16-faucet-c-processed.png)

![C funded](process/17-node-c-funded.png)

## 4. Problems I ran into

### Receiver C's channel got stuck

This is where things went wrong. I opened C's 499 CKB channel with Bottle.
The tutorial says that can take up to a minute. Mine submitted the funding
transaction (0x60ffa1…) and then just sat on "Waiting for channel state",
with the log repeating "existing or new channel is still progressing".

![C stuck waiting](process/18-attempt1-c-channel-stuck.png)

![Still progressing, never finishing](process/19-attempt1-c-still-progressing.png)

I reloaded the page, hoping it would kick things along. The tutorial found my
saved nodes and restored them, which was a nice touch, but C's channel still
said "Not opened".

![Nodes restored, C still not opened](process/20-attempt1-restored-c-not-opened.png)

I opened A's channel on the same set to see if it was a general problem. A
got further, up to AWAITING_TX_SIGNATURES. But with C going nowhere, there
was no point continuing with those nodes.

![A's channel opening](process/21-attempt1-a-channel-opening.png)

![A waiting for signatures](process/22-attempt1-a-awaiting-signatures.png)

So I started again with a completely fresh pair of nodes. I never found out
why the first C stalled, and I'd rather not guess. What I can say is that
the second time, both channels opened cleanly.

### Smaller things that tripped me up

- **The 499 vs 400 gap.** I funded A's channel with 499 CKB, but the
  tutorial said A could only spend 400. I had to work out for myself that
  about 99 CKB stays locked in the channel, most likely as a reserve. A
  one-line note in the tutorial would have saved me the head-scratching.
- **A vague status message.** "Existing or new channel is still progressing"
  tells you to wait, but not how long, or what to do if it never finishes.
  When my channel got stuck, I had nothing to go on.
- **The faucet showing 0.00 CKB** on first load (see above). Harmless, but
  alarming for a second.

## 5. Second attempt: both channels ready

With fresh nodes, A's channel reached CHANNEL_READY first, then C's went
through every stage.

![A ready, C opening](process/23-attempt2-a-ready-c-opening.png)

![C collaborating on funding](process/24-c-collaborating-funding.png)

![C waiting for signatures](process/25-c-awaiting-signatures.png)

And finally, both channels ready:

![Both channels ready](process/26-both-ready-initial-liquidity.png)

| Channel | My side | Bottle's side |
|---|---|---|
| A–B | A: 400 CKB | 151 CKB |
| B–C | C: 400 CKB | 151 CKB |

The maximum invoice showed 151 CKB, not 400. More on why in the "What I
learnt" section.

## 6. Quest 1: A pays C through Bottle

C created a 1 CKB invoice. It comes out as one long string starting with
`fibt`, which is the Testnet invoice format. The tutorial showed the amount
and a maximum routing fee of 1 CKB before I hit send.

![Invoice ready to pay](process/27-pay-invoice.png)

For a few seconds the payment was "Inflight" while it travelled
A → Bottle → C.

![Payment pending](process/28-payment-pending.png)

Then it landed:

![Payment completed](quest1/01-outcome-payment-completed.png)

- Amount received: **1 CKB**
- Routing fee: **0.005 CKB** (I'd allowed up to 1 CKB)
- Route: Sender A → Bottle B → Receiver C
- Invoice status: **Paid**
- Payment hash: `0xd2f75c964158d7fa1f09909196aeac527006709b2eb9fcc619b92f7e550ac80f`

## 7. Quest 2: sending it back

After Quest 1, the bars had shifted:

![Liquidity before the return payment](quest2/01-liquidity-before-return.png)

| Channel | My side | Bottle's side |
|---|---|---|
| A–B | A: 398.995 CKB | 152.005 CKB |
| B–C | C: 401 CKB | 150 CKB |

A paid 1.005 (the payment plus the fee) into the A–B channel, and Bottle
passed exactly 1 on to C. Bottle's side toward C dropped from 151 to 150, so
C could now receive 1 CKB less than before.

To fix that, I opened "Restore receiving liquidity" and sent 1 CKB from C
back to A. The panel said the most I could return was 152.005 CKB, which is
exactly Bottle's side of the A–B channel. Same rule as before, just in
reverse.

![Restore panel, 1 CKB](quest2/02-restore-panel-1ckb.png)

![Return payment pending](quest2/03-return-pending.png)

![Liquidity restored](quest2/04-liquidity-restored-with-hash.png)

- **1 CKB received by A**
- Routing fee: **0.005 CKB** again
- Return payment hash: `0x34fd3349046a0d747841f8e32fef929e2bcb1424f29faaf41c205542c8048a34`

And afterwards:

![Liquidity after the return payment](quest2/05-liquidity-after-return.png)

| Channel | My side | Bottle's side |
|---|---|---|
| A–B | A: 399.995 CKB | 151.005 CKB |
| B–C | C: 399.995 CKB | 151.005 CKB |

Bottle's side toward C is back up to 151.005, so C can receive a little more
than it could at the very start.

The whole run in one table:

| Stage | A–B: A | A–B: Bottle | B–C: Bottle | B–C: C | Max invoice |
|---|---|---|---|---|---|
| Channels ready | 400 | 151 | 151 | 400 | 151 |
| After Quest 1 | 398.995 | 152.005 | 150 | 401 | 150 |
| After Quest 2 | 399.995 | 151.005 | 151.005 | 399.995 | 151.005 |

## 8. What I learnt

**Having money in a channel doesn't mean you can receive.** This was the big
one. C had 400 CKB, but that's money C can *send*. To *receive*, the node on
the other end (Bottle) needs funds on its side. Bottle only had 151 on the
B–C channel, so that was C's ceiling, no matter how much C held.

**A route is only as strong as its weakest hop.** A could send 400 into the
A–B channel, but Bottle could only pass 151 along to C. The smaller number
wins.

**Fees land on the first hop.** Bottle took in 1.005 from A but only passed
1 to C. The 0.005 difference is Bottle's cut for forwarding. It happened
again on the way back, so Bottle finished 0.01 CKB up, and A and C each
finished 0.005 down.

**Reverse payments move money, they don't create it.** Every channel added
up to 551 CKB at every single stage. Sending money back from C didn't add
anything new. It just pushed Bottle's balance back toward C's side, which is
what gave C its receiving room back.

**There are three different "costs", and they're not the same thing:**
- the on-chain fee to open a channel: A's wallet went from 10,000 to
  9,500.99999536, so about 0.00000464 CKB
- the routing fee per payment: 0.005 CKB
- the channel reserve: about 99 CKB locked, not spent

**Multi-hop is what makes this usable.** A and C never opened a channel with
each other, yet money moved between them in seconds. If everyone needed a
direct channel with everyone they pay, nobody would bother. One
well-connected node in the middle changes that.

---

Testnet funds only. No private keys, seed phrases or browser data in this
repo.
