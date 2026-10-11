# CLAUDE.md — 온에어 (溫 On-Air) 따뜻한 안내방송

클라우드 Claude Code 세션용 안내서. 이 PC의 메모·시험 파일은 클라우드에 없으니, 여기 적힌 것과 저장소 코드만 믿고 작업할 것.

## 누구를 위한 앱인가
- 아파트·지식산업센터·학교·식당·공항 등 안내방송을 아나운서처럼 따뜻하게 만들어 읽어 주는 무료 웹앱.
- 라이브: https://s95722462-cell.github.io/ONAIR/ (main 에 push = GitHub Pages 배포)
- 사용자는 한국인 비개발자. **답은 한국어, 쉬운 말로.** 화면 글·설명이 길거나 어수선한 것을 싫어함.
- **새 기능·큰 변경은 기획안(목적·화면·단계·결정할 것)을 먼저 보여 주고 승인받은 뒤 만들 것.** 작은 수정·버그는 바로 해도 됨.

## 원본 위치 (중요)
- 사용자 PC에도 작업용 사본이 있음: `Desktop\자주 쓰는 것\웹앱 원본\온에어_안내방송.html`.
- **이제부터 저장소의 `index.html` 이 원본(source of truth).** 클라우드 세션이 저장소를 직접 고치기 때문.
- PC 세션은 로컬에서 고치기 전에 반드시 `git pull` 후 저장소 index.html 을 사본으로 복사해서 시작할 것 (거꾸로 덮어쓰지 말 것).

## 구조
- `index.html` 하나 (CSS + `(() => { ... })()` IIFE 스크립트, 약 42만 자). `manifest.webmanifest`, `sw.js`, `icons/`.
- **push 할 때마다 `sw.js` 의 `CACHE` 번호를 올릴 것** (지금 `onair-v13` → 다음 `onair-v14`). 안 올리면 사용자 기기에 옛 화면이 남음.
- 외부 라이브러리는 cdnjs 에서 정확한 버전만, 필요할 때 늦게 불러옴: lamejs 1.2.1(MP3), qrcodejs 1.0.0(QR).
- 저장: localStorage 키 접두사 `onair_` (`ls.get/ls.set` 도우미, 공간 부족 알림). 백업(💼)은 `onair_*` 전부를 JSON 으로.
- 스크립트가 IIFE 라 내부 변수는 밖에서 안 보임 → 시험은 화면·localStorage 로 확인하거나, 시험용 사본에서만 `window.__t` 로 노출 (저장소에 넣지 말 것).

## 주요 기능과 코드 위치 (이름으로 검색)
- 장소 `PLACES` (14곳, 4개 국어 sender/aud) + 장소별 템플릿 70여 종(한·영·일·중 직접 작성). `{aud}` `{sender}` `{attime}` 치환, 조사 자동(은/는, 으로/로).
- 인사말·끝인사 `HEADS`, 사용자 고친 값 `S.heads[장소][언어]{open,urgent,close,again}` (`headTxt/headSet/fullText`).
- 고친 문장 유지 `keepEdit` → `S.tplEdits`, `regen()` 끝에 `applyEdits()`.
- 🌐 다른 언어에도 적용 / 추가 언어 27개 `MORE_LANGS` — 브라우저 Translator API(기기 안 무료), 캐시 `onair_trcache`, `trText`, 번호 두 번 부르기 `dupCall`.
- 읽기 `say()` → `speakAs()`(발음 사전, "14번"→한자어 수 등, 읽을 때만) → 🎙 억양 `prosodyPlan` 조각 발화. 표시 `[강조]…[/] [천천히] [빠르게] [높게] [낮게] [작게] [쉼] [긴쉼]`, 공지문·카톡·번역엔 `unmark()`.
- 멈춤은 `cancelN` (쉼·차임 중에도 바로).
- 목소리: Edge Natural(★★★ 온라인)이 가장 좋음. 한국어 기본 `DEFAULT_VOICE` = Brian Multilingual. 각 언어 목록 뒤 '· 다국어'.
- 아이폰: WebKit 은 누른 순간에만 speak 허용 → `IOS` + `primeSpeech()`(터치 때 무음 한 번 읽기). 아이폰은 iOS 목소리만(한국어 '유나').
- 안드로이드: **Chrome 만 웹 음성 목소리가 있음**. Edge·삼성 인터넷은 0개 → `AND_EDGE` 로 "Chrome 으로 열어 주세요".
- 탭 `nav#tabs` (방송·예약/자동·파일/인쇄·기록·설정, 조건부 🧾 대기 `qTab`), 휴대폰 아래 고정 줄 `#oaBar`.
- 🖥 방송실 모드 `#room` (즐겨찾기 큰 타일·번호 패드, Esc 나가기).
- 🖥 호출 번호판 `?board` 창 — `onair_board` + storage 이벤트로 같은 PC 다른 창 갱신 (`boardPush`).
- 📱 QR 휴대폰 알림: 손님 화면 `?wait=<매장코드>[&n=번호]`, `phonePush`/`shopListen`(손님 '확인했어요' 되받기).
- 🧾 대기 순번: 카운터 QR `?q=<코드>`, 번호는 매장 PC 한 창이 `navigator.locks` 로 정함.
- 예약 방송·⏱ 카운트다운·📚 방송 묶음, 💾 음성 파일(getDisplayMedia 시스템 오디오 → WAV/MP3), 📄 공지문 인쇄, 💬 카톡 복사, 📊 기록 CSV(BOM), 📘 앱 안 설명서 `#manual`.

