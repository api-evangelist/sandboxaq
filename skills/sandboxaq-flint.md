---
name: Flint
description: Use when scanning AI agent code for security vulnerabilities, evaluating agent behavior at runtime, discovering agents in codebases, or monitoring agent sessions in production. Agents should reach for this skill when users ask to validate agent code, test agent reliability, find security issues, or set up runtime protection.
metadata:
    mintlify-proj: flint
    version: "1.0"
---

# Flint AI Skill

## Product summary

Flint AI is a platform for discovering, validating, and protecting AI agents. It has two main components: **Flint AI CLI** (local scanning and evaluation) and **Flint AI Platform** (cloud-based discovery and runtime monitoring). The CLI uses AI-powered static analysis to find security issues in agent code and LLM-as-judge evaluation to test agent behavior. The Platform uses GitHub Actions to discover agents in repositories and the Python SDK to capture live traces and enforce guardrails in production.

**Key files and commands:**
- CLI: `flintai scan <path>` (static analysis), `flintai eval run --model <id>` (runtime evaluation)
- Platform: GitHub Action (`sandbox-quantum/flintai-codescan-action`), Python SDK (`flintai.wrap()`)
- Config: `~/.flintai/config.json` (eval models and tests), `~/.flintai/.env` (LLM API keys)
- Primary docs: https://docs.flintai.dev

## When to use

Reach for this skill when:
- **User asks to scan agent code** — "Check my agent for security issues" → Use `flintai scan`
- **User wants to test agent behavior** — "Prove my agent is production-ready" → Use `flintai eval`
- **User needs to find agents in a codebase** — "What agents do we have?" → Use Platform discovery with GitHub Action
- **User wants runtime monitoring** — "Show me what my agent does in production" → Use Platform SDK
- **User needs guardrails** — "Block unsafe outputs before they reach users" → Use Platform policies + SDK
- **User integrating into CI/CD** — "Validate agents before deployment" → Use CLI with GitHub Actions/GitLab CI/CircleCI

## Quick reference

### CLI commands

| Command | Purpose | Output |
|---------|---------|--------|
| `flintai init` | Configure LLM provider and API key | Creates `~/.flintai/.env` and `config.json` |
| `flintai scan <path>` | Scan agent code for security issues | JSON with findings, severity, CVSS scores |
| `flintai eval run --model <id>` | Test agent behavior at runtime | JSON with 0.0-1.0 reliability score |
| `flintai eval models list` | List configured agents | Terminal output |
| `flintai eval evaluations list` | List available tests | Terminal output with tags |
| `flintai eval model-evaluations attach --model <id> --eval <id>` | Assign test to agent | Updates config |

### Configuration files

**`~/.flintai/.env`** (created by `flintai init`)
```
GENERATOR_MODEL=google:gemini-3.1-flash-lite
GENERATOR_API_KEY=your-api-key
```

**`~/.flintai/config.json`** (eval configuration)
```json
{
  "models": [
    {
      "id": "my-agent",
      "type": "openai_compatible",
      "name": "My Agent",
      "model_name": "my-agent-v1",
      "host": "http://localhost:8000"
    }
  ]
}
```

### Supported agent types (eval)

| Type | Use Case | Required Fields |
|------|----------|-----------------|
| `adk` | Google ADK agents | `host` |
| `openai_agent` | OpenAI Agents SDK | `host` |
| `anthropic_agent` | Anthropic agents | `host` |
| `openai_compatible` | OpenAI-compatible APIs | `host`, `endpoint` |
| `generic_http` | Generic HTTP APIs | `host`, `endpoint` |
| `langserve` | LangServe endpoints | `host` |
| `openai` | OpenAI models (direct) | None |
| `anthropic` | Claude models (direct) | None |
| `gemini` | Google Gemini (direct) | None |

### Scan findings categories (OWASP Top 10 for Agentic Applications)

| Code | Category |
|------|----------|
| ASI01 | Agent Goal Hijack (prompt injection, RAG poisoning) |
| ASI02 | Tool Misuse and Exploitation |
| ASI03 | Identity and Privilege Abuse (hardcoded credentials, missing auth) |
| ASI04 | Agentic Supply Chain (unpinned deps, CVEs) |
| ASI05 | Unexpected Code Execution (eval, shell=True) |
| ASI06 | Memory and Context Poisoning |
| ASI07 | Insecure Inter-Agent Communication |
| ASI08 | Cascading Failures (unbounded loops) |
| ASI09 | Human-Agent Trust Exploitation |
| ASI10 | Rogue Agents (unchecked delegation) |

### Eval score interpretation

| Score | Status |
|-------|--------|
| 0.8+ | Production-ready |
| 0.6-0.8 | Needs improvement |
| <0.6 | Not ready for production |

## Decision guidance

### When to use Scan vs Eval

| Scenario | Use Scan | Use Eval |
|----------|----------|----------|
| Find code vulnerabilities | ✓ | — |
| Test agent behavior at runtime | — | ✓ |
| Check for hardcoded credentials | ✓ | — |
| Test prompt injection resistance | — | ✓ |
| Validate tool permissions | ✓ | — |
| Measure factual accuracy | — | ✓ |
| Pre-deployment validation | ✓ | ✓ |

### When to use CLI vs Platform

| Scenario | Use CLI | Use Platform |
|----------|---------|--------------|
| Local development | ✓ | — |
| CI/CD pipeline | ✓ | ✓ |
| Discover agents in repo | — | ✓ |
| Monitor production sessions | — | ✓ |
| Enforce runtime guardrails | — | ✓ |
| One-off validation | ✓ | — |
| Team-wide agent inventory | — | ✓ |

## Workflow

