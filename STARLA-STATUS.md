# Starla v99 — recovery status (16 Nov 2026)

## 1. Where things stand

| Part | State |
|---|---|
| Relay on Cloudflare | **Live and verified with real Gemini** (v99) |
| Updated website file | **Ready and tested, not yet published** — I could not push to GitHub from this session |
| Live site (GitHub Pages) | Still the 13 Nov file. It has no relay address, so Starla AI on the live site is unchanged until you upload the new file |

Not "everything is live": the backend is, the frontend publication is waiting on one upload by you (section 6).

## 2. Recovery baseline

| File | What it was | SHA-256 (first 16) |
|---|---|---|
| `talent-star.html` (your attachment) | v97 page, byte-identical to the file published on GitHub Pages; relay address empty | `704287d1c3deecc2` |
| `starla-gemini-relay.js` (your attachment) | v97 relay, identical to my 13 Nov delivery | `4d03ff4ad824d05b` |

Both are kept untouched. I rebuilt the lost v98 changes from them. Proof that the rebuild is faithful: backup source + the v98 changes, minified, is byte-identical to the code that was running on Cloudflare (`19b474cdc966b1d0…`).

## 3. What I found on Cloudflare, and what I did (each step approved by you)

| Time (Dubai, 16 Nov) | Event |
|---|---|
| 10:21 | You added the secret `GEMINI_API_KEY` in the dashboard. Correct Worker, correct name. |
| 10:32 | A Wrangler deploy replaced the Worker with a **static copy of the website**. This removed the relay code, its three settings and the secret. The relay URL served the website; every Starla request would have failed. |
| 11:34 | Restored relay v98 from the stored version, with your secret carried over (its value was never read). Allowed sites updated. |
| 11:40 | Gemini test: every request stalled (8 of 8). Not an authentication problem: Google rejects an invalid key instantly, and it did not. |
| 11:42 | Changed `GEMINI_MODEL` to `gemini-3.5-flash`. Gemini started answering, about 6 in 10. |
| 12:42 | Deployed relay **v99**: a stalled attempt is dropped after a few seconds and retried once. |

Likely cause of the 10:32 accident: `wrangler deploy` was run from the website folder using the relay's name. Please don't run Wrangler for this Worker; if you ever must, only from the `relay` folder (see `wrangler.jsonc`).

One side effect I left alone: that Wrangler deploy switched on Worker logs. The relay only logs error classes and status codes, never messages, files or keys, and Cloudflare's request logs don't include request bodies. You can turn logs off under the Worker's Settings → Observability.

## 4. Verified relay details

| Item | Value |
|---|---|
| Account | Waseem.asghar12@gmail.com's Account |
| Worker / environment | `starla-gemini-relay` (single production environment) |
| **Base URL** | `https://starla-gemini-relay.talent-star.workers.dev/` |
| Chat + attachments | `POST /` on the base URL, JSON. There is no `/chat` or `/api` route. |
| Health | `GET /` or `GET /health` → `{"service":"starla-relay","ok":true,"version":"v99"}` |
| Status checks | `POST /` `{"mode":"ping"}` (relay only) · `{"mode":"selftest"}` (one tiny real Gemini request) |
| Version | v99, deployment `12d1d2fd-af5f-4dea-94a2-f681817f99b7`, Worker version 7 |
| Provider | Google AI Studio (Gemini Developer API) |
| Settings (not secret) | `GEMINI_MODEL=gemini-3.5-flash` · `RATE_LIMIT_PER_MIN=20` · `ALLOWED_ORIGINS=https://waseemasghar12.github.io,https://talent-star.com,https://www.talent-star.com` |
| Secrets (names only) | `GEMINI_API_KEY` |
| Archive settings, scheduled jobs | None |

Request: `{ message, lang, page, history:[{role,text}], attachments:[{id,name,mimeType,data}] }` → `{ reply, attachments:[…] }` or `{ error:{ code } }`.

The delivered `starla-gemini-relay.js` is the source of the running code (the Worker holds its minified build; checksum verified at upload).

### The five checks you asked to keep separate

| | Check | Result |
|---|---|---|
| A | Secret binding exists | ✔ `GEMINI_API_KEY` bound to the active version |
| B | Worker reachable | ✔ health 200, v99 |
| C | Gemini authentication | ✔ real requests answered |
| D | Model available | ✔ `gemini-3.5-flash`. ✘ `gemini-flash-latest` (stalled 8 of 8 from this Worker) |
| E | Frontend can use the relay | ✔ under the real site origin in a test browser. **Not yet on the published site.** |

