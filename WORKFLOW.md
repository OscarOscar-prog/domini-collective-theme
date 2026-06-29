# Domini Collective — Shopify Build & Deploy Workflow

**Purpose:** The practical, corrected process for designing and shipping changes to the Domini Collective theme using Claude Code. This supersedes the earlier "Shopify Build Pipeline" setup notes where the two disagree — see [What changed](#what-changed-from-the-original-setup-notes) at the bottom.

---

## 1. The architecture (as actually wired)

| Component | What it is | This setup |
|---|---|---|
| **Shopify** | The live store + any draft themes | Live theme (published) **and** a draft theme named `domini-collective-theme/dev` |
| **GitHub** | Version-controlled copy of the theme code | `OscarOscar-prog/domini-collective-theme` |
| **Local folder** | A clone where edits actually happen | `/Users/tomrudkin/Project/Shopify /domini-collective-theme` |

**How they talk:**
- Local ⇄ GitHub via **Git** (`pull`, `commit`, `push`).
- GitHub → Shopify via Shopify's **native GitHub integration**, which auto-syncs each connected branch to its matching theme. There is **no manual deploy and no manual "push to theme" step.**

**Branch → theme mapping (the key fact):**

| Branch | Connected Shopify theme | Effect of pushing |
|---|---|---|
| `main` | **Live, published** theme | **Goes live.** Merging into `main` *is* the publish. |
| `dev` | Draft theme `domini-collective-theme/dev` (unpublished) | Safe. Updates the draft only — live site untouched. |

> The original setup notes called the working branch "staging." In reality it is **`dev`**, and it has its own draft theme. Treat `dev` as the staging branch.

---

## 2. What Claude Code can / can't do here

**Can:** read & edit any theme file; search the whole codebase; run Git (`status`/`diff`/`log`/`add`/`commit`/`push`, `checkout -b`); make coordinated multi-file changes; explain a diff in plain language before committing.

**Must NOT do unsupervised:**
- **Merge `dev` → `main`.** That merge is the live publish — the human review before it is the *only* safety check in this pipeline, so it is mandatory.
- Force-push or rewrite history on a shared branch.
- Touch Shopify account settings, billing, or app permissions.

**Not currently available:** local preview via `shopify theme dev` — the Shopify CLI is **not installed** and the store isn't authenticated locally. Until that's set up, previewing happens through the **`dev` draft theme in admin** (Section 3, Step 5). Optionally install it later: `brew install shopify-cli` or `npm i -g @shopify/cli @shopify/theme`, then `shopify auth login`.

**Standing instructions are saved, not re-pasted.** The brand guidelines and working rules (on-brand colours/fonts/spacing/tone; brand voice not generic marketing; desktop-primary but mobile-safe; prefer native Shopify elements over one-off custom blocks) are persisted in Claude Code's project memory. Reconfirm them for any major piece of work, but they no longer need pasting in every session.

---

## 3. SOP — designing and shipping a change

### Step 1 — Open Claude Code in the project folder
Point it at `/Users/tomrudkin/Project/Shopify /domini-collective-theme`. **Confirm the branch is `dev`, not `main`,** before anything else (`git branch --show-current`).

### Step 2 — (Re)confirm the standing guardrails
For any significant work, restate the brand + build rules so output stays on-brand and native-first. (Saved in memory; reconfirm for big tasks.)

### Step 3 — Give a specific brief
Concrete beats vague. State what's being built, its purpose, audience, must-haves, and a reference page/section for tone and layout. Use bracketed placeholders for anything to fill in. The more specific the brief, the less Claude fills gaps with off-brand assumptions.

### Step 4 — Review the plan / diff before any commit
Have Claude show what it's about to do (and the diff). Catch wrong assumptions here, before they're in code.

### Step 5 — Push to `dev` and preview on the draft theme
1. Claude commits **only the files for this change** (not local agent config like `.claude/`) and pushes to `origin/dev`.
2. The GitHub integration syncs to the `domini-collective-theme/dev` draft theme (seconds to ~a minute).
3. In admin → **Online Store → Themes → `domini-collective-theme/dev`** → **Preview** (or **Customize** to edit the draft).
4. **If the change is a new page template**, it is not yet a visible page. In **Pages**, create the page, then assign the new template in the *Theme template* dropdown. Add any images via the theme editor (sections show a placeholder until then).

### Step 6 — Stress-test on the draft (do not skip)
- View at **mobile, tablet, and desktop** widths.
- Test more than one case (e.g. product with many variants vs none; long title vs short).
- Confirm **other pages/sections weren't affected**.
- Click **every interactive element** (buttons, dropdowns, filters, add-to-cart).
- Check page load — no oversized images or broken assets.

### Step 7 — Merge `dev` → `main` to publish
Only once Step 6 is genuinely clean. **This merge is the live deploy — there is no further publish step.** Requires a deliberate, reviewed human go-ahead; Claude does not do this unsupervised.

---

## 4. Notes for anyone other than Tom
Acting on changes directly (not just reading this) requires your own push access to `OscarOscar-prog/domini-collective-theme`. Tom is currently the sole collaborator, so set that up first.

---

## What changed from the original setup notes

| # | Original notes said | Correction (what's actually true) |
|---|---|---|
| 1 | Working branch is **"staging"** | It's **`dev`**, connected to a draft theme **`domini-collective-theme/dev`**. |
| 2 | "Push the theme / pull this file into the live theme" to view | **Never push to live to preview.** You `git push` to `dev`; the GitHub integration auto-syncs to the **draft** theme, which you preview in admin. No manual theme push exists. |
| 3 | Claude can run `shopify theme dev` for local preview | The Shopify **CLI is not installed** and the store isn't authenticated, so local preview isn't available yet. Preview via the `dev` draft theme instead (or install the CLI). |
| 4 | "No separate Shopify theme ID is tracked" (live identified by name) | Still name-based — and there are now **two** named themes to know: the live one and the draft **`domini-collective-theme/dev`**. |
| 5 | Standing instructions must be given each session | They're now **saved in Claude Code's project memory** — persistent across sessions; just reconfirm for major work. |
| 6 | (Not mentioned) | A synced **`templates/page.*.json` does not create a visible page by itself** — you must create the Page in admin, assign the template, and add images in the editor. |
| 7 | (Not mentioned) | **Don't commit `.claude/`** (local agent config) into the theme repo — commit only the theme files for the change. |
| 8 | (Not mentioned) | Git is committing as an auto-generated identity (`tomrudkin@Toms-Laptop.local`). Set `git config --global user.email` to your real GitHub email for correct attribution. |
