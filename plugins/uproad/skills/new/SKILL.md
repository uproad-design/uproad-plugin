---
description: Interview the user about a client brief until the requirements are settled, write a spec, build a working HTML prototype, review it locally with them, then ask whether it goes out as a public share link or stays private to the workspace, and push to Uproad. Use when someone wants to build a proposal, mock-up, prototype, or 提案・たたき台 for a client.
---

# Take a brief to a client-ready link

Four phases with **two hard stops**. Do not run them together. The stops exist because a spec the user has not corrected produces a prototype built on the wrong assumptions, and a prototype they have not seen should not reach a client.

```
1. 要件を詰める (interview)
2. Spec を書く ────────────── STOP: 人間が直す
3. プロトタイプを作る → ローカルで見せる ── STOP: 人間がレビューする
4. 置き場所を確かめる → 公開/非公開を選ぶ → Uproad に push → 共有リンク、または非公開のまま
```

Work in Japanese with the user throughout.

---

## Phase 1 — Interview until the requirements are settled

**Interview relentlessly. Walk down each branch of the decision tree, resolving dependencies between decisions one by one.** Do not accept a thin brief and start building.

- **One question at a time.** Wait for the answer before asking the next.
- **Always give your recommended answer**, with the reason. A question without a recommendation makes the user do your thinking.
- **If a question can be answered by looking, look instead of asking.** Read the notes they pointed at, existing designs in the repo, `list_designs`, the current site. Never ask for something you can find.
- **Order questions by what they unlock.** Ask the decision that changes the most downstream choices first — who the client is and what they will decide from this, before colour and copy.

Stop when the answers no longer change what you would build. That is the shared understanding you are after — not an exhausted user. If a branch genuinely does not matter yet, say so and move on rather than asking about it.

Typical branches worth walking: 誰が見て何を判断するのか / 対象ユーザーと利用場面 / 必要な画面と遷移 / 各画面で判断に必要な情報 / 既存デザインやトンマナの制約 / 今回スコープに入れないもの.

## Phase 2 — Write the spec, then STOP

Create the design directory and write the spec:

```
designs/<slug>/spec.md
```

`<slug>` is short, lowercase, hyphenated, and names the thing — `acme-lp`, `billing-dashboard`. **One directory per design**, so several designs sit side by side without colliding.

Sections, in Japanese:

- **背景・目的** — the problem, and how this gets judged
- **対象ユーザー** — who uses it, in what situation
- **画面と導線** — each screen and how they connect
- **機能要件** — numbered, so an engineer can implement against them
- **確認事項** — everything still assumed or undecided

**確認事項 is what earns the review.** Anything you decided on the client's behalf goes here as a question.

Then **stop and ask the user to read and correct it.** Say plainly that you will not start building until they reply. Apply their corrections and confirm before moving on.

## Phase 3 — Build, serve locally, then STOP

Write the prototype to:

```
designs/<slug>/index.html
```

- **One self-contained file.** Inline CSS and JS. No build step and no local asset files — Uproad serves this file on its own later, so anything it references from disk is gone.
- Avoid external CDNs where you reasonably can. The client may open this on a train.
- Make it look finished. Real copy, plausible data, no `Lorem ipsum` and no `TODO`.
- Responsive unless told otherwise.
- Wire up the interactions the client is being asked to judge (tabs, modals, form states). Fake the data behind them.
- **When the prototype has more than one screen in the one file**, give each screen its own container — `<section data-screen="driver-loading" hidden>` — and switch screens by toggling `hidden`, never by rebuilding the markup with `innerHTML`. Give every screen a heading of its own (`h1`–`h4`). Uproad pins a comment to "this element on this screen"; screens that can be told apart this way keep each pin on the screen it was left on, instead of surfacing on every screen that shares the structure.

Then start a static server **from the `designs/` root** so every design is browsable, and **leave it running in the background**:

```bash
python3 -m http.server 4173 --directory designs
```

Pick another port if 4173 is taken. Give the user the direct URL (`http://localhost:4173/<slug>/`) and **stop, asking them to look at it.**

Apply what they come back with and show them again. **Stay in this loop until they say it is good.** Do not push a prototype the user has not approved — the next step puts it in front of their client.

## Phase 4 — Decide where it lives and who can see it, then push

Only after approval.

### 4a. Find where this design lives

The connection may reach more than one workspace (by default it reaches every workspace the user belongs to; the consent screen can narrow it). Nothing is remembered between sessions — Uproad itself is the record of where a design lives — so look it up:

