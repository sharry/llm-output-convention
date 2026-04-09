# RFC: `--llm` Flag Convention for CLI Tools

**Status:** Draft

**Author:** Youssef Ben Sadik

**Created:** 09 April 2026

## Abstract

AI coding agents (Claude Code, Codex, OpenCode, etc.) increasingly rely on CLI tools to perform tasks — running tests, linting, building, deploying. These tools produce output designed for human consumption: colored text, progress bars, decorative formatting, verbose stack traces. This wastes tokens, injects noise, and degrades agent performance.

This RFC proposes a lightweight, voluntary convention: a `--llm` flag and `LLM_OUTPUT` environment variable that CLI tools can adopt to emit LLM-optimized output — concise, plain-text, high-signal, and free of decoration.

This follows the precedent of `--verbose` and `--quiet`: no formal spec body, no compliance authority, just an obviously useful convention that spreads through adoption.

## Motivation

### The problem

When an AI agent runs `jest`, it might receive:

```
 PASS  src/utils.test.ts
 PASS  src/helpers.test.ts
 PASS  src/format.test.ts
 FAIL  src/auth.test.ts
  ● should reject expired tokens

    expect(received).toBe(expected)

    Expected: 401
    Received: 200

      40 |   const res = await request(app).get('/protected').set('Authorization', expiredToken);
      41 |
    > 42 |   expect(res.status).toBe(401);
         |                      ^
      43 |
      44 | });

      at Object.<anonymous> (src/auth.test.ts:42:22)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)

  ● should refresh token on 403

    TypeError: refreshToken is not a function

      13 |   if (response.status === 403) {
      14 |     const newToken = await refreshToken();
    > 15 |     return retry(config, newToken);
         |                         ^
      16 |   }

      at handleResponse (src/api.ts:15:25)
      at processTicksAndRejections (node:internal/process/task_queues:95:5)

Test Suites: 1 failed, 3 passed, 4 total
Tests:       2 failed, 48 passed, 50 total
Snapshots:   0 total
Time:        3.842 s
Ran all test suites.
```

The agent doesn't need the banner, the PASS lines, the decorative arrows, the code context window, or the summary table. It needs to know what failed and why — nothing more.

### Why `--json` isn't the answer

Many tools offer `--json` output. This is designed for machine parsing by scripts and pipelines. It's a poor fit for LLMs because:

- **It's verbose.** JSON representations of test results can be 10-50x larger than a concise text summary.
- **It's structured for traversal, not comprehension.** An LLM doesn't iterate over keys; it reads sequentially.
- **It includes everything.** JSON output typically contains all data — passing tests, timing, metadata — because scripts might need any of it. LLMs almost never do.

### Why `--quiet` isn't the answer

Quiet mode suppresses too much. An agent that gets only an exit code can tell *that* something failed, but not *what* or *why*. It still needs diagnostic information — just not decorative information.

### The gap

There is currently no convention for output that is:

- **Concise** — minimal tokens, maximum signal
- **Diagnostic** — enough to understand and fix the problem
- **Plain text** — no ANSI codes, no box drawing, no spinners
- **Stable** — consistent enough to pattern-match across runs
- **Selective** — omits success details, focuses on failures and actionable info

This RFC fills that gap.

## Specification

### 1. The `--llm` flag

CLI tools SHOULD support a `--llm` flag that switches output to LLM-optimized mode.

```bash
jest --llm
eslint --llm
cargo build --llm
```

The flag name is `--llm` because:

- It is specific and self-documenting.
- It has no known conflicts with existing CLI tools.
- It accurately describes the target consumer of the output.
- It is extensible to future values (see §5).

### 2. The `LLM_OUTPUT` environment variable

Tools SHOULD also check for the `LLM_OUTPUT` environment variable. This allows AI agent harnesses to set the mode globally for all subprocesses without modifying individual commands.

```bash
export LLM_OUTPUT=1
jest          # behaves as if --llm was passed
eslint        # behaves as if --llm was passed
```

**Precedence:** The `--llm` flag, if explicitly passed, MUST take priority over the environment variable. Explicitly passing a standard flag (e.g., `--json`, `--verbose`) SHOULD take priority over `LLM_OUTPUT`.

### 3. Output style guide

Tools implementing `--llm` SHOULD follow these principles:

#### 3.1 Strip decoration

Remove all elements that exist for human visual comfort:

- ANSI color/style codes
- Progress bars and spinners
- Box-drawing characters and ASCII art
- Banners, logos, version headers
- Decorative separators and blank lines

#### 3.2 Report failures, not successes

