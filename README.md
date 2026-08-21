# Bill Upload — Accounts Payable

The bill upload flow: **Accounts Payable → All Bills · Needs Review · Uploads**, the
bulk upload dialog, and the create-bill screen it hands off to.

Built from Karbon — AI Accountant, Figma file `9sL5Vrc6a3NzoLve2o9zgr`, frames
`23917:5544` (Sales list), `23917:5758` (Bulk Upload Bills) and `23917:6009` (Accounts
Payable / Needs Review). Tokens are the file's own variables; the PDF glyph, the
dropzone glyph, minimize-2 and the shopping-bag are the exported assets, inlined into
the SVG sprite at their own viewBoxes. The topbar lockup is the supplied white PNG, inlined as a data URI (source kept at `assets/logo.png`, not loaded from there). Token *names* are carried from the Accounts
Payable and Sync Queue prototypes so the three stay one system.

Everything lives in `index.html` — no build step, no dependencies.

```
open index.html
```

## Try it

Open it and you land on **Needs Review** with nothing in it — that is the first thing
this prototype is about. Press **Upload Bills**, then **Load a sample batch of 7 bills**.
Seven files run through the pipeline: two land in Needs Review, four fail for four
different reasons, and `month-end-batch.pdf` stops to ask where its bills start. Close the dialog. Refresh the page. The failures are still there.

Each failure states its reason on one line and opens on the chevron for the fix and
the controls. The file waiting on a split stays open — it is the decision the screen
exists to put in front of you.

The prototype console, bottom left, runs the batch, adds a multi-bill PDF or a single
failed file, or resets to a fresh workspace.

> **The upload and extraction are simulated.** No file leaves your machine and no OCR
> runs. Files you pick yourself are checked for real on format and size; everything else
> follows a script so that every state is reachable in a demo. Upload state is kept in
> this browser's `localStorage` under `aia.billupload.v1`.

## What it fixes

Five problems in the flow as it stands, plus the Bill Splitter from its PRD.

### 1. The empty Needs Review tab explains the product

It used to say *"Bills on the way"* — a status message standing in for an explanation,
and a wrong one when nothing had been uploaded. It is now a numbered four-step guide in
the order the work actually happens: **upload → AI reads → review and correct → sync to
Tally**. Step 1 carries the live **Upload Bills** button; step 3 is marked as the tab
you are looking at.

The guide does not disappear the moment an upload starts. While files are in flight the
same four steps stay, the lede changes to say how many are being read, and step 2 takes
the highlight — so the tab never goes blank at the exact moment a new user is waiting to
see what happens next.

### 2. Every failure carries its reason and the action that helps

A `Retry` button on a password-protected PDF is a dead end: retrying cannot succeed, and
offering it is what made the old screen unhelpful. Each failure now names what went
wrong, what to do about it, and offers only the actions that can actually resolve it.

| Reason | What we say | Action offered |
|---|---|---|
| Password-protected PDF | Remove the password and upload again | Upload the unlocked file · Remove |
| Over the 500 MB limit | Split it, or upload each bill separately | Upload a different file · Remove |
| Unsupported file type | We read PDF, JPG and PNG | Upload a different file · Remove |
| Scan too blurred | Re-scan at 300 DPI, or enter it by hand | Enter this bill by hand · Remove |
| Duplicate | Names the voucher it duplicates | View PUR/26-27/181 · Remove |
| Connection dropped | Nothing was saved; only this file is re-sent | **Retry upload** · Remove |
| Extraction timed out | The file is safe | **Retry upload** · Remove |
| Stopped when the page closed | This file was still transferring | **Retry upload** · Remove |

Only the last three are retryable, and only those three show a Retry.

*Enter this bill by hand* is not a dead end either — it opens Create Bill with the
unreadable file attached and a note saying nothing was extracted from it.

### 3. Failed uploads persist

Uploads are written to `localStorage` as they change state, so closing the dialog,
navigating away or refreshing the page loses nothing. Three places keep a failure
visible:

- a **strip above the tabs**, present anywhere in Accounts Payable, naming the files and
  offering *See what went wrong* (dismissible, and it comes back for new failures)
- a **red count on the Uploads tab**, alongside the total
- the **Uploads tab itself**, with a Failed status filter, failures sorted to the
  top, and the reason on the row

