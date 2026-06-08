# 칼림바 연주 웹앱 바이브코딩 프롬프트

아래 문서는 코딩 AI, Cursor, Claude Code, Bolt, Lovable, Replit Agent 같은 바이브코딩 도구에 그대로 붙여넣어 사용할 수 있는 개발 프롬프트입니다. 목표는 **17키/21키 칼림바 선택 화면**과 **키보드로 실제처럼 연주하는 칼림바 화면**을 갖춘 웹앱을 구현하는 것입니다.

---

## 1. 최종 목표 요약

웹앱을 2개의 주요 페이지로 만든다.

1. **첫 번째 페이지: 칼림바 선택 화면**
   - 사용자가 `17키 칼림바` 또는 `21키 칼림바` 중 하나를 선택한다.
   - 귀여운 칼림바 카드 2개를 보여준다.
   - 각 카드를 클릭/탭하면 선택된 칼림바 연주 화면으로 이동한다.

2. **두 번째 페이지: 칼림바 연주 화면**
   - 선택한 칼림바의 실제 키 배열을 V자 형태로 보여준다.
   - 사용자가 숫자키를 누르면 해당 음이 재생된다.
   - `C4~B4` 기본 옥타브는 숫자키만 누른다.
   - `C5 이상` 높은음은 `Shift + 숫자키`로 연주한다.
   - `B3 이하` 낮은음은 `Ctrl + 숫자키`로 연주한다.
   - 키보드 입력뿐 아니라 화면의 칼림바 키를 마우스/터치로 눌러도 소리가 난다.
   - 누른 키는 화면에서 즉시 하이라이트되어야 한다.

---

## 2. 추천 기술 스택

아래 스택으로 구현한다.

- Frontend: `React + TypeScript + Vite`
- Routing: `react-router-dom`
- Styling: 기본 CSS 또는 CSS Modules
- Audio: 브라우저 `Web Audio API`
- Test: `Vitest` 권장
- Backend: 사용하지 않음
- 외부 이미지 의존: 가급적 없음. 칼림바 일러스트와 키는 CSS/SVG/HTML로 그린다.

---

## 3. 한 번에 붙여넣는 전체 개발 프롬프트

아래 프롬프트를 코딩 AI에 그대로 입력한다.

