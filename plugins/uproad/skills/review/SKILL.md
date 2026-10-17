---
description: Read the unresolved client comments on an Uproad design, apply the changes, show them to you locally before anything reaches the client, then push the new version and mark the ones you actually handled as resolved. Use when a client has left feedback or 赤入れ on a shared prototype and it needs folding back in.
---

# Close one review round

The client left comments on a link you sent. This skill turns that into a new version they can look at, and leaves an honest record of what was and was not done.

**There is one hard stop**: the fixes are shown to you locally before they replace what the client sees. The link is already in their inbox, so a wrong fix reaches them immediately.

## 1. Read what came back

1. `list_designs` — find the design if the user did not name one. Match on title or `external_key`. Every row carries its `workspace`; the list spans every workspace the connection can reach, so if the same title or key turns up in two workspaces, **ask which one** rather than picking. Everything after this is keyed by `design_id`, which already pins the workspace.
2. `list_comments` with `unresolved_only: true`. Each item is one thread, newest first: `body` is the message that opened it and `replies` holds the rest, oldest first.

If nothing is unresolved, say so and stop. Do not invent work.

Read all of them before touching anything, replies included. Clients contradict themselves across comments, and the second one usually wins — better to notice that now than to implement both. Inside a thread the same applies: the last word often revises the first, and a `team` reply may already have agreed a direction with the client.

`author` is `client` (someone reviewing through the share link) or `team` (a member of the workspace). Names are not included. A `team` reply is your own side answering — treat it as something already agreed, not as a new request.

Sort each comment into one of three piles, and say which pile you put things in:

- **Change requests** — do them
- **Questions** — the client is asking, not instructing. Answer one only when the answer is already settled in the spec, the prototype or an earlier `team` reply; draft that answer for step 3. When the answer needs a decision you should not make alone (budget, scope, dates), do not reply — hand the question to the user
- **Out of scope or contradictory** — anything that needs a decision you should not make alone (budget, scope, something that breaks a stated requirement). Surface it, do not quietly implement or quietly drop it

### Find what each comment points at

A misread pin produces a confidently wrong fix, so locate every pinned comment in the HTML before changing anything. `pin` says where the client clicked; it is `null` for a comment about the whole page.

- **`pin.element.selector`** — the CSS selector of the element they clicked. Look it up in the HTML from `get_design_html` (step 2). Selectors are positional (`nth-of-type`), so they can drift when the markup changes.
- **`pin.element.text`** — that element's visible text (up to 120 characters). Use it to confirm the selector landed on the right element, and search the HTML for it when the selector no longer matches. When the two disagree, trust the text.
- **`pin.element.offset`** — where inside the element they clicked, from 0 to 1 (`x` 0 = left edge, `y` 0 = top edge). It matters for large elements: a card, an image, a table.
- **`pin.screen`** — which screen of a multi-screen prototype it was left on: `container` is the screen's container selector, `location` the hash (`#pricing`). Change the element on that screen, not one with the same structure on another screen.
- **`pin.page`** — the position as a 0–1 ratio of the whole page. Rough; use it only when `element` is `null`.
- **`pin.viewport`** — the reviewer's window size. Around 390 wide means they were on a phone: look at the layout at that width before deciding what is wrong, because the same element can be fine on a desktop.
- **`version` / `on_current_version`** — `false` means it was left on an older version. Check whether the current one already deals with it before changing anything, and say which version it came from.

`pin_x` / `pin_y` are older copies of `pin.page`; ignore them.

When a pin still cannot be placed, or could mean two different elements, say which one you think it refers to and why, and ask before changing it.

## 2. Make the changes

1. `get_design_html` — always start from what is actually deployed, not from whatever is on disk. Someone else may have pushed since. Write it into `designs/<slug>/index.html`, which is where designs live (one directory each, matching `/uproad:new`). Derive `<slug>` from the design's `external_key` when it has one.
2. Apply the changes. Keep the edits tight; unrelated refactoring makes the next round harder to review.
3. If the feedback changes what the thing *does* rather than how it looks, the spec is now wrong too. Fetch it with `get_doc` (`spec.md`) and update `designs/<slug>/spec.md`. **A spec that silently drifts from the prototype is worse than no spec** — the next engineer trusts it. Do not push it yet; it goes out with the prototype in step 4.

