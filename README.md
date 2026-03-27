# backlink-screening

A Codex skill for repeatable backlink screening through a three-round workflow:

- Round 1: Semrush bulk analysis on the `3ue` platform
- Round 2: Semrush domain overview review
- Round 3: manual live-site review

The skill is designed for recurring operational use. It confirms default rules with the user when needed, reuses project-local `3ue` credentials, organizes each run into a timestamped folder, and returns every round's output files instead of only the final shortlist.

## Included Files

- `SKILL.md`
- `agents/openai.yaml`

## Default Workflow

1. Ask whether to use the default screening rules if the user did not provide rules.
2. Run first-round batch analysis with `访问量 >= 35000`.
3. Run second-round domain overview filtering with:
   - `Authority Score >= 35`
   - `自然流量 >= 100000`
   - no `链接农场`
   - no `些许垃圾迹象`
4. Optionally run third-round manual review and classify each site as `通过` / `拒绝` / `存疑`.
5. Return all round outputs plus the final shortlist.

## Platform Requirement

This skill is written specifically for the `3ue` Semrush workflow:

- `https://dash.3ue.com/zh-Hans/#/page/m/home`
- `https://sem.3ue.com/`

## Credentials

The skill stores project-local credentials at:

- `.auth/3ue-credentials.json`

## Installation

Copy the `backlink-screening` skill folder into your Codex skills directory, for example:

```bash
mkdir -p "${CODEX_HOME:-$HOME/.codex}/skills"
cp -R backlink-screening "${CODEX_HOME:-$HOME/.codex}/skills/"
```
