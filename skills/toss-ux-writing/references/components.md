# Component guidance

## Contents

1. CTA buttons
2. Dialogs and confirmations
3. Titles
4. Error messages
5. Toasts
6. Push notifications
7. Input labels and questions
8. Empty states *(추론)*

---

## 1. CTA buttons

- The label tells the user what they will do or where they will go next.
  Repeating the screen's value pitch in the CTA, so the user can't predict
  the next screen, is a dark pattern that fails Toss's release review.
  - `4,000원 받고 금리 3%받는 통장 만들기` (next step opens a partner
    securities app) → `이동하기`
- A single action CTA: verb + `기` (`동의하기`, `전화하기`, `문의하기`,
  `금액 직접 입력하기`, `보러 가기`).
- Pure progression to the next screen: `다음`, `확인`.
- Short established nouns can stand alone (`업데이트`, `확인`).
- Don't stack an exaggerated or duplicate helper line above the CTA; it blurs
  the button's role.
- No period.

## 2. Dialogs and confirmations

- The left (dismiss) button is always `닫기`. `취소` can be misread as
  cancelling the task the user is doing.
- Confirmation title: `~할까요?` (`카드를 해지할까요?`), not `~하시겠어요?`.
- The primary button names the action in the title (`해지하기`), not `확인`,
  when the action is consequential *(추론)*.
- Exit/abandon dialogs: no loss framing. `다음에 할까요?` plus a neutral
  statement of the benefit, buttons `닫기` / `나가기`.
- Always leave a way out; blocking sheets on entry or on back are dark
  patterns.

## 3. Titles

- Lead with the conclusion or the action that resolves it:
  `앱을 업데이트해주세요`, `전화로만 취소할 수 있어요`,
  `하루에 한 번만 신청할 수 있어요`, `4.5% 금리 혜택만 받을 수 있어요`.
- Bottom-sheet titles fit on one line.
- If the title says enough, don't add a subtitle.
- No period on titles.

## 4. Error messages

Toss's stance is "navigating error": the message exists to get the user to
the next screen. It covers the situation, the reason when it helps, and what
to do. Check these six principles in order:

1. **The best error is one that never happens.** Ask whether the error needs
   to exist. Toss stopped showing `내 연락처로는 돈을 보낼 수 없어요.` by hiding
   the user's own contact from the send list. When you can't change the
   product, say so in the notes as a suggestion.
2. **Use the right component.** Toasts vanish in 3–5 seconds; anything the
   user must read or act on goes in a dialog. Don't use alarming red for
   non-urgent cases.
   - 토스트 `대출 심사 진행 중으로 KB저축은행 고객센터(0000-0000)로 취소 신청해주세요.` →
     다이얼로그 제목 `전화로만 취소할 수 있어요` /
     `이미 심사를 시작했어요. 대출 신청을 취소하려면 KB저축은행 고객센터(0000-0000)로 문의해주세요.` /
     `닫기` `전화하기`
3. **Tell users how to fix it themselves** — the most important principle.
   - 토스트 `생체 인증에 실패했어요.` → 다이얼로그 `생체 인증 정보에 문제가 생겼어요` /
     `같은 문제가 반복된다면 앱을 껐다 켜거나 삭제하고 다시 설치해주세요.` / `확인`
4. **Use the user's language.** No status codes, system terms, or vague
   categories.
   - `알 수 없는 종류의 신분증입니다.` → `기간이 만료된 신분증이에요`
   - Don't: `Request failed with status code 500`, `세션이 만료되었습니다.`
     (e.g. → `오랫동안 사용하지 않아 로그아웃했어요` / `다시 로그인해주세요` *(추론 예시)*)
5. **Make the fix easy.** Attach the solution as a button.
   - `하루에 한 번만 신청할 수 있어요` … `고객센터(1599-4905)로 문의해주세요.` /
     `확인` → same copy with `닫기` `문의하기`
6. **Minimize negative feelings.** Say what is possible.
   - `신용 관리 서비스를 이용할 수 없는 연령입니다.` →
     `신용점수는 만 19세 이상부터 확인할 수 있어요.`

When security requires vagueness, still tell the user whether they can fix
it themselves: `비밀번호를 틀렸어요` → `김토스님의 정보가 아니에요`.

Unknown or server errors with no user remedy: say what happened in plain
words and what to try (`잠시 후 다시 시도해주세요`), without blaming the user
*(추론)*.

## 5. Toasts

- Short completion or feedback only; if it needs explanation, use a dialog.
- Pattern: object + active past 해요체 + period. `카드를 연결했어요.`,
  `알림을 설정했어요.`
- No celebration, emoji, or follow-up question for routine completions.

## 6. Push notifications

- Let the user predict the content; summarize the core in one line.
- Prefer plain text over emoji, special symbols, pronoun teasers
  (`이것은 무엇일까요?`), or 초성 quizzes. If the content differs from what
  the push implied, it feels like clickbait.
- No exaggeration to boost clicks.
- Facts over memes (Universal words); empathy where the moment carries
  emotion (Find hidden emotion).
- Long notices: one sentence plus "자세한 내용은 알림을 눌러 확인해보세요."
  - Long multi-section 종료 notice → `토스 P2P 분산투자 서비스가 10월 23일(금)부터 종료돼요. 자세한 내용은 알림을 눌러 확인해보세요.`
- Check for provider-centric value: is this what the user cares about, or
  what we want to say?

## 7. Input labels and questions

- Ask with the wanted information as the subject: `대출 목적이 무엇인가요?`,
  `매달 보험료는 얼마인가요?`.
- Turn form labels into the question the user answers *(2차 출처)*:
  `받는 분` → `어디로 돈을 보낼까요?`, `금액` → `얼마나 보낼까요?`.
  Use this for step-by-step flows; dense forms may keep short labels.
- Selection: `신고 이유를 선택해주세요`.
- `-시나요?` only for surveys or when the user must judge their own situation.

## 8. Empty states *(추론)*

Toss publishes no empty-state rule. Apply the principles:

- Don't fill the space with filler sentences (Remove empty sentences).
- Say what the user can do, positively, instead of only `~이 없어요`
  (`아직 예약한 곳이 없어요` is acceptable as a fact; pair it with a CTA
  such as `예약하러 가기`).
- One line plus one CTA is usually enough.