```text
너는 시니어 프론트엔드 엔지니어이자 인터랙티브 음악 웹앱 개발자다.
React + TypeScript + Vite로 “칼림바 연주 웹앱”을 완성도 있게 구현해줘.

핵심 요구사항:
1. 페이지는 2종류다.
   - `/` : 칼림바 선택 페이지
   - `/play/17` : 17키 칼림바 연주 페이지
   - `/play/21` : 21키 칼림바 연주 페이지
2. 첫 화면에서는 17키 칼림바와 21키 칼림바 중 하나를 선택한다.
   - 귀여운 캐릭터형 칼림바 카드 2개를 보여줘.
   - 각 카드는 버튼처럼 동작해야 하고, hover/active/focus 상태가 있어야 한다.
   - 카드를 클릭하면 해당 연주 페이지로 이동한다.
3. 연주 화면에서는 선택한 칼림바의 실제 키를 V자 배열로 보여준다.
   - 17키는 17개의 tine/key를 렌더링한다.
   - 21키는 21개의 tine/key를 렌더링한다.
   - 각 키에는 음 이름, 숫자악보 표기, 키보드 단축키를 표시한다.
4. 키보드 매핑은 반드시 아래 규칙을 따른다.
   - C4~B4 기본 음역: 숫자키만 사용
     - 1=C4, 2=D4, 3=E4, 4=F4, 5=G4, 6=A4, 7=B4
   - C5 이상 높은음: Shift + 숫자키 사용
     - Shift+1=C5, Shift+2=D5, Shift+3=E5, Shift+4=F5, Shift+5=G5, Shift+6=A5, Shift+7=B5, Shift+8=C6, Shift+9=D6, Shift+0=E6
   - B3 이하 낮은음: Ctrl + 숫자키 사용
     - 21키 칼림바에서 Ctrl+4=F3, Ctrl+5=G3, Ctrl+6=A3, Ctrl+7=B3
     - 17키 칼림바에는 낮은음 F3~B3가 없으므로 Ctrl+4~7은 아무 소리도 내지 않는다.
5. KeyboardEvent 처리 시 반드시 `event.key`가 아니라 `event.code`를 기준으로 숫자키를 판별한다.
   - 이유: Shift+1을 누르면 `event.key`가 `!`가 될 수 있기 때문이다.
   - `Digit1`~`Digit0`와 `Numpad1`~`Numpad0`를 모두 지원한다.
6. Ctrl+Shift+숫자처럼 Ctrl과 Shift가 동시에 눌린 경우는 오입력으로 보고 음을 재생하지 않는다.
7. Alt 또는 Meta 키가 눌린 경우도 브라우저/OS 단축키 충돌을 피하기 위해 음을 재생하지 않는다.
8. `event.repeat`가 true인 반복 keydown은 무시한다. 단, 빠르게 같은 키를 여러 번 따로 누르면 매번 재생되어야 한다.
9. 브라우저 기본 단축키와 충돌할 수 있으므로 연주에 사용된 조합에서는 가능한 범위에서 `preventDefault()`를 호출한다.
10. 화면의 칼림바 키를 마우스 또는 터치로 눌러도 동일한 `playNote(noteId)`가 실행되어야 한다.
11. 오디오 엔진은 Web Audio API로 구현한다.
    - 별도 mp3 파일 없이도 동작해야 한다.
    - 기본은 합성음으로 만든 칼림바 톤이다.
    - 각 음은 fundamental + 약한 overtone 2~3개를 섞고, 짧은 attack과 긴 decay envelope을 준다.
    - 여러 음을 동시에 또는 빠르게 누를 수 있게 polyphony를 허용한다.
    - 첫 사용자 입력 시 AudioContext를 resume해서 브라우저 autoplay 제한을 해결한다.
12. 음 데이터는 하드코딩이 아니라 `src/data/kalimbas.ts` 같은 단일 데이터 파일에 명확히 정의한다.
13. 17키 칼림바의 음역은 C4~E6이다.
14. 21키 칼림바의 음역은 F3~E6이다.
15. 실제 칼림바처럼 낮은 음이 중앙에 있고 양쪽으로 번갈아 높은 음이 배치되는 V자형 physical order를 사용한다.
16. UI는 한국어로 작성한다.
17. 모바일에서도 터치로 연주할 수 있게 반응형으로 만든다.
18. 접근성을 고려한다.
    - 칼림바 키는 button 요소 또는 role=button으로 구현한다.
    - aria-label에 음 이름과 단축키를 포함한다.
    - 키보드 포커스 스타일을 제공한다.
19. 선택한 칼림바 타입은 localStorage에 저장해도 좋다. 단, URL `/play/17`, `/play/21`가 항상 source of truth가 되게 한다.
20. 잘못된 URL `/play/abc`는 선택 화면으로 돌려보내거나 404 안내를 보여준다.

구현할 파일 구조 예시:
- src/main.tsx
- src/App.tsx
- src/routes.tsx 또는 App 내부 라우팅
- src/data/kalimbas.ts
- src/audio/KalimbaAudioEngine.ts
- src/utils/keyboard.ts
- src/pages/SelectKalimbaPage.tsx
- src/pages/PlayKalimbaPage.tsx
- src/components/KalimbaCard.tsx
- src/components/KalimbaInstrument.tsx
- src/components/KalimbaTine.tsx
- src/components/KeyboardGuide.tsx
- src/styles/global.css
- src/utils/keyboard.test.ts

중요 데이터 명세:

17키 pitch order:
C4, D4, E4, F4, G4, A4, B4, C5, D5, E5, F5, G5, A5, B5, C6, D6, E6

17키 physical left-to-right order:
D6, B5, G5, E5, C5, A4, F4, D4, C4, E4, G4, B4, D5, F5, A5, C6, E6

21키 pitch order:
F3, G3, A3, B3, C4, D4, E4, F4, G4, A4, B4, C5, D5, E5, F5, G5, A5, B5, C6, D6, E6

21키 physical left-to-right order:
D6, B5, G5, E5, C5, A4, F4, D4, B3, G3, F3, A3, C4, E4, G4, B4, D5, F5, A5, C6, E6

주요 음 주파수:
F3=174.61, G3=196.00, A3=220.00, B3=246.94,
C4=261.63, D4=293.66, E4=329.63, F4=349.23, G4=392.00, A4=440.00, B4=493.88,
C5=523.25, D5=587.33, E5=659.25, F5=698.46, G5=783.99, A5=880.00, B5=987.77,
C6=1046.50, D6=1174.66, E6=1318.51

숫자악보 표기:
- F3=.4, G3=.5, A3=.6, B3=.7
- C4=1, D4=2, E4=3, F4=4, G4=5, A4=6, B4=7
- C5=1', D5=2', E5=3', F5=4', G5=5', A5=6', B5=7'
- C6=1'', D6=2'', E6=3''

키보드 매핑:
- C4: Digit1/Numpad1
- D4: Digit2/Numpad2
- E4: Digit3/Numpad3
- F4: Digit4/Numpad4
- G4: Digit5/Numpad5
- A4: Digit6/Numpad6
- B4: Digit7/Numpad7
- C5: Shift + Digit1/Numpad1
- D5: Shift + Digit2/Numpad2
- E5: Shift + Digit3/Numpad3
- F5: Shift + Digit4/Numpad4
- G5: Shift + Digit5/Numpad5
- A5: Shift + Digit6/Numpad6
- B5: Shift + Digit7/Numpad7
- C6: Shift + Digit8/Numpad8
- D6: Shift + Digit9/Numpad9
- E6: Shift + Digit0/Numpad0
- F3: Ctrl + Digit4/Numpad4, 21키에서만
- G3: Ctrl + Digit5/Numpad5, 21키에서만
- A3: Ctrl + Digit6/Numpad6, 21키에서만
- B3: Ctrl + Digit7/Numpad7, 21키에서만

구현 순서:
1. Vite + React + TypeScript 프로젝트 기본 구조를 만든다.
2. 전역 스타일과 기본 레이아웃을 만든다.
3. `kalimbas.ts`에 17키/21키 음 데이터, physical order, 키보드 매핑 정보를 정의한다.
4. `keyboard.ts`에 KeyboardEvent를 noteId로 변환하는 순수 함수를 만든다.
5. `KalimbaAudioEngine.ts`에 Web Audio API 기반 playNote 함수를 만든다.
6. 선택 페이지를 만든다.
7. 연주 페이지를 만든다.
8. 칼림바 키 컴포넌트를 만들고 physical order대로 렌더링한다.
9. window keydown listener를 연주 페이지 mount 시 등록하고 unmount 시 해제한다.
10. 클릭/터치 연주를 연결한다.
11. 시각적 pressed 애니메이션을 구현한다.
12. KeyboardGuide를 만들어 사용자가 어떤 키를 눌러야 하는지 볼 수 있게 한다.
13. 모바일 반응형 스타일을 적용한다.
14. keyboard 매핑에 대한 단위 테스트를 작성한다.
15. `npm run build`가 성공하는지 확인한다.
16. 가능한 경우 `npm run test`도 성공하게 만든다.

세부 구현 지침:
- `getDigitFromCode(code: string): '0'|'1'|...|'9'|null` 함수를 만든다.
- `getNoteIdFromKeyboardEvent(event, kalimbaType)` 또는 테스트 가능한 순수 함수 `resolveKeyboardShortcut({ code, shiftKey, ctrlKey, altKey, metaKey }, kalimbaType)`를 만든다.
- Ctrl과 Shift가 동시에 true면 null을 반환한다.
- altKey 또는 metaKey가 true면 null을 반환한다.
- normal octave는 modifier가 없어야만 동작한다.
- high octave는 shiftKey만 true여야 동작한다.
- low octave는 ctrlKey만 true여야 동작한다.
- 숫자키 8, 9, 0은 Shift 조합에서만 C6, D6, E6에 사용한다. modifier 없이 8, 9, 0을 누르면 아무 소리도 내지 않는다.
- 21키 low note는 Ctrl+4~7만 동작한다.
- 17키에서는 Ctrl+4~7도 null이다.

오디오 합성 지침:
- `KalimbaAudioEngine` 클래스를 만든다.
- 생성자에서 AudioContext와 masterGain을 준비한다.
- `ensureReady()`에서 suspended 상태의 context를 resume한다.
- `playFrequency(frequency: number, velocity = 1)`를 만든다.
- 각 음마다 GainNode envelope을 만든다.
- attack은 0.005~0.015초, decay는 1.5~2.8초 정도로 둔다.
- fundamental oscillator는 sine 또는 triangle을 사용한다.
- overtone은 frequency * 2.01, frequency * 3.02 정도를 아주 작은 gain으로 섞는다.
- 최종 gain은 masterGain으로 연결한다.
- note가 끝나면 oscillator를 stop하고 노드를 정리한다.
- 클릭/키보드 입력마다 새 voice를 생성해 polyphony가 가능해야 한다.

UI 디자인 지침:
- 전체 앱은 따뜻하고 귀여운 느낌으로 만든다.
- 배경은 밝은 크림색 또는 은은한 그라데이션을 사용한다.
- 선택 화면은 중앙 정렬된 제목, 설명, 카드 2개로 구성한다.
- 칼림바 카드는 둥근 사각형 body, 작은 얼굴, 팔/다리 느낌의 장식을 CSS로 표현해도 좋다.
- 연주 화면의 칼림바 body는 둥근 나무판 느낌으로 만든다.
- 금속 키는 세로 막대 형태로 만들고, V자형으로 길이가 달라지게 한다.
- 눌린 키는 살짝 아래로 이동하고 밝게 빛나는 효과를 준다.
- 현재 연주된 음 이름을 상단 또는 하단에 잠깐 표시한다.
- 화면 하단에는 키보드 매핑 가이드를 표시한다.
- 볼륨 슬라이더를 제공한다.
- 홈으로 돌아가기 버튼을 제공한다.

접근성 지침:
- 모든 클릭 가능한 요소는 키보드 포커스가 가능해야 한다.
- 포커스 링이 명확해야 한다.
- 칼림바 tine에는 `aria-label="C4, 숫자키 1"` 같은 형태로 설명을 넣는다.
- 연주된 음을 시각적으로만 알리지 말고, `aria-live="polite"` 영역에 최근 음 이름을 업데이트한다.

테스트 지침:
- `resolveKeyboardShortcut` 테스트를 반드시 작성한다.
- 테스트 케이스:
  - Digit1 + no modifier + 17 => C4
  - Digit7 + no modifier + 17 => B4
  - Digit8 + no modifier + 17 => null
  - Shift+Digit1 + 17 => C5
  - Shift+Digit0 + 17 => E6
  - Ctrl+Digit4 + 17 => null
  - Ctrl+Digit4 + 21 => F3
  - Ctrl+Digit7 + 21 => B3
  - Ctrl+Shift+Digit4 + 21 => null
  - Alt+Digit1 + 21 => null
  - Meta+Digit1 + 21 => null
  - Numpad1 + no modifier + 21 => C4
- 데이터 테스트:
  - 17키 note 개수는 17개
  - 21키 note 개수는 21개
  - physical order에 없는 noteId가 없어야 한다.
  - 모든 note는 frequency, shortcutLabel, notation을 가져야 한다.

완료 기준:
- `/`에서 17키/21키 선택이 가능하다.
- `/play/17`에서 17키 칼림바가 렌더링된다.
- `/play/21`에서 21키 칼림바가 렌더링된다.
- 숫자키, Shift+숫자키, Ctrl+숫자키 매핑이 위 명세와 정확히 일치한다.
- 키보드와 마우스/터치 모두 소리가 난다.
- 누른 키가 즉시 시각적으로 반응한다.
- 브라우저 새로고침 후에도 URL에 맞는 칼림바가 표시된다.
- `npm run build`가 성공한다.
- TypeScript 에러가 없어야 한다.
- 테스트를 작성했다면 `npm run test`가 성공한다.

구현 중 모호한 부분은 임의로 단순하게 결정하되, 위 키보드 매핑과 페이지 구조는 절대 바꾸지 마라.
최종 답변에는 구현된 주요 파일, 실행 방법, 테스트 방법, 남은 개선 가능성을 간단히 정리해줘.
```

