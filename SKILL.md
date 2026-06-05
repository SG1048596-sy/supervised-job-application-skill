---
name: supervised-job-application
description: Supervised job-search and application workflow for platforms such as BOSS Zhipin, LinkedIn, or job boards. Use when Codex needs to screen jobs, read JDs, compare them with a resume, rank opportunities, draft outreach, operate a browser for job applications, click apply/contact only with explicit per-job confirmation, or manage a controlled daily application pipeline. Do not use for unsupervised bulk spam, CAPTCHA bypass, credential handling, or platform-rule evasion.
---

# Supervised Job Application

## Agent Compatibility

Use this skill as an agent-agnostic operating procedure. It is written for Codex skills, but Claude Code or any other capable agent can follow the same process by reading this `SKILL.md` file and its `references/` files.

Do not assume a specific browser tool. Use whatever the current agent environment provides, while preserving the same safety gates: read-only extraction first, visible browser automation only when needed, and explicit confirmation before each external side effect.

## Operating Principle

Treat job application actions as external side effects. Reading listings, extracting JD text, scoring fit, and drafting messages are safe. Clicking apply, submit, send, or contact buttons can transmit the user's resume or job intent and requires explicit confirmation for the specific job.

Never run unsupervised bulk applications. Build a supervised pipeline where the user can see each target, the reason for applying, and the message before any side effect.

## Workflow

1. Run the initial intake before touching any job board.
2. Confirm the target search criteria: role keywords, city, salary floor, experience range, work mode, excluded industries, and daily cap.
3. Confirm the resume source: pasted text, local file, online resume, or user-provided summary. Do not read private files or online resumes unless the user explicitly authorizes that source.
4. Offer a validation run after intake and before formal applications. Default to a dry-run on an unrelated role/company to verify browser control, search, JD extraction, and button-state detection without applying.
5. Collect job candidates with the lowest-risk method available:
   - Prefer read-only DOM/CDP extraction over visible mouse automation when a local browser is available.
   - Use visible browser automation only when read-only extraction cannot operate the page.
   - Let the user handle login, CAPTCHA, SMS, QR, face verification, and other human checks.
6. Normalize listings into: company, role, salary, city/district, experience, education, keywords, URL, recruiter status, and source page.
7. Score fit against the resume and target criteria. Prioritize jobs that match role intent, salary floor, experience range, location, and resume proof points.
8. For each shortlisted job, read the JD before applying. Extract responsibilities, hard requirements, nice-to-have requirements, risk flags, and recruiter/company context.
9. Draft an outreach message based on JD-resume overlap. Keep it short, concrete, and human. Avoid phone, WeChat, email, ID numbers, home address, or other private data unless the user explicitly requests it.
10. Present the job, fit summary, risk notes, and drafted message to the user. Ask for explicit confirmation before any click that can apply, contact, submit, upload, or send.
11. After the user confirms a side-effect action, perform only the confirmed action. Stop and report the resulting page state before continuing.
12. Log outcomes in the conversation: applied/contacted, default message sent, custom message sent, blocked by verification, needs resume update, skipped, or duplicate.

## Initial Intake

Ask these questions at first use unless the user has already provided the answer:

```text
Target role keywords:
Target city/cities:
Minimum acceptable salary:
Experience range:
Education/work-mode constraints:
Industries or companies to avoid:
Daily application cap:
Resume source to use:
Should I only shortlist, or also help apply after per-job confirmation?
```

Use concise phrasing in chat. For example:

```text
先确认投递规则：目标岗位、城市、最低薪资、经验范围、简历来源、每天最多投几个？如果你没特别要求，我会先只筛选并生成话术，每个岗位确认后再点投递/沟通。
```

Do not proceed to browser actions until role, city, salary floor, and resume source are known.

## Validation Run

After the user provides the target criteria and resume source, proactively offer a validation run before formal applications.

Default validation mode:

1. Search for an unrelated role or industry in the same city, such as coffee, retail, hospitality, operations, or another obviously non-target field.
2. Read the listing cards and one JD.
3. Confirm that the agent can identify company, role, salary, location, recruiter, button state, and page transitions.
4. Do not click apply/contact/send in dry-run mode.
5. Report what was validated and any remaining risk before formal applications begin.

Use this prompt:

