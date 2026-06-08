# 🎵 칼림바 연주 웹앱 — 통합 바이브코딩 프롬프트 (FINAL)

> ChatGPT·Claude·Gemini 세 프롬프트의 장점을 통합한 **결정판**입니다.
> - **데이터 모델 + 순수함수 + 테스트 케이스 + 접근성 + 엣지케이스**(ChatGPT 강점)
> - **실물 칼림바 이미지 위에 투명 오버레이 건반**(Gemini 강점)
> - **"같은 숫자키 = 같은 계이름" 매핑 + Web Audio 합성 레시피 + 기술적 함정 정리**(Claude 강점)
>
> 여기에 이번 요청 2가지를 **최우선 규격**으로 추가했습니다.
> 1. **앱이 9:16 화면 전체에 가득 차도록**
> 2. **17키·21키 칼림바 원본 이미지를 왜곡 없이 그대로 표시**

---

## 0. 이 문서 사용법 & 이미지 자산 준비 (먼저 읽기 ★)

1. 이 마크다운 전체를 복사해 AI 코딩 도구(Claude, Cursor, v0, Bolt, Lovable, Replit Agent 등)에 붙여넣습니다.
2. "위 명세대로 **단일 `index.html` + `assets/` 폴더** 구조의 칼림바 웹앱을 완성해줘"라고 요청합니다.
3. 결과를 받은 뒤 **16번 체크리스트**로 검수/보완을 요청합니다.

**이미지 자산 (★ 반드시 이 매핑대로 사용)** — 한글·날짜 파일명은 경로 오류가 잦으니 아래처럼 단순 영문명으로 바꿔 `assets/`에 넣고, 코드에서는 영문명을 참조하세요.

| 원본 업로드 파일 | 코드에서 쓸 파일명 | 용도 |
|---|---|---|
| `ChatGPT_Image_...02_38_22.png` (귀여운 17키 캐릭터) | `assets/char-17.png` | **선택 화면** 17키 카드 |
| `ChatGPT_Image_...02_38_40.png` (귀여운 21키 캐릭터) | `assets/char-21.png` | **선택 화면** 21키 카드 |
| `17키_칼림바_고양이_01.png` (주황 고양이 실물) | `assets/play-17.png` | **연주 화면** 17키 본체 |
| `21키_칼림바_일러스트_02.png` (벚꽃 우드 실물) | `assets/play-21.png` | **연주 화면** 21키 본체 |

> 이 4개 이미지는 **앱의 핵심 비주얼**이다. 절대 새로 그리거나 대체하지 말고, **원본 그대로** 사용한다(4번 규칙 참고).

---

## 1. 프로젝트 개요 & 핵심 요구사항

브라우저에서 **키보드와 마우스/터치로 칼림바(엄지 피아노)를 연주**하는 인터랙티브 웹앱.

- **화면 1 (선택)**: `17키 칼림바` / `21키 칼림바`를 귀여운 캐릭터 카드로 고른다.
- **화면 2 (연주)**: 선택한 칼림바의 **실물 사진**이 9:16 화면에 크게 표시되고, 그 위 투명 건반과 키보드로 연주한다.
- **입력 규칙(핵심)**: 숫자키=가운데 음 / `Shift`+숫자=높은음(C5↑) / `Ctrl`+숫자=낮은음(B3↓, 21키 전용).
- **사운드**: Web Audio API **합성**(mp3 없이 동작, 추후 샘플 교체 쉽게 구조화).
- **톤**: 따뜻하고 귀여운(카와이) 분위기. 둥근 모서리, 부드러운 색감, 경쾌한 마이크로 인터랙션.

---

## 2. 기술 스택 & 결과물 형태

- **권장(기본): 단일 `index.html`** (HTML + CSS + Vanilla JS, 빌드 불필요) + `assets/` 이미지 폴더.
  - 더블클릭으로 바로 실행, 배포·공유 간단, 이미지 드롭만으로 적용.
- 사운드: **Web Audio API** (`AudioContext`, `OscillatorNode`, `GainNode`, `BiquadFilterNode`).
- 코드 품질(아래는 단일 파일 안에서도 지킨다):
  - 음 데이터는 **데이터 객체/배열로 분리**(하드코딩 분산 금지).
  - 키 입력→음 변환은 **테스트 가능한 순수 함수**로.
  - **접근성**(role/aria/focus) 반영.
