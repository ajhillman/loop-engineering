# ANDREW-NOTES.md: loop-engineering on the Mac Mini

Owner: Andrew Hillman
Repo: github.com/cobusgreyling/loop-engineering (MIT)
Local path: ~/code/loop-engineering
Set up: 2026-06-30 on Apple-Mac-Mini-AJH-467

## What this repo is

Loop engineering is the practice of building a scheduled system that prompts
an AI coding agent for you, instead of prompting it by hand each time. A loop
runs on a cadence, reads a state file, does work through sub-agents with a
maker/checker split, and hands ambiguous items back to a human. This repo is a
reference: patterns, checklists, safety docs, starter kits, and two local CLIs.

Two pieces matter day to day:
- loop-audit: scores any project 0 to 100 and assigns a Loop Readiness level
  L0 to L3, with suggestions.
- loop-init: scaffolds a starter kit (skills, STATE.md, LOOP.md, budget file,
  constraints file) for a chosen pattern and tool.

## The two CLIs (installed and tested)

Both build from the local repo. The npm packages are not published yet, so they
run from ~/code/loop-engineering, not from npm.

Status on this machine:
- loop-audit v1.5.2, 12 of 12 tests pass, CLI runs.
- loop-init v1.3.1, 9 of 9 tests pass, CLI runs.

Shell wrappers were added to ~/.zshrc (a timestamped backup sits next to it).
Run them from any folder:

    loop-audit .                 # score the current project
    loop-audit . --suggest       # score plus copy-paste fixes
    loop-audit . --md            # markdown report
    loop-init <dir> --pattern daily-triage --tool claude   # scaffold a starter

## Build gotcha on this Mac (worth knowing)

Your shell exports NODE_ENV=production. npm reads that and omits
devDependencies, so npm ci and npm install both leave out TypeScript and the
build (tsc) fails with "tsc: command not found". The CLIs still run because the
dist/ folder ships prebuilt, but the tests need the compiler.

The fix used here, and the one to reuse after any git pull:

    cd ~/code/loop-engineering/tools/loop-audit && NODE_ENV=development npm install --include=dev && npm run build
    cd ~/code/loop-engineering/tools/loop-init  && NODE_ENV=development npm install --include=dev && npm run build

That is a per-command override. Nothing global changed.

## How this applies to your stack

Your themed-sites cluster lives at ~/Desktop/Projects/themed-sites-2026-06-07/.
It holds around fifteen static sites (fixcancer, fixcancers, andrewhillman-org,
andrewhillmanrecord, andrewjhillman, discseel, aiapro, eachfocus, hillmanv, and
others), git-tracked, deployed through Cloudflare Wrangler (deploy_all.sh,
_redeploy_llms.sh, .wrangler). After each deploy round you want to know, without
checking by hand, whether any live page broke: dead links, missing images, or a
regression against the last known-good state.

That is the Daily Triage pattern. Report first, act later.

## The daily-triage loop plan (documented, not enabled)

Goal: once a day, crawl the live themed-sites URLs, flag broken links, missing
or oversized images, and regressions, then write a prioritized report to
STATE.md. No auto-fix. No deploy. You read the report.

Runner at zero API cost: drive the loop with Claude Code under your existing
subscription, or with OpenHands pointed at a local Ollama model through
OmniRoute / llm-router (~/code/llm-router, OLLAMA_MODELS on G-DRIVE). All
inference stays local or inside your subscription, so no metered per-call spend.

Inputs the triage step reads each run:
- The live URL list for each site in the cluster.
- A link and image checker pass (HTTP status per link, image load per page).
- The prior STATE.md (what was already known).
- Optional: the last deploy log from deploy_all.sh.

Output each run:
- STATE.md updated with four sections: High Priority (broken now), Watch List
  (slow or suspicious), Recent Noise (ignored this run), Post-Run Critique
  (what to tune next time).
- loop-run-log.md appended with a one-line record.

### Ready-to-run commands

1. Scaffold the starter into a separate triage workspace, which keeps loop
   files out of the site repo:

    loop-init ~/Desktop/Projects/themed-sites-triage --pattern daily-triage --tool claude

   Add --dry-run first to preview what it writes before writing anything.

2. Fill in the site URL list and a checker command inside the generated
   loop-triage skill (.claude/skills/loop-triage).

3. Score readiness and read the suggestions:

    loop-audit ~/Desktop/Projects/themed-sites-triage --suggest

4. Run one triage pass by hand, read STATE.md, tune the skill. Repeat for a
   week before you consider any schedule.

### Human gates (keep these)

- No auto-edit of site source, no auto-deploy, no Wrangler calls from the loop.
  Report only.
- Denylist for any later action phase: .env, .env.*, wrangler.toml,
  .wrangler/**, deploy_all.sh, _redeploy*.sh, and anything matching
  secrets/credentials/keys.
- Anything unresolved three days running goes to you with full context.
- Token cap 100k per day (loop-budget.md). Kill switch: pause the scheduler.

## Nothing was scheduled

No autonomous loop is enabled. No cron, no scheduled task, no OpenHands job, and
no Claude Code /loop was started. Everything above is a plan plus commands you
run when you choose to. Turning a self-running agent loose on your repos is a
separate decision and needs your explicit go.

## Where this fits with what you already run

- OpenHands stays your autonomous executor. Loop engineering is the control
  layer around it: the schedule, the STATE.md spine, the maker/checker split,
  and the safety gates. This repo gives you the patterns and the readiness
  score. OpenHands does the work once you promote a loop past report-only.
- Your existing scheduled-task setup is where a promoted loop would eventually
  fire. Until you promote it, nothing here touches that setup.

## Quick reference

    loop-audit .                                score current dir
    loop-audit . --suggest --md                 full report
    loop-init <dir> -p daily-triage -t claude   scaffold a starter
    Repo docs: patterns/daily-triage.md, docs/safety.md, docs/loop-design-checklist.md
