# open-lovable review notes

Reviewed at upstream commit `806b8072f28afea11fa2b028cac1449eff74c511` (merge of PR #229, `bump-next-15.5.27`), shallow clone, read-only. Dependencies were not installed and the app was not built or run, so these findings come from reading the code.

## Security

1. **No authentication on any API route (high).** There is no `middleware.ts` and no auth check in the routes. Routes that spend credentials include `generate-ai-code-stream` (LLM keys), `scrape-url-enhanced` and `scrape-screenshot` (Firecrawl), and `create-ai-sandbox*` (sandbox quota). On a publicly reachable deployment, anyone can run up the operator's bills. Run locally, or put auth in front of it.

2. **Shared global sandbox (medium).** `run-command` and related routes read `global.activeSandbox`. With more than one user, they all act on the same sandbox, so one user can read or change another's generated files and run commands in it.

3. **Arbitrary command execution in the sandbox (medium, by design).** `run-command` splits the submitted string on whitespace and passes the first token as `cmd`. It does not use a shell, so shell metacharacters don't inject, but any binary the sandbox has can run. Acceptable only inside a disposable sandbox and only with the auth issue above fixed.

4. **Scraping arbitrary URLs with no allowlist (low).** `scrape-url-enhanced` forwards any `url` to Firecrawl without validation. Firecrawl does the fetch, so this is mostly a credit-spend issue rather than internal SSRF, but it still allows abuse of the operator's Firecrawl quota.

5. **No secrets found in tracked files.** A regex scan for common key formats (OpenAI-style `sk-`, AWS `AKIA`, Firecrawl-style `fc-`) found nothing. `.env.example` contains only placeholders.

## Quality

6. **Test scripts point at missing files.** `package.json` defines `test:api` and `test:code` (both run `tests/*.test.js`, which doesn't exist) and `test:all`, which calls `test:integration` (not defined). `pnpm test:all` fails.

7. **Stray line in `.env.example`.** Line 25 is a bare `=======`, a leftover merge marker. It's harmless to dotenv but misleading.

8. **Two lockfiles.** `bun.lock`, `package-lock.json` and `pnpm-lock.yaml` are all committed. Pick one package manager and remove the rest to avoid version drift.

9. **Regex-based code parsing.** Generated code is extracted with many regexes (`apply-ai-code*`, `generate-ai-code-stream`, `page.tsx`). Model output that drifts from the expected format can silently drop files. Worth a schema or a stronger parser if it's kept.

10. **Setup docs disagree.** `README.md` lists only the direct provider keys. `.env.example` also documents `AI_GATEWAY_API_KEY` and `E2B`. The README should be brought in line with `.env.example`.

## Verdict for installing as a skill

Reasonable as a local development tool and as a reference for building a Lovable-style app with Firecrawl. Not suitable for public hosting without authentication, a per-user sandbox model, and a spending cap.