## 중계 (ntfy.sh)
- `RELAY` = 늘 `https://ntfy.sh/`. 시험 때만 `window.ONAIR_RELAY = "http://127.0.0.1:포트/"` (localhost·file 에서 열었을 때만 허용).
- 주제: `onair-<매장코드>` (호출·ack), `onair-q-<코드>` / `onair-q-<코드>-in` (대기 순번). 매장 코드 = `onair_shopId` 랜덤 20자.
- **주제 이름 = 비밀번호.** 실제 매장 코드·채널 id 를 커밋·로그·문서에 남기지 말 것. 무료 한도 하루 약 250건.
- 보류된 기획: '📲 입주사 알림'(ntfy 앱 구독 푸시) — 사용자가 다시 꺼내면 범위를 한 번 더 확인하고 시작.

## 규칙
- 한국어 화면 글은 짧게. 장황한 안내문 금지.
- 휴대폰 먼저(아이폰 느낌): 누르는 칸 ≥44px, 입력칸 글자 ≥16px(아이폰 확대 방지), `touch-action: manipulation`, 320~430px 폭에서 가로 넘침 없음 (`.places` 가로 띠는 예외).
- 색은 `:root` 토큰 + `prefers-color-scheme` 다크 모드.
- 내려받기·공유 파일 이름은 영문·숫자만 (아이폰이 한글 파일명을 자모로 쪼갬).
- 비밀·개인정보를 저장소에 넣지 말 것. 대기열·QR 에는 번호·인원·시각만 (이름·전화번호 X).
- 화면 하단 '문의' 연락처는 사용자가 원해서 넣은 것 — 지우지 말 것.

## 시험 (클라우드에서)
- 사용자 PC의 시험 스크립트(e2e.py 등)는 저장소에 없음. 필요하면 `tests/` 를 만들어 작게 넣을 것.
- 헤드리스 Chromium/Edge + Playwright 또는 CDP. **소리는 늘 음소거** (`--mute-audio`).
- `speechSynthesis.speak` 를 init script 로 가로채 읽은 문장·rate·voice 를 배열에 모으고 실제로 틀지 말 것 (`onend` 는 바로 호출).
- 중계 기능은 로컬 가짜 ntfy(POST 저장 + `/sse` 내보내기) + `window.ONAIR_RELAY`.
- 확인할 것: JS 오류 0, 모든 장소 × 방송 문장 생성, 4개 국어 문장, 320/375/390px 가로 넘침, 탭 전환, 방송실·`?board`·`?wait=`·`?q=` 화면, 백업 저장/불러오기, 억양 표시가 공지문·카톡에 안 남는지.
- 클라우드 리눅스 Chromium 에는 Edge Natural 목소리가 없음 → 목소리 품질·발음은 사용자 PC Edge 에서 확인 부탁.

## Git
- 커밋 메시지는 한국어로 무엇이 바뀌었는지 (예: "대기 순번을 별도 탭으로: …").
- 시작 전 `git pull --rebase`, 강제 push 금지. main 에 push = 배포.
- push 뒤 라이브 확인: `https://s95722462-cell.github.io/ONAIR/sw.js` 의 CACHE 이름이 새 번호인지, 페이지 JS 오류 없는지.
