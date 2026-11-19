# Campus Stores

Campus Stores is a work sample set at a fictional campus media equipment desk. You start on a Friday afternoon with a club collecting a screening kit and you finish on Saturday morning.

The change I made is in the Desk book: the handover note is now drafted from the loan register instead of being typed from memory at the end of the shift.

## Running it

Open `index.html` in a browser. If you would rather serve the folder:

```sh
python3 -m http.server 8000 --bind 127.0.0.1
```

Then go to `http://localhost:8000`. There is no build step, no dependencies and nothing loads from the network. Progress is kept in the browser, and `Restart sample shift` puts the sample records back.

## A short route through it

1. **Requests** → Courtyard screening → **View form**. The approved kit is a projector, a speaker and a USB-C to HDMI adapter, because the laptop only has USB-C ports.
2. **Loan register** → **New issue slip**, three times, all three to Kabir Sen, due `Fri 8 pm`. That is how the desk signs a kit out today: one form per unit.
3. **Move to return time.** Record the projector and the speaker back in. Leave the adapter on the register, because Leela still has it.
4. **Desk book.** The card at the top is the new handover draft. Press **Use draft**, correct the name, save it.
5. **Move to next morning** and open the Desk book again. The draft is now the morning shift's list. Return the adapter from the Loan register and the draft closes itself out.

At 7:30 pm the draft reads:

```
Friday · 7:30 pm · handover draft
Back this shift: L-402 PR-04 HDMI projector; L-403 SP-02 Portable speaker.
Still out (2):
- MI-09 Wireless microphone · L-401 · Tara Shah · due Sat 10 am · RQ-209 Debate Union
- AD-15 USB-C → HDMI adapter · L-404 · Kabir Sen · due Sat 9 am · RQ-214 Film Society
Not issued yet: RQ-217 Photo Circle · 1 item · due Sat noon.
Record check: 2 closed this shift, 2 open.
```

## A moment the app handles well

At 7:30 Kabir comes back with two of the three items. The register keeps one slip per physical unit, so recording the return closes `L-402` and `L-403` and leaves `L-404` exactly as it was. Nothing has to pretend the whole request came back.

I kept that. Folding the register down to one row per request would read better and would lose the record of which unit is out and who signed for it, which is the thing the desk needs when the adapter does not come back.

## A moment I would change

Same return, but in the Desk book. The app is precise about what is out and then hands you an empty textarea. So the person closing the desk types it out from a screen one tab away.

Everything the note needs is already in the register: unit, slip number, collector, due time, request. Nothing is missing except the draft.

What gets written at 7:30 with a queue at the window is "adapter still out, Leela brings it tomorrow". True, and it drops `L-404`, `AD-15`, `RQ-214` and `Sat 9 am`. The morning shift is then left working out which of two identical adapters is the one still out.

## What I built

The Desk book opens with a draft assembled from the register (`handoverDraft()` and the `.draftcard` block in `app.js`, styles in `styles.css`):

- **Back this shift**: the slips closed during the current phase. Returns now stamp `at` so this is a fact rather than a guess.
- **Still out**: every open slip with unit, slip number, collector, due time, request and club.
- **Not issued yet**: approved requests with no slip at all, so the morning shift can see the noon camera collection instead of finding out about it.
- **Record check**: how many slips closed today and how many are still open. One line, so the desk does not have to count rows.

It is a draft, not a note. **Use draft** puts the text in the note field and the desk edits it and saves it as their own. Saved notes carry `drafting: true`, so drafted notes can be told apart from typed ones later.

### Why there is no model call

The register already has the facts and the only generative part would be the wording. At a lending desk a wrong due time is a student losing borrowing privileges, so I would rather the draft be deterministic, instant and auditable. If the desk's free text messages need to feed into the note later, that is the point to add extraction, with this version as the baseline to measure against.

### What I deliberately did not change

Issuing one approved kit still takes three slips. That is the most annoying part of the pickup, and I left it because it is a separate question: does a request own a kit, or does a slip own a unit? Changing both in the same pass would make it impossible to tell which change moved the numbers.

I also did not prefill the note automatically. The draft sits above the field and has to be accepted, because an unreviewed machine sentence should not look like the desk's own handover.

## Where the draft is wrong

This is the part worth writing down.

**A slip names the collector, not the holder.** `L-404` says Kabir Sen because Kabir signed for the kit, but Leela Das is the one bringing the adapter back. The register cannot know that, so the card says to check the names and the desk fixes it in step 4.

The rest of the cases I would turn into a small test set before calling this shippable:

- **Item back, slip still open.** The draft says the adapter is still out when it is on the shelf. That blocks an over-issue rather than causing one, but it is wrong.
- **Two identical adapters.** `AD-12` and `AD-15` look the same on the counter. The draft prints the tag, so the desk checks the sticker instead of the shape.
- **Late return.** An item due `Sat 9 am` and returned at 9:04 reads the same as one returned at 8:30. There is no lateness field yet.
- **Return before the current phase.** The draft trusts `at` on the slip. Any return recorded by an older version of the data has no stamp and falls into "still out" until it is closed again.

On success: a saved handover note that names every open slip, written without the desk opening the Loan register. It is checkable here, because the draft knows the open set, so the note has to contain all of it, and the edits between draft and saved note say how much work the desk had to do. If they only fix names, the draft is carrying the facts.

## The one thing I would watch at a real counter

One Friday close, with a queue at the window and a student waiting to hand something back. I would time it from opening the Desk book to saving the note, and count what the desk changes in between.

If they keep the structure and only correct names, then the draft is doing the work and should be extended to the morning shift too. If they clear the draft and write their own short sentence, the facts are right and the wording is wrong, so the fix is the phrasing, not more data. If they never open the Desk book at all, the handover is happening somewhere else and the empty textarea was never the problem.

## Source

- `index.html`: layout, the candidate brief, bundled icons.
- `app.js`: sample records and behaviour. The draft is `handoverDraft()` and the card at the top of `deskbook()`.
- `styles.css`: layout and the draft card styling, using the existing memo colours.

Icon licence notices are in [third-party/lucide-LICENSE.txt](third-party/lucide-LICENSE.txt).
