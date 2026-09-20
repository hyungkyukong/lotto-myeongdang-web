# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

유튜브 댓글 자동 답글이 안내하는 공개 웹페이지. "로또 명당"과 같은 와인×골드 테마로
지난주 당첨번호와 이번주 추천 조합 5게임을 보여준다. 프로젝트 공통 규칙은 상위
`../CLAUDE.md`에 있다. GitHub Pages로 배포하는 **별도의 작은 저장소**다 —
`automation/`(구글 인증정보 보관)와 절대 같은 저장소로 만들지 않는다.

## 구조

- `index.html`: 정적 페이지. `./data.json`을 fetch해서 렌더링한다. 빌드 단계 없음(순수 HTML+CSS+JS 한 파일).
- `data.json`: `shorts/assets/lotto-theme/fetch_data.mjs`가 만드는 그 파일을 그대로 복사해 온 것.
  `recommend.games`(6개짜리 추천 조합 5세트), `recommend.carryoverSource`(이월수 출처),
  `recommend.neglectedPool`(소외수 후보)까지 포함한다 — `lottoRecommender.js`의
  `generateGames()`를 서버(fetch_data.mjs) 쪽에서 이미 실행해 둔 결과다.

## 데이터 흐름과 "다시 뽑기"

`generateGames()`의 계산은 두 단계로 나뉜다: (1) 이월수/소외수 **후보 풀**을 정하는 부분
(결정적, `analyzeFrequency` 기반)과 (2) 그 풀에서 `Math.random()`으로 실제 6개를 뽑는 부분
(비결정적). data.json에는 이미 뽑힌 게임 5세트가 들어있어 처음 로드 시 바로 보여줄 수
있고, 페이지의 "다시 뽑기" 버튼은 `index.html` 안에 **(2)만 그대로 포팅한 JS**로
`carryoverSource.numbers`/`neglectedPool`을 이용해 그 자리에서 재조합한다 — 서버 재호출도
없고 로직도 하나만 존재한다(포팅한 부분은 `lottoRecommender.js`의 `generateGames`/
`pickRandom`과 정확히 같은 알고리즘이므로, 그쪽을 고치면 여기도 같이 고친다).

## 배포

GitHub Pages, `main` 브랜치 루트에서 배포. 매주 `automation/weekly_run.mjs`가 실행 후
1) 새로 만든 `data.json`을 이 폴더로 복사, 2) `git add -A && git commit && git push`.
**이 저장소에는 `client_secret.json`/`token.json` 등 인증정보가 절대 들어가면 안 된다** —
애초에 그런 파일이 존재하지 않는 별도 저장소이므로 구조적으로 안전하지만, 나중에 다른
파일을 옮겨올 때 이 점을 항상 확인한다.

## 규칙

- 디스클레이머(로또는 무작위, 통계 참고용, 당첨 보장 안 함)를 페이지에서 항상 보이게 유지한다.
- 이월수/소외수 표시는 "참고 기준일 뿐 당첨 확률을 높이지 않는다"는 걸 명확히 한다
  (`lottoRecommender.js` 파일 상단 주석 참고).
- 색상 팔레트는 `shorts/reference/lotto-shorts.md` 5절의 값(와인 `#2E0A13`, 금 `#D8B46A`,
  공식 5색 `#FBC400`/`#69C8F2`/`#FF7272`/`#AAAAAA`/`#B0D840`)과 통일한다 — 영상과 웹사이트가
  같은 브랜드로 보이게 하는 게 목적이다.