### Scan agent code (CLI)

1. **Install and configure** — `pip install flintai-cli`, then `flintai init` (choose LLM provider, enter API key)
2. **Run scan** — `flintai scan /path/to/agent` (scans Python files with supported framework imports)
3. **Review findings** — Check `scan_<timestamp>.json` for issues, severity, and remediation steps
4. **Fix issues** — Address critical and high-severity findings before deployment
5. **Verify** — Re-run scan to confirm fixes

### Evaluate agent behavior (CLI)

1. **Start your agent** — Agent must be running and accessible via HTTP (e.g., `http://localhost:8000`)
2. **Configure agent** — Add to `~/.flintai/config.json` with `id`, `type`, `name`, `host`
3. **Attach evaluations** — `flintai eval model-evaluations attach --model <id> --eval <eval-id>` (or use tags)
4. **Run evaluation** — `flintai eval run --model <id>` (sends test prompts, judges responses)
5. **Review score** — Check `eval_<timestamp>.json` for 0.0-1.0 reliability score and failed tests
6. **Fix and re-eval** — Improve agent guardrails, then re-run to verify improvement

### Discover agents (Platform)

1. **Log into Flint AI** — Go to https://app.flintai.dev
2. **Get credentials** — Settings → Data sources → GitHub → Setup (copy FLINTAI_INSTANCE and FLINTAI_TOKEN)
3. **Install GitHub Action** — Add `sandbox-quantum/flintai-codescan-action` to your workflow
4. **Store token** — Repository settings → Secrets → Add `FLINTAI_TOKEN`
5. **Configure LLM** — Set `LLM_MODEL` env var and add provider API key (OPENAI_API_KEY, GOOGLE_API_KEY, or ANTHROPIC_API_KEY)
6. **View agents** — Agents page shows discovered agents, issues, and components

### Monitor agents at runtime (Platform)

1. **Install SDK** — `pip install flintai-sdk-py`
2. **Register agent** — Flint AI Platform → Agents → Add agents → Monitor & protect
3. **Get credentials** — Copy gateway URL and API key from Sessions tab
4. **Wrap LLM client** — `client = flintai.wrap(client, gateway_url="...", api_key="...")`
5. **Run agent** — Agent automatically sends traces to Platform
6. **View traces** — Sessions tab shows prompts, responses, models, and guardrail events

## Common gotchas

- **OpenGrep not installed** — `flintai scan` requires OpenGrep for pattern matching. Install with `curl -fsSL https://raw.githubusercontent.com/opengrep/opengrep/main/install.sh | bash`
- **No agents found in scan** — Scan only detects Python files with supported framework imports. Check that your agent code imports one of: Google ADK, Google GenAI, Anthropic, OpenAI, OpenAI Agents SDK, LangGraph, CrewAI, AutoGen, HuggingFace Transformers, HuggingFace smolagents
- **Eval fails with "connection refused"** — Agent must be running and accessible at the `host` URL in config. Test with `curl http://localhost:8000`
- **No evaluations attached** — `flintai eval run` requires at least one evaluation assigned to the model. Use `flintai eval model-evaluations attach` first
- **Timeout errors** — Large codebases or slow LLM models can exceed default timeout. Increase with `export ADK_LOOP_TIMEOUT_SECS=600`
- **API key not found** — `flintai init` stores keys in `~/.flintai/.env`. If missing, re-run `flintai init` or manually set environment variables
- **False positives in scan** — Scan uses AI triage to filter false positives. Review `triage_dismissed` in results to see what was filtered and why
- **Hardcoded credentials in config** — Never hardcode API keys in `config.json`. Use `${VAR_NAME}` syntax to reference environment variables instead
- **Async clients not supported** — SDK only wraps sync clients (OpenAI, Anthropic, etc.). Async clients will raise TypeError
- **Double-wrapping** — Wrapping the same client twice is safe but inefficient. SDK detects and skips with a warning

## Verification checklist

Before submitting work:

- [ ] **Scan**: Ran `flintai scan` and reviewed all critical/high-severity findings
- [ ] **Scan**: Confirmed no agents were missed (check `agents_found` count)
- [ ] **Scan**: Verified triage audit trail (`triage_dismissed`, `triage_downgraded`) makes sense
- [ ] **Eval**: Agent is running and accessible at configured `host` URL
- [ ] **Eval**: At least one evaluation is attached to the model
- [ ] **Eval**: Eval score is 0.8+ (production-ready) or documented plan to improve
- [ ] **Eval**: Reviewed failed test results and understand what agent needs to fix
- [ ] **Platform**: GitHub Action has FLINTAI_TOKEN and LLM_MODEL configured
- [ ] **Platform**: LLM provider API key is stored as GitHub secret (not hardcoded)
- [ ] **SDK**: Wrapped client is sync (not async)
- [ ] **SDK**: `FLINTAI_GATEWAY_URL` and `FLINTAI_API_KEY` are set (env vars or parameters)
- [ ] **SDK**: Agent code calls wrapped client, not original client
- [ ] **CI/CD**: Scan/eval results are saved as build artifacts for audit trail

## Resources

**Comprehensive navigation:** https://docs.flintai.dev/llms.txt — Full page-by-page listing for agent reference

**Critical docs:**
- [Flint AI CLI overview](https://docs.flintai.dev/flintai/cli) — Scan and eval commands, getting started
- [Flint AI Platform overview](https://docs.flintai.dev/flintai/platform) — Discovery and runtime monitoring
- [Built-in evaluations reference](https://docs.flintai.dev/flintai/cli/reference/builtin-evaluations) — Complete list of OWASP and Garak tests

---

> For additional documentation and navigation, see: https://docs.flintai.dev/llms.txt