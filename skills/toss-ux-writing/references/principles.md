# Core values and writing principles

Source: toss.tech "토스의 8가지 라이팅 원칙들" (2022-11). Toss built these from
A/B test winners ("이렇게 썼더니 이기더라"), so treat them as what works for
users, not taste.

Toss structures its writing as a pyramid: core values → principles →
guidelines → templates → systems. Principles are the judgment axes; the
deterministic rules in `rules.md` are guidelines. When a guideline and a
principle seem to conflict, the principle (and above it, the value) wins.

## Core values

| Value | Meaning |
|---|---|
| Clear 명확한 | No hard or vague words; understood in one pass; written for the user, not the maker. |
| Concise 간결한 | Users don't read everything. Only what is needed, only as much as needed. |
| Casual 친근한 | Replace finance/IT jargon and stiff nuance with easy, kind words. The defining trait of Toss's voice. |
| Respect 존중하는 | Never hide costs or inflate benefits, even at a short-term cost to metrics. |
| Emotional 공감하는 | Empathize with what the user feels in the moment. |

## The 8 principles (as review questions)

### 1. Predictable hint — 다음 화면을 예상할 수 있는 힌트가 있는가?
Explaining value is not enough; say what happens next. Users who can't
predict the next screen feel anxious.
- `다른 곳에서 후불결제 하기` (leads to a brand-request survey) →
  `내가 쓰는 브랜드 신청하기`

### 2. Weed cutting — 의미 없는 단어를 모두 제거했는가?
A "weed" is a word whose presence changes nothing. Removing meaningless words
matters more than cutting character count. One message per sentence.
- `앞으로 받을 배당금` → `받을 배당금`
- `혹시` is a weed by Toss guideline.

### 3. Remove empty sentences — 의미 없는 문장을 모두 제거했는가?
Weeds at screen level: sentences that repeat the title or exist because the
space felt empty.
- 제목 `요즘 20대가 자주 쓰는 인기 서비스 6개` + 부제
  `발견한 인기 서비스를 써볼 수 있어요.` → keep the title, delete the subtitle.

### 4. Focus on key message — 정말 중요한 메시지만 전달하고 있는가?
Weeds between screens: drop what the user already knows; keep only what they
must check now. Omitting can be the kinder choice.
- Options in a bottom sheet "얼마를 갚을까요?":
  `대출금의 원금을 갚습니다` + long description /
  `한 달치의 원금과 이자를 미리 갚을게요` + description →
  `금액 직접 입력하기` / `다음 달 이자 미리 내기`, descriptions removed.

### 5. Easy to speak — 이해하기 어려운 용어나 표현을 사용하지 않았는가?
It is "easy to *speak*", not read: read it aloud. Sino-Korean, written style,
and long breaths sound wrong spoken. Remove jargon; short breaths; one
message per sentence.
- `미국 외 다른 나라 투자자들이 시차로 인한 거래의 불편함을 덜 수 있도록, 정규장 시작 전에 거래할 수 있는 시간을 프리마켓이라 해요. 토스증권 프리마켓을 통해 예약 주문이 가능하며, 실제 거래(체결)는 정규장에 이뤄져요.`
  → `정규장이 열리기 전에 거래할 수 있는 시간이에요. 시차로 주문하기 어려울 때 프리마켓에 예약을 걸어두면, 정규장이 열렸을 때 주문이 체결돼요.`

### 6. Suggest over force — 특정 행동을 강요하거나 공포감을 주고 있지 않은가?
Let users feel they have a choice. Announce benefits without "you'll lose
out" nuance; use a neutral tone.
- Exit dialog: 제목 `지금 그만두면 무제한 연 1% 이자를 받을 수 없어요` /
  `이어서 하기` `나가기` → 제목 `다음에 할까요?` /
  본문 `토스증권 계좌는 한도 없이 연 1% 이자를 매달 받을 수 있어요.` /
  `닫기` `나가기`

### 7. Universal words — 모두가 이해할 수 있고 모두에게 무해한가?
Avoid words whose understanding varies by age, education, or industry —
especially trends, slang, memes. Wit is welcome if it stays refined and
universal.
- Push `소리 지…르지 말고 박수 쳐! 🥳` → `지난밤 나스닥 4% 상승한 이유는?`

### 8. Find hidden emotion — 정보 전달을 넘어 사용자의 감정에 공감했는가?
Find the emotion behind the information. Paying off a loan feels like a
reward for effort; users said the push "힘이 됐다", and metrics improved.
- 제목 `신용 관리` / `김토스님, 대출 잔액이 달라졌어요. 잔액을 확인하세요.` →
  제목 `드디어 대출 끝🎉` / `김토스님, 갚느라 고생 많으셨어요! 신용 점수가 달라졌는지 확인해보세요.`

Emotion fits moments that carry one. A routine completion toast should not
celebrate: `카드 연결 끝! 이제 써볼까요? 😆` was a Don't; `카드를 연결했어요.`
was the Do.

## Screen-level flow

Even when every sentence passes, a screen can break flow or repeat itself
across title, body, and CTA. Read the whole screen (and adjacent screens) in
order as one narrative before finishing.

## Consistency

An early Toss principle was Keep Consistency: the same thing has the same
name everywhere. When rewriting one screen, match the product's existing
terms unless the user asks to change them product-wide.
