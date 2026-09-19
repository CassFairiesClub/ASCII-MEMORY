# ASCII-MEMORY — ASCII art inscribed in Chia transaction memos

*ASCII-MEMORY by Cass - 2026*

<img width="1641" height="1000" alt="image" src="https://github.com/user-attachments/assets/99650f9c-ae09-453b-a771-858de16b4c77" />

ASCII-MEMORY turns a picture into ASCII art small enough to live inside a Chia transaction memo,
inscribes it on-chain with your wallet, and reads it back. It is five static HTML pages with no
build step and no server, packaged as one sandboxed app for the [Sage wallet](https://github.com/xch-dev/sage).

## Install

Sage 0.13 or later can install the app two ways. Both install the same files and ask for the same permissions.

### From URL (recommended)

1. In Sage, open **Apps** and choose to install an app from a URL.
2. Enter **`https://cassxch.com/ascii_memory/`** and approve the permissions it asks for.

The advantage of a URL install is **updates**: Sage checks that address for new versions and offers
to update when a new release is published there, so you don't have to download anything.

### From the zip file

1. Download **[`ascii-memory-v3.9.1.zip`](ascii-memory-v3.9.1.zip)** from this repository.
2. In Sage, install it from the zip file and approve the permissions it asks for.

A zip install never checks for updates: to update, download the new zip and install it again.
If you switch to the URL install, remove the zip copy afterwards so you don't run two copies with separate settings.

SHA-256 of the zip:

```
75401f1313fa1bf913ef5bb1667d28e624cc8175b869c68b156d1cf010e60eb0
```

Either way, the app opens on the **Home** page, which explains it. The other tabs are **The Wall**,
**My Memos**, **Generate** and **View**.

**If a URL install fails with "app directory already exists, cannot create"**, an earlier attempt left
a folder behind. Close Sage, delete the folder starting with `url-cassxch-com-` in
`%APPDATA%\com.rigidnetwork.sage\apps\` (Windows), then install again.

### What the app contains

| Page | What it does |
|---|---|
| **Home** | what the app does, and two generator presets: The Wall (1 XCH) and My Memos (1 mojo) |
| **The Wall** | the newest inscriptions of at least 1 XCH sent to `xch1khfj4wj6wt783wxzysgt5etws3hy2s03025k0m3dy707ul8x9hnq2w0cym` |
| **My Memos** | the same, for your wallet's first unhardened address, with no XCH minimum |
| **Generate** | PNG/JPEG → ASCII art that fits a byte budget, hand editing, terminal styles, and sending through Sage |
| **View** | a coin ID or pending transaction ID → the inscription, in its stored style, with PNG export; or any XCH address → every inscription it received, any amount |

The pages are plain HTML and JavaScript inside the zip, so you can read every line before installing.
The build script and the optional browser bridge are not published here.

## Amounts

Every mojo or XCH figure in the app goes through `amounts.js` and is shown in both units with the locale's separators, for example `1.5 XCH (1,500,000,000,000 mojo)`.

- **Exact:** conversion uses BigInt, so amounts beyond 2^53 mojo stay exact, and trailing zeros are trimmed (`0.0001458 XCH`, not `0.000145800000`).
- **Covered:** the fee and amount fields, the suggested fee, the Sage balance, the whole transaction review (amount, fee, inputs, outputs), the wall notes and the 1 XCH minimum, and The Wall's and My Memos' per-card amounts and totals.

## The Wall

`feed.html` runs `viewer.js` in feed mode (`<body data-page="feed">`), so the cards are the same as in the viewer: stored terminal style, title, text size, PNG export, plus the amount sent and an **Open in View →** link.

- **How it finds inscriptions:** it decodes the bech32m address to a puzzle hash and lists that address's coins with `get_coin_records_by_puzzle_hash`, newest first. For each coin it reads the memos from the spend that created it (`get_block_spends_with_conditions`), the same way the viewer does.
- **Minimum 1 XCH:** only coins of at least 1 XCH (1,000,000,000,000 mojo) count. Smaller coins are skipped before their block is read, so they cost no extra API calls.
- **What counts as ASCII art:** a memo list containing a `memoart:v1;` style memo or a packed `asciiz:` memo, or printable multi-line ASCII (at least 3 lines, at least 8 columns wide). Change coins and plain payments are skipped.
- **Posting to The Wall:** on the Generate tab, **The Wall preset** fills in the wall address and sets the amount to 1 XCH. A note under the amount confirms whether the transaction will appear on The Wall, and warns that the XCH goes to the wall owner. **My Memos preset** fills in your own first address and sets the amount to 1 mojo.
- **Totals and options:** a counter shows the total number of eligible inscriptions ever sent to the address and the XCH they sent, with a **+N new** badge when new ones arrive (click it to dismiss). **Show last** picks how many cards to show (5, 10, 20, 50 or All) and **Sort** picks **Newest first** or **Highest XCH first**, where ties go to the newest. Both choices are remembered on this device.
- **Monitoring:** it checks every 60 seconds, but only while the page is visible, and when it comes back into view if the last check is stale. **Refresh now** checks immediately. Each check asks only for coins from the last seen block height on, so a check with nothing new is one API call.
- **Index and memory:** to count every eligible inscription, The Wall indexes the address's whole history of coins of 1 XCH or more, reading each block once. The index (coin ID, amount, height, time) is kept in the device's local storage, since confirmed coins never change, so later launches only check for new coins. Memo text is fetched only for the cards on screen and kept for the session. The index is rebuilt automatically if the wall address or minimum changes.
- **Limits:** one check reads at most 200 new coins' blocks and continues automatically where it stopped. Mainnet only. The address and the minimum live in `app-config.js` inside the package, shared by the Generate and The Wall tabs.

## My Memos

`myfeed.html` is the same feed machinery pointed at your own wallet instead of the shared address:

- **Address:** the wallet's **first unhardened address** (derivation index 0), read from Sage with `wallet.getDerivations({hardened: false, offset: 0, limit: 1})`. Outside Sage there is no wallet to ask, so `myfeed.html?address=xch1…` points the page at any address.
- **No minimum:** every coin at that address is checked, whatever the amount.
- **Separate index:** stored per address, so several wallets keep separate indexes, with its own Show last and Sort settings.
- **XCHandles:** the Generate tab's receive address also accepts an [XCHandle](https://xchandles.com) such as `@cassfairiesclub` (mainnet only). It is looked up on `api.xchandles.com` as you type, and the address it points at (its NFT's owner address) is shown under the field, used for the transaction and shown again in the review before you sign. Unregistered or expired handles show an error and can't be sent to.
- **Sending to it:** the Generate tab has two presets, **The Wall preset** (shared address, sets the amount to 1 XCH) and **My Memos preset** (your first unhardened address, sets the amount to 1 mojo). You can still change the amount or address after using one. The second replaces the old *Use my Sage address* button, which filled Sage's rotating receive address and so did not show up in My Memos.

## The Sage App

The app is sandboxed by Sage: scripts run only from the package, network access is limited to the
hosts below, and it cannot open outside links or download files.

### Permissions Sage will ask for

- **Network:** only `https://api.coinset.org` and `https://testnet11.api.coinset.org`, the public full-node API the View, The Wall and My Memos tabs read from, and `https://api.xchandles.com`, to turn an `@handle` typed in Generate's receive address or in View into an address.
- **`wallet.get_sync_status`:** balance, receive address and which network you are on.
- **`wallet.get_derivations`:** your wallet's first unhardened address, which My Memos watches.
- **`wallet.send_xch`:** inscriptions. Sage still shows its own approval dialog for every send, and the app never sees your keys.
- **`storage.persistent_webview`:** a Sage 0.13.0 workaround, see below. The app only stores display settings and its feed index on your device.

### Tabs and drafts

- The tabs are separate pages inside the app, so switching tabs reloads the page. To avoid losing work, the Generate tab keeps a session draft: all settings, the address/amount/fee, any hand edits, and a copy of the image (downscaled to 1600 px, WebP). The draft is restored when you come back.
- The View tab remembers the last inscription it showed and caches completed lookups for the session (up to 6, oldest dropped first if storage runs out). Coming back from Generate redraws the art without calling coinset.org; a **Reload from chain** button forces a fresh lookup. Pending transactions and empty or failed lookups are never cached.
- **Any address:** type an `xch1…` (or `txch1…`) address or an XCHandle such as `@cassfairiesclub` instead of an ID, or use the **The Wall's address** example button, to list every ASCII art inscription that address received, whatever the amount, with the same count, **Show last** and **Sort** controls as The Wall. A handle is looked up on `api.xchandles.com` and shown next to the address it points at. The address or handle goes in the link (`viewer.html#xch1…` or `viewer.html#@name`), and each address keeps its own index on the device, so reopening it only checks for new coins. A busy address can take a while the first time: each check reads at most 200 coins' blocks and then continues on its own.
- **Just-sent inscriptions:** a transaction can take a few seconds to reach coinset.org's mempool. If an ID is not found yet, or its coin is unspent with no memos yet, View looks it up again every 5 seconds for up to 2 minutes, with a countdown. Once the transaction is in the mempool, it waits for confirmation and switches to the permanent coin link.
- After **Send with Sage…**, the result has an **Open in viewer →** link that opens the View tab on the new inscription.

### Sending from the app

- **Send with Sage…** calls `wallet.sendXch`. Sage shows its own approval dialog, then signs and submits. **Approve within 30 seconds**, or Sage expires the request and nothing is sent.
- The package includes `sage-runtime-bridge.js` from the official [`sage-app-sdk@0.13.0`](https://www.npmjs.com/package/sage-app-sdk) (Apache-2.0, license included), which sets up `window.__SAGE__`. It is an ES module bundle and must load with `type="module"`: as a classic script its top-level classes (`Image`, `Window`, …) overwrite browser globals, and image loading breaks.
- Sage may intercept drag-and-drop, so pick the image with a click or paste it.

### Sandbox workarounds

- **Links and downloads:** the webview can't open outside links or download files, so explorer links are plain text and the download buttons and reference-wallet export are hidden. Inside the app the PNG buttons are replaced by **PNG…**, which opens the image in a popup: right-click it → Copy image / Save image as.
- **Clipboard:** the async Clipboard API is refused, and Sage gives apps no clipboard or file method. `copy-helpers.js` makes Copy buttons use the webview's selection copy (`execCommand('copy')`), then the Clipboard API, and as a last resort open the text pre-selected so you can press Ctrl+C.
- **Scripts:** they may only load from the package (`script-src 'self'`), so all code lives in `.js` files. The build fails if an inline script or `on…=` handler sneaks back in.
- **Hashing:** if the WebCrypto API is unavailable, coin IDs are hashed with a built-in pure-JS SHA-256.
- **Incognito storage:** the manifest requests `storage.persistent_webview`, even though the app stores nothing beyond the session draft. Sage 0.13.0 on Windows fails its own "incognito storage" sandbox test (fixed after that release in [xch-dev/sage#841](https://github.com/xch-dev/sage/pull/841)) and blocks every app that runs in incognito mode. Requesting persistent storage makes Sage check its persistent-storage test instead. Once a Sage release includes the fix, you can remove the capability.

## Running the pages outside Sage

Unzip the package and open `index.html` (Generate), `viewer.html`, `feed.html` or `myfeed.html` in a
browser. Everything that only reads the chain works: View and The Wall call the public coinset.org API
directly, and the generator converts images, edits art and counts bytes locally.

Two things differ outside Sage:

- **Sending** needs the wallet, so instead of **Send with Sage…** the generator shows the Chia
  reference-wallet export: fill in the address, amount and fee, download `send_transaction.json`, then

  ```bash
  chia rpc wallet send_transaction -j send_transaction.json
  ```

  The response contains `transaction_id`. Paste it into the View tab while it is in the mempool, and the
  page switches to the permanent coin ID once it confirms. Use `#testnet11:0x…` for testnet.
- **My Memos** cannot ask Sage for your address there. `myfeed.html?address=xch1…` points it at any address.

## Size limits (why ~457 KB)

Chia sets no size limit on memos themselves; they are bytes inside a `CREATE_COIN` condition. The real limits are cost-based:

- every byte in a transaction costs **12,000** cost units
- the mempool accepts at most **5.5 billion** cost per transaction (half of the 11 billion block limit)
- so a simple XCH send can carry about **457,000 bytes** of memo in total

Fees are optional while the mempool is not full. When it is full, the minimum is about 5 mojo per cost unit, or 60,000 mojo per memo byte (10 KB ≈ 0.0006 XCH). Memos over ~100 KB take up a big share of a block and may wait longer to be included.

## Memo format

- `memos[0]`: the art as plain UTF-8 ASCII, lines separated by `\n`, readable on any explorer
  - or, in **Pack** mode, `asciiz:` + base64(raw DEFLATE(art)): roughly 2–3× more detail, and only the viewer can decode it
- `memos[1]` (optional): a short title
- last memo: display settings, e.g. `memoart:v1;theme=green-crt` (about 26 bytes, counted in the byte budget)

The settings memo keeps its original `memoart:v1;` prefix, so inscriptions made before the ASCII-MEMORY rename still open with their style.

### Text size

Every page that shows art (The Wall, My Memos, Generate, View) has a text size control above it: **Fit**, **A−**, a slider (2–40 px) and **A+**.
- Every page starts at a fixed **4 px**, with Fit off.
- **Fit** sizes the art to the width of its box, and the slider shows that size.
- Moving the slider or pressing A−/A+ switches to a fixed size; large sizes scroll inside the art box. Ticking **Fit** again returns to automatic sizing.
- In the generator the size also applies to the text editor; in the viewer it applies to every card.
- On The Wall, My Memos and an address in View, each card is only as wide as its art needs and the cards wrap into as many per row as fit, so one wide artwork doesn't force the other rows down to one card. **Crop tall cards** (on by default) keeps long inscriptions, such as pure text, in a 600 px box with a scroll bar; untick it to show them full height. With **Fit**, each card takes the full width.
- Each page remembers the choice in the browser's local storage. The shared code is in `text-size.js`.
- This only changes the on-screen display. The viewer's PNG size is set separately, and nothing about text size is stored on-chain.

### Editing the art by hand

In the generator, **Edit text** turns the preview into an editable text area (same font and terminal style), and you can also start from scratch with no image.
- **Overwrite typing** (on by default) makes typed characters replace the ones under the cursor, like a terminal, so columns stay aligned. At the end of a line, typing extends it. Undo works.
- Once you change the text, it replaces the image-generated art. Image controls pause, while title, style, Pack and the byte budget still apply, and byte count and cost update as you type.
- **Discard edits**, or loading a new image, goes back to generating from the image.
- Non-ASCII characters are allowed but flagged: they use 2–4 bytes each and may not line up in every font.

### Terminal styles

The generator's **Terminal style** picker restyles the preview and stores the style id in the settings memo. The viewer's **Style** picker defaults to **Auto (from memo)**, so art opens the way it was made, and you can still override it. PNG exports use the same colours, glow and scanlines.

| id | Style |
|---|---|
| `paper` | dark ink on light (used for inscriptions without a settings memo) |
| `classic` | white on black terminal (the generator's default) |
| `green-crt` | green phosphor CRT with glow and scanlines |
| `amber-crt` | amber CRT with glow and scanlines |
| `ubuntu` | Ubuntu terminal (aubergine) |
| `powershell` | Windows PowerShell blue |
| `c64` | Commodore 64 |
| `solarized-dark` | Solarized Dark |

- The styles are defined once, in `terminal-themes.js` inside the package. Only the id goes on-chain, and an unknown id falls back to `paper`.
- Choosing a dark style ticks **Invert brightness** automatically, so bright areas of the image become dense characters that glow on the dark background. You can still untick it.
- Fonts come from the viewer's system (for example Ubuntu Mono if it's installed). The Sage sandbox doesn't allow loading web fonts.

## Why the viewer asks for a coin ID

A wallet `transaction_id` is the hash of the spend bundle, and it is not stored on-chain once the transaction confirms. Coins, however, are permanent. The viewer resolves memos like this:

1. `get_coin_record_by_name` → find the block where the coin was created and the block where it was spent
2. `get_block_record_by_height` → `get_block_spends_with_conditions` → read the memo lists from the `CREATE_COIN` (opcode 51) conditions
3. if the ID is not a coin, `get_mempool_item_by_tx_id` → wait for the input coin to be spent, then repeat step 1

## Licence

MIT. `sage-runtime-bridge.js` inside the package is part of the
[sage-app-sdk](https://www.npmjs.com/package/sage-app-sdk) and stays under Apache-2.0; its licence
travels with it in the zip as `LICENSE-sage-app-sdk.txt`.

*ASCII-MEMORY by Cass - 2026*