- (선택) React+TypeScript+Vite 구조를 원하면 **부록 B** 참고. 단, 본 명세는 단일 파일 기준.

---

## 3. ★ 화면/레이아웃 규격 — 9:16 풀스크린 (이번 요청 1)

앱은 **세로 9:16 비율의 한 화면**을 기준으로 하고, **뷰포트를 가득 채운다.**

- 앱 루트(`#app`)는 **항상 9:16 비율**을 유지하며, 화면에 맞춰 최대 크기로 중앙 배치된다.
  - 모바일 세로: 화면 전체를 채움.
  - 데스크톱/가로: 9:16 패널이 화면 높이에 맞춰 중앙에 크게 표시되고, 바깥 여백은 어두운/은은한 배경으로 채운다(레터박스).
- 모바일 주소창 대응으로 **`100dvh`** 사용(폴백 `100vh`).
- 화면 1·2는 각각 `#app` 안에서 `position:absolute; inset:0;`로 **9:16를 가득** 채운다. 빈 흰 여백이 보이면 안 된다(테마 배경으로 채움).

**레퍼런스 CSS**
```css
* { box-sizing: border-box; }
html, body { margin: 0; height: 100%; }
body {
  display: grid; place-items: center;
  min-height: 100dvh;
  background: #15130F;           /* 데스크톱 레터박스 배경 */
  font-family: "Pretendard", system-ui, sans-serif;
}
#app {
  position: relative;
  width:  min(100vw, calc(100dvh * 9 / 16));
  height: min(100dvh, calc(100vw * 16 / 9));
  overflow: hidden;
  background: var(--bg);         /* 앱 내부 배경(크림) */
  box-shadow: 0 20px 60px rgba(0,0,0,.35);
}
.screen { position: absolute; inset: 0; display: flex; flex-direction: column; }
.screen[hidden] { display: none; }
```

> 모든 폰트 크기·간격·이미지 크기는 `#app` 크기에 비례하도록 `%`, `vmin`, `clamp()`, `cqw`(컨테이너 쿼리) 등을 사용해 **어떤 화면에서도 비율이 깨지지 않게** 한다.

---

## 4. ★ 이미지 자산 사용 규칙 — 원본 무왜곡 (이번 요청 2)

**0번 표의 원본 이미지를 변형 없이 그대로 표시한다.**

- 모든 칼림바 이미지는 **종횡비를 절대 변경하지 않는다**(가로/세로 따로 늘이기 금지).
- 표시는 **`object-fit: contain`** 으로 한다 → 이미지가 **잘리거나 찌그러지지 않고 전체가** 보인다.
  - (참고) `cover`는 가장자리를 잘라내므로 "그대로 표시" 요구에 어긋난다. 기본은 `contain`.
- 이미지가 차지하고 남는 공간은 **테마 배경색/패턴으로 채워** 화면이 비어 보이지 않게 한다.
- **연주 화면 사진은 9:16 안에서 최대한 크게**(가용 영역 높이에 맞춰) 키우되, 비율은 유지한다.
- 실물 사진(`play-17.png`, `play-21.png`)은 저해상도일 수 있다. 부드럽게 보이도록 `image-rendering:auto`를 쓰고, 업스케일로 인한 흐림은 감수한다(원본 유지가 우선). 더 선명한 고해상 사진이 있으면 교체만 하면 되도록 `<img src>`만 바꾸면 되게 구조화한다.

**연주 화면의 사진 + 오버레이 정렬 방법(중요)**
사진 위에 투명 건반을 정확히 얹기 위해, **사진과 같은 비율의 래퍼**를 만들고 그 안에서 **백분율 좌표**로 건반을 배치한다. 래퍼가 사진과 동일 비율이므로 화면 크기가 바뀌어도 건반이 사진과 함께 정확히 스케일된다.

```css
.kalimba-stage{                 /* 사진과 동일한 종횡비로 고정 */
  position: relative;
  margin: auto;
  height: 100%;                 /* 가용 영역을 꽉 채우도록 */
  max-width: 100%;
  aspect-ratio: 218 / 277;      /* 17키 사진 비율 (21키는 232/297) */
}
.kalimba-photo{                 /* 원본 무왜곡 */
  width: 100%; height: 100%;
  object-fit: contain;
  display: block; user-select: none; -webkit-user-drag: none;
}
.tine-layer{ position: absolute; inset: 0; }   /* 투명 건반 레이어 */
.tine{ position: absolute; background: transparent; border: 0; cursor: pointer; }
```

