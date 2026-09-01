# Mobile Dev Agent Starter Kit

**[Live showcase / field manual →](https://jenyago.github.io/mobile-dev-agent-starter-kit/)**

A multi-agent "company" for Claude Code, built specifically for shipping iOS/macOS apps as a solo or small-team developer. Instead of one generalist assistant trying to hold research, design, code, QA, App Store mechanics, and marketing in its head at once, this kit splits that work into 13 specialist subagents that hand off to each other in a fixed pipeline, with two mandatory quality gates before anything reaches the App Store.

## Why a multi-agent company beats one generalist assistant

A single Claude Code session asked to "ship this app" tends to do everything in one undifferentiated pass: it designs a screen, writes the Swift, calls it QA'd because the build compiled, and submits. Each of those steps has a different failure mode, and a generalist doing all of them back-to-back has no reason to distrust its own prior step.

Splitting the work into named roles fixes this in two ways:

1. **Each role carries only the context it needs.** The Dev agent doesn't need App Store Connect submission mechanics in its context; the Release agent doesn't need SwiftUI layout rules. Smaller, focused context means fewer mistakes per role.
2. **The gates are adversarial by design.** QA never reviews its own build. A dedicated ship-critic agent exists purely to be skeptical of "ready to submit" claims — including claims from QA and Release themselves — before a submission goes out. The same agent that built a feature is structurally prevented from being the one who declares it done.

The result is closer to how a real small studio works: research decides if an idea is worth building, a PM coordinates, specialists execute, and nothing reaches the store without an independent sign-off from someone (or something) that didn't write the code.

## What's inside

- **`PIPELINE.md`** — the full org chart and handoff flow, as prose and as a diagram, with the two mandatory gates called out explicitly.
- **`agents/`** — 13 standalone Claude Code subagent definitions: CEO, Researcher, BA, PM, Dev, Designer, UX, QA, Release, ASO, Marketing, Growth, ship-critic.
- **`rules/mobile-dev-lessons.md`** — about 20 hard-won technical lessons for iOS/macOS App Store shipping, each stated as a rule with its reasoning, no incident story attached.
- **`rules/definition-of-shipped.md`** — a three-part bar for what "shipped" actually means, so "submitted for review" stops getting reported as done.

## Install

1. Copy the `agents/` folder's contents into your own project's `.claude/agents/` directory.
2. Copy `rules/mobile-dev-lessons.md` and `rules/definition-of-shipped.md` into wherever your project keeps auto-loaded or on-demand rules (e.g. `.claude/rules/`), or just paste their contents into your `CLAUDE.md`.
3. Read `PIPELINE.md` once so you know which agent to invoke for what, and which two steps are non-negotiable gates.
4. Each agent's frontmatter lists the tools it expects (Read/Grep/Glob/Bash/Edit/Write/WebFetch, plus `Agent` for the ones that orchestrate others). If you have an App Store Connect MCP server or an XcodeBuild MCP server connected, the agents that mention ASC or Xcode operations will use them automatically; otherwise they fall back to the raw REST API or CLI commands described inline.
5. Adjust the placeholder project list, bundle ID pattern, and contact details in `Dev.md`, `QA.md`, and `Release.md` to your own apps before running anything for real.

Nothing in this kit is tied to a specific company, app, or account — every agent reads its live state from your own project files (an `apps.json`-style registry is assumed but not included; recreate one for your own portfolio, or point the agents at whatever registry format you already use).

## One developer's real production setup

This pipeline wasn't designed on a whiteboard — it's the refined version of a working setup used to actually ship apps to the App Store, across enough submissions and enough rejections that the gates and the lessons file both earned their place the hard way. Nothing here is theoretical: every gate exists because skipping it once caused a real problem (a broken paywall that shipped anyway, a submission that silently orphaned itself, a rejection that sat unanswered because nobody checked a reactive-only reply thread). The specifics have been stripped — no app names, no numbers, no dates — but the shape of the pipeline and the lessons in `rules/mobile-dev-lessons.md` are exactly what survived contact with the real App Store review process.
