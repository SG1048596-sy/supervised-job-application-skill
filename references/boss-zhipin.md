# BOSS Zhipin Playbook

Use this reference when working on BOSS Zhipin/Boss直聘.

## Initial Use Checklist

Before opening BOSS, ask or confirm:

```text
目标岗位关键词是什么？
目标城市是哪里？
最低接受薪资是多少？
经验范围是什么？
使用哪份简历作为匹配依据？
今天最多投递/沟通几个岗位？
是否允许每个岗位确认后点击“立即沟通”？
```

If role, city, salary floor, or resume source is missing, ask before searching. If the user asks for bulk automatic delivery, reframe the task as supervised shortlist plus per-job confirmation.

## Validation Run Before Formal Applications

After intake is complete and before formal applications, proactively suggest a dry-run on an unrelated role/company.

Purpose:

- Verify that BOSS search works in the controlled browser.
- Verify job-card extraction, JD extraction, salary glyph normalization, and recruiter/company parsing.
- Verify button-state detection such as `立即沟通`, `继续沟通`, `已发送`, app prompt, or resume-completion prompt.
- Avoid burning a real target job during setup.

Default dry-run rule:

- Use an unrelated role such as `咖啡师`, `店员`, `酒店前台`, or another clearly non-target field in the same city.
- Read listings and one JD only.
- Do not click `立即沟通`, `继续沟通`, `投递`, or `发送`.
- Report whether the automation path is stable enough to begin real target applications.

If the user asks to actually submit/contact an unrelated company for validation, warn that this can create a low-quality application record and may affect account quality. Proceed only when the user explicitly names the company/role and confirms the exact action.

## Browser Strategy

- Prefer a dedicated local Chrome profile with `--remote-debugging-port` when the user authorizes local browser control.
- Verify the controlled profile with `chrome://version`; the profile path should be the dedicated profile path the user expects.
- The Codex in-app browser may show blank pages or fail to render BOSS verification widgets.
- Use read-only Chrome DevTools Protocol extraction for lists and JD text when possible; it is less visually disruptive and may reduce verification triggers.
- Use visible mouse events only when a real click is needed and the user has confirmed the action.

## Security Verification

- Do not bypass CAPTCHA, sliders, SMS, QR login, face verification, or security prompts.
- Ask the user to complete verification manually.
- After verification, re-read the target page and confirm the current company, job, URL, and button state before continuing.

## BOSS-Specific Behavior

- Salary digits may be rendered with custom glyphs. If extracted text contains private-use glyphs, map them before ranking. In one observed session, glyphs mapped as: `=0`, `=1`, `=2`, `=3`, `=4`, `=5`, `=6`, `=7`, `=8`, `=9`.
- `立即沟通` can immediately create a chat and send BOSS's default greeting, then change to `继续沟通`.
- Do not click `立即沟通` as a preview. Treat it as applying/contacting.
- After clicking, inspect whether the page shows `已发送`, `继续沟通`, a chat panel, an app-download prompt, or a resume-completion prompt.
- If a custom message is desired, draft it separately and ask for confirmation before sending it as a follow-up message.

## Chinese Label Fallbacks For Windows Agents

Some Windows agents or shells may read UTF-8 Markdown without BOM as garbled text. When that happens, use the English meaning or Unicode escape to identify BOSS UI labels:

| Meaning | Chinese label | Unicode escape |
| --- | --- | --- |
| security verification | `安全验证` | `\u5b89\u5168\u9a8c\u8bc1` |
| login/register | `登录/注册` | `\u767b\u5f55/\u6ce8\u518c` |
| profile path | `个人资料路径` | `\u4e2a\u4eba\u8d44\u6599\u8def\u5f84` |
| contact now | `立即沟通` | `\u7acb\u5373\u6c9f\u901a` |
| continue chat | `继续沟通` | `\u7ee7\u7eed\u6c9f\u901a` |
| sent | `已发送` | `\u5df2\u53d1\u9001` |
| send | `发送` | `\u53d1\u9001` |
| apply/deliver resume | `投递` | `\u6295\u9012` |

If Chinese labels appear garbled in terminal output, do not trust the garbled text. Re-read with UTF-8 mode, use browser DOM text directly, or match by role, position, URL, and the Unicode escape meaning above.

## Observed Failure Modes And Fixes

Use these as operational guardrails:

1. Blank in-app browser
   - Symptom: the user sees a blank page, while the backend reports `安全验证`.
   - Fix: do not rely on the in-app browser for BOSS. Use a local Chrome profile with user-visible login and manual verification.

2. Wrong Chrome/profile confusion
   - Symptom: the user says they are logged in, but the agent sees `登录/注册` or security verification.
   - Fix: ask the user to open `chrome://version` and compare `个人资料路径/Profile Path` with the controlled profile path.

3. Stale automation tab
   - Symptom: an automation wrapper reports old `安全验证` tabs while the user sees a live job page.
   - Fix: inspect `http://127.0.0.1:<debug-port>/json` and target the active BOSS `page` entry directly.

4. Read-only extraction succeeds while visual automation is unreliable
   - Symptom: page state is visible, but wrappers cannot click or snapshot the right tab.
   - Fix: use Chrome DevTools Protocol to read `document.body.innerText`, job cards, JD text, and button state. Keep the user updated because this may not be visible on screen.

5. Fake DOM clicks fail
   - Symptom: dispatching JavaScript `MouseEvent('click')` does not trigger `立即沟通`.
   - Fix: after explicit confirmation, bring the page to front, scroll the real button into view, compute its viewport center, and use native mouse events through the browser/DevTools input API.

6. `立即沟通` is not a preview
   - Symptom: clicking it immediately sends the platform's default greeting.
   - Fix: treat the click as the application/contact event. Ask the user before clicking, then confirm whether `继续沟通` or `已发送` appears.

7. Custom outreach must often be a follow-up
   - Symptom: default BOSS greeting is sent before a custom message can be entered.
   - Fix: generate the custom JD-resume message separately, show it to the user, and send it only after confirmation as a second message.

## Minimum Per-Job Report

Before applying/contacting, show:

```text
Company:
Role:
Salary:
Location:
Fit:
Risk/notes:
Action requested:
Message draft:
```

After the action, show:

```text
Result:
Evidence seen:
Next step:
```