---

## 4. 단계별 바이브코딩 프롬프트

한 번에 전체 구현이 불안정할 때는 아래 프롬프트를 순서대로 입력한다.

---

### Step 1. 프로젝트 생성과 기본 구조

```text
React + TypeScript + Vite 기반으로 칼림바 연주 웹앱 프로젝트를 시작해줘.
라우팅은 `/`, `/play/17`, `/play/21` 구조로 잡아줘.
아직 오디오는 구현하지 말고, 다음 파일 구조를 먼저 만들어줘.

- src/main.tsx
- src/App.tsx
- src/data/kalimbas.ts
- src/audio/KalimbaAudioEngine.ts
- src/utils/keyboard.ts
- src/pages/SelectKalimbaPage.tsx
- src/pages/PlayKalimbaPage.tsx
- src/components/KalimbaCard.tsx
- src/components/KalimbaInstrument.tsx
- src/components/KalimbaTine.tsx
- src/components/KeyboardGuide.tsx
- src/styles/global.css

전역 스타일은 따뜻하고 귀여운 분위기의 기본 레이아웃만 잡아줘.
TypeScript strict 모드에서 오류가 없게 해줘.
```

---

### Step 2. 칼림바 데이터 모델 작성

```text
`src/data/kalimbas.ts`에 17키와 21키 칼림바 데이터를 완성해줘.
각 note 객체는 최소한 다음 속성을 가져야 한다.

- id: 예: "C4"
- noteName: 예: "C4"
- frequency: number
- notation: 숫자악보 표기, 예: "1", "1'", ".4"
- shortcutLabel: 예: "1", "Shift+1", "Ctrl+4"
- keyboardGroup: "low" | "middle" | "high"

17키 pitch order:
C4, D4, E4, F4, G4, A4, B4, C5, D5, E5, F5, G5, A5, B5, C6, D6, E6

17키 physical left-to-right order:
D6, B5, G5, E5, C5, A4, F4, D4, C4, E4, G4, B4, D5, F5, A5, C6, E6

21키 pitch order:
F3, G3, A3, B3, C4, D4, E4, F4, G4, A4, B4, C5, D5, E5, F5, G5, A5, B5, C6, D6, E6

21키 physical left-to-right order:
D6, B5, G5, E5, C5, A4, F4, D4, B3, G3, F3, A3, C4, E4, G4, B4, D5, F5, A5, C6, E6

주파수:
F3=174.61, G3=196.00, A3=220.00, B3=246.94,
C4=261.63, D4=293.66, E4=329.63, F4=349.23, G4=392.00, A4=440.00, B4=493.88,
C5=523.25, D5=587.33, E5=659.25, F5=698.46, G5=783.99, A5=880.00, B5=987.77,
C6=1046.50, D6=1174.66, E6=1318.51

주의:
데이터 중복을 최소화하고, physicalOrder는 note id 배열로 관리해줘.
렌더링 시 physicalOrder를 기준으로 notes를 찾아 그려야 한다.
```

