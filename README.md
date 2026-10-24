# Rise Social Studio

A local studio that drafts Slovak social posts for rise.sk from public sources, with a person approving each one before export.

Public read-only site: https://rise-sk.github.io/rise-social/ | [Visual playbook](https://rise-sk.github.io/rise-social/visual-playbook/) | [Content plan](https://rise-sk.github.io/rise-social/content-plan/)

<p>
  <img src="docs/images/desktop.png" alt="The public read-only site: the rise.sk brand context every draft and image must follow" width="74%">
  <img src="docs/images/phone.png" alt="The public site at phone width" width="23%">
</p>

## Why it exists

The studio prepares rise.sk posts about its software and products for LinkedIn, Instagram and Facebook. Each post has to be accurate, traceable to public sources and read by a person before it goes out. The studio does the research and drafting in steps, keeps the evidence next to every claim, and stops at a human approval before anything is exported or scheduled. It is built for the rise.sk team that writes and approves these posts.

## What it does

- Starts from one topic. By default it writes one post. In auto mode it writes a campaign of 2 to 3 posts only when the research has enough distinct evidence. `--mode campaign` asks for a campaign directly.
- Stops high-risk briefs before drafting. Client results, metrics, regulated topics, competitors, non-public stories, named people and generated images need a human brief review first.
- Keeps a claim ledger built from public sources only. Each claim has a stable ID, its source URL, the checked date, a risk level and an expiry date one year after the check.
- Writes separate LinkedIn, Instagram and Facebook versions in Slovak, in the formal form, following `docs/EDITORIAL_RULES_SK.md`.
- Renders the carousel slides as 1080 x 1350 px PNG files and a PDF, both with Playwright. The PDF is the document format for LinkedIn.
- Runs one independent critique and one final validation. The author and critic roles alternate between the Codex and Claude CLIs from run to run.
- Export needs an approval in the browser. The approval is a SHA-256 digest of the draft pack, so any change to text, sources, visuals or rights needs a new one.
- Optional: YouTrack task sync (dry run unless `--apply`), Buffer drafts and scheduling after a separate approval, and Cloudinary media for Buffer. Manual export works without them.

## Stack

| Part | Technology |
|---|---|
| Language and runtime | TypeScript 5.9.3 on Node.js 24 or newer, npm 11 or newer |
| Review app | Next.js 15.5.28, React 19.2.7 |
| CLI | Commander 14.0.3, run with tsx 4.23.1 |
| Validation | Zod 4.4.3 |
| Storage | SQLite through Node's built-in `node:sqlite`, file `data/studio.sqlite` |
| Rendering and export | Playwright (playwright-core 1.61.1), archiver 7.0.1 |
| Source reading | cheerio 1.2.0, for HTML from public sources |
| Public site | Next.js static export for GitHub Pages |
| Tests and lint | Vitest 4.1.10, Playwright Test 1.61.1, ESLint 9.39.5 |
| AI runners | Codex CLI and Claude CLI, installed and signed in on your machine |
| Optional services | JetBrains YouTrack, Buffer GraphQL API, Cloudinary |

## Run it locally

You need Node.js 24 or newer. For real briefs you also need the Codex and Claude CLIs, signed in. `npm run rise-social -- doctor` lists what is missing.

```bash
npm ci
npx playwright install chromium   # needed for PNG and PDF export
npm run dev
```

Open http://127.0.0.1:4173/review. With no run selected, the page opens a demo run, so you can look around the review flow without any model call.

To prepare a run from your own topic:

```bash
npm run rise-social -- prepare "<topic>" --goal consideration
npm run rise-social -- review   # opens the review page (uses macOS open)
```

Settings for the optional services go in `.env.local`, copied from `.env.example`. The studio runs without them.

## Checks

```bash
npm run verify        # tests, lint, typecheck, contract checks and build
npm run test:e2e      # browser tests, needs the Chromium install above
npm run verify:push   # verify, plus the Git LFS size check and the production dependency audit
npm run verify:pages  # what CI runs for the GitHub Pages site
npm run hooks:install # enables the pre-push hook in .githooks
```

## How it runs

- Local only: `dev` and `start` bind to 127.0.0.1:4173, and the server accepts changes only from that origin. Runs, drafts and exports stay in git-ignored files under `data/`.
- Public site: `public-site/` is a static Next.js export under the `/rise-social` base path. Pull requests run the checks. A push to `main` runs them again, builds the site and publishes it to the `gh-pages` branch of this repository.
- The studio is not deployed anywhere. It runs on the machine of the person who approves the content, and GitHub Pages serves only the read-only site.

## Repo map

| Path | What is in it |
|---|---|
| `src/` | Review app, CLI, domain rules, AI runners, rendering, export and publishing |
| `public-site/` | Static site published to GitHub Pages |
| `brand/` | Brand tokens, Slovak voice, asset manifests and visual playbooks |
| `.agents/skills/` | Skills for each workflow step and each platform |
| `.agentic/` | Shared workflow contract: stages, human gates and evals |
| `content/approved/` | Archived approved packs, PNG and PDF files in Git LFS |
| `docs/` | Workflow, editorial rules, platform playbook and YouTrack workflow, mostly in Slovak |
| `scripts/` | Contract, content, LFS and public-output checks |
| `tests/` | Vitest and Playwright tests |
| `youtrack/templates/` | YouTrack issue templates |

Read [AGENTS.md](AGENTS.md) first for the rules that apply to people and agents, then [docs/WORKFLOW.md](docs/WORKFLOW.md) for the step-by-step flow.
