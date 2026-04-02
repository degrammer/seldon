# Plan: Add Gemini Judge Support to Seldon

## Summary

Add a new external judge runner for Google Gemini, allowing teams to use Gemini as an independent reviewer alongside the existing OpenAI and Anthropic judges.

## Phases

### Phase 1 — Create the Gemini judge runner

Create `judges/gemini.sh` following the same pattern as `judges/openai.sh`:

- Accept `[--focus <mode>] <plan-file> [supporting-files...]`
- Use the Gemini REST API via `curl`
- Require `GEMINI_API_KEY` environment variable
- Default model: `gemini-2.5-pro`
- Read the schema from `seldon.schema.json` for structured output
- Return JSON matching the schema to stdout

### Phase 2 — Update the skill configuration

- Update `SKILL.md` in the project root to mention Gemini as a judge option
- Add Gemini to the judge detection logic in `skills/seldon/judge-runner.sh`
- Update the `config.yaml` to register the new judge provider

### Phase 3 — Update documentation

- Add a Gemini section to `README.md` under External Judges
- Add environment variable table (JUDGE_MODEL default: gemini-2.5-pro)
- Update the install instructions in `docs/setup-guide.md`

### Phase 4 — Testing

- Add integration tests in `tests/judges/gemini.test.sh`
- Validate JSON output against `seldon.schema.json`
- Test all 6 focus modes with the Gemini judge
- Run the existing test suite in `tests/` to ensure no regressions

## Dependencies

- Google Cloud SDK for authentication fallback
- `jq` for JSON parsing (already in the project's `package.json` dependencies)
- Python 3.x for JSON escaping (already used by other judges)

## Rollout

1. Merge to `develop` branch for internal testing
2. Update `CHANGELOG.md` with the new feature
3. Tag a new release to trigger the CI/CD pipeline in `.github/workflows/ci.yml`