For tools that check or validate (test runners, linters, type checkers, builds):

- **DO** report each failure with location and reason.
- **DO NOT** enumerate passing/successful items.
- **DO** include a one-line summary count (e.g., `FAIL 2/50`).

#### 3.3 Compress stack traces

Stack traces are useful but typically over-detailed. In `--llm` mode:

- **DO** include the error message and type.
- **DO** include the top 1-3 frames originating in user code.
- **DO NOT** include frames from `node_modules`, standard library internals, or runtime machinery.
- **DO NOT** include code context windows (the `>` arrow source display).

#### 3.4 Use a flat, scannable structure

Prefer a consistent, line-oriented format:

```
STATUS summary
--- location description
  detail
  detail
--- location description
  detail
```

Specific formatting is left to each tool (see §4), but the output should be immediately understandable by reading top-to-bottom with no nesting beyond one level.

#### 3.5 Preserve essential information

`--llm` is not `--quiet`. The output MUST contain enough information for an LLM to take corrective action. The test: **could a competent developer fix the problem given only this output and access to the codebase?**

#### 3.6 Plain text only

Output MUST be valid UTF-8 plain text. No ANSI escape sequences, no terminal control characters.

### 4. Reference: Jest example

**Standard output** — as shown in the Motivation section above.

**`jest --llm` output:**

```
FAIL 2/50

--- src/auth.test.ts:42 "should reject expired tokens"
expected: 401
received: 200

--- src/auth.test.ts:67 "should refresh token on 403"
TypeError: refreshToken is not a function
  at handleResponse (src/api.ts:15)
```

**Design notes:**

| Decision | Rationale |
|---|---|
| No PASS lines | Agent only needs to act on failures |
| No code context | Agent can read the file itself if needed |
| One-level stack | Top user-code frame is almost always sufficient |
| Quoted test name | Helps the agent match output to specific test |
| `expected`/`received` preserved | Core diagnostic info for assertion failures |

### 5. Extensibility

The `--llm` flag and `LLM_OUTPUT` variable start as simple booleans. Future evolution MAY support values for different profiles:

```bash
jest --llm=compact     # minimal (default, equivalent to --llm)
jest --llm=verbose     # more context, still LLM-optimized
jest --llm=json        # structured JSON, but filtered for relevance
```

```bash
LLM_OUTPUT=compact
LLM_OUTPUT=verbose
```

Tools SHOULD treat `--llm` with no value and `LLM_OUTPUT=1` as equivalent to the default compact mode. Tools SHOULD treat unrecognized values as the default compact mode rather than erroring.

This extensibility is reserved for future use. Implementations today SHOULD only support the boolean form.

## Adoption path

This convention succeeds through voluntary adoption, not standardization. The suggested path:

1. **Build reference implementations** — Jest, ESLint, and pytest are high-impact starting points due to their frequent use in AI-assisted development.
2. **Propose upstream PRs** to popular tools, or ship as wrapper plugins.
3. **Agent harnesses adopt `LLM_OUTPUT`** — Claude Code, OpenCode, Codex, and similar tools set the env var by default.

Community contributions for additional tools are welcome.

## FAQ

**Why not just post-process existing output?**
Agent harnesses could strip output, but that wastes tokens (the LLM still receives and processes the noise) and is fragile against format changes. The tool itself knows best what's signal and what's decoration.

**Why a new flag instead of `--json` with filtering?**
JSON is designed for programmatic traversal. LLM output is designed for sequential reading. They are different targets with different optimal formats. Bolting LLM optimization onto JSON would produce worse results than a purpose-built text format.

**Will this fragment CLI output into too many modes?**
Most tools already support 2-4 output modes (default, `--verbose`, `--quiet`, `--json`). Adding one more is marginal complexity for significant value. The modes also map to clearly distinct consumers: humans (default/verbose), scripts (json/quiet), and LLMs (llm).

**Why `--llm` and not `--ai`?**
`--ai` is too broad and likely to conflict. The target consumer is specifically large language models reading output as text context. `--llm` is precise, has no known conflicts, and is immediately understood.

**What about non-LLM AI agents?**
The output style guide optimizes for text-in-context consumption, which applies to any system reading plain text — LLMs, future reasoning engines, or human-in-the-loop pipelines. The name `--llm` is chosen for clarity today; the convention is useful beyond the name.

## References

- `--verbose` / `--quiet` — de facto CLI conventions, no formal spec
- `git --porcelain` — precedent for alternative output modes targeting non-human consumers
- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) — AI coding agent that executes CLI tools
- [OpenAI Codex CLI](https://github.com/openai/codex) — AI coding agent