---

## 5. 공통 디자인 시스템

업로드 영상·캐릭터의 **귀엽고 따뜻한 톤**을 따른다.

**색상(CSS 변수)**

| 토큰 | 값 | 용도 |
|---|---|---|
| `--bg` | `#FBF6EE` | 앱 배경(크림) |
| `--green` | `#6FAE4A` → `#5C9A3C` | 선택 라벨/포인트 |
| `--brown` | `#5A4034` | 텍스트·우드 톤 |
| `--silver` | `#E2E2E6`→`#A9A9B0` | 금속 느낌 보조 |
| `--glow` | `#FFD66B` | 연주 시 하이라이트 |
| `--ink` | `#3A2E28` | 본문 텍스트 |

**스타일 원칙**: 큰 라운드(16~28px), 부드러운 그림자, 클릭 시 살짝 튀는 bounce, 화면 전환은 0.25~0.35s **페이드/슬라이드**. `prefers-reduced-motion`이면 애니메이션 최소화.

---

## 6. [화면 1] 선택 화면 — 캐릭터 카드

**레이아웃(9:16 세로 가득)**
- 상단: 앱 타이틀(예: "🎵 칼림바 연주하기") + 한 줄 안내("연주할 칼림바를 골라주세요").
- 본문: **세로로 2개의 큰 카드**(9:16 높이를 고르게 분할).
  - **17키 카드**: `assets/char-17.png`(무왜곡, `object-fit:contain`) + 하단 **초록 라운드 라벨 "17키 칼림바"**.
  - **21키 카드**: `assets/char-21.png` + 라벨 **"21키 칼림바"**.

**상호작용**
- hover/focus: `scale(1.04)` + 떠오르는 그림자(+가벼운 bounce).
- 클릭/Enter/Space:
  1. 선택 카드 강조(확대·완전 채도), 반대 카드 흐리게(`opacity:.4`).
  2. 클릭 지점 **물결(ripple)** 효과.
  3. 0.3~0.5s 후 **연주 화면으로 페이드 전환**(선택값 `17|21` 전달).
- 접근성: 카드는 `role="button"`, `tabindex="0"`, `aria-label="17키 칼림바 선택"`. 또렷한 포커스 링.

---

## 7. [화면 2] 연주 화면 — 실물 사진 + 오버레이 건반

**레이아웃(9:16 세로 가득)**
- **상단 바(슬림)**: 좌측 `← 다시 선택`(화면 1 복귀), 가운데 칼림바 이름("17키 칼림바"), 우측 **볼륨 슬라이더**.
- **메인(가장 크게)**: `.kalimba-stage` 안에 **선택된 실물 사진**(`play-17.png` 또는 `play-21.png`)을 **무왜곡·최대 크기**로 표시(4번 규칙).
- **하단(접이식 가능)**: **키 안내 패널** — "숫자키=가운데 / ⇧+숫자=높은음 / ⌃+숫자=낮은음(21키)" 요약 + 매핑 표 토글. **최근 연주음 배지**를 잠깐 띄움.

**오버레이 건반(투명 hotspot)**
- 사진은 **장식 배경**으로 두고, 그 위에 **정확한 음 매핑을 가진 투명 건반 N개**(17 또는 21)를 **물리 배열 순서(9번)대로 좌→우** 배치한다.
- 각 건반은 **세로 막대(컬럼)** 형태의 투명 버튼. **건반 영역(tine band)** 을 N등분해 균등 배치한다.
- **건반 영역 기본 좌표(백분율, 사진 기준 / 미세조정 가능)**:
  - 17키(`play-17.png`): `{ left: 7%, right: 93%, top: 6%, bottom: 60% }`
  - 21키(`play-21.png`): `{ left: 6%, right: 94%, top: 6%, bottom: 64% }`
  - 위 값은 시작점이다. 실제 사진과 어긋나면 **이 한 객체만** 보고 시각적으로 미세조정한다.
- 누르면(키보드/클릭 공통): 해당 컬럼에 **노란 글로우 + 물결**이 잠깐 뜨고 소리 재생. **사진은 절대 변형하지 않는다.**
- 접근성: 각 건반 `role="button"`, `aria-label="C4, 단축키 1"`. 마우스·터치·키보드 모두 같은 `playNote(noteId)` 실행.

