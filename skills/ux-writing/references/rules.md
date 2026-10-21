# Deterministic rules

Source: Toss's Apps-in-Toss UI/UX guide, "UX 라이팅" section, which Toss uses
as a release review standard. Examples are Toss's own unless marked *(추론)*.
Each rule has published exceptions; check them before applying the rule.

## Contents

1. 해요체
2. 능동형
3. 긍정형
4. 캐주얼한 경어
5. 명사 나열 쓰지 않기 / 한자어 풀어쓰기
6. Handling everyday Sino-Korean nouns such as 예약 *(추론)*
7. 표기
8. Quick substitution table

---

## 1. 해요체

All product copy uses 해요체, in every situation, including error messages.
No 합쇼체 (`~습니다`, `~입니다`, `~합니다`, `~십시오`).

| Before | After |
|---|---|
| 김토스님에게 1원을 보냅니다 | 김토스님에게 1원을 보낼게요 |
| 세션이 만료되었습니다. | (사용자 언어로 다시 쓰기 — components.md 에러 참고) |
| 카드 연결을 완료했습니다. | 카드를 연결했어요. |

Use `~할게요` for what Toss is about to do, `~했어요` for what is done,
`~해요` for facts, `~할까요?` for confirmations, `~해주세요` for requests.

## 2. 능동형

Prefer active sentences; passive only in specific situations.

| Sub-rule | Before | After |
|---|---|---|
| `됐어요` → `했어요` | 알림이 설정됐어요. | 알림을 설정했어요. |
| drop `~었` passive/auxiliary | 새 비밀번호로 바뀌었어요 | 새 비밀번호로 바꿨어요 |
| use a direct verb | 카드론 평균 금리가 13%로 높아졌어요 | 카드론 평균 금리가 13%로 올랐어요 |

Decide the subject by who acted. If the user did it, the user is the implied
subject (`예약했어요`). If a third party did it, name it rather than hiding
it behind a passive (`가게에서 예약을 확정했어요` *(추론)*).

**Passive is fine when:**

- A service ends or a period expires — it emphasizes the subject and conveys
  "종료/만료" precisely (`서비스가 종료돼요`, `기간이 만료됐어요`). But do not
  use `종료돼요` for things that end on a regular cycle.
- Telling the user about a consequence of their action (연체, 해지, 적용) — the
  passive makes the cause-and-effect clear.
- Reassuring the user in sensitive moments (e.g. 정보 수집 안내).

## 3. 긍정형

Reduce negative communication. `안 돼요`, `없어요` → `~하면 할 수 있어요`.
Users scan; a negative headline reads as "the whole product is unusable".

| Before | After |
|---|---|
| 보호자가 허락하기 전에는 가입할 수 없어요 | 보호자가 허락해야 가입할 수 있어요 |
| 제목: 지금 버전에서는 쓸 수 없어요 / 본문: 생체 인증을 쓰려면 앱을 최신 버전으로 업데이트 해주세요. | 제목: 앱을 업데이트해주세요 / 본문: 생체 인증을 쓰려면 최신 버전이 필요해요. |
| 제목: 계좌 개설 혜택은 받을 수 없어요 | 제목: 4.5% 금리 혜택만 받을 수 있어요 |
| 제목: 모임지원금 없이 모임통장을 만들까요? / 본문: 지금 받지 않으면 모임지원금을 받을 수 없어요. / 버튼: 받지 않고 만들기 | 제목: 약관에 동의하면 모임지원금을 받을 수 있어요 / 버튼: 동의하기 |

Related, from the error principles: prefer `~할 수 있어요` (suggestion) over
`~해야 해요` (demand).

**Negative is fine when** the user must clearly understand something negative:

- The service cannot be used for policy reasons — say so and give the reason.
- Only some features are unavailable — name which ones, so the result of the
  user's choice is clear.
- Reassurance (e.g. `비밀번호는 저장하지 않아요`).

Double negatives (`~하지 않으면 안 돼요`) are a form of negative framing;
rewrite them positively *(추론)*.

## 4. 캐주얼한 경어

No excessive honorifics: drop subject-honorific `-시-`, `께`, `계시다`,
`여쭈다`. 해요체 itself stays polite; `드려요`, `~해주세요`, and `~님` remain.

| Sub-rule | Before | After |
|---|---|---|
| drop `-시-` | 카드를 해지하시겠어요? | 카드를 해지할까요? |
| drop `-시-` | 오늘 토스뱅크 시작하시는 분에게 5,000원을 드려요 | 오늘 토스뱅크를 시작하면 5,000원을 드려요 |
| `계시다` → `있다` | 자동차를 가지고 계시나요? | 자동차가 있나요? |
| `계시다` → restructure | 매달 보험료 얼마씩 내고 계시나요? | 매달 보험료는 얼마인가요? |
| `여쭈다` → `확인하다/묻다` | 안전한 개통을 위해 몇 가지 다시 여쭤볼게요 | 안전한 개통을 위해 몇 가지 다시 확인할게요 |
| `께` → `에게` | 토스뱅크카드가 김토스님께 날아가고 있어요 | 토스뱅크카드가 김토스님에게 날아가고 있어요 |
| awkward after dropping `-시-` → make the wanted info the subject | 어떤 목적으로 대출받으시나요? | 대출 목적이 무엇인가요? |
| input question → instruction | 어떤 이유로 신고하시나요? | 신고 이유를 선택해주세요 |