---

### Step 3. 키보드 매핑 유틸 구현

```text
`src/utils/keyboard.ts`에 키보드 입력을 칼림바 noteId로 변환하는 순수 함수를 구현해줘.

필수 함수:
- getDigitFromCode(code: string): '0'|'1'|'2'|'3'|'4'|'5'|'6'|'7'|'8'|'9'|null
- resolveKeyboardShortcut(input, kalimbaType): string | null

input 구조:
{
  code: string;
  shiftKey: boolean;
  ctrlKey: boolean;
  altKey: boolean;
  metaKey: boolean;
}

규칙:
1. event.key가 아니라 event.code 기준으로 처리한다.
2. Digit1~Digit0, Numpad1~Numpad0를 지원한다.
3. Alt 또는 Meta가 눌렸으면 null.
4. Ctrl과 Shift가 동시에 눌렸으면 null.
5. modifier 없는 1~7은 C4~B4.
6. modifier 없는 8, 9, 0은 null.
7. Shift+1~7은 C5~B5.
8. Shift+8=C6, Shift+9=D6, Shift+0=E6.
9. Ctrl+4=F3, Ctrl+5=G3, Ctrl+6=A3, Ctrl+7=B3. 단, 21키에서만 동작한다.
10. 17키에서 Ctrl+4~7은 null.

Vitest로 이 함수의 테스트도 작성해줘.
```

