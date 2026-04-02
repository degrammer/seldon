<p align="center">
  <img src="seldon.png" alt="Seldon" width="200">
  <br><br>
  <em>Analyzes implementation plans the way Hari Seldon analyzed civilizations — by checking structural assumptions against reality before things go wrong.</em>
  <br><br>
  <a href="#install">Install</a> &bull;
  <a href="#usage">Usage</a> &bull;
  <a href="#focus-modes">Focus Modes</a> &bull;
  <a href="#external-judges">External Judges</a> &bull;
  <a href="#output-schema">Output Schema</a>
</p>

---

Feed Seldon a plan, spec, or proposal. It reads the document, inspects your codebase for evidence, and returns a verdict: **approve**, **approve with changes**, or **request major revision**. Every finding includes severity, evidence from your workspace, and file references with line numbers.

Works out of the box as an inline skill for [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Plug in [Codex](#codex), [OpenAI](#openai-api), or [Anthropic](#anthropic-api) as an external judge for true model independence. Or [write your own](#writing-your-own-judge).

## Install

**Claude Code plugin (recommended):**

```bash
# Download the latest release and install
curl -sL https://github.com/degrammer/seldon/releases/latest/download/seldon-1.0.0.zip -o seldon.zip
claude plugin install ./seldon.zip
```

Or download from the [Releases](https://github.com/degrammer/seldon/releases) page and run `claude plugin install ./seldon-*.zip`.

**Via skills.sh:**

```bash
npx skills add degrammer/seldon
```

**Manual:**

```bash
# Clone into your Claude Code skills directory
git clone https://github.com/degrammer/seldon.git ~/.claude/skills/seldon
```

**Project-level** (shared with your team):

```bash
git clone https://github.com/degrammer/seldon.git .claude/skills/seldon
```

## Usage

```
/seldon my-plan.md
/seldon --focus architecture docs/migration-plan.md
/seldon --focus safety spec.md supporting-context.md
```

### What happens

1. Seldon reads your plan file (and any supporting files)
2. Inspects the workspace to verify claims — file paths, APIs, dependencies, config, schema
3. Evaluates against a rubric: repo fit, correctness, sequencing, evaluation, safety
4. Returns a structured verdict with a visual confidence bar:

```
🟡 Confidence  ████████████████░░░░  0.82  (moderate)
```

### Example output

```
Judge: codex via judge-runner.sh
Verdict: approve_with_changes

Summary: Plan is sound but assumes a migration path that doesn't exist yet.

🟡 Confidence  ████████████████░░░░  0.82  (moderate)

Strengths:
- Clear phasing with realistic scope per step
- Good rollback strategy for the data migration

Blocking findings:

  high — Migration depends on schema v3 which hasn't been created
  Why: Step 2 cannot begin without this prerequisite
  Evidence: No v3 migration file exists in prisma/migrations/
  Refs: prisma/schema.prisma:42, docs/plan.md:18

Open questions:
- Is the external billing API rate limit sufficient for the proposed batch size?
```

## Focus Modes

Focus modes weight the review toward specific concerns. Default is `balanced`.

| Mode | Emphasis |
|------|----------|
| `balanced` | All rubric dimensions evenly |
| `architecture` | Service boundaries, dependencies, migration risk, hidden integration work |
| `evaluation` | Success criteria, regression detection, testability of quality claims |
| `product` | User-visible failure modes, sequencing, scope realism |
| `operations` | Rollout, alerting, rollback, failure handling, maintenance burden |
| `safety` | Privacy, security, hallucination controls, access assumptions |

```bash
/seldon --focus safety docs/auth-redesign.md
```

## External Judges

By default, Seldon runs **inline** — the current agent performs the review. For true model independence, plug in an external judge.

### Setup

1. Pick a judge from the `judges/` directory
2. Copy it to `judge-runner.sh` in the skill root
3. That's it — Seldon auto-detects it

```bash
cd ~/.claude/skills/seldon  # or .claude/skills/seldon

# Option A: Codex (requires @openai/codex CLI)
cp judges/codex.sh judge-runner.sh

# Option B: OpenAI API (requires OPENAI_API_KEY)
cp judges/openai.sh judge-runner.sh

# Option C: Anthropic API (requires ANTHROPIC_API_KEY)
cp judges/anthropic.sh judge-runner.sh
```

To switch back to inline review, remove `judge-runner.sh`.

### Codex

Uses the OpenAI Codex CLI with full workspace access (read-only sandbox) and schema-enforced structured output.

**Prerequisites:**
```bash
npm install -g @openai/codex
```

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `JUDGE_MODEL` | `gpt-5.4` | Codex model |
| `JUDGE_REASONING` | `xhigh` | Reasoning effort |
| `JUDGE_WEB_SEARCH` | `cached` | Web search mode |

### OpenAI API

Direct API call to OpenAI Chat Completions. Sends plan content in the prompt (no workspace access).

**Prerequisites:**
```bash
export OPENAI_API_KEY=sk-...
```

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `JUDGE_MODEL` | `gpt-4o` | Model to use |

### Anthropic API

Uses a separate Claude instance for a second opinion within the Anthropic ecosystem.

**Prerequisites:**
```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

| Environment Variable | Default | Description |
|---------------------|---------|-------------|
| `JUDGE_MODEL` | `claude-sonnet-4-6-20250514` | Model to use |

### Writing Your Own Judge

Create a `judge-runner.sh` that:

1. Accepts `[--focus <mode>] <plan-file> [supporting-files...]`
2. Outputs JSON matching `seldon.schema.json` to stdout
3. Exits 0 on success, non-zero on failure (errors to stderr)

See `judges/codex.sh` for a full reference implementation.

## Output Schema

See [`seldon.schema.json`](seldon.schema.json) for the complete JSON Schema. Structure:

```json
{
  "verdict": "approve | approve_with_changes | request_major_revision",
  "summary": "1-3 sentence assessment",
  "confidence": 0.82,
  "strengths": ["..."],
  "blocking_findings": [
    {
      "severity": "critical | high | medium | low",
      "title": "Short description of the issue",
      "why_it_matters": "Impact if unaddressed",
      "evidence": "What was found in the workspace",
      "references": ["src/api.ts:42", "docs/plan.md:18"]
    }
  ],
  "non_blocking_findings": [],
  "open_questions": ["Things that couldn't be verified locally"]
}
```

### Confidence ranges

| Range | Label |
|-------|-------|
| 0.90 - 1.00 | 🟢 High confidence |
| 0.70 - 0.89 | 🟡 Moderate confidence |
| 0.50 - 0.69 | 🟠 Low confidence |
| 0.00 - 0.49 | 🔴 Very low confidence |

## Compatibility

Seldon works with any agent that supports `SKILL.md` skills:

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
- [Gemini CLI](https://github.com/google-gemini/gemini-cli)
- Any agent supporting the skills.sh ecosystem

## Name

Named after [Hari Seldon](https://en.wikipedia.org/wiki/Hari_Seldon) from Isaac Asimov's *Foundation* series. Seldon developed psychohistory — a science that predicted the future of civilizations by analyzing structural assumptions against reality. At critical decision points, a holographic Seldon would appear and say: *"If you're seeing this, here's what you got wrong."*

That's what `/seldon` does for your implementation plans.

## License

MIT