> 건반은 **논리적 음 위치**이므로 사진의 금속 막대와 1:1 픽셀 정합이 아니어도 된다(스톡 사진의 실제 막대 수가 17/21과 다를 수 있음). **키보드 연주가 1차 수단**이며 어떤 경우에도 정확히 동작해야 한다.

---

## 8. 키보드 입력 규칙 (핵심 ★ 통합 결정판)

세 음역을 **수식키**로 구분한다.

| 입력 | 음역 |
|---|---|
| `숫자키` 단독 | 가운데 옥타브 C4~B4 |
| `Shift + 숫자키` | 높은음 C5 이상 |
| `Ctrl + 숫자키` | 낮은음 B3 이하 (**21키 전용**) |

**설계 원칙 — "같은 숫자키 = 같은 계이름(도레미), 수식키 = 옥타브"**
> 1~7 = 도·레·미·파·솔·라·시(C·D·E·F·G·A·B). 수식키가 옥타브 결정: 단독=가운데, Shift=위, Ctrl=아래.
> 예) **4번 키 = 항상 '파'** → `Ctrl+4=F3`, `4=F4`, `Shift+4=F5`.
> 가장 높은 C6·D6·E6은 숫자키가 부족하므로 `Shift+8/9/0`에 배정한다.

---

## 9. 음 데이터 (17키 / 21키)

> **표준 C장조.** 가운데가 가장 낮은 음(가장 긴 tine), 좌우로 갈수록 높아짐.
> 21키 = 17키에 **저음 4개(F3·G3·A3·B3)** 를 가운데에 추가한 구조(음역 F3~E6). 17키 음역은 C4~E6.

**물리 배열 (화면 좌→우 순서)**
```js
// 17키 (총 17개)
const ORDER_17 = ["D6","B5","G5","E5","C5","A4","F4","D4",
                  "C4",                       // 가운데(가장 낮음)
                  "E4","G4","B4","D5","F5","A5","C6","E6"];

// 21키 (총 21개)
const ORDER_21 = ["D6","B5","G5","E5","C5","A4","F4","D4",
                  "B3","G3","F3","A3",        // 가운데 저음 4개(F3가 가장 낮음)
                  "C4","E4","G4","B4","D5","F5","A5","C6","E6"];
```

**주파수(Hz) & 숫자악보**
```js
const NOTE = {
  // note: [frequency, 숫자악보]
  F3:[174.61,".4"], G3:[196.00,".5"], A3:[220.00,".6"], B3:[246.94,".7"],
  C4:[261.63,"1"], D4:[293.66,"2"], E4:[329.63,"3"], F4:[349.23,"4"],
  G4:[392.00,"5"], A4:[440.00,"6"], B4:[493.88,"7"],
  C5:[523.25,"1'"], D5:[587.33,"2'"], E5:[659.25,"3'"], F5:[698.46,"4'"],
  G5:[783.99,"5'"], A5:[880.00,"6'"], B5:[987.77,"7'"],
  C6:[1046.50,"1''"], D6:[1174.66,"2''"], E6:[1318.51,"3''"]
};
// 또는 공식: freq = 440 * 2^((midi-69)/12)
// MIDI: F3=53,G3=55,A3=57,B3=59, C4=60..B4=71, C5=72..B5=83, C6=84,D6=86,E6=88
```

---

## 10. 키 ↔ 음 매핑 완성표

**가운데 — 숫자키 단독 (공통)**: `1`=C4, `2`=D4, `3`=E4, `4`=F4, `5`=G4, `6`=A4, `7`=B4. (`8/9/0` 단독 = 무음)

**높은음 — Shift+숫자 (공통)**

| 키 | 음 | 키 | 음 |
|---|---|---|---|
| ⇧1 | C5 | ⇧6 | A5 |
| ⇧2 | D5 | ⇧7 | B5 |
| ⇧3 | E5 | ⇧8 | C6 |
| ⇧4 | F5 | ⇧9 | D6 |
| ⇧5 | G5 | ⇧0 | E6 |

**낮은음 — Ctrl+숫자 (★ 21키에서만)**: `⌃4`=F3, `⌃5`=G3, `⌃6`=A3, `⌃7`=B3. (17키에선 무시, `⌃1/2/3/8/9/0`도 무음)