---

### Step 4. Web Audio 기반 칼림바 사운드 구현

```text
`src/audio/KalimbaAudioEngine.ts`를 구현해줘.
별도 mp3 파일 없이 Web Audio API 합성음만으로 칼림바처럼 들리게 만들어줘.

요구사항:
- AudioContext는 lazy initialization 또는 클래스 생성 시 준비한다.
- 첫 사용자 입력에서 context.resume()을 호출할 수 있게 ensureReady()를 제공한다.
- setVolume(value: number)를 제공한다. value 범위는 0~1이다.
- playFrequency(frequency: number, velocity?: number)를 제공한다.
- playNote(note) 또는 playNoteById(noteId, kalimbaData)를 제공해도 좋다.
- 각 음은 짧은 attack, 긴 decay를 가진 pluck envelope을 사용한다.
- fundamental + overtone 2~3개를 섞는다.
- 여러 음이 겹쳐서 들릴 수 있게 매번 독립 voice를 생성한다.
- 노이즈가 심하지 않게 gain을 적절히 낮춘다.
- TypeScript 오류 없이 작성한다.
```

---

### Step 5. 선택 페이지 UI 구현

```text
`SelectKalimbaPage`와 `KalimbaCard`를 구현해줘.

UI 요구사항:
- 제목: “칼림바를 선택하세요”
- 설명: “17키 또는 21키 칼림바를 골라 바로 연주해보세요.”
- 17키 카드와 21키 카드 2개를 보여준다.
- 각 카드는 귀여운 칼림바 캐릭터 느낌으로 표현한다.
- 버튼 라벨은 “17키 칼림바”, “21키 칼림바”로 한다.
- 클릭하면 각각 `/play/17`, `/play/21`로 이동한다.
- hover, active, focus-visible 상태를 CSS로 구현한다.
- 모바일에서는 카드가 세로로 쌓이고, 데스크톱에서는 나란히 보이게 한다.
```

