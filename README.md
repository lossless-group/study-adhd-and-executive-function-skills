# ADHD and Executive Function Skills: Study

## The question

> When an agent skill is written to compensate for ADHD or other
> executive-function limits (short attention span, task initiation, working
> memory, time blindness, hyperfocus, drift), **where does the skill put the
> intervention**? Does it shape the agent's own output, steer the user's task
> flow, or keep state between sessions? Which rules and trigger heuristics
> have the authors converged on?

The trigger was [`ayghri/i-have-adhd`](https://github.com/ayghri/i-have-adhd)
(~59k★), which shows that a short, opinionated output-style skill can change
how every response reads. The rest of this collection maps how far the idea
goes beyond formatting.

This study does not cover clinical ADHD tooling or general productivity apps.
It is limited to **agent skills, hooks, and agent runtimes** where executive
function support is the stated purpose.

**Who it's for.** This is intel for two of our products:
[`ai-labs/hope-ai/`](../../hope-ai), the agent skill for the ChoiceCenter
100-day Personal Strategic Plan, and [`ai-labs/kay-os/`](../../kay-os), a
colony of personal-assistant agents for ADHD/2e all-stars, starting with a
hosted Hermes agent.

## The design space at a glance

| Bet | Entry |
|---|---|
| **Output shape only.** One always-on ruleset: answer first, numbered steps, capped lists, minute estimates. Packaged for ~10 harnesses, with evals and a SessionStart opt-in hook | [i-have-adhd](./i-have-adhd) |
| **Executive-function taxonomy.** 24 small skills, mostly one per EF construct (inhibition, set-shifting, working memory, time blindness, task initiation, …), each with numeric trigger heuristics, plus a manual `nd` router and synced always-on templates | [nd-skills](./nd-skills) |
| **Hooks and time awareness.** A date-injection hook plus SQLite "nudges" that fire on prompt submit; journaling into Obsidian as cross-session memory | [claude-adhd-skills](./claude-adhd-skills) |
| **Session enforcer with persistent state.** `/adhd` writes an outcome before any code; parks drift into per-workstream files under `~/.claude/adhd/` | [adhd-claude-skill](./adhd-claude-skill) |
| **Agent as a whole external EF system.** Produces a daily HTML workbench; "if the user cannot copy-paste it, click it, or check it off, it does not belong" | [audhd-executive-function](./audhd-executive-function) |
| **Companion plus adaptive profile.** Coaching skill (en/zh) plus a co-triggering `adhd-profile-tracker` that maintains a consented user profile across sessions | [adhd-copilot](./adhd-copilot) |
| **Behavioral coaching.** Micro-steps, launch rituals, time calibration, self-blame recovery; written for a non-Claude harness (DeepSeek Harness) | [dsh-adhd-copilot](./dsh-adhd-copilot) |
| **Ritual reviews over an external system.** GTD triage/review and zettelkasten review over org-mode files; agent asks and critiques, the user does the writing | [adhd-skills](./adhd-skills) |
| **Learning domain.** Study coach built on retrieval practice and spaced review; state lives in a pasted "Progress Card" because chat has no files | [adhd-study-coach](./adhd-study-coach) |

Deep per-entry write-ups live in [`context-v/profiles/`](./context-v/profiles).
Every claim in them cites a pinned `path:line`.

## What the profiles found

**The answer to the question.** The interventions fall into three places,
and no entry covers all three:

| Locus | Entries | What it buys | What it costs |
|---|---|---|---|
| **The reply itself** | i-have-adhd, talk-normal (ours), dsh-adhd-copilot | Works in any harness, needs no state | Can't see time, can't remember, can't follow up |
| **The session's task flow** | nd-skills, adhd-claude-skill, adhd-copilot, adhd-skills, adhd-study-coach | Catches drift, spirals, and stuck loops as they happen | The model has to keep its own counts, and nothing runs between sessions |
| **State and perception outside the chat** | claude-adhd-skills (clock + SQLite hooks), audhd-executive-function (reads CRM/inbox/calendar), adhd-claude-skill (`~/.claude/adhd/`), adhd-copilot / adhd-study-coach (user-carried file or card) | Time awareness, memory across sessions, nudges | Setup, and a new source of interruption unless it's capped |

**Patterns that recur across authors who never cite each other:**

- **A two-minute first step.** "Start" is the bottleneck, so every coach ends
  on one small concrete action: i-have-adhd's under-two-minute next step,
  adhd-copilot's 2-minute first step, dsh-adhd-copilot's 30-second action.
- **Cap the choices at three.** nd-skills, adhd-copilot, dsh-adhd-copilot,
  and audhd-executive-function (which bans options outright).
- **Count, don't feel.** The triggers are numeric: 8 research turns, 10
  hyperfocus turns, 3 repeated failures, 4+ open threads, <15% word overlap,
  3 nudges a day. The full table is in
  [`Profile__ND-Skills`](./context-v/profiles/Profile__ND-Skills.md).
- **Park, don't refuse.** Tangents are captured somewhere trusted
  (adhd-claude-skill's per-workstream `parked.md`, biz-weekly-review's single
  front) instead of being followed or blocked.
- **Score effort, not outcomes, with shame-free language.** audhd-executive-function
  ("track effort only", a banned-word table), nd-skills' `biz-weekly-review`
  ("counts of things sent, shipped, published"), dsh-adhd-copilot ("showing
  up counts as a win"). hope-ai's PSP arrived at the same rule on its own.
- **Consent before memory.** adhd-copilot proposes profile updates at
  session end and writes only on approval. nd-interrupt-budget's budget is
  "never inferred". adhd-skills reads the user's org files but never writes them.

**Warnings from the source:**

- **Prompt-only counters drift.** Without a hook or host signal, the model
  keeps its own turn counts. nd-skills' README calls turn counts "a weak
  proxy" and lists the host signals it would rather have (idle time, front
  app, clock). Only claude-adhd-skills actually feeds the agent the clock.
- **Evals mostly check that rules exist, not that they're followed.**
  nd-skills' regression check is phrase matching. i-have-adhd's blind-judged
  eval improved every dimension and still failed its own release gate.
- **Always-on stacks conflict.** talk-normal bans next-step menus. nd-skills'
  closure gate and hyperfocus guard offer them, and i-have-adhd *requires* a
  closing next action. Pick one closer rule per agent.
- **Rules contradict themselves across files.** nd-skills restates working
  memory every 15 turns in the skill and every 5 in the always-on block. The
  block wins, because it's what loads.
- **Compiling judgment into code loses it.** dsh-adhd-copilot's runtime is a
  keyword router, and the Markdown method ships alongside it unloaded.

## Intel for hope-ai

The profiles each compared their entry to the PSP skill
(`hope-ai/context-v/agent-skills/personal-strategic-plan/SKILL.md`).
What's worth taking:

1. **A Progress Card for single-chat mode** (adhd-study-coach). PSP's no-files
   fallback is "stay in this chat". A plain-text card the member pastes into
   the next chat survives a new conversation, and PSP's "give them the exact
   YAML to paste" is already most of the way there.
2. **A clock** (claude-adhd-skills). PSP can't tell morning from evening
   unless told. A date hook, or a "what time is it for you?" opener on
   surfaces without hooks, lets it pick the check-in mode itself.
3. **Park new goals instead of pushing back** (adhd-claude-skill). When a
   member wants to add a habit mid-program, log it and raise it at the weekly
   review instead of only pushing back with "protect them from their own ambition".
4. **What worked, not just what was done** (adhd-copilot). Add "Effective /
   Rejected / Own hacks" to the member's file, written only after they
   approve a proposed change.
5. **Check-in language** (audhd-executive-function, dsh-adhd-copilot). A
   banned-word table, keeping completed items visible, emotion first after a
   missed day, and a morning menu tiered by energy.
6. **Stalled goals need an explicit decision** (adhd-skills, ADR 0012):
   never default-reschedule. Add a 14-day deadline horizon scan to the weekly review.
7. **If check-ins ever go out unprompted, cap them** (nd-skills'
   `nd-interrupt-budget`): silent by default, with a nudge budget the member
   sets.
8. **Already aligned:** never miss twice, effort over outcome, one goal at a
   time. nd-skills' `biz-weekly-review` adds the diagnosis "two misses in a
   row is a design problem (cue gone, too big, reward dead)".

## Intel for kay-os

The profiles didn't look at kay-os directly. These map their findings onto
the plan in `kay-os/context-v/explorations/`: a hosted Hermes agent that
writes its own skills, reachable by phone, with a git-backed record in
Cloudflare Artifacts.

1. **The colony's baseline skill set already exists.** nd-skills' 25 MIT
   skills are already in the `SKILL.md` format Hermes reads. Import them as
   the read-only baseline, but fix the 5-vs-15-turn working-memory conflict
   and choose one closer rule first.
2. **Hermes can do what the prompt-only skills can't.** Cron and the
   messaging gateway give it a clock and a way to reach the person between
   sessions, which no entry here has except through hooks. That makes
   `nd-interrupt-budget` (silent by default, user-set budget) a requirement,
   not an option.
3. **A learning loop needs consent.** adhd-copilot's propose-then-write
   profile is the pattern for Hermes' self-written skills and notes: show a
   proposed change, write it only on approval, keep it in git so it can be undone.
4. **Pick a daily surface.** audhd-executive-function's daily page (every
   item is pre-written text, a link, or a checkbox) is the far end of "the
   agent *is* the executive function". It suits a member who wants the agent to
   act, not coach. adhd-skills' write boundary suits one who wants the agent
   only to ask. Decide per member, and record it in the profile.
