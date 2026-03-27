---
name: backlink-screening
description: "Use when the user provides a backlink or domain CSV/list and wants it screened through a repeatable three-round workflow: first-round Semrush bulk analysis, second-round domain overview review, and optional third-round manual site quality review with organized result files."
---

# Backlink Screening

Use this skill when the user sends a backlink list, domain list, or CSV and asks to filter, review, or screen candidates for outreach or link building.

This skill is for repeated operational work. The agent should run the requested rounds end to end, organize outputs into a dedicated run folder, troubleshoot ordinary blockers, and return every round's result files to the user.

## Trigger Conditions

Trigger this skill when the user:
- uploads or mentions a CSV of backlinks or domains
- says things like `筛选外链`、`筛选域名`、`跑一轮外链筛选`、`按默认规则筛选`
- wants Semrush-based filtering plus manual website review
- asks to continue from round 1 to round 2 or round 3

Do not use this skill for general SEO discussion with no concrete list to process.

## Required Tools

- Use [$playwright-stealth-cli](/Users/mac/.codex/skills/playwright-stealth-cli/SKILL.md) for all browser automation.
- Stay in CLI workflow. Do not switch to ordinary Playwright workflow.
- Use branded Chrome with `--channel chrome`, not default Chromium.
- Prefer a persistent Chrome profile stored in the current project directory.

## Required Platform

This workflow must use the `3ue` platform for Semrush access.

Required platform URLs:
- login/home: `https://dash.3ue.com/zh-Hans/#/page/m/home`
- Semrush entry: `https://sem.3ue.com/`

Do not replace `3ue` with another Semrush mirror or another SEO platform unless the user explicitly changes the requirement.

## Credential Handling

On the first run for a given workspace or project:
- ask the user for the `3ue` account and password before starting
- do not assume credentials

After the user provides credentials:
- save them for reuse in a local project credential file so later runs do not need to ask again
- always use the fixed project-local path `.auth/3ue-credentials.json`
- keep the saved credentials tied to this workflow and reuse them on future runs automatically

Credential file rules:
- create the `.auth/` directory under the current project if it does not exist
- save credentials in JSON format
- use this structure:

```json
{
  "platform": "3ue",
  "login_url": "https://dash.3ue.com/zh-Hans/#/page/m/home",
  "semrush_url": "https://sem.3ue.com/",
  "username": "the-user-account",
  "password": "the-user-password"
}
```

- on later runs, read `.auth/3ue-credentials.json` first
- if the file exists and login succeeds, do not ask the user again
- if the file exists but login fails, ask the user for updated credentials and overwrite the file

If the user already provided credentials earlier in the same project, do not ask again unless login fails or the user requests a change.

## Default Confirmation

If the user does not provide explicit screening rules, pause before running and ask:

`是否使用默认规则筛选？默认规则如下：`

- 第一轮：访问量 `>= 35000`
- 第二轮：`Authority Score >= 35`，`自然流量 >= 100000`，不能标记 `链接农场`，不能标记 `些许垃圾迹象`
- 第三轮：人工开站复核，结论为 `通过 / 拒绝 / 存疑`

If the user provides custom rules, use them instead of defaults. Do not silently mix custom rules with default rules unless the user explicitly asks for that.

## Run Folder Organization

Before running, create a dedicated run folder inside the current project. Use a clear timestamped name, for example:

- `runs/2026-03-27-backlink-screening/`

Use subfolders:
- `input/`
- `round1-bulk-analysis/`
- `round2-domain-overview/`
- `round3-site-review/`
- `final/`

Copy or generate working files into the run folder rather than overwriting the user's original file. Keep raw exports and filtered outputs from every round.

## Workflow

### 1. Intake

1. Read the user-provided file or list.
2. Identify whether the input is backlinks, root domains, or URLs that need domain extraction.
3. Check whether a saved `3ue` credential file already exists in the current project.
4. The credential file path must be `.auth/3ue-credentials.json`.
5. If the credential file does not exist, ask the user for the `3ue` account and password, then save them in that JSON file for reuse.
6. If the credential file exists, try it first.
7. If login fails with saved credentials, ask the user for updated credentials and overwrite the file.
8. Confirm rules if the user did not provide them.
9. Count valid rows.
10. If the first-round bulk analysis input exceeds `100` entries, split it into batches of at most `100`.

### 2. Round 1: Semrush Bulk Analysis

Use Semrush `流量与市场 -> 行业与批量分析`.

Workflow:
1. Open the platform with persistent Chrome profile in the project directory.
2. Log in to `3ue` with the saved project credentials if needed.
3. Navigate to the batch analysis page.
4. Upload the current batch CSV.
5. Treat upload as successful only if the left-side rich input area has been auto-filled with domains.
6. Do not trust the right-side `文件已上传` message by itself.
7. After the left-side area is filled, click the rich input area once to trigger focus.
8. Only after that, click `分析`.
9. Wait for readable results.
10. Export raw batch results.
11. Merge all batch results if there are multiple batches.
12. Apply round-1 filtering.

