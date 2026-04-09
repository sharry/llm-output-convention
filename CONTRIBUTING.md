# Contributing

This convention succeeds through adoption, not authority. Every contribution — a new tool example, a reference implementation, or a refinement to the spec — moves the needle. Here's how to help.

## Propose `--llm` output for a tool

This is the most impactful contribution. Pick a CLI tool that AI agents frequently use and show what its `--llm` output should look like.

1. Create a file in `examples/` named after the tool (e.g., `examples/mypy.md`).
2. Follow this structure:

```markdown
# Tool Name

## Standard output
\`\`\`
(paste real-world output from a typical run with some failures)
\`\`\`

## `--llm` output
\`\`\`
(proposed LLM-optimized output for the same run)
\`\`\`

## Design decisions
| Decision | Rationale |
|---|---|
| What you changed | Why it helps an LLM |
```

3. Open a PR.

**High-impact tools we'd love examples for:**

- [ ] ESLint
- [ ] mypy / pyright
- [ ] cargo build / cargo test
- [ ] go test
- [ ] tsc (TypeScript compiler)
- [ ] rubocop
- [ ] docker build
- [ ] terraform plan
- [ ] kubectl

Don't limit yourself to this list. If an AI agent runs it, it's a candidate.

## Build a reference implementation

A working plugin or PR to an actual tool is worth more than any spec. Some approachable starting points:

- **Jest custom reporter** — Jest's reporter API makes this possible without forking.
- **pytest plugin** — pytest's plugin system is similarly extensible.
- **ESLint formatter** — ESLint supports custom output formatters.

Place implementations in `implementations/<tool>/` with their own README explaining how to install and use them.

## Improve the spec

The RFC itself is a living document. If you see a gap, an ambiguity, or a better way to phrase something, open an issue or PR. Some areas that would benefit from discussion:

- **Output format for specific tool categories** (build tools vs. linters vs. test runners vs. package managers) — are there category-specific patterns worth codifying?
- **The `--llm=verbose` and `--llm=json` extension** — when should these be specced out?
- **Edge cases** — interactive prompts, streaming output, multi-command pipelines.

## Spread the word

Adoption is everything. You can help by:

- **Filing issues on tools you use** requesting `--llm` support, linking back to this repo.
- **Sharing in relevant communities** — developer tooling forums, agent-building communities, tool-specific Discords/forums.
- **Adding the badge** to tools that adopt the convention:

```markdown
[![LLM Output](https://img.shields.io/badge/--llm-supported-blue)](https://github.com/sharry/llm-output-convention)
```

## Guidelines

- Keep PRs focused. One tool per example PR, one idea per spec change.
- Test your examples against real output. Don't invent hypothetical CLI output — run the tool, capture the real thing, then write the `--llm` version.
- Prioritize agent workflows. The question is always: "What does an LLM need to take the next correct action?" Not what looks clean, not what's minimal for its own sake — what's *useful*.

## Code of conduct

Be kind, be constructive. This is a small idea that could make a lot of developers' (and agents') lives easier. Let's keep the discussion focused on that.
