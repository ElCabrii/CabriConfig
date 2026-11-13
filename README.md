# CabriConfig

A personal Claude Code plugin: an Opus team lead that plans, delegates, reviews and makes small changes, plus four Sonnet workers (frontend, app, QA, cyber). Each agent only loads the skills its work needs, parallel workers run in their own git worktrees, and finished worktrees get cleaned up.

## Agents

| Agent | Model / effort | Preloaded skills | Reads from the library when needed |
|---|---|---|---|
| `team-lead` (main session) | Opus / xhigh | invokes `ponytail` at session start | `improve` |
| `frontend-worker` | Sonnet / xhigh | `ponytail`, `find-docs` | frontend-design, shadcn, Vercel React/composition/view-transitions/web-design/React Native, agent-browser |
| `app-worker` | Sonnet / xhigh | `ponytail`, `find-docs` | Better Auth, TanStack AI, Cloudflare, Vercel deploy/CLI/optimize, React best practices, anti-slop installer |
| `qa-worker` | Sonnet / xhigh | `ponytail`, `find-docs` | agent-browser |
| `cyber-worker` | Sonnet / xhigh | `ponytail`, `find-docs` | Cloudflare security-audit (full audits only), Better Auth security, Workers best practices; runs Claude Code's built-in `/security-review` for branch/PR reviews |

Every agent can research on its own with Context7 (`ctx7`) and the web.

## How skills are split

- **`skills/`**: installed as normal plugin skills, so every agent sees their names. Kept small: `ponytail`, `find-docs`, and Matt Pocock's productivity skills (`grill-me`, `grilling`, `handoff`, `teach`, `to-questionnaire`, `wait-what`, `writing-for-agents`; five of these are slash commands only and never reach the model's skill list).
- **`library/`**: not scanned by Claude Code, so no agent sees these until its own prompt tells it to read one with `${CLAUDE_PLUGIN_ROOT}/library/...`. The lead never pays for skills only workers use.

Two platform facts shaped this:
- Skill preloading (`skills:` in agent frontmatter) works for subagents but not for the main-session agent, so the lead invokes `ponytail` itself.
- Built-in `security-review` can't be preloaded: it runs `git diff origin/HEAD...` as soon as it loads, which blocks the agent from starting when `origin/HEAD` is missing. `cyber-worker` invokes it on demand instead.

## Worktrees

The lead starts parallel workers with `isolation: "worktree"`. When a worker's commits are on a pushed branch with a PR, or merged into the branch being integrated, the lead runs `git worktree remove` + `git worktree prune`. `/cabriconfig:sweep` does the same check across the repo and asks before touching anything with uncommitted or unpushed work.

## CLIs the agents are told about

`git`, `gh`, `wrangler`, `shadcn`, `tanstack`, `npx auth@latest` (Better Auth), `ctx7`, `agent-browser`.

## Install

```bash
claude plugin marketplace add D:/CabriConfig
```

```bash
claude plugin install cabriconfig@cabriconfig
```

Make the lead your main session by adding this to `~/.claude/settings.json`:

```json
{ "agent": "cabriconfig:team-lead" }
```

To try it without changing settings: `claude --agent cabriconfig:team-lead`.

## Credits

Every skill in `skills/` and `library/` is copied unmodified from its authors' repository at the commit below, with its license kept next to it. All credit for that content goes to them.

| Skill(s) | Author | Source | Commit | License |
|---|---|---|---|---|
| `ponytail` | Dietrich Gebert | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | `96608e0` | MIT |
| `find-docs` | Upstash | [upstash/context7](https://github.com/upstash/context7/tree/master/skills/find-docs) | `5c4a4b4` | MIT |
| productivity skills | Matt Pocock | [mattpocock/skills](https://github.com/mattpocock/skills/tree/main/skills/productivity) | `5863806` | MIT |
| `frontend-design` | Anthropic | [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/frontend-design) | `2c5958e` | Apache-2.0 |
| `shadcn` | shadcn | [shadcn-ui/ui](https://github.com/shadcn-ui/ui/tree/main/skills/shadcn) | `e62c943` | MIT |
| `improve` | shadcn | [shadcn/improve](https://github.com/shadcn/improve) | `cac56e1` | MIT |
| Vercel agent skills | Vercel | [vercel-labs/agent-skills](https://github.com/vercel-labs/agent-skills) | `1a2c79d` | MIT (per README) |
| `agent-browser` | Vercel Labs | [vercel-labs/agent-browser](https://github.com/vercel-labs/agent-browser) | `2f533ec` | Apache-2.0 |
| Better Auth skills | Better Auth | [better-auth/skills](https://github.com/better-auth/skills) | `20c9e88` | none published (see `library/better-auth/LICENSE-NOTE.md`) |
| `tanstack-ai` | TanStack | [TanStack/ai](https://github.com/TanStack/ai/tree/main/skills/tanstack-ai) | `1888233` | MIT |
| Cloudflare skills | Cloudflare | [cloudflare/skills](https://github.com/cloudflare/skills/tree/main/skills) | `cbf893b` | Apache-2.0 |
| `security-audit` | Cloudflare | [cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill) | `c1c8a8c` | MIT |
| `install-anti-slop` | Dillon Mulroy | [dmmulroy/anti-slop](https://github.com/dmmulroy/anti-slop) | `13909c9` | MIT |

pdfcn ([shadcn-labs/pdfcn](https://github.com/shadcn-labs/pdfcn), MIT) ships no agent skill, so `frontend-worker` is pointed at its docs instead.

Keep this repository private: the Better Auth skills have no published license.
