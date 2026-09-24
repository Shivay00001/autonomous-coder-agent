# autonomous-coder-agent

Internal batch tooling for cleaning up and documenting the owner's own GitHub
repositories. It is a single script (`autonomous_coder_agent.py`) — not a product,
not a framework.

## What it actually does

1. Lists repos for the authenticated `gh` CLI user.
2. Clones each repo not yet recorded in `processed_repos.json`.
3. Runs a syntax/build check (Python `py_compile`, Node `npm run build` if present).
4. If the check fails, sends the failing file + error to an LLM and asks for a fix,
   then re-checks (bounded retries).
5. Asks the LLM to write a fresh `README.md` based on the repo's actual code.
6. Commits and **force-pushes to `main`**, records the repo as processed, deletes
   the local clone.

## Setup

```bash
pip install openai g4f
gh auth login
export AUTONOMOUS_CODER_API_KEY="<your-key>"
python autonomous_coder_agent.py
```

The API key is read from the `AUTONOMOUS_CODER_API_KEY` environment variable —
never hardcode it in the script. (Older commits in this repo's git history contain
hardcoded keys; those keys must be treated as compromised and rotated. History was
intentionally not rewritten.)

## Model configuration

- Default base URL: `https://api.hcnsec.cn/v1` (a third-party OpenAI-compatible
  relay), default model `kimi-k3`. Point `API_BASE_URL`/`MODEL_NAME` at your own
  provider before running.
- Fallback chain: primary OpenAI-compatible client → g4f → Pollinations →
  AiHubMix → HuggingFace inference. `g4f` is an unofficial community package, not
  affiliated with OpenAI; treat its outputs accordingly.

## Limitations and warnings

- **Destructive by design**: it force-pushes to `main`. Only run it against
  repositories you own, and only with a clean backup/remote you can recover from.
- Fix quality depends entirely on the model; every auto-fix should be reviewed in
  the commit history.
- Requires `gh` CLI authentication and network access to your LLM provider.
- State lives in local `processed_repos.json` / `quality_report.txt`; deleting
  them re-processes everything.