1. `list_designs` (no arguments). Every row carries its `workspace` and `external_key`. Look for a row whose `external_key` is exactly `designs/<slug>/index.html`.
2. **Exactly one match** → this is a re-push. Its `workspace` is where it goes; nothing to ask. Note its `is_public` for 4b.
3. **No match** → this is a new design. `list_workspaces`. One workspace → use it, say nothing. **More than one → ask once, in Japanese, which workspace it goes to**, with a recommendation: the one whose name matches the client from Phase 1, otherwise the one the user's other designs are in. Wait for the answer.
4. **Two or more matches** (the same key exists in two workspaces) → ask which one. Never pick: pushing to the wrong one puts a new version behind a link some other client already has.

Carry the chosen workspace's `id` into 4c. Passing the id rather than the name avoids the case where two workspaces share a name.

### 4b. Ask 公開 or 非公開 — before anything is pushed

Ask once, in Japanese, and wait for the answer. Give your recommendation with it:

- **公開** — Uproad issues a share link. Anyone holding the URL opens the prototype without an account and can leave pinned comments. This is the point when there is a client on the other end.
- **非公開** — the design and the spec live in Uproad with no share link. Only signed-in members of the workspace can open it. For 社内の下書き, or anything not ready to leave the building.

Recommend 公開 when the user has talked about a client or an outside reviewer, 非公開 when this is theirs alone for now.

Ask **before** `push_design`, not once the link exists. Through these tools publishing only goes one way: `get_share_link` makes a design public and nothing here makes it private again — that is a toggle on the design's page in the app.

One case the answer cannot fix: if 4a found the design and it is **already public** (`is_public`), say so before pushing. The version you are about to push replaces what that existing link shows, and answering 非公開 now does not retract a link the client already has.

### 4c. Push

1. `list_projects` with `workspace` set to the id from 4a — if there is more than one project, pick the one matching the client and say which you picked. One project, or no obvious match: leave `project` off.
2. `push_design` — the contents of `designs/<slug>/index.html`, with `external_key` set to that same path, `workspace` set to the id from 4a, and a one-line Japanese `note`. For a new design write 「初版」 plus a few words on what it is (「初版: 料金ページの3画面」); for a re-push say what changed, as `/uproad:review` does (「料金表を2プランに変更」). Leave version numbers out. **The key is what makes re-running safe**: the same key adds a version to the same design instead of creating a duplicate, so a link already sent to the client keeps working. The workspace is required even on a re-push, because the same key could exist in another workspace too.
3. `push_doc` — the spec, at `spec.md`. It takes `design_id`, so no workspace here.

A design created this way starts 非公開. Nothing is reachable from outside the workspace until 4d publishes it.

### 4d. Hand over

**公開を選んだ場合** — `get_share_link` publishes the design and returns the URL. Safe to call repeatedly.

Report, in Japanese:

- **The share link**, on its own line
- What you built, in two or three lines
- **確認事項** — repeat them; this is what you want the client to respond to
- The client needs no account, and can click anywhere on the prototype to leave a pinned comment

**非公開を選んだ場合** — **do not call `get_share_link`.** Publishing is the whole job of that tool, and there is no undo through the tools.

Report, in Japanese:

- **The design's page in Uproad**, on its own line — `<Uproad URL>/designs/<design_id>`, using the id `push_design` returned. `<Uproad URL>` is `https://uproad.design` unless the plugin is configured against another instance
- Say plainly that this page needs a sign-in to the workspace, so it is not a link to send to a client
- What you built, in two or three lines
- **確認事項** — repeat them, so they can be settled before this goes out
- 公開 later is one toggle on that page, or re-run this skill and answer 公開

Then offer `/uproad:review` for when the comments come back.

## When things fail

- **401** — the connection was revoked (Settings → Personal tokens in Uproad) or never made. Tell the user to run `/mcp`, pick uproad and choose Authenticate; the browser opens Uproad's consent screen. Nothing is stored in a file.
- **`workspace is required`** — the connection reaches more than one workspace and `push_design` was called without `workspace`. The error lists the candidates; go back to 4a rather than guessing.
- **`workspace not found`** — the connection cannot reach that workspace (not a member, or it was narrowed on the consent screen). `list_workspaces` shows what it can reach.
- **`title is required`** — you passed neither `design_id` nor a matching `external_key` and no title. Give a title on first push.
- **`project not found`** — projects are never created implicitly. Run `list_projects` and use an existing name, or create it in the app.
- **quota exceeded** — the workspace hit its storage limit. Free is 200MB.
- **公開してしまったが非公開に戻したい** — the tools cannot take it back. Open the design in the app and turn the 公開 toggle off; the slug is kept, so publishing again later reuses the same URL.
- **Port already in use** — another design's server is probably still running. Reuse it if it is serving the same `designs/` root; otherwise pick a free port.