Anything left mid-transfer when the page closed reopens as *"Upload stopped when the
page was closed"* — retryable — rather than as a row frozen at 47%.

The dialog says so before you close it: *"4 failed files stay in Uploads — closing
this or refreshing the page will not lose them."*

### 4. Attachment-only bills are labelled, and it is a choice

Attaching a document in Create Bill used to be one ambiguous upload button that quietly
did one of two very different things. It is now two named routes:

- **Read this bill** — we extract vendor, GSTIN, dates, amounts and tax, and fill the form
- **Attach only** — the file is stored as evidence, nothing is read, every field is yours

Whichever you pick, the document pane carries a standing note saying which it was, with
a one-click switch to the other. Saved bills carry the distinction into **All Bills** as
a tag in the Bill File column — *Attachment only* or *Read* — so nobody assumes figures
were checked against a document that was never opened.

Dropping a file straight onto the pane attaches it without reading, and says so: a drop
is not a statement about whether you want extraction.

### 5. Sub-page errors land on the field

An error summary at the top of a form with three sub-pages tells you a problem exists
and not where. Validation now shows in four places at once:

- **the field** — red border, and the message underneath it
- **the sub-page tab** — a count badge, so an error on a sub-page you are not looking at
  is still visible
- **the summary** — how many, grouped by sub-page, each group a button
- **on jump** — the summary and the Save button both move you to the first unresolved
  field on that sub-page, focus it, and flash it

Counts fall as you type, not only when you press Save again.

### 6. Bill Splitter

Implements `ai-accountant-prds.vercel.app/bill-pdf-split/`. Suppliers send several
invoices in one PDF at month-end, and a bill is capped at 12 pages so extraction stays
reliable — so a multi-invoice PDF either failed the page limit or posted as one voucher
with mixed details.

**Where it sits.** A check step now runs between upload and extraction: *uploading →
checking pages → reading*. Most files pass straight through and never see the dialog. A
PDF that looks like several bills stops at **Needs split review** and behaves exactly
like a failure in every way that matters — the reason is on the row, it survives close
and refresh, it has its own strip, its own tab count and its own status filter. Amber
rather than red: nothing is broken, a decision is waiting.

**The dialog.** 920px — wider than the 742 upload dialog because it holds page previews,
identical in every other respect.

- Pages are grouped under the bill they belong to, each group showing its range, its
  supplier and invoice number, and *Remove split* to merge it upwards
- Each page carries *Start a new bill here* and *Exclude page* / *Restore page*
- The right rail, **Bills to create**, is a read-only summary that scrolls to a bill —
  there is one place to change a boundary, not two
- The blank separator page arrives **excluded automatically**, clearly labelled as ours
  to have done, and restorable
- An **uncertain boundary** asks its question inside the bill it affects — *It starts a
  new bill* / *It belongs to this bill* — and blocks Confirm until answered
- A bill over the 12-page limit says it will be created as ordered parts of the same
  bill, with the page counts, rather than blocking

**On confirm** each bill enters as its own upload — `month-end-batch · Bill 2 of 3.pdf`,
carrying its page range and its source — and runs the normal extraction into Needs
Review. The parent stays in Uploads marked *Split into 3 bills*. **Cancel creates
nothing** and leaves the file waiting. Nothing here approves anything.

If detection fails, that is an ordinary failure with a readable reason and a *Retry
upload* that re-runs detection.

The sample batch carries a 14-page `month-end-batch.pdf` that exercises all of it: two
confident bills, a blank separator auto-excluded, an annexure on page 6 flagged
uncertain, and a third bill of seven pages. Remove enough splits and you push it past
the page limit and see the parts rule fire.

## What came from the Figma

Where the design file and the current build disagreed, the Figma won.