5. **Park by workstream.** adhd-claude-skill's routing of tangents to the
   *right* project's inbox maps onto one Artifacts repo per workstream.
6. **Replies on a phone screen.** i-have-adhd's rules (answer first,
   capped lists, minute estimates, a two-minute next step) are the output
   contract for the messaging gateway.

## What we are looking at, repo by repo

Working checklist per entry:

- **Locus of intervention.** Does it change the agent's output, the user's
  task flow, or persistent state?
- **Triggering.** Is it user-invoked (`/command`), model-invoked from the
  description, always-on via a hook, or co-triggered with another skill?
- **Trigger heuristics.** What concrete thresholds does it use (turn counts,
  repeated errors, word overlap, list caps)?
- **State.** Is there none, files under `~/.claude/`, SQLite, an external
  vault (Obsidian, org-mode), or a user-carried card?
- **Grounding.** Does it cite research (Barkley, Ramsay & Rostain,
  learning science), or is it written from lived experience?
- **Tone rules.** Shame-free language, "never lecture", one-tap dismiss,
  "never blocks". How each one avoids becoming a nag.
- **Packaging.** Single `SKILL.md`, multi-skill repo, plugin manifests per
  harness, evals.

## References

### [i-have-adhd](./i-have-adhd)

Profile: [`Profile__I-Have-ADHD.md`](./context-v/profiles/Profile__I-Have-ADHD.md)

