# atrium

> Multi-project agent platform — central coordination space where separate workspaces meet.

**Status (S18, 2026-05-21):** Working external-peer bridge in production. Framework extraction in progress; first reference implementation (atrium-mailbox transport + external-peer-cycle worker + dual-loop wake mechanism + protocol v0.2 + drift detector) live and autonomous in a private deployment. Public extraction next.

---

## What is atrium

A framework for operating Claude Code (and other agent runtimes like Codex, OpenCode, OpenClaw, Hermes) across multiple distinct projects with shared infrastructure but isolated content.

The metaphor: in real buildings, the atrium is the open central space where different wings meet. In this framework, the central event-spine + contexts registry + librarian sub-agents is where firm work, brand work, product work, and research work meet — without their content mixing.

## Why this exists

If you operate AI agents across multiple distinct projects (e.g., a research firm + a consumer product + a personal brand + meta-research about how it all works), you face problems that single-project agent setups don't:

- **Coordination drift** — what does session A know that session B should know?
- **Memory leakage** — content from one project bleeding into another
- **Doctrine accumulation** — principles you developed in project A apply to project B but how do they get there?
- **Authorship attribution** — when an agent writes to shared memory, which human's agent made that change?
- **Audit + governance** — for high-stakes decisions, you need a separate read-only auditor distinct from the writer

Existing agent frameworks (gstack, OpenClaw, Letta, Mem0, Zep, Hermes, etc.) solve adjacent problems brilliantly but don't address this specific multi-project coordination layer.

atrium does.

## What's novel here

A landscape sweep across 85 agent platforms in May 2026 identified three architectural patterns where no surveyed system has a public implementation:

1. **Read-only auditor sub-agent as a separate role.** A `memory-librarian-<project>` sub-agent with capabilities locked at the harness level (Read/Grep/Glob only — cannot Write), firing at session-close to audit memory layers for drift and contradictions. Memento blurs auditor+writer; Anthropic Dreaming is write-time; everyone else punts.

2. **3-tier memory promotion authority with attribution.** Every memory change is classified into one of (a) auto-applied (mechanical), (b) operator-proposed (medium-bar, draft + approve), or (c) operator-only (high-bar, identity changes). Every doctrine change is attributable.

3. **Author-tagged writes (cryptographic per-human identity).** Every memory record carries cryptographic identity of which human's agent produced it. Audit log is append-only and tamper-evident. *(Currently designed in spec v0.3 — not yet shipped.)*

Plus two doctrinal disciplines no surveyed framework enforces:

- **Anti-spiral rule**: every agent dispatch must produce one of {research artifact, tool, action, operator answer}. Prevents drift into meta-work.
- **Information asymmetry by design**: in three-critic dispatch (e.g., fact-checker / risk-challenger / portfolio-skeptic), each critic sees deliberately scoped subset of context. Confirmation bias engineered out, not just discouraged.

## What's running today (S18 milestone)

The first concrete instance of the framework — the **external-peer bridge** — is live and autonomous as of 2026-05-21. The bridge solves a specific problem: letting an external AI agent (ChatGPT, via its GitHub connector) participate as a peer in a multi-Claude-Code agent organization without giving it direct API access to the local environment.

The architecture (all built in this run):

```
    External peer (ChatGPT)
            ↓ writes markdown message
    GitHub mailbox repo (private)
            ↓ pulled on cycle wake
    external-peer-cycle (partner-cycle subtype)
            ↓ atomic claim + push-or-revert
    Local recipient (claude-frontier / firm / linkedin / freshframe)
            ↓ processes + writes reply
    Same path back to external peer
```

Components shipped:

- **Transport** — separate private GitHub repo as the mailbox. Markdown messages with YAML frontmatter, organized into `inbox/`, `claimed/`, `done/`, `threads/`. State transitions = git commits; audit trail = git log. Append-only JSONL event log per repo.
- **Protocol v0.2** — message schema with required + optional fields (id, thread_id, from, to, status, created_at, priority, expires_at, dependencies, routing_hints, protocol_version). Grandfather logic for v0.1 messages.
- **Validator** — `validate.py` runs in GitHub Actions on every push; rejects malformed messages, warns on protocol-version drift, checks definition-of-done strictness.
- **Worker cycle** — `external-peer-cycle` skill (a partner-cycle subtype) that wakes on a self-tuning cadence (270s / 600s / 14400s ladder) and processes the mailbox. Includes Step 7.5 event-spine production (emits cross-project events for the foreign repo), Step 7.6 CI verification (post-push), and per-project fanout (Phase 1.1, gated on enablement).
- **Supervisor** — `partner-cycle-supervisor` reads the worker heartbeat at 5× worker cadence; restarts the worker if stale. For autonomous execution to stop, BOTH chains must die simultaneously.
- **Drift detector** — `version_drift_check.py` catches the recurring failure pattern where a spec version is bumped but downstream docs still reference the old version. Wired into the routine maintenance step of partner-cycles.
- **Cross-project event spine** — shared event log on the operator's local filesystem; events from one project's commits get routed to other projects that should see them (`affects_projects` / `may_affect_projects` semantics).

What this proves:

- **External AI peers can participate without API access.** A markdown file + a git push is enough.
- **Atomic claim discipline survives external peers.** `git push --force-with-lease` rejects races; failed pushes revert local commits.
- **The autonomous loop self-perpetuates.** The cycle calls `ScheduleWakeup` on itself; the harness wakes a fresh Claude session that picks up state from a heartbeat JSON file. No daemon, no cron, no `claude -p` headless.
- **Git author identity ≠ message author.** Frontmatter `from:` is the trust anchor until cryptographic identity is shipped (v0.3 roadmap).

