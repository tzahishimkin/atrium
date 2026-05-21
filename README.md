# atrium

> Multi-project agent platform — central coordination space where separate workspaces meet.

**Status:** early. Scaffold only. Framework extraction in progress.

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

## Status (May 2026)

This is the public scaffold. The framework code, skills, principles, and tools were developed in a private workspace and are being progressively extracted, sanitized, and published here.

**Not yet in this repo:**
- `skills/` — session lifecycle templates, partner-cycle workers, wake-reader, supervisor
- `agents/` — memory-librarian template, hiring-manager pattern
- `tools/` — session_registry, curator_helper, contexts_lib, cross_session_dashboard
- `docs/principles/` — Principles 16-21 (Memory Architecture, Spec Follows Shipped Reality, Continuous Cycles via ScheduleWakeup, Workspace-Aware Tool Architecture, Event-Spine Coordination)
- `docs/PATTERN.md` — partner-cycle pattern spec
- `docs/PLATFORM_SPEC.md` — RFC-2119 MUST clauses
- `docs/DEPLOY.md` — install + initialize
- `setup` — gstack-style installer script

Coming over the next sessions.

## Roadmap

- **v0.1 (current)**: scaffold + README + LICENSE
- **v0.2**: session lifecycle skills + memory librarian sub-agent template + Principle 16 (Memory Architecture)
- **v0.3**: partner-cycle pattern + autonomous loops via ScheduleWakeup
- **v0.4**: contexts.yaml registry + multi-project disambiguation
- **v0.5**: event-spine + cross-session inbox
- **v1.0**: full framework + DEPLOY.md + installer + working examples

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