`ayghri/i-have-adhd` · MIT · pinned at `v0.4.1`. The anchor reference. The
rules live in `skills/i-have-adhd/SKILL.md` and start from "five facts" about
how ADHD changes reading. It is the most heavily packaged entry: Claude,
Codex, Cursor, OpenCode, Gemini, Qwen, and Kimi manifests, plus `evals/`,
`tests/`, and a `hooks/always-on.mjs` SessionStart hook that injects the
ruleset only when `~/.claude/.i-have-adhd-always` exists. It is here as the
reference for the *output-shape* approach and for cross-harness distribution.
Several forks exist (see candidates); this is the upstream.

### [nd-skills](./nd-skills)

Profile: [`Profile__ND-Skills.md`](./context-v/profiles/Profile__ND-Skills.md)

`jpoindexter/nd-skills` · MIT · pinned at `v2.1.0-4-g8b697d1`. This
replaces `jpoindexter/neurodivergent-agent-skills`, an older repo by the
same author that now only points here. It has 25 skills: 24 behaviors,
mostly named after an executive-function construct and each with numeric
triggers, plus a manual `nd` router. Examples: `nd-research-gate` fires
after 8+ research turns with no concrete output, `nd-setshift` after the
same error 3+ times, `nd-choicereduce` caps options at 3, and
`nd-taskboundary` uses <15% word overlap to detect a topic shift. The
v2.2.0 "companion pack" adds `autism-layer`, `biz-weekly-review`,
`nd-interrupt-budget`, and `nd-inbox-digest`. The repo also has synced
always-on templates for Claude Code and Codex, deterministic
behavior-regression evals, and a redacted conversation-audit script. This
is the closest thing in the collection to a *taxonomy*, and it is the one
to read when deciding how to split executive-function support into skills.