| | Was | Now |
|---|---|---|
| Page title | Purchases | **Accounts Payable** |
| Third tab | Bill Uploads | **Uploads** |
| Header actions | Sync · Create Bill · Upload Bills | **Sync · Upload Bills (split) · ⋮** — Create Bill moved into the split menu, alongside *Retry all failed uploads* |
| Company chip | white pill on the chrome | dark `#303850` chip, teal badge |
| Current nav item | indigo text, semibold | `#f6f7ff` ground, black text — indigo is spent on actions, not navigation |
| Page title type | 24px semibold | 24/32 **regular** (H4) |
| Tabs | one rule under the strip | a rule per tab, 1px resting / 2px indigo active, 30px side padding |
| Table | sort arrows, "View" | **filter funnels, "Columns", Sync status, Action** — the Sales table pattern |
| List rows | 34px tile, 14px name | **40px tile, 16/24 name, 14/20 detail, 25/19 padding, `#e5e7eb` border** |
| Progress | 4px bar, grey caption | **8px bar on `#e5e7eb`, indigo fill, 10px indigo percentage** |
| Dialog | 820px, generic | **742px, `#bac3f4` border, `#fbfbfe` header, 164px dashed dropzone, minimise control, Cancel All** |
| Dropzone copy | "up to 500 MB per file / PDF/PNG/JPG/JPEG" | **"Upload up to 100 Bills at once / Supported formats: PDF/JPG/PNG"** |
| File icon | generic document | the exported **PDF glyph** |
| Sidebar | 9 items incl. Tax & Compliance | the Figma's **8**, in its order |

Two components in the Figma were adopted for behaviour, not just for looks:

- **The minimise control** in the dialog header is what the "uploads keep running in the
  background" behaviour needed — it closes the dialog without stopping the run, and the
  strip on the page takes over reporting.
- **Cancel All** now has a real outcome: cancelled files stay in the list saying they
  were cancelled and offering to start again, rather than vanishing.

The Figma's pre-queued **Sample file** card is how the sample batch is presented — six
bills, badged, running the full pipeline.

## The visual direction — "Ledger"

The tokens are the Figma's and are not negotiable, so the character had to come
from composition, density and scale instead. It is taken from the audience's own
material world rather than from other web apps: **Tally and Indian ledger
stationery are ruled, not carded** — hairline rules, a fixed column rhythm, a
quantity column down the left, tabular figures, and the key map printed where
you can see it.

**The signature is the pages gutter.** Every document list in the module leads
with its page count set large in tabular numerals. It is real data, never an
ornamental 01/02/03: it unifies the review queue, the upload log and the bills
inside a split, and it is what makes the splitter legible at a glance, because a
14-page file now *looks* different from a 2-page one.

What changed, and why:

| | Was | Now |
|---|---|---|
| Lists | rounded bordered cards, tinted icon tiles, ~86px a row | hairline-ruled rows on a fixed three-column rhythm, ~52px — **ten visible rows on the target screen instead of six** |
| Status | a coloured pill on every row | a tracked capital and a glyph; colour never carries it alone |
| Failures | a full red card ground | white ground, a red rule, a red figure in the quantity column — four failures used to paint half the screen |
| Attention | two stacked tinted strips, 130px | one ruled bar of claims, ~40px — that is two queue rows bought back |
| Empty state | a four-card grid with numbered circles | a ruled four-column sequence; the live step marked by an indigo rule |
| Upload dialog | its own card language | the same ruled list as the tab, carrying the Figma card's own values |
| Invalid fields | a pink wash per field, so the card read pink | red border, red inset rule, message — the wash removed |
| Page proxies | grey bars in a box | drawn as invoice stationery — masthead, ruled table, totals rule |
| Page actions | two stacked underlined links per card | two 26px icon buttons, always visible, never hover-only |
| Shortcuts | none | a **persistent legend** at the foot, and the keys are real: `U` `N` `1` `2` `3` `/` `Esc` |
| Table at 1280 | Total Amount clipped behind a scrollbar | Creation Date drops below 1400px so the decision column always fits |

Every type size snaps to the Figma's scale (10·11·12·13·14·16·20·24 — plus the
two steps proposed below) and every spacing value to 4·8·12·16·24.

### Revision — 20 Aug 2026: the direction, executed at full volume

The Ledger direction above was right and was being rendered at about 60% of its
own volume, which reads as an admin template with a quirk rather than as a
design. This pass turned it up. Scope was the **Uploads tab and the chrome it
shares** — page head, tabs, toolbar, attention strip, buttons and fields — so
All Bills, Needs Review and the dialogs inherited the same fixes. No new hue,
no restructured flow.