### Model change (needs your awareness)
You asked to keep `gemini-flash-latest` unless evidence required a change. Evidence: 0 of 8 requests answered with it; public reports from late October describe the same stalls and 503 errors for that alias (now Gemini 3.8 Flash) on free-tier keys called from Cloudflare Workers. To go back: Worker → Settings → Variables → `GEMINI_MODEL`.

### Remaining relay risk: your key is on Google's free tier
Google returned "quota exceeded" (429) three times during testing, after roughly ten requests in a minute. Starla shows an honest "busy, try again in a minute" message when that happens, but on a public site it will happen often. Enabling billing on the key's Google project removes the low limit and also changes Google's data terms (paid-tier content is not used to improve Google's products). This is your decision and your action in Google AI Studio.

### Rollback
- Dashboard → the Worker → Deployments → pick a version → Rollback.
- Version 6 = v98 with `gemini-3.5-flash`; version 5 = v98 with `gemini-flash-latest`.
- **Never roll back to version 3** (the accidental website copy).
- Website: set `proxyUrl: ""` in the page to switch Starla AI off cleanly.

## 5. What changed in the website file

**Starla relay**
- `proxyUrl` is now the verified relay URL: one setting for desktop and mobile. A placeholder such as `your-worker-name.workers.dev` is never treated as configured.
- An obsolete relay address saved in a browser by an old `!gemini relay` command is removed. No other storage is touched.
- `!gemini status` shows: effective relay URL and its source; provider; model as reported by the relay; relay connectivity (passed / failed / not checked) with relay version and whether a key is present; Gemini upstream (not checked until `!gemini test`). A URL alone never produces a tick. No secret appears.
- `!gemini test` sends one tiny real request and reports the result.

**Privacy Policy** (existing topic; one English and one Arabic copy; numbering 1–9 intact; points 3, 4, 6, 8, 9 unchanged)
- **Point 1 "How This Website Operates"** — your wording, plus two sentences I added because the check required them: Starla chat settings and anonymous usage counts are kept on the device and the open conversation only until the tab closes; the website sets no cookies and the Botpress component may keep its own session identifiers.
- **Point 2 "Forms"** — it was lost; restored with the wording you approved earlier.
- **Point 5 "Who Processes Information"** — your wording, plus one sentence: "Files sent to Starla AI are not saved to those Google Drive folders."
- **Point 7 "Processing Outside the UAE"** — your wording, unchanged.
- **Point 9** — was already in the backup; unchanged.
- Attachment notice — unchanged and correct: "Files are processed securely for analysis and cleared when your chat ends, please read our Privacy Policy", linked to the policy topic.

What the checks showed (measured on a real page load):
- On load the page contacts Google Fonts and Botpress.
- The page sets no cookies.
- Botpress stores its own session identifiers in the browser's session storage.
- The site's own stored items are language, Starla voice speed, emoji preferences, anonymous usage counts, and the open chat (session only).

**Footer card "Ecosystem & Community"**
- Removed from the markup, in English and Arabic: the "Careers & Job Board's" heading, its container, the "Also explore job opportunities on…" sentence and its links (5 per language).
- Removed the style rules and the animation used only by that block (10 lines).
- Added the Star Animation as a decorative layer behind the content. I used the copy already embedded in the page; it is byte-identical to your uploaded `Star_Animation.svg`. The asset has no background shape and no script (55 self-contained animations), so nothing had to be stripped. No corrected SVG is needed.
- The layer is 20% opaque, cannot be clicked or focused, is hidden from screen readers, mirrors in Arabic, pauses off-screen and is static under reduced motion. Its IDs use their own prefix, so it doesn't collide with the About and Careers instances.
- One spacing rule: the two remaining blocks sit at the top and bottom of the card instead of being spread three ways. On desktop the card stays square (the grid requires it); on mobile it is 94 px shorter.

Nothing else changed. A line-by-line comparison with the backup shows edits only in those three areas (26 lines removed, 46 added).

## 6. Publishing (your action)

**Repository:** `waseemasghar12/talent-star`, branch `main`. GitHub Pages serves `index.html` from it (the published file is byte-identical to `index.html` on `main`). There is no build workflow.

I prepared the commit, but this session has no write access to the repository: both the Git push and the GitHub API were refused. Nothing was committed.

To publish:
1. Download `index.html` from this delivery (it is the same file as `talent-star.html`).
2. On github.com open the repository → **Add file → Upload files** → drop `index.html` → **Commit changes** (to `main`).
3. Wait a minute or two, open `https://waseemasghar12.github.io/talent-star/` and hard-refresh.
4. Open Starla, switch to AI, type `!gemini test`. Expect "Relay connectivity: ✓ passed" and "Gemini upstream: ✓ passed". Then try it on your phone.

Optional: also upload `starla-gemini-relay.js` and `wrangler.jsonc` into a `relay` folder so the repository holds a copy. Alternatively, reconnect GitHub for Claude in claude.ai settings and I can push for you.

## 7. Test report

**Runtime-tested**

| Area | Result |
|---|---|
| Relay v99, direct, real Gemini | Health, ping, origin rules (GitHub Pages ✔, talent-star.com ✔, file-opened ✘, others ✘), self-test 4/4, general chat answered, **real image read correctly** ("ZX-41", "Tuesday", "Blue Falcon" from a synthetic picture). Google quota errors returned as safe codes. |
| Relay code, mocked model | 28 / 28, including stall-then-retry, 503-then-retry, invalid key, bad requests, no key in URLs. |
| Page under the real GitHub Pages origin → live relay → real Gemini (test browser) | 17 checks passed. One further check used a question that Starla's own rules answered locally (existing behaviour), so I repeated it with other questions, which Gemini answered. Covered: status and test commands; company question answered locally with no relay call; general questions in English and Arabic answered by Gemini; image analysed and follow-up answered; previews cleared at chat end; iPhone 13 and Pixel 7 profiles; Bot mode mounts Botpress. |
| Page offline (footer, policy, status, regression) | 46 / 46: footer on desktop, tablet, mobile, Arabic and reduced motion; policy points and Arabic; status honesty with relay unreachable and reachable; late answer after a fresh chat never appears; forms, Sheet endpoints, field names, Botpress and contact links identical to the backup; all routes load without errors. |

**Verified by source inspection**
- No API key, placeholder or archive reference in the HTML or relay source.
- Careers upload and Drive folder mapping code untouched (the comparison shows no change there).
- Voice features untouched.

**Not tested or blocked**
- The **published** site with the new file — blocked until you upload it.
- Real iOS Safari and Android Chrome — phone results are Chromium with phone profiles. Please run step 4 above on your phone.
- App wrappers — no build available. A wrapper with its own origin would need that origin added to `ALLOWED_ORIGINS`.
- Text-file analysis against real Gemini — the one attempt hit Google's quota; it passes with the mocked model.
- Real form submissions and Careers uploads — not sent, by standing instruction.
- GitHub Pages settings page — the API for it is blocked here; inferred from the published file.

## 8. Privacy: still open for you or a legal reviewer

1. **Chat-end deletion.** True for the browser and for the relay (it stores nothing). Not controlled by Talent Star at Google: on the free tier Google may keep and use content to improve its products. The notice and point 9 point to Google's terms, but enabling billing is what actually changes this.
2. **Transfer arrangements (point 7).** The paragraph says Talent Star is responsible for putting required arrangements in place. I found no evidence that any exist: no confirmed data-processing agreement with Google (account type unknown), Botpress (plan unknown; its public agreement says data is processed in the United States) or Cloudflare. Accurate wording does not replace them.
3. **Point 6 Retention** still shows the placeholders "[6 to 12 months]" and "[24 months]". Outside this task; unchanged.
4. **Website Terms §5** still calls Starla "a scripted guidance tool". Outside this task; unchanged.
5. **Arabic** points 1, 2, 5, 7 are my translations of legal text and need review by an Arabic-speaking legal reviewer.

I am not a lawyer; items 1–3 should be settled before you rely on the policy.

## 9. Files in this delivery

- `index.html` and `talent-star.html` — the same updated page under both names.
- `starla-gemini-relay.js` — relay source, v99, matching the live Worker.
- `wrangler.jsonc` — configuration matching the live Worker (reference only).
- `STARLA-STATUS.md` — this document.

Sources: [Gemini 3.8 Flash stalls and 503s from Cloudflare Workers (Google AI forum)](https://discuss.ai.google.dev/t/investigation-of-api-instability-with-gemini-3-8-flash-intermittent-http-503-errors-and-transport-timeouts/196264) · [gemini-flash-latest resolves to an overloaded model](https://github.com/trypsynth/backtalk/issues/298) · [Botpress Data Processing Agreement](https://botpress.com/legal/data-processing-agreement) · [Gemini API Additional Terms of Service](https://ai.google.dev/gemini-api/terms)
