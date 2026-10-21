---
name: toss-ux-writing
description: Write, rewrite, or review Korean UI copy by Toss (토스) UX writing rules — 해요체, active voice, positive framing, casual honorifics, no noun stacking, weed cutting, predictable CTAs, and Toss's error-message principles — and report every change as a Before/After table with the rule applied. Use whenever the user wants Korean product copy improved or written: a single phrase such as "예약" or "결제가 완료되었습니다", button labels, toasts, dialogs, error messages, push notifications, empty states, UI strings inside React/TSX/i18n files, or copy for a described screen. Triggers on "토스 스타일로", "토스처럼", "UX 라이팅", "문구 다듬어줘", "문구 바꿔줘", "버튼 문구", "에러 메시지 써줘", "카피 검수", "마이크로카피", "해요체로", and on any request to make Korean UI text clearer or friendlier, even when Toss is not named.
---

# Toss UX Writing

Toss writes for people who scan, not read. Every rule below serves that: say
the one thing the user needs, in words they would say out loud, so they know
what happens next. Apply the rules as judgment, not as find-and-replace. A
mechanical swap that breaks meaning or misleads the user violates the deeper
values (Clear, Respect) that the surface rules exist to serve.

Write all copy and explanations in Korean.

## Reference files

Read these before producing copy; they hold the rules, exceptions, and the
real Toss Before/After examples the rules are calibrated on.

- `references/rules.md` — the deterministic rules from Toss's published guide
  (해요체, 능동형, 긍정형, 캐주얼한 경어, 명사 나열 금지, 표기) with their
  exceptions, plus how to handle common Sino-Korean nouns like 예약. Always read.
- `references/principles.md` — the 5 core values and 8 writing principles with
  examples. Read for anything beyond a one-word fix.
- `references/components.md` — per-component guidance: CTA buttons, dialogs,
  titles, error messages, toasts, push, input questions, empty states. Read
  when the copy belongs to a known component, and always for errors.
- `references/sources.md` — provenance. Read only when the user asks where a
  rule comes from.

Rules marked *(추론)* in the references are heuristics inferred from Toss's
examples, not published rules. When one decides a change, say so in the
rationale column.

## Pick the mode

The user may ask for one of four things. Identify it from the request; mixed
requests are fine.

1. **Rewrite a phrase** — the user gives text ("예약", "결제가 완료되었습니다").
2. **Fix copy in code** — the user points at files, a component, or a diff.
3. **Write new copy** — the user describes a screen or situation.
4. **Review copy** — the user wants violations found, not necessarily fixed.

### Context first

The right copy depends on where it appears and who acts. "예약" as a button is
`예약하기`; as a completion toast it is `예약했어요.`; as a list header it may
stay `예약`. Before rewriting, determine:

- **Component**: button, title, body, toast, dialog, error, push, input
  label/question, list item, tab, empty state.
- **Actor**: did the user do it (`예약했어요`), or did the system or a third
  party (`예약이 확정됐어요` is only acceptable under a passive-voice
  exception; otherwise restructure)?
- **Next screen**: what pressing a button actually leads to.

Infer these from code, surrounding copy, or the request. When a bare phrase
arrives with no context, do not stall on questions: give the most likely
component's version and add rows for the other plausible components, each
labeled with its context. Ask only when the meaning itself is ambiguous in a
way that changes the copy.

### Mode 2: copy in code

- Find user-facing strings: JSX text, `aria-label`, `placeholder`, `title`,
  toast/dialog/alert calls, i18n resource values. Leave alone log messages,
  analytics event names, test IDs, keys, enum values, and comments.
- Preserve interpolations (`{name}`, `${count}`, `%s`, `{{amount}}`), markup,
  and i18n keys exactly. Rewrite around them; check that Korean particles
  still agree with the interpolated value (`{name}님에게`), and keep particle
  helpers if the codebase uses them.
- Keep consistency within the product: if the codebase already settled a
  term (e.g. `송금` vs `보내기`) across many screens, follow the codebase and
  mention the alternative rather than changing one screen alone.
- When the user asked for fixes, edit the files, then report the table with
  `file:line` in the location column. When they asked for a review, report
  without editing.

### Mode 3: new copy

Write the full set the screen needs — title, body, CTA, secondary button,
toast/error states — then run the review pass on your own draft before
presenting. For a flow, read the screens in order and remove repetition
between them (Focus on key message).

## Review pass

Run these in order on every string, whether rewriting, writing, or reviewing.
Earlier steps can make later ones moot.

1. **Should this text exist?** For errors: can the error be prevented instead?
   For subtitles and helper text: does it repeat the title or fill space?
   Deleting is a valid After (`(삭제)`).
2. **Errors** follow the error principles in `references/components.md`:
   situation, reason when helpful, what the user can do, and a button for it.
3. **Deterministic rules** from `references/rules.md`, checking each rule's
   exception list before applying: 해요체 → 능동형 and `돼요` → 경어 (`-시-`,
   `께`, `계시다`, `여쭈다`) → noun stacking and Sino-Korean → 긍정형.
4. **Weeds**: words that change nothing (`혹시`, `앞으로`, `해당`, `관련`),
   then redundant sentences on the screen.
5. **CTA**: the label names the action or destination, not the value pitch;
   the dialog's dismiss button is `닫기`, never `취소`.
6. **Tone**: no pressure or fear of loss, no memes or insider slang, no
   exaggeration or clickbait; acknowledge the user's feeling where the moment
   carries one (repayment done, failure after effort).
7. **Punctuation**: no period on titles and buttons; periods on body
   sentences and toasts.
8. **Read it aloud.** If it would sound stiff spoken to a friend-but-customer,
   it is not done.

## Output

Always report with a Before/After table. One row per string changed or flagged.

```markdown
| 위치 | Before | After | 적용 원칙 |
|---|---|---|---|
| 완료 토스트 | 예약이 완료되었습니다. | 예약했어요. | 해요체, 능동형, 명사 나열 풀기 |
| 버튼 | 예약 | 예약하기 | CTA는 동사+기 |
| 상세 화면 제목 | 예약 내역 확인 | 예약 내역 | 잡초 제거 (추론) |
```

- **위치**: component (and `file:line` for code). For bare phrases, the assumed
  context.
- **Before** exactly as given; **After** ready to paste. For deletions write
  `(삭제)`.
- **적용 원칙**: short rule names from the references, comma-separated. Mark
  inferred rules with `(추론)`.
- When two Afters are both good and depend on product context, put the
  recommended one in the table and list the alternative in a short note under
  it, with when to prefer it.
- For review mode with no fix requested, use the same table; After holds the
  suggested fix and an extra `문제` column is fine.
- Under the table, add at most a few lines: assumptions made, strings left
  unchanged on purpose because an exception applied, and anything the user
  must decide (e.g. legal wording that must stay).

Do not rewrite legally mandated wording (약관, 고지 문구, regulatory
disclosures) without flagging it; suggest, and note that legal review is needed.