```text
正式投递前，我建议先做一次验证：找一个和你当前经验不相关的岗位，只验证搜索、读取JD、识别按钮和页面状态，不投递。这样可以确认工具链稳定，也避免误投。是否进行这个 dry-run？
```

If the user explicitly wants a real unrelated application as a test, explain the risk first: it can send their resume/intent to an irrelevant employer and may lower account quality. Only proceed if the user names the exact company/role and explicitly confirms the side-effect action for that job. Never choose an unrelated employer yourself for real submission.

## Confirmation Rules

Require explicit per-job confirmation before:

- Clicking buttons like `Apply`, `Submit`, `Send`, `Contact`, `Message`, `立即沟通`, `继续沟通`, `投递`, or similar.
- Uploading or selecting a resume file.
- Sending a custom message.
- Accepting permissions, saving credentials, or changing profile/resume settings.

The confirmation must identify the company, role, and action. Example: `Allow clicking Wudong Technology immediate contact for the multimodal data evaluation role; ask again before sending a custom message.`

If a platform button is known or suspected to auto-send a default message, say so before clicking and treat the click itself as the send/apply action.

## BOSS Zhipin Notes

Read `references/boss-zhipin.md` when working on BOSS Zhipin or when the user mentions Boss/BOSS/Zhipin/直聘.

Core rules from observed behavior:

- The in-app browser may render blank pages or get stuck on security verification. Prefer a dedicated local Chrome profile with remote debugging when the user authorizes it.
- BOSS can trigger security verification frequently. The user must complete CAPTCHA, slider, SMS, QR, or face checks.
- `立即沟通` can immediately send the platform default greeting and change to `继续沟通`. It is not a harmless preview button.
- A custom JD-tailored message may need to be sent as a second chat message after the default contact message. Confirm before sending it.

## Reuse Lessons From The 2026-06-05 BOSS Session

Record these lessons for other agents, including Claude Code-style agents, so they do not repeat the same mistakes:

- The in-app browser can appear blank to the user while the automation backend sees a security-verification page. Do not tell the user they are on the wrong page until verifying the actual controlled browser/profile.
- A normal Chrome login and a dedicated remote-debugging Chrome profile can have different cookies. Verify `chrome://version` and the profile path before assuming the user is logged into the controlled browser.
- Browser automation wrappers can get stuck on stale tabs. If the user sees a different page, inspect all Chrome DevTools targets and select the actual job page.
- Read-only CDP extraction can successfully read job cards and JD text even when visible automation is unreliable.
- BOSS salary digits may use custom glyphs; normalize them before scoring jobs.
- DOM-dispatched fake clicks may not trigger BOSS buttons. Native mouse events can work, but only use them after per-job confirmation.
- `立即沟通` was observed to automatically send BOSS's default greeting, not open a harmless editor. Treat it as the application/contact action.
- Keep the user informed in chat for each step because background CDP reads may not be visible on their screen.

## Fit Scoring

Use this simple score unless the user gives a different rubric:

- Role match: 0-30
- Salary and location match: 0-20
- Experience and education match: 0-15
- Resume evidence for core JD requirements: 0-25
- Company/recruiter quality and risk flags: -10 to +10

Output concise labels:

- `Priority`: strong match and worth applying now.
- `Maybe`: plausible but missing proof or lower salary.
- `Skip`: weak match, salary below floor, unrelated role, obvious risk, duplicate, or platform-only noise.

## Outreach Message Rules

Draft messages in the user's language unless the JD is clearly in another language. Prefer 60-120 Chinese characters for BOSS-style chats.

Message pattern:

```text
您好，我想了解这个岗位。我关注[JD方向]，有[简历证据1]、[简历证据2]相关经验，能做[JD任务]。想进一步了解团队方向、工作内容和用工形式。
```

If resume evidence is weak, do not fabricate. Use cautious wording such as `我关注...方向，有...基础/经验，想进一步了解...`.

## Stop Conditions

Stop and ask the user when:

- The page asks for CAPTCHA, SMS, QR, face verification, password, or security confirmation.
- A click may apply, send, upload, or expose private data and the user has not confirmed that specific action.
- The platform prompts for resume completion or profile changes.
- The page state is ambiguous and continuing could duplicate a message or apply twice.
- The user asks for mass unsupervised applications. Offer a supervised shortlist-and-confirm workflow instead.