**참고 매핑 객체**
```js
const MID  = { Digit1:"C4",Digit2:"D4",Digit3:"E4",Digit4:"F4",Digit5:"G4",Digit6:"A4",Digit7:"B4" };
const HIGH = { Digit1:"C5",Digit2:"D5",Digit3:"E5",Digit4:"F5",Digit5:"G5",Digit6:"A5",Digit7:"B5",
               Digit8:"C6",Digit9:"D6",Digit0:"E6" };
const LOW  = { Digit4:"F3",Digit5:"G3",Digit6:"A3",Digit7:"B3" }; // 21키 전용
```

---

## 11. 키 입력 처리 로직 (순수 함수 + 엣지케이스 + 테스트)

**반드시 지킬 규칙**
1. **`event.code` 사용**(`event.key` 금지). `Shift+1`은 `key`가 `"!"`로 바뀌기 때문. **`Digit1~0`과 `Numpad1~0` 둘 다** 지원.
2. **`Alt` 또는 `Meta`(⌘)** 가 눌리면 → 무음(OS/브라우저 단축키 충돌 방지).
3. **`Ctrl`+`Shift` 동시** → 무음(오입력 처리).
4. 수식키 없는 `8/9/0` → 무음. `Shift+8/9/0`만 C6/D6/E6.
5. `Ctrl`+숫자는 **21키에서만** 동작(`Ctrl+4~7`→F3~B3). 17키는 전부 무음.
6. **`event.repeat`(꾹 누름)** 무시. 단, 같은 키를 따로 빠르게 여러 번 누르면 매번 재생.
7. 매핑된 조합에서 **`event.preventDefault()`** 호출(탭 전환·저장 등 차단).
8. 포커스가 `input/textarea/select/contenteditable` 안이면 연주하지 않음.

**테스트 가능한 순수 함수(이 시그니처로 구현)**
```js
// resolveShortcut({code, shiftKey, ctrlKey, altKey, metaKey}, type:"17"|"21") -> noteId | null
function digitOf(code){
  const m = /^(?:Digit|Numpad)([0-9])$/.exec(code);
  return m ? m[1] : null;            // "0".."9" | null
}
const MAP = {
  MID:{1:"C4",2:"D4",3:"E4",4:"F4",5:"G4",6:"A4",7:"B4"},
  HIGH:{1:"C5",2:"D5",3:"E5",4:"F5",5:"G5",6:"A5",7:"B5",8:"C6",9:"D6",0:"E6"},
  LOW:{4:"F3",5:"G3",6:"A3",7:"B3"}
};
function resolveShortcut(e, type){
  if (e.altKey || e.metaKey) return null;
  if (e.shiftKey && e.ctrlKey) return null;
  const d = digitOf(e.code); if (d === null) return null;
  if (e.shiftKey) return MAP.HIGH[d] ?? null;
  if (e.ctrlKey)  return type === "21" ? (MAP.LOW[d] ?? null) : null;
  return MAP.MID[d] ?? null;
}
```

**필수 테스트 케이스**
- `Digit1` (수식X, 17) → `C4`
- `Digit7` (수식X) → `B4`
- `Digit8` (수식X) → `null`
- `Shift+Digit1` → `C5`, `Shift+Digit0` → `E6`
- `Ctrl+Digit4` (17) → `null`
- `Ctrl+Digit4` (21) → `F3`, `Ctrl+Digit7` (21) → `B3`
- `Ctrl+Shift+Digit4` → `null`
- `Alt+Digit1` / `Meta+Digit1` → `null`
- `Numpad1` (수식X) → `C4`

데이터 검증: 17키 음 17개·21키 음 21개, 배열의 모든 noteId가 `NOTE`에 존재.

---

## 12. 사운드 엔진 (Web Audio 합성)

**목표 음색**: 밝고 맑은 금속 어택 + 부드러운 감쇠, 낮은 음일수록 길게. **폴리포니**(동시·연속 발음). mp3 없이 동작, 추후 샘플 교체 쉽게.

**필수 사항**
- `AudioContext`는 **1회 생성·재사용**. **첫 사용자 입력(키/클릭) 시 `ctx.resume()`**.
- **마스터 게인 + 볼륨 슬라이더**(0~1). 음마다 독립 voice 생성 후 종료 시 정리.
- 음 = 기본음 + 약한 배음(2x, 3.01x, 5.4x). **빠른 어택(5~15ms) + 지수 감쇠(약 1.4~2.6s)**. 로우패스로 고역 정리.