---

### Step 6. 연주 페이지 UI와 칼림바 렌더링 구현

```text
`PlayKalimbaPage`, `KalimbaInstrument`, `KalimbaTine`, `KeyboardGuide`를 구현해줘.

요구사항:
- URL 파라미터가 17이면 17키 데이터, 21이면 21키 데이터를 사용한다.
- 잘못된 파라미터면 선택 페이지로 이동시키거나 안내를 보여준다.
- 상단에는 뒤로가기/홈 버튼, 현재 칼림바 이름, 볼륨 슬라이더를 둔다.
- 중앙에는 칼림바 body를 보여준다.
- 칼림바 키는 physicalOrder 기준으로 left-to-right 렌더링한다.
- 각 키에는 noteName, notation, shortcutLabel을 표시한다.
- 키 길이는 중앙 쪽이 길고 양쪽이 짧아 보이도록 V자 형태로 만든다.
- 키를 클릭/터치하면 해당 음이 재생된다.
- 최근 연주한 음을 화면에 표시한다.
- 하단에는 키보드 매핑 가이드를 표 또는 카드 형태로 보여준다.
```

---

### Step 7. 키보드 연주 연결

```text
연주 페이지에서 window keydown 이벤트를 연결해줘.

구현 요구사항:
- mount 시 keydown listener를 등록하고 unmount 시 제거한다.
- event.repeat가 true이면 무시한다.
- resolveKeyboardShortcut에 event.code, shiftKey, ctrlKey, altKey, metaKey를 넘긴다.
- 반환된 noteId가 있으면 해당 note를 찾아 playNote를 실행한다.
- 사용된 키 조합에는 가능한 범위에서 event.preventDefault()를 호출한다.
- 연주된 noteId를 active state에 넣어 150~220ms 정도 하이라이트한다.
- 같은 음을 빠르게 여러 번 누르면 하이라이트 타이머가 갱신되어야 한다.
- 입력 포커스가 input, textarea, select, contenteditable 안에 있을 때는 연주하지 않는다.
```

---

### Step 8. 시각 효과와 반응형 polish

```text
앱 전체 디자인을 다듬어줘.

요구사항:
- 따뜻하고 귀여운 분위기.
- 선택 화면의 칼림바 카드는 캐릭터처럼 보이게 얼굴, 볼, 손 모양을 CSS로 표현한다.
- 연주 화면의 칼림바는 나무판 느낌의 rounded body와 금속 tine 느낌을 낸다.
- 눌린 tine은 아래로 살짝 움직이고 glow 효과가 난다.
- 최근 연주 음은 작은 배지로 나타났다 사라지게 한다.
- 모바일에서도 화면 밖으로 넘치지 않게 instrument를 scale 또는 responsive width로 처리한다.
- 터치 대상은 너무 작지 않게 한다.
- prefers-reduced-motion 사용자를 위해 과한 애니메이션을 줄인다.
```

---

### Step 9. 테스트와 품질 점검

```text
테스트와 품질 점검을 추가해줘.

필수 테스트:
- getDigitFromCode가 Digit/Numpad 숫자를 정확히 반환한다.
- resolveKeyboardShortcut이 17키/21키별로 정확한 noteId를 반환한다.
- Ctrl+Shift, Alt, Meta 조합은 null이다.
- 17키에서는 low note Ctrl+4~7이 null이다.
- 21키에서는 Ctrl+4~7이 F3~B3로 매핑된다.
- Shift+0은 E6이다.
- modifier 없는 8/9/0은 null이다.

가능하면 데이터 검증 테스트도 작성해줘.
- 17키 notes length는 17.
- 21키 notes length는 21.
- physicalOrder length가 타입별 키 개수와 일치한다.
- physicalOrder의 모든 noteId가 notes에 존재한다.

마지막으로 다음 명령이 통과되게 해줘.
- npm run build
- npm run test
```

---

## 5. 핵심 데이터 명세 상세표

### 5.1 17키 칼림바 음역과 키보드 매핑