Round-1 default rule:
- `访问量 >= 35000`

Round-1 hard requirements:
- if left-side domains did not appear, do not click `分析`; re-upload or reload first
- if a file has more than `100` entries, split it before submission
- if tab switching happened, re-check the current page before clicking anything

### 3. Round 2: Domain Overview Review

Only use the `域名概览` page.

Workflow:
1. Take the domains that passed round 1.
2. Open each domain in `域名概览`.
3. Review one domain at a time.
4. Record the result immediately after each domain.
5. Read:
   - `Authority Score`
   - `自然流量`
   - risk markers shown near the `Authority Score` block
6. Use only what is shown on the `域名概览` page, especially the `Authority Score` block, to decide risk markers.
7. Filter according to the active round-2 rule set.

Round-2 default rule:
- `Authority Score >= 35`
- `自然流量 >= 100000`
- no `链接农场`
- no `些许垃圾迹象`

### 4. Round 3: Manual Site Review

Take the websites that passed round 2 and manually inspect the live sites.

Workflow:
1. Open the homepage.
2. Open at least two more internal pages.
3. Prefer a mix of:
   - article/content page
   - category/archive page
   - About, Contact, or Privacy page
4. Judge the site as one of:
   - `通过`
   - `拒绝`
   - `存疑`
5. Record visited pages and notes immediately after each site.
6. Make sure at least one checked page provides a meaningful recency signal, such as a publish date or update date, whenever the site exposes one.

Round-3 review criteria:
- site opens normally
- homepage and inner pages look like one coherent operating website
- content feels like real articles/news, not stitched filler
- site has basic legitimate pages such as About, Contact, Privacy
- site appears recently updated
- topic and sections are coherent instead of random

Likely `通过`:
- clear brand or positioning
- homepage, category, and article pages feel consistent
- readable content that does not look mass-generated
- recent updates
- looks like a living news, magazine, niche, tool, or small content site

Likely `拒绝`:
- obvious content farm or fake news/magazine site
- extremely mixed topics with no positioning
- heavy celebrity/net worth/family gossip filler
- adult, gambling, or gray-market content
- obvious paid-post or sponsored soft-article site
- repeated page titles/body templates
- looks like lead-gen, sales landing pages, or low-trust utility shells

Likely `存疑`:
- site does not open
- stuck on CAPTCHA or Cloudflare checks
- too little content to judge
- suspicious but not enough evidence
- unstable site state prevents review

## Expected Agent Behavior

- Run the full workflow end to end when feasible.
- Do not stop at planning if the user asked to run the task.
- Troubleshoot ordinary blockers yourself:
  - stale refs
  - tab switching issues
  - upload worked but batch input did not focus
  - slow page loads
  - retriable navigation issues
- After `tab-select`, verify the current page again before acting. Prefer `snapshot`, `screenshot`, or URL confirmation.
- Prefer CLI-first recovery tactics before asking the user for help.
- If a platform limitation or account restriction genuinely blocks progress, explain the blocker briefly and state exactly what was already completed.
- Close the CLI-controlled browser at the end and verify with `playwright-stealth list` that the session is `closed`.

## Known Pitfalls

- `tab-select` can succeed while the visible page state is still stale. Reconfirm URL or page content before clicking.
- On the round-1 page, `文件已上传` alone is not enough. The left-side rich input must show the domains, or analysis may return no results.
- The round-1 rich input behaves like a rich-text/tag editor. After upload and left-side fill, click it once to trigger focus before clicking `分析`.
- Use the same persistent profile for the whole run. Mixing fresh sessions increases login drift and inconsistent page behavior.
- `3ue` may show a device-limit page. If that happens, check for leftover sessions first, then ask the user to free another device only if the limit is still active.
- Red console or network errors do not automatically mean the task failed; judge by whether the page produced the required result.

## Output Requirements

Always deliver all round outputs, not only the final file.

At minimum, provide:
- round-1 raw or merged result file
- round-1 filtered result file
- round-2 full review result file
- round-2 filtered result file
- round-3 full manual review result file if round 3 was run
- final shortlist file

In the response, summarize:
- which rules were used
- total counts per round
- pass counts per round
- any missing or unreadable sites

## Notes

- Keep browser profile under the current project directory, not a random temporary path, unless the user explicitly wants a temporary session.
- Keep `3ue` credentials in `.auth/3ue-credentials.json` so future runs in the same project can reuse them.
- Use the same persistent profile throughout a run for stability.
- If the user only asks for round 1 or round 2, run only the requested scope, but still organize files in the run folder.
- When manual review is requested, do not replace page-by-page review with a blind batch script.