**레퍼런스 구현**
```js
const ctx = new (window.AudioContext || window.webkitAudioContext)();
const master = ctx.createGain(); master.gain.value = 0.8; master.connect(ctx.destination);
const setVolume = v => master.gain.value = Math.max(0, Math.min(1, v));
const freqOf = n => NOTE[n][0];

function playNote(noteName){
  const f = freqOf(noteName); if (!f) return;
  if (ctx.state === "suspended") ctx.resume();        // ★ autoplay 해제
  const now = ctx.currentTime;
  const midi = 69 + 12*Math.log2(f/440);
  const dur = Math.max(1.4, 2.6 - (midi - 53) * 0.025); // 낮을수록 길게

  const lp = ctx.createBiquadFilter(); lp.type="lowpass"; lp.frequency.value = Math.min(7000, f*8);
  const env = ctx.createGain(); env.connect(lp).connect(master);
  env.gain.setValueAtTime(0.0001, now);
  env.gain.exponentialRampToValueAtTime(0.9, now + 0.006);  // 빠른 어택
  env.gain.exponentialRampToValueAtTime(0.0001, now + dur); // 지수 감쇠

  [[1,1.0],[2,0.45],[3.01,0.22],[5.4,0.08]].forEach(([mul,g])=>{
    const o = ctx.createOscillator(); o.type="sine"; o.frequency.value = f*mul;
    const og = ctx.createGain(); og.gain.value = g;
    o.connect(og).connect(env); o.start(now); o.stop(now + dur + 0.1);
  });
}
```
> (선택) 더 사실적인 소리는 음별 샘플(.wav/.mp3) 로드로 교체 가능하게 `playNote`만 갈아끼우면 되도록 분리한다.

---

## 13. 상호작용 & 시각 효과

- **연주 피드백(키보드/클릭 공통)**: 해당 건반 컬럼에 **노란 글로우 + 물결**이 0.2초 뜨고 사라짐. **사진은 변형 금지**(오버레이만 반응).
- `keydown`에서 소리+효과, `keyup`(또는 타이머)으로 해제. `event.repeat` 무시.
- **최근 연주음 배지**(예: "도 (C4)")를 상단/하단에 잠깐 표시 후 페이드아웃.
- (접근성) 시각 효과와 별개로 `aria-live="polite"` 영역에 최근 음 이름을 갱신.

---

## 14. 접근성 & 반응형

- 모든 클릭 요소는 키보드 포커스 가능 + 또렷한 포커스 링.
- 칼림바 건반/카드: `role="button"`, `aria-label`(음+단축키 / 선택 안내).
- `aria-live`로 최근 음 안내. `prefers-reduced-motion` 시 애니메이션 축소.
- 터치 타깃은 충분히 크게. 가로/세로·다양한 해상도에서 9:16 비율과 이미지 무왜곡 유지.

---

## 15. 단계별 개발 순서 (Phase 1~7)

> 각 단계 후 동작 확인하고 진행.

1. **뼈대 & 9:16 풀스크린**: `#app`(9:16, 3번 CSS) + `.screen` 2개(선택/연주) 토글 + `← 다시 선택`. 데스크톱 레터박스 확인.
2. **데이터**: `ORDER_17/21`, `NOTE`, `MAP` 정의(9·10번). 17키 17개·21키 21개 검증.
3. **키 입력 순수함수 + 테스트**: `digitOf`, `resolveShortcut`(11번) + 테스트 케이스 통과.
4. **사운드 엔진**: `playNote`, 마스터 게인/볼륨, 첫 입력 `resume`(12번). 폴리포니 확인.
5. **선택 화면**: `char-17/21.png` 무왜곡 카드 + 초록 라벨 + hover/ripple/페이드 전환(6번).
6. **연주 화면 + 오버레이**: `play-17/21.png` 무왜곡 최대 표시 + `.kalimba-stage`(사진 비율) + 투명 건반 N개를 tine band(7번 좌표) 균등 배치 + 키보드/클릭 연결 + 글로우/물결 + 최근음 배지.
7. **마감**: 접근성(role/aria/focus), `prefers-reduced-motion`, 키 안내 패널/볼륨, 다양한 화면 점검·polish.

---

## 16. 완성 체크리스트

