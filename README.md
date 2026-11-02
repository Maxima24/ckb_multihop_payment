# Paying across Fiber without a direct channel

This is my submission for the CKBoost Fiber multi-hop campaign. The idea:
run two Fiber nodes in my browser, Sender A and Receiver C, and get a payment
from one to the other through Bottle, a public node, even though A and C
never open a channel with each other. Then send it back the other way and
watch what happens to the liquidity.

I used the hosted demo: https://fiber.world/docs/build/multi-hop-invoice
(Testnet only, obviously.)

It took me two attempts. The first set of nodes got stuck, which I explain
further down.

## For reviewers: where the proof is

- **Quest 1** (payment completed, invoice Paid, hash):
  [quest1/01-outcome-payment-completed.png](quest1/01-outcome-payment-completed.png)
- **Quest 2, before** the return payment:
  [quest2/01-liquidity-before-return.png](quest2/01-liquidity-before-return.png)
- **Quest 2, liquidity restored** (1 CKB received by A, return hash):
  [quest2/04-liquidity-restored-with-hash.png](quest2/04-liquidity-restored-with-hash.png)
- **Quest 2, after** the return payment:
  [quest2/05-liquidity-after-return.png](quest2/05-liquidity-after-return.png)

## The part that didn't work first time

Setting up the nodes and funding them went fine. I grabbed 10,000 Testnet CKB
from the faucet for each node (A's claim went through at 14:37, C's at 14:38)
and both balances showed up in the tutorial within about 30 seconds. Funny
moment: the faucet page first loaded showing a balance of 0.00 CKB, and for
a second I thought it was empty. It just hadn't loaded yet.

The trouble started when I opened C's channel with Bottle. The tutorial says
opening can take up to a minute. Mine got as far as submitting the funding
transaction (0x60ffa1…), then sat on "Waiting for channel state" with the log
repeating "existing or new channel is still progressing". It never moved past
that.

I reloaded the page to see if that would kick it. The tutorial found my saved
nodes and restored them, which was nice, but C's channel still showed
"Not opened". I tried opening A's channel on the same set and it got further,
up to AWAITING_TX_SIGNATURES, but with C stuck there was no point carrying on
with those nodes.

So I started over with a completely fresh pair of nodes. This time both
channels went through every stage to CHANNEL_READY without a problem, and
everything after that worked first time. I never found out why the first C
node stalled. I can only say what I saw.

The screenshots from both attempts are in `process/`, labelled attempt1 and
attempt2.

## Quest 1: A pays C through Bottle

Once both channels were ready, the tutorial showed this:

| Channel | My side | Bottle's side |
|---|---|---|
| A–B | A: 400 CKB | 151 CKB |
| B–C | C: 400 CKB | 151 CKB |

The maximum invoice was 151 CKB, not 400. That confused me for a minute
until it clicked. C's 400 CKB is money C can *send*. For C to *receive*,
Bottle has to have funds on its side of the B–C channel, and Bottle only had
151 there. That's the ceiling.

I made C create a 1 CKB invoice (it comes out as a long `fibt…` string),
paid it from A, and it settled in a few seconds.

![Payment completed](quest1/01-outcome-payment-completed.png)

- Amount received: 1 CKB
- Routing fee: **0.005 CKB**. I'd allowed up to 1 CKB, so the actual fee was
  tiny.
- Invoice status: Paid
- Payment hash: `0xd2f75c964158d7fa1f09909196aeac527006709b2eb9fcc619b92f7e550ac80f`

## Quest 2: sending it back

After Quest 1, the bars looked like this:

![Before the return payment](quest2/01-liquidity-before-return.png)

| Channel | My side | Bottle's side |
|---|---|---|
| A–B | A: 398.995 CKB | 152.005 CKB |
| B–C | C: 401 CKB | 150 CKB |

A paid 1.005 (the payment plus the fee) and Bottle passed exactly 1 on to C.
Bottle's side toward C dropped to 150, so C could now receive one CKB less
than before.