| 음 | 숫자악보 | 키보드 | 주파수 |
|---|---:|---|---:|
| C4 | 1 | 1 | 261.63 |
| D4 | 2 | 2 | 293.66 |
| E4 | 3 | 3 | 329.63 |
| F4 | 4 | 4 | 349.23 |
| G4 | 5 | 5 | 392.00 |
| A4 | 6 | 6 | 440.00 |
| B4 | 7 | 7 | 493.88 |
| C5 | 1' | Shift+1 | 523.25 |
| D5 | 2' | Shift+2 | 587.33 |
| E5 | 3' | Shift+3 | 659.25 |
| F5 | 4' | Shift+4 | 698.46 |
| G5 | 5' | Shift+5 | 783.99 |
| A5 | 6' | Shift+6 | 880.00 |
| B5 | 7' | Shift+7 | 987.77 |
| C6 | 1'' | Shift+8 | 1046.50 |
| D6 | 2'' | Shift+9 | 1174.66 |
| E6 | 3'' | Shift+0 | 1318.51 |

### 5.2 17키 칼림바 실제 화면 배열

왼쪽에서 오른쪽 순서:

```text
D6, B5, G5, E5, C5, A4, F4, D4, C4, E4, G4, B4, D5, F5, A5, C6, E6
```

---

### 5.3 21키 칼림바 음역과 키보드 매핑

| 음 | 숫자악보 | 키보드 | 주파수 |
|---|---:|---|---:|
| F3 | .4 | Ctrl+4 | 174.61 |
| G3 | .5 | Ctrl+5 | 196.00 |
| A3 | .6 | Ctrl+6 | 220.00 |
| B3 | .7 | Ctrl+7 | 246.94 |
| C4 | 1 | 1 | 261.63 |
| D4 | 2 | 2 | 293.66 |
| E4 | 3 | 3 | 329.63 |
| F4 | 4 | 4 | 349.23 |
| G4 | 5 | 5 | 392.00 |
| A4 | 6 | 6 | 440.00 |
| B4 | 7 | 7 | 493.88 |
| C5 | 1' | Shift+1 | 523.25 |
| D5 | 2' | Shift+2 | 587.33 |
| E5 | 3' | Shift+3 | 659.25 |
| F5 | 4' | Shift+4 | 698.46 |
| G5 | 5' | Shift+5 | 783.99 |
| A5 | 6' | Shift+6 | 880.00 |
| B5 | 7' | Shift+7 | 987.77 |
| C6 | 1'' | Shift+8 | 1046.50 |
| D6 | 2'' | Shift+9 | 1174.66 |
| E6 | 3'' | Shift+0 | 1318.51 |

### 5.4 21키 칼림바 실제 화면 배열

왼쪽에서 오른쪽 순서:

```text
D6, B5, G5, E5, C5, A4, F4, D4, B3, G3, F3, A3, C4, E4, G4, B4, D5, F5, A5, C6, E6
```

---

## 6. 구현 시 주의할 함정

### 6.1 `event.key` 대신 `event.code` 사용

`Shift+1`을 누르면 키보드 배열에 따라 `event.key`가 `!`처럼 들어올 수 있다. 따라서 반드시 물리 키 기준인 `event.code`를 사용한다.

예시:

```ts
function getDigitFromCode(code: string): Digit | null {
  if (/^Digit[0-9]$/.test(code)) return code.replace('Digit', '') as Digit;
  if (/^Numpad[0-9]$/.test(code)) return code.replace('Numpad', '') as Digit;
  return null;
}
```

### 6.2 Ctrl+숫자 브라우저 단축키 충돌

일부 브라우저에서는 `Ctrl+숫자`가 탭 이동 같은 단축키로 쓰일 수 있다. 그래도 요구사항에 따라 `Ctrl+4~7`을 구현하되, 다음을 함께 처리한다.

- 해당 조합에서 가능한 경우 `event.preventDefault()` 호출
- 마우스/터치 클릭 연주를 항상 제공
- 화면 가이드에 낮은음 단축키를 명확히 표시

### 6.3 숫자 8, 9, 0 처리

기본음은 `1~7`만 사용한다. 따라서 modifier 없이 `8`, `9`, `0`을 눌러도 음이 나면 안 된다.

- `Shift+8 = C6`
- `Shift+9 = D6`
- `Shift+0 = E6`

### 6.4 Ctrl과 Shift 동시 입력

`Ctrl+Shift+숫자`는 의도하지 않은 입력으로 보고 아무 음도 재생하지 않는다.

---

## 7. 예시 TypeScript 데이터 형태