### [claude-adhd-skills](./claude-adhd-skills)

Profile: [`Profile__Claude-ADHD-Skills.md`](./context-v/profiles/Profile__Claude-ADHD-Skills.md)

`ravila4/claude-adhd-skills` · MIT. The *hooks* reference. A date hook gives
the agent time awareness, and `hooks/check_alerts.py` /
`add_alert.py` / `ack_alert.py` back the `nudge` skill with SQLite,
surfacing reminders on prompt submission. `skills/nudge/SKILL.md` is
explicit about what nudges can't do: they need user input to fire, so they
can't monitor processes. `daily-journal` and `obsidian-vault` use an
Obsidian vault as memory between sessions.

### [adhd-claude-skill](./adhd-claude-skill)

Profile: [`Profile__ADHD-Claude-Skill.md`](./context-v/profiles/Profile__ADHD-Claude-Skill.md)

`Leonw98/adhd-claude-skill` · MIT. A single `skill/SKILL.md` with `/adhd`
subcommands (`init`, `park`, `parked`, `done`, `mission`, `workstreams`). It
requires a written outcome before code moves, routes off-topic ideas to the
*right* workstream's `parked.md` (not the current one), and surfaces parked
items at the next session start. It is here for its file-based state layout
(`~/.claude/adhd/workstreams.json`, `sessions/current.json`) and its
"park, don't follow" drift pattern.

### [audhd-executive-function](./audhd-executive-function)

Profile: [`Profile__AuDHD-Executive-Function.md`](./context-v/profiles/Profile__AuDHD-Executive-Function.md)

`assafkip/audhd-executive-function` · MIT. Written for founders with AUDHD
(ADHD + autism). The agent's daily output is a full HTML workbench, not a
briefing, with inline copy buttons and no cross-references (rule A1:
"never say 'see section above'"). It reads `references/research.md` and a
per-user `references/user-profile.md` first. It is here as the far end of
the spectrum: the agent *is* the external executive function.

### [adhd-copilot](./adhd-copilot)

Profile: [`Profile__ADHD-Copilot.md`](./context-v/profiles/Profile__ADHD-Copilot.md)

`dsx0511/adhd-copilot` · **no license file** (read-only reference). Three
skills: `adhd-copilot-en`, `adhd-copilot-zh`, and `adhd-profile-tracker`.
The copilot handles daily stuck states and acts as a "monitoring layer"
during research discussions, stepping in only when divergence spirals. The
profile tracker co-triggers, loads a profile at session start, observes
patterns, and *proposes* updates at session end with consent. It is here for
the co-triggered, adaptive-profile pattern.

### [dsh-adhd-copilot](./dsh-adhd-copilot)

Profile: [`Profile__DSH-ADHD-Copilot.md`](./context-v/profiles/Profile__DSH-ADHD-Copilot.md)