**`-시나요? / -셨나요?` are allowed, sparingly, when:**

- Asking about the user's own context, to reduce confusion
  (`방금 로그인하셨나요?` *(추론 예시)*).
- Toss lacks the information and must ask the user to judge their situation.
- Toss needs the user's goodwill, as in surveys.

## 5. `{명사}+{명사}` 쓰지 않기 / 한자어 풀어쓰기

Stacked nouns and Sino-Korean nominal phrases read like a form. Unfold them
into verbs. If unfolding is hard, at least use `{명사}가 {명사}해서`.

| Before | After |
|---|---|
| 이자 환불을 받았어요 | 이자를 돌려받았어요 |
| 내일 카드값이 결제될 예정이에요 | 내일은 카드값 나가는 날이에요 |
| 잔액 부족으로 구매하지 못했어요 | 잔액이 부족해서 구매하지 못했어요 |
| 카드 연결 완료 | 카드를 연결했어요. |
| 5000원 송금이 완료되었습니다 | 5000원을 보냈어요 |

Also unfold written-style connectors Toss marked in red in its Easy-to-speak
example: `~로 인한`, `~을 통해`, `~가 가능하며`, `~ 외`, `~ 시작 전에`.

| Before | After |
|---|---|
| 토스증권 프리마켓을 통해 예약 주문이 가능하며, 실제 거래(체결)는 정규장에 이뤄져요. | 시차로 주문하기 어려울 때 프리마켓에 예약을 걸어두면, 정규장이 열렸을 때 주문이 체결돼요. |

A Sino-Korean word that is genuinely clearer than its native alternative may
stay. Distinguish everyday words (예약, 송금, 결제, 확인) from industry jargon
(체결, 원리금, 상환, 이체 한도 초과). Replace jargon; restructure everyday
words only when they appear as stacked nouns or nominal endings.

## 6. Handling everyday nouns such as 예약 *(추론)*

Toss kept the word 예약 in its own rewrite (`예약을 걸어두면`). The problem is
never the noun itself but how it is used. Inferred from the rules above:

1. **Nominal endings and noun stacks → verbs.** `예약 완료` → `예약했어요.`,
   `예약 변경이 완료되었습니다` → `예약을 바꿨어요.`, `예약 취소 처리되었습니다`
   → `예약을 취소했어요.`
2. **Buttons → verb + 기, naming the action.** `예약` → `예약하기`;
   `예약 취소` → `예약 취소하기`. If the next screen is actually a date picker,
   the more predictable label may be `날짜 고르기`.
3. **Passive → active by actor.** `예약되었습니다` → `예약했어요.` (user did it).
4. **Say what the user experiences** when a plainer phrasing exists.
   `예약 송금` → `보낼 날짜를 정해두면 그날 보내드려요`, mirroring
   `결제될 예정` → `카드값 나가는 날`.
5. **Labels and headers may stay nouns.** A tab or section header like `예약`
   or `예약 내역` is fine; do not force a sentence into a navigation label.
6. **Swap for a plainer verb** when one exists and the product does not
   already standardize the noun: `송금하기` → `보내기`, `조회하기` → `확인하기`
   or `보기`, `입력하기` → `적기` only when it reads naturally.

## 7. 표기

- `되어요` → `돼요` always (narrow mobile screens). `되었어요` → `됐어요`.
- `업데이트 해주세요` → `업데이트해주세요` (붙여 씀, as in Toss's example).
- Titles and buttons: no period. Body sentences and toasts: period.
- Numbers and units in Korean the user would say: `3min` → `3분`.

## 8. Quick substitution table

Starting points, not blind replacements. Always re-read the full sentence.

| Before | After |
|---|---|
| ~습니다 / ~입니다 | ~어요 / ~이에요 |
| ~하시겠어요? / ~하시겠습니까? | ~할까요? |
| ~하시는 분에게 | ~하면 |
| ~께 | ~에게 |
| 계시나요? | 있나요? |
| 여쭤볼게요 | 확인할게요 / 물어볼게요 |
| ~이/가 ~됐어요 (user did it) | ~을/를 ~했어요 |
| 되어요 | 돼요 |
| ~ 완료되었습니다 / ~ 완료 | ~했어요. |
| ~ 부족으로 | ~이/가 부족해서 |
| ~로 인해 / ~로 인한 | ~해서 / (재구성) |
| ~을 통해 | ~에서 / ~로 / (재구성) |
| ~가 가능해요 | ~할 수 있어요 |
| ~할 수 없어요 (혜택 일부만 불가) | ~만 ~할 수 있어요 |
| ~해야 해요 (권유 상황) | ~할 수 있어요 / ~하면 돼요 |
| 다이얼로그 `취소` | `닫기` |
| 혹시, 앞으로, 해당, 관련 (의미 없을 때) | (삭제) |