## 3. Show it locally, then STOP

Serve the designs directory and **leave the server running in the background**:

```bash
python3 -m http.server 4173 --directory designs
```

If that port is already serving the same `designs/` root, reuse it rather than starting a second one. Otherwise pick a free port.

Give the user the direct URL (`http://localhost:4173/<slug>/`) and **stop, asking them to check the fixes.** Point out specifically what changed, so they know where to look.

Show the reply drafts (see "Write the replies and the change note" below) in the same message, so the user approves the words the client will read along with the fix.

**Do not push until they approve.** The next step replaces what the client is currently looking at behind a link that is already in their inbox — a wrong fix arrives faster than a right one. Apply what comes back and show them again, staying in this loop until they say it is good.

## 4. Push, reply, and mark resolved

Do these in this order. The replies come after the push so that a failed push never leaves a reply saying something was fixed.

1. `push_design` with the **same `design_id`**, and a `note` (see below). This adds a version; the share link the client already has keeps working and now shows the new one.
2. `push_doc` for any document you changed (the spec is at `spec.md`).
3. `reply_comment` on every thread you are answering — the ones you addressed, the ones you did not, and the questions you could answer. Use the approved drafts. No notification is sent for a reply; the client reads it the next time they open the link.
4. `resolve_comment` — **only for comments you actually addressed.** Leave questions and out-of-scope items unresolved. Resolving something you did not do erases the client's request without them knowing.

### Write the replies and the change note

The client reads both. The note goes into the 「対応済み」 email to the people who left comments. Write them in Japanese, plainly, as the team.

**Reply** (`reply_comment`, 1–3 sentences):

- Addressed → say what changed, naming the element the way the client sees it: 「『申請を開始する』ボタンの色を濃くしました。」
- Not addressed → say why in one sentence, and leave the thread open: 「予算に関わるため、こちらで確認してからお返事します。」
- A question you could answer → answer it directly.
- Do not write version numbers, selectors, the word 「AI」, or internal reasons. Do not write a reply for something you only plan to do.

**Note** (`note` on `push_design`): one line saying what changed, like a commit message: 「料金表を3プランから2プランに変更」. It is shown in the version list and in the email, so keep it short and leave out version numbers. Optional on the server, but always set it here.

## 5. Report

In Japanese:

- **対応した** — comment → what changed, one line each. Name the element the way the client sees it (its text, e.g. 「申請を開始する」ボタン), not by its selector
- **返信した** — which threads got a reply, one line each. Say that replies send no notification
- **確認したい** — the questions you did not answer, quoted, so the user can answer them. Say plainly that these are still open in Uproad and need a reply from the user
- **対応していない** — with the reason. Say it out loud rather than leaving it to be discovered
- The share link is unchanged — say so. Clients worry that a new version means a new URL

If nothing needs the client's input, say the round is closed.

## When things fail

- **404 on the design** — the connection cannot reach the workspace it lives in (not a member, or the reach was narrowed on the consent screen when signing in). `list_workspaces` shows what it can see; to widen it, revoke the connection under Settings → Personal tokens and sign in again with `/mcp`.
- **`reply_comment` fails** — the thread may belong to another design or have been deleted. Re-run `list_comments`, and tell the user which replies were not posted rather than skipping them silently.
- **A comment will not resolve** — it may already be resolved, or belong to another design. Re-run `list_comments` and check.
- **Port already in use** — another design's server may already be serving the same `designs/` root; reuse it. Otherwise pick a free port.
- **The prototype has moved on since the comment** — a comment pinned to an element that no longer exists is still real feedback. Search for `pin.element.text` first; if it is gone too, say which version it was left against (`version`) and ask rather than guessing.