**레이아웃/이미지(이번 요청 핵심)**
- [ ] 앱이 **9:16 비율**로 뷰포트를 가득 채운다(모바일 풀, 데스크톱 중앙 레터박스).
- [ ] 화면에 빈 흰 여백 없이 테마 배경으로 채워진다.
- [ ] 선택 화면 캐릭터 이미지가 **왜곡 없이**(contain) 표시된다.
- [ ] 연주 화면 실물 사진이 **왜곡·잘림 없이** 9:16 안에서 **최대 크기**로 표시된다.
- [ ] 투명 건반이 사진과 함께 정확히 스케일된다(화면 크기 바뀌어도 어긋나지 않음).

**연주/매핑**
- [ ] 17키 17개 / 21키 21개 건반이 물리 배열대로 좌→우 배치.
- [ ] `1~7`→C4~B4, `Shift+1~0`→C5~E6, `Ctrl+4~7`→F3~B3(21키만), 17키 Ctrl 무음.
- [ ] `같은 숫자=같은 계이름`(`⌃4=F3,4=F4,⇧4=F5`).
- [ ] `8/9/0` 단독·`Ctrl+Shift`·`Alt`·`Meta` 무음. `Numpad`도 동작.
- [ ] 키 꾹 눌러도 연타 안 됨, 빠른 반복은 매번 재생.
- [ ] `Ctrl+숫자`로 브라우저 탭 안 바뀜.
- [ ] 키보드·마우스·터치 모두 소리. 폴리포니. 첫 입력에서 정상 발음(resume).
- [ ] 누른 건반 즉시 시각 반응 + 최근음 배지.

**품질**
- [ ] 순수함수 테스트 케이스 통과. 데이터 개수 검증 통과.
- [ ] 포커스 링/`aria-label`/`aria-live` 동작. `prefers-reduced-motion` 반영.

---

## 17. (선택) 확장 아이디어

녹음/재생, 숫자악보 입력→자동 연주, 동요 연습 모드(반짝반짝 작은별 등), 리버브/딜레이, 메트로놈, 다크 모드, 실물 mp3 샘플 교체, PWA 설치, 키 힌트 표시 토글.

---

## 부록 A. 그대로 복붙하는 압축 프롬프트