`zimai233/dsh-adhd-copilot` · MIT. A behavioral-coaching skill for
DeepSeek Harness, mostly in Chinese, with frontmatter metadata
(`type: behavioral-coaching`). It covers micro-steps, launch rituals, time
calibration, and recovery from self-blame. It is here as a non-Claude
harness data point and for its "lower the cost of starting" framing, which
avoids asking for self-discipline.

### [adhd-skills](./adhd-skills)

Profile: [`Profile__ADHD-Skills.md`](./context-v/profiles/Profile__ADHD-Skills.md)

`RobotDisco/adhd-skills` · CC-BY-SA-4.0 · default branch `trunk`. The README
calls it personal, not general-purpose. It contains `gtd/` (triage, review)
and `brain/` (weekly review, note processing) skills over org-mode files,
each with a target duration ("Target time: 30 minutes. Name it if the
session runs long") and a fixed ritual order. It is here for the *ritual*
pattern, where the agent asks and critiques but the user does the cognitive
work.

### [adhd-study-coach](./adhd-study-coach)

Profile: [`Profile__ADHD-Study-Coach.md`](./context-v/profiles/Profile__ADHD-Study-Coach.md)

`itsAalaa7/adhd-study-coach` · MIT. A study companion where each rule cites
learning-science or ADHD research with honest strength ratings. Its thesis
is that learning science already knows what works and ADHD research explains
why it's hard to *keep doing*. It is here for the "Progress Card" pattern:
state is a plain-text block the user carries between chats, treated as data,
not instructions. It also ships a harness-agnostic `universal/` variant.

## Candidates: surfaced, not yet pinned

- **Forks of i-have-adhd**: `murrayj426/adhd-skill` (action-first,
  plain-language) and `afrokob/adhd-skill` (adds an ASD-STE100
  simplified-English rule). Pin only if the diff from upstream matters.
- **`Citability/claude-code-adhd-skills`** (MIT): `mirror` and `librarian`,
  described as an unedited copy of another author's (ChudiNnorukam) daily
  skills. The original repo returns 404, so provenance is unclear.
- **Standalone agent apps, not skills**: `anyan001121-dot/focusflow`
  (LangGraph + Streamlit: brain dump, focus mode, interruption capture,
  resume) and `ihddirmas/offload-agent` (Strands SDK capture-and-route).
  Relevant if the question widens to EF *agents* rather than skills.
- **`adhd-daily-planner`**: appears in registry mirrors
  (`majiayu000/claude-skill-registry`, `erichowens/some_claude_skills`).
  Find the canonical source before pinning.
- **No license, thin**: `harrrrychen/adhd-skills`,
  `houchenfeng/openclaw-adhd-skills`, `oanogin/adhd-skill`.

## Sub-inquiries

1. Compare `i-have-adhd`'s always-on output rules with
   `nd-sensory-load` / `nd-working-memory`. Where do output-shape rules and
   EF-construct skills overlap, and where do they conflict?
2. Collect every numeric trigger threshold across the collection (turn
   counts, error repeats, list caps, durations) into one table.
3. Compare state models: none → user-carried card → `~/.claude/` files →
   SQLite → external vault. What does each buy, and what does it cost the
   user to maintain?
4. Compare anti-nag mechanics: "never blocks", one-tap dismiss, consent
   before profile writes, "stop adhd mode". How does each keep the agent
   from becoming one more source of interruption?

Notes go in `context-v/inquiry/`, cited by path
(`studies/adhd-and-executive-function-skills/<repo>/<file>:<line>`), not
paraphrased as prose.

## Related

- `ai-labs/hope-ai/`: PSP agent skill; overlaps on habits, pacing, and
  check-in tone
- `ai-labs/kay-os/`: personal-assistant agent colony for ADHD/2e all-stars;
  see its `context-v/explorations/` for the Hermes and Artifacts plans
- `context-v/agent-skills/talk-normal/` (anchor root): an existing
  output-style skill in this tree
- [[../agent-harnesses]]: how harnesses load skills and run hooks (the
  mechanics these skills depend on)
- [[../open-specs-and-standards]]: the `SKILL.md` spec itself