아래는 코딩 AI가 참고할 수 있는 데이터 구조 예시다. 실제 구현에서는 더 깔끔하게 분리해도 된다.

```ts
export type KalimbaType = '17' | '21';
export type KeyboardGroup = 'low' | 'middle' | 'high';

export interface KalimbaNote {
  id: string;
  noteName: string;
  frequency: number;
  notation: string;
  shortcutLabel: string;
  keyboardGroup: KeyboardGroup;
}

export interface KalimbaConfig {
  type: KalimbaType;
  name: string;
  description: string;
  notes: KalimbaNote[];
  physicalOrder: string[];
}
```

---

## 8. 예시 키보드 매핑 로직

```ts
type Digit = '0'|'1'|'2'|'3'|'4'|'5'|'6'|'7'|'8'|'9';
type KalimbaType = '17' | '21';

interface KeyboardInputLike {
  code: string;
  shiftKey: boolean;
  ctrlKey: boolean;
  altKey: boolean;
  metaKey: boolean;
}

const MIDDLE_MAP: Partial<Record<Digit, string>> = {
  '1': 'C4',
  '2': 'D4',
  '3': 'E4',
  '4': 'F4',
  '5': 'G4',
  '6': 'A4',
  '7': 'B4',
};

const HIGH_MAP: Partial<Record<Digit, string>> = {
  '1': 'C5',
  '2': 'D5',
  '3': 'E5',
  '4': 'F5',
  '5': 'G5',
  '6': 'A5',
  '7': 'B5',
  '8': 'C6',
  '9': 'D6',
  '0': 'E6',
};

const LOW_MAP_21: Partial<Record<Digit, string>> = {
  '4': 'F3',
  '5': 'G3',
  '6': 'A3',
  '7': 'B3',
};

export function resolveKeyboardShortcut(
  input: KeyboardInputLike,
  kalimbaType: KalimbaType,
): string | null {
  if (input.altKey || input.metaKey) return null;
  if (input.shiftKey && input.ctrlKey) return null;

  const digit = getDigitFromCode(input.code);
  if (!digit) return null;

  if (input.shiftKey) {
    return HIGH_MAP[digit] ?? null;
  }

  if (input.ctrlKey) {
    if (kalimbaType !== '21') return null;
    return LOW_MAP_21[digit] ?? null;
  }

  return MIDDLE_MAP[digit] ?? null;
}
```

---

## 9. 완료 후 확인 체크리스트

- [ ] `/`에서 17키/21키 선택 카드가 보인다.
- [ ] 17키 카드를 누르면 `/play/17`로 이동한다.
- [ ] 21키 카드를 누르면 `/play/21`로 이동한다.
- [ ] `/play/17`에서 키가 정확히 17개 보인다.
- [ ] `/play/21`에서 키가 정확히 21개 보인다.
- [ ] 17키 physical order가 명세와 일치한다.
- [ ] 21키 physical order가 명세와 일치한다.
- [ ] `1~7`은 C4~B4를 연주한다.
- [ ] `Shift+1~7`은 C5~B5를 연주한다.
- [ ] `Shift+8`, `Shift+9`, `Shift+0`은 C6, D6, E6을 연주한다.
- [ ] 21키에서 `Ctrl+4~7`은 F3~B3를 연주한다.
- [ ] 17키에서 `Ctrl+4~7`은 소리가 나지 않는다.
- [ ] modifier 없는 `8`, `9`, `0`은 소리가 나지 않는다.
- [ ] `Ctrl+Shift+숫자`는 소리가 나지 않는다.
- [ ] 키보드로 누른 키가 화면에서 하이라이트된다.
- [ ] 마우스/터치로 칼림바 키를 눌러도 소리가 난다.
- [ ] 볼륨 슬라이더가 동작한다.
- [ ] 모바일에서 화면이 깨지지 않는다.
- [ ] `npm run build`가 성공한다.
- [ ] 테스트를 작성했다면 `npm run test`가 성공한다.

---

## 10. 추가 개선 아이디어

필수 구현이 끝난 뒤에는 다음 기능을 확장할 수 있다.

- 녹음/재생 기능
- 연주 기록 MIDI-like 이벤트 저장
- 악보 입력 모드
- 숫자악보 자동 재생 모드
- 다크 모드
- 실제 칼림바 샘플 mp3를 `/public/sounds`에 넣고 합성음 대신 샘플 재생
- 리버브/딜레이 효과
- 메트로놈
- 즐겨찾기 악보
- PWA 설치 지원