## Status (May 2026)

This is the public scaffold. The framework code, skills, principles, and tools were developed in a private workspace and are being progressively extracted, sanitized, and published here.

**The canonical architecture doc lives in the private workspace today** (will be extracted to `docs/PLATFORM_ARCHITECTURE.md` here once sequencing is ratified). It captures the 3-layer model that atrium implements:

- **Layer 1 — atrium itself** (this repo): the OSS framework, installable
- **Layer 2 — holding company** (private deployment): platform infrastructure + cross-portfolio coordination (VP of Hiring, shared registry, event spine, external-peer bridge)
- **Layer 3 — operating businesses** (private deployment, multiple): each has its own identity, culture, doctrine, employees (Hires), and partner cycle

**Not yet in this repo (extraction queue):**
- `docs/PLATFORM_ARCHITECTURE.md` — the 3-layer holding-company model (top-level architecture)
- `docs/PARTNER_CYCLE_PATTERN.md` — partner-cycle pattern spec v0.8 (includes external-peer-cycle as 5th instance + cadence ladder + sole-pusher rule)
- `docs/MULTI_AGENT_PATTERN.md` — sub-agent doctrine (Contractor / Hire / cmux-sibling; 2-tier broker via VP of Hiring + per-project Hiring Managers; final-report schema)
- `docs/PROTOCOL.md` — atrium-mailbox message schema v0.2
- `docs/PLATFORM_SPEC.md` — RFC-2119 MUST clauses
- `docs/principles/` — Principles 16-21 (Memory Architecture, Spec Follows Shipped Reality, Continuous Cycles via ScheduleWakeup, Workspace-Aware Tool Architecture, Continuous Discovery, Event-Spine Coordination)
- `skills/` — session lifecycle templates, partner-cycle workers (`external-peer-cycle` first), wake-reader, supervisor
- `agents/` — memory-librarian template, vp-hiring template, hiring-manager template
- `tools/` — session_registry (with AQ-2 hierarchical sub-agent IDs), contexts_lib (with AQ-1 display labels), cross_session_dashboard, validate.py, forward_reply.py, atrium_inflight_check.py, version_drift_check.py
- `docs/DEPLOY.md` — install + initialize
- `setup` — gstack-style installer script

Coming over the next sessions.

## Roadmap

- **v0.1** (DONE, S17): scaffold + README + LICENSE
- **v0.1.5** (DONE, S18 2026-05-21): first reference implementation deployed (external-peer bridge: atrium-mailbox + external-peer-cycle + dual-loop supervisor + protocol v0.2 + validator + drift detector). Not yet extracted to this repo; lives in a private workspace.
- **v0.1.6** (DONE, S19 2026-05-22): multi-agent platform v1.0 doctrine landed in the private workspace — PLATFORM_ARCHITECTURE.md (3-layer holding-company model) + MULTI_AGENT_PATTERN.md (Contractor / Hire / cmux-sibling, 2-tier broker via VP of Hiring + per-project HMs) + AQ-1 display labels + AQ-2 hierarchical sub-agent IDs + AQ-3 final-report schema. Generic enough for extraction; awaiting sequencing decision.
- **v0.2** (next): extract session lifecycle skills + memory librarian sub-agent template + Principle 16 (Memory Architecture) + the external-peer-cycle as the first concrete skill.
- **v0.3**: partner-cycle pattern formalized + autonomous loops via ScheduleWakeup spec + signed-commit attestation (cryptographic identity for external peers).
- **v0.4**: contexts.yaml registry + multi-project disambiguation + cross-project fanout (Phase 1.1).
- **v0.5**: event-spine + cross-session inbox + Anthropic Routines integration (webhook wake replacing polling).
- **v1.0**: full framework + DEPLOY.md + installer + working multi-project example.

## Inspiration + acknowledgments

The patterns here emerged from running a multi-project AI operation in production over several months, but they sit alongside (and explicitly reference) prior work:

- **[gstack](https://github.com/garrytan/gstack)** by Garry Tan — Claude Code skill stack; reference for SKILL.md template generation + distribution polish.
- **[OpenClaw](https://github.com/openclaw/openclaw)** by Peter Steinberger — agent runtime substrate that this framework can sit on top of.
- **[Memento-Skills](https://github.com/Memento-Teams/Memento-Skills)** — academic continual-learning agent framework; the Read→Execute→Reflect→Write loop on markdown skills is convergent with this framework's Principle 17 (Spec Follows Shipped Reality).
- **[Zep / Graphiti](https://github.com/getzep/graphiti)** by Daniel Chalef — bi-temporal knowledge graph; reference for how memory queries should eventually work.
- **[Letta](https://github.com/letta-ai/letta)** — stateful agent runtime + sleep-time compute paper (arxiv 2504.13171); reference for idle-triggered self-review.
- **Anthropic Managed Agents + Dreaming** — closest commercial analog to this framework's librarian pattern.
- **Andrej Karpathy's "Software 3.0" framing** — LLM as new OS. The 6-layer memory architecture in this framework maps directly onto LLM-OS components (identity = boot ROM, doctrine = kernel, state = RAM, feedback = syslog, KB = filesystem, routing = syscall table).

## License

MIT. See [LICENSE](LICENSE).

## Author

[@tzahishimkin](https://github.com/tzahishimkin)