```
키보드와 마우스/터치로 연주하는 "칼림바(엄지 피아노) 웹앱"을 외부 라이브러리 없이 단일 index.html(HTML+CSS+Vanilla JS) + assets/ 이미지 폴더로 만들어줘. 사운드는 Web Audio API 합성으로.

[★ 화면 규격 — 9:16 풀스크린]
- 앱 루트(#app)는 항상 9:16 비율로 뷰포트를 가득 채움. 모바일은 전체, 데스크톱은 화면 높이에 맞춘 9:16 패널을 중앙에 두고 바깥은 어두운 배경(레터박스). 100dvh 사용. 빈 흰 여백 금지(테마 배경으로 채움).
- #app { width:min(100vw, calc(100dvh*9/16)); height:min(100dvh, calc(100vw*16/9)); position:relative; overflow:hidden; }

[★ 이미지 — 원본 무왜곡으로 그대로]
- assets/char-17.png(귀여운 17키 캐릭터), assets/char-21.png(귀여운 21키 캐릭터) → 선택 화면 카드.
- assets/play-17.png(주황 고양이 실물), assets/play-21.png(벚꽃 우드 실물) → 연주 화면 본체.
- 모든 이미지는 종횡비 유지 + object-fit:contain(잘림·찌그러짐 금지, 전체가 보이게). 연주 사진은 9:16 안에서 최대 크기로. 남는 공간은 테마 배경.

[화면 흐름]
- 화면1(선택): char-17/21.png 카드 2개(무왜곡) + 초록 라벨 "17키 칼림바"/"21키 칼림바", hover bounce + 클릭 ripple + 페이드 전환.
- 화면2(연주): 선택한 실물 사진을 무왜곡 최대 표시. "← 다시 선택" 버튼, 볼륨 슬라이더, 최근음 배지, 키 안내 패널.

[연주 화면 오버레이 건반]
- 사진은 장식 배경. 그 위에 사진과 같은 비율의 .kalimba-stage(17키 aspect 218/277, 21키 232/297)를 만들고, 그 안에 투명 건반 N개를 백분율 좌표로 균등 배치(키보드 연주가 1차, 클릭은 보조). tine band 기본값 17키 {left7%,right93%,top6%,bottom60%}, 21키 {left6%,right94%,top6%,bottom64%}(미세조정 가능). 누르면 노란 글로우+물결, 사진은 변형 금지.

[건반 물리 배열 좌→우]
- 17키(C4~E6): D6 B5 G5 E5 C5 A4 F4 D4 [C4=가운데] E4 G4 B4 D5 F5 A5 C6 E6
- 21키(F3~E6): D6 B5 G5 E5 C5 A4 F4 D4 [B3 G3 F3 A3=가운데 저음4] C4 E4 G4 B4 D5 F5 A5 C6 E6

[키 입력 규칙 — 핵심]
- 숫자 1~7 = 계이름(1=도C,2=레D,3=미E,4=파F,5=솔G,6=라A,7=시B). 수식키=옥타브.
- 단독:1~7=C4~B4 / Shift:1~7=C5~B5, 8=C6,9=D6,0=E6 / Ctrl(21키만):4=F3,5=G3,6=A3,7=B3. 같은 숫자=같은 계이름(⌃4=F3,4=F4,⇧4=F5).
- 반드시 event.code(Digit1~0 + Numpad1~0)와 shiftKey/ctrlKey로 판별(Shift+1은 key가 "!"라서). Alt/Meta 또는 Ctrl+Shift 동시 → 무음. 8/9/0 단독 무음. 17키 Ctrl 무음. event.repeat 무시. 매핑된 키는 preventDefault. input/textarea 포커스 시 연주 안 함.
- 순수함수 resolveShortcut({code,shiftKey,ctrlKey,altKey,metaKey}, "17"|"21")로 구현하고 테스트 케이스도 작성: Digit1→C4, Digit8→null, Shift+Digit0→E6, Ctrl+Digit4(21)→F3, Ctrl+Digit4(17)→null, Ctrl+Shift+Digit4→null, Alt/Meta+Digit1→null, Numpad1→C4.

[사운드]
- AudioContext 1회 생성·재사용, 첫 입력에 ctx.resume(). 마스터 게인+볼륨 슬라이더(0~1). 음=기본음+배음(2x,3.01x,5.4x), 빠른 어택(~6ms)+지수 감쇠(낮을수록 길게 1.4~2.6s), 로우패스. 폴리포니. 주파수=440*2^((midi-69)/12). MIDI: F3=53,G3=55,A3=57,B3=59,C4=60..B4=71,C5=72..B5=83,C6=84,D6=86,E6=88.

[접근성/반응형] role=button + aria-label(음+단축키), 포커스 링, aria-live로 최근음, prefers-reduced-motion 반영. 어떤 화면에서도 9:16 비율과 이미지 무왜곡 유지.

색: 크림 배경 #FBF6EE, 초록 #6FAE4A, 브라운 #5A4034, 노란 하이라이트 #FFD66B. 따뜻하고 귀여운 톤. 완료 후 주요 구현/실행법/체크리스트를 요약해줘.
```

---

## 부록 B. (선택) React + TypeScript 구조로 만들 경우

단일 파일 대신 견고한 구조를 원하면 아래 구성을 따른다(매핑·데이터·규격은 본문과 동일).

```
src/
  data/kalimbas.ts          # ORDER_17/21, NOTE, MAP, KalimbaConfig 타입
  utils/keyboard.ts         # digitOf, resolveShortcut (+ keyboard.test.ts)
  audio/KalimbaAudioEngine.ts  # AudioContext, master gain, playNote, setVolume
  pages/SelectScreen.tsx    # char-17/21.png 카드
  pages/PlayScreen.tsx      # play-17/21.png + 오버레이 건반(9:16)
  components/KalimbaStage.tsx, Tine.tsx, KeyboardGuide.tsx
  styles/global.css         # #app 9:16 풀스크린, .kalimba-stage 등
public/assets/char-17.png, char-21.png, play-17.png, play-21.png
```
- 라우팅: `/`, `/play/17`, `/play/21`(URL이 source of truth, 잘못된 값은 선택 화면으로).
- `resolveShortcut`는 **Vitest**로 부록 A의 테스트 케이스를 검증.
- 이미지·9:16·무왜곡 규칙(3·4번)은 동일하게 적용.

---

*이 명세대로 만들면 9:16 화면을 가득 채우고, 17·21키 원본 이미지를 왜곡 없이 그대로 보여주며, 정확한 키 매핑으로 연주되는 칼림바 웹앱이 완성됩니다.* 🎶
