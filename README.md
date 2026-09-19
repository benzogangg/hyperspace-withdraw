# Hyperspace escrow withdraw

**Open the page: https://benzogangg.github.io/hyperspace-withdraw/**

It's served by GitHub Pages straight from this repository, so the page is exactly the code you see here.

A single static page that lets a former **Hyperspace** user withdraw the SOL still sitting in their
bid escrow. Unofficial and not affiliated with Hyperspace or Tensor.

## Why this exists

Hyperspace kept bid deposits in a per-user escrow account owned by its program
`HYPERfwdTjyJ2SCaKHmpF2MtrXqWxrsotYDsTrshHWq8`. The website no longer offers a withdraw button, so
many of these escrows still hold SOL.

The program itself still works. Its trading instructions call Metaplex Token Metadata, and those
calls have failed since the Metaplex update of November 2025. `Withdraw` doesn't touch NFTs or
Metaplex, so it still succeeds. This page sends exactly that instruction.

## What you sign

One instruction, `Withdraw` (discriminator `b712469c946da122` = first 8 bytes of
`sha256("global:withdraw")`):

| # | account | role |
|---|---------|------|
| 0 | your wallet | signer |
| 1 | your wallet | receiver of the SOL |
| 2 | your escrow = PDA `["hyperspace", auction house, your wallet]` | source |
| 3 | `FEL1Z3EjUEbET9miT2p3S8qK1K11stCzN5KLaqZZ976d` | Hyperspace's auction-house authority (stored in the auction house account data). Read-only, not a signer, receives nothing |
| 4 | `5pdaXth4ijgDCeYDKgSx3jAbN7m8h4gy1LRCErAAN1LM` | Hyperspace auction house |
| 5–7 | Token program, System program, Rent sysvar | |

Data: discriminator + escrow bump (1 byte) + amount (u64, little-endian) = the full escrow balance.

There are no token approvals, no authority changes and no transfers to anyone other than you. You
pay your own network fee (0.000005 SOL). If anyone else signs, the program rejects the transaction
(`ConstraintSeeds`, error 2006), because the escrow address is derived from your wallet.

## How to use it

1. Open the page on a computer with Phantom, Solflare or Backpack. On a phone, open it in your wallet
   app's built-in browser.
2. **Optional:** before connecting, use "Check any address" to see your escrow balance. This is
   read-only and asks for no signature.
3. Press **Connect wallet**. The page finds your escrow and simulates the withdrawal.
4. Press **Withdraw to my wallet**. Your wallet shows its own preview. It should be **+N SOL to you**
   with nothing sent out except the network fee. If it shows anything else, don't sign.

## How to verify this page yourself

- **All the logic is in [`app.js`](app.js)** (about 165 lines). The transaction is built in
  `withdrawIx()`.
- **No hidden third-party code.** The page loads only its own files. [`web3.iife.min.js`](web3.iife.min.js)
  is the unmodified `lib/index.iife.min.js` from the official npm package `@solana/web3.js@1.98.0`,
  SHA-256 `89c3d6c27cae93882d78153ef7f23c8d50f53f5e3f0b72666bb909c6a4dc9fac`. To check it:
  ```bash
  npm pack @solana/web3.js@1.98.0 && tar xzf solana-web3.js-1.98.0.tgz
  shasum -a 256 package/lib/index.iife.min.js web3.iife.min.js
  ```
- **The page can't send your data anywhere.** A Content-Security-Policy in [`index.html`](index.html) allows
  network requests only to two public Solana RPCs (`solana-rpc.publicnode.com` and
  `api.mainnet-beta.solana.com`). The browser blocks everything else.
- **Compare with real withdrawals made by other users.** These are ordinary successful Hyperspace
  withdrawals, not made with this page. They have the same instruction and the same 8 accounts in the
  same order, only the wallet and escrow differ:
  - [`qk52YWP8…F9sM`](https://solscan.io/tx/qk52YWP84LqsgJADVRuD4erbTFTvBkrR1oTNih6QbF8vSv9h1SMh3TEvh8B1NKV6tPuGWwhecE8EvgLb5bJF9sM) (2026-09-14)
  - [`63jbKADR…V4Gx`](https://solscan.io/tx/63jbKADREqyauV8zqrsXQKwJFLCK4GJkFVoPZwrUE1ve3JAj3zcbfreSchjnKuHdrLJT9bditaKoebkVUja1V4Gx) (2026-09-11)
- **Your wallet is the final check.** Phantom, Solflare and Backpack simulate the transaction
  themselves before you sign, independently of this page. Sign only if the preview shows SOL coming
  in to you and nothing going out except the ~0.000005 SOL fee.

## Safety measures in the page

- **One official address:** `https://benzogangg.github.io/hyperspace-withdraw/`. Copies on other domains
  aren't this project, even if they look the same.
- **Safety check before signing** (`assertSafe()` in `app.js`). The page refuses to hand your wallet
  anything other than exactly one Hyperspace `Withdraw` from your own escrow to your own wallet,
  with you as the fee payer.
- **Anti-clickjacking** ([`antiframe.js`](antiframe.js)). The page stays blank when another site
  embeds it in a frame.
- **Locked network access.** The Content-Security-Policy allows only the page's own files and two
  public Solana RPCs.
- **Protected history.** The `main` branch can't be force-pushed or deleted, so every change to
  the code stays visible in the commit history.

## If someone sent you this link

You don't need to trust them. Don't take anyone's word, this README included. Check your escrow
balance on Solscan first, read `app.js`, and let your wallet's own preview decide. Nobody running this
page ever asks for your seed phrase or private key, and nobody asks you for payment up front. If
someone does, it's a scam.
- **Run it locally.** Download this repository and serve the folder, e.g. with
  `python3 -m http.server`. You don't have to trust the hosted copy.

## License

MIT, see [LICENSE](LICENSE).