To fix that, I opened "Restore receiving liquidity" and sent 1 CKB from C
back to A. The panel said the maximum I could return was 152.005 CKB, which
is exactly Bottle's side of the A–B channel. Same idea as before, just the
other direction.

![Restore panel](quest2/02-restore-panel-1ckb.png)

![Return pending](quest2/03-return-pending.png)

![Liquidity restored](quest2/04-liquidity-restored-with-hash.png)

- 1 CKB received by A
- Routing fee: 0.005 CKB again
- Return payment hash: `0x34fd3349046a0d747841f8e32fef929e2bcb1424f29faaf41c205542c8048a34`

And afterwards:

![After the return payment](quest2/05-liquidity-after-return.png)

| Channel | My side | Bottle's side |
|---|---|---|
| A–B | A: 399.995 CKB | 151.005 CKB |
| B–C | C: 399.995 CKB | 151.005 CKB |

Bottle's side toward C went back up to 151.005, so C can receive slightly
more than it could at the very start. A and C each ended 0.005 CKB short
because each one paid one fee. Bottle is 0.01 CKB up. Every channel still
adds up to 551 CKB, so nothing new came in. The money just moved around.

## Things I noticed along the way

- **I funded 499 CKB, but A could only spend 400.** Roughly 99 CKB is locked
  in the channel, most likely as a reserve. I'd have liked the tutorial to
  explain this upfront instead of leaving me to work it out.
- **The on-chain fee is a completely different thing from the routing fee.**
  Opening the channel took A's wallet from 10,000 to 9,500.99999536, so the
  on-chain cost was about 0.00000464 CKB. The routing fee was 0.005 CKB per
  payment.
- **The bars showed round numbers until the first payment.** Then the
  decimals appeared. I'd prefer exact amounts from the start.

## Setup screenshots

<details>
<summary>Starting the nodes</summary>

![Prepare nodes](process/01-prepare-nodes.png)

![Starting nodes](process/02-starting-nodes.png)

![Node A running](process/03-node-a-running.png)

</details>

<details>
<summary>Funding A</summary>

![A zero balance](process/04-fund-a-zero-balance.png)

![Faucet loading](process/05-faucet-loading.png)

![Faucet with A's address](process/06-faucet-a-address.png)

![A claim success](process/07-faucet-a-claim-success.png)

![A claim pending](process/08-faucet-a-pending.png)

![A claim processed](process/09-faucet-a-processed.png)

![A funded](process/10-node-a-funded.png)

</details>

<details>
<summary>Funding C</summary>

![C zero balance](process/11-fund-c-zero-balance.png)

![C address copied](process/12-c-address-copied.png)

![Faucet with C's address](process/13-faucet-c-address.png)

![C claim success](process/14-faucet-c-claim-success.png)

![C claim pending](process/15-faucet-c-pending.png)

![C claim processed](process/16-faucet-c-processed.png)

![C funded](process/17-node-c-funded.png)

</details>

<details>
<summary>Attempt 1: C's channel gets stuck</summary>

![C channel stuck](process/18-attempt1-c-channel-stuck.png)

![C still progressing](process/19-attempt1-c-still-progressing.png)

![Restored, C not opened](process/20-attempt1-restored-c-not-opened.png)

![A channel opening](process/21-attempt1-a-channel-opening.png)

![A awaiting signatures](process/22-attempt1-a-awaiting-signatures.png)

</details>

<details>
<summary>Attempt 2: both channels ready</summary>

![A ready, C opening](process/23-attempt2-a-ready-c-opening.png)

![C collaborating funding](process/24-c-collaborating-funding.png)

![C awaiting signatures](process/25-c-awaiting-signatures.png)

![Both channels ready](process/26-both-ready-initial-liquidity.png)

</details>

<details>
<summary>Paying the invoice</summary>

![Pay invoice](process/27-pay-invoice.png)

![Payment pending](process/28-payment-pending.png)

</details>

---

Testnet only. No private keys, seed phrases or browser data in this repo.y