| | Was | Now |
|---|---|---|
| Ground | one flat white sheet for chrome and content alike | two planes: nav, head and tabs on white, every document list raised onto its own plane over `--bg-primary`. The attention strip and the tab rule used to be two identical rule-bands 40px apart, with no way to tell furniture from work |
| The pages gutter | 24px in a 54px column | **40px in a 64px column** — 2.9× the filename, which is where a signature starts. It was a number that had come loose; it is now the spine of the list |
| Failures | every one opened by default, ~180px a row | the reason sits on the resting line in ordinary ink; the fix opens on a disclosure. **62px a row, six of a seven-file batch on screen against three** |
| Red | figure, `FAILED` capital, a bold red sentence and a red hairline — four jobs a row | two: the figure and the status capital. A fix is instructions, not an alarm |
| Page name | 24px/400 — lighter than a row's own heading | 28px/600, tracked in. The screen's title no longer loses to one of its rows |
| Row actions | solid, outline and a red underlined `Remove` link | one button language at 32px; `Remove` is a real bordered control that reddens on hover |
| Fields | 34px against 36px buttons | one 36px height scale, inset shadow, indigo focus ring |
| `Sample file` badge | an indigo chip on all thirteen rows | muted micro-caps, set to be ignored |
| File count | two bold coloured numbers | tracked capitals with rules between — a caption, not a second alarm |
| Motion | none | 40ms staggered row entrance capped at eight, disclosure and strip fades, transform/opacity only |
| Console vs legend | the console sat on top of the shortcut legend | cleared to `bottom:52px` |

**Elevation reverses an earlier call.** The original note said elevation should
be near-zero because ledger paper has no shadows. That is true of paper and
false of a screen: with nothing raised, the strip demanding a decision looked
exactly like the tab rule below it. There are now exactly two levels — `--e1`
for a content plane, `--e2` for the one bar that overrides it — both tinted
with `--chrome` rather than black so they stay warm-neutral. Nothing gets a
third.

### Extension proposal — six tokens, for you to accept or reject

Six values this module needs that the sampled set does not carry. They sit in a
labelled block at the top of `:root` and nothing else depends on them, so the
block can be deleted wholesale. **Not one new hue** — every value is a step on a
scale the file already has, or `--chrome` at low alpha.

| Token | Value | Why |
|---|---|---|
| `--t-title` | `28px` | the page name. 1.17 off 24, matching the file's existing 12→14→16→20→24 rhythm |
| `--t-display` | `40px` | the quantity column. The one place the design spends scale |
| `--e1` | `0 1px 2px rgba(33,38,60,.05), 0 1px 1px rgba(33,38,60,.04)` | a content plane |
| `--e2` | `0 6px 16px rgba(33,38,60,.08), 0 1px 3px rgba(33,38,60,.05)` | the attention strip, above it |
| `--radius-plane` | `10px` | a full-width plane; the 6px control radius nests inside it correctly |
| `--ease` / `--dur` | `cubic-bezier(.2,.7,.3,1)` / `.18s` | one easing pair for the module |

If they land in the Figma variables, the Accounts Payable and Sync Queue
prototypes should pick them up too, so the three stay one system.

**Two conflicts worth settling, flagged rather than silently resolved:**

1. `Platform constraints.md` sets an absolute **12px floor** for UI text. The
   Figma's own `Caption/Caption-1` token is **10px**, and documented components
   use it — the "Sample file" badge, the upload percentage. I followed the
   constraint for anything that is a sentence to read, and the Figma for badges,
   counts, unit markers and labels inside artwork. Worth a decision.
2. The Figma documents the upload dialog's file as a **card**. With seven files
   in the dialog that language fought the queue behind it, so the dialog uses the
   ruled list and carries the card's type, colour and progress values inside it.
   A deliberate deviation, not an oversight.

## Notes

- Fluid from 1440 down to 320; the table clears all nine columns without horizontal
  scroll at 1440 and scrolls inside its own container below that. Indian digit grouping,
  `DD-MM-YYYY` dates, tabular numerals on every figure.
  **The reference screenshots show `9 Jul 2026`; `Product/Platform constraints.md` says
  `DD-MM-YYYY`, and this prototype follows the constraint.**
- Tabs and sub-tabs are arrow-key navigable, Esc closes the dialog, every failure action
  is a real button. No hover-only affordances.
- Status is never carried by colour alone — every pill and tag pairs its colour with an
  icon and a word.
- The failure catalogue is the `FAILURES` object near the top of the `<script>` block,
  and the scripted batch is `BATCH` beside it. Both are meant to be edited.
