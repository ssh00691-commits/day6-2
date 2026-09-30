# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

영어 단어장 웹앱: 외울 영어 단어를 저장하고 휴대폰으로 복습한다. 앱 전체가 정적 파일 `index.html` 하나(HTML + CSS + JS 인라인)이며 빌드·린트·테스트 도구는 없다. 요구사항은 `PRD.md`, 단계별 작업 목록은 `STEPS.md`(1 추가·체크·삭제 → 2 탭·개수 → 3 디자인, 모두 완료)에 있다. 문서는 한국어로 쓴다.

## Running

로컬 서버(localhost)로 확인한다. `file://`로 열지 않는다.

```bash
python -m http.server 8123
```

http://localhost:8123/index.html 로 접속. 포트 8000은 다른 프로젝트 서버가 쓰는 경우가 있어 8123을 쓴다. 접속 전 `<title>`이 "영어 단어장"인지 확인할 것(다른 서버가 응답하고 있을 수 있음).

## Architecture

- **백엔드 없음.** 브라우저가 Supabase JS(CDN, `@supabase/supabase-js@2`)로 Supabase 프로젝트 **`day06-2`**(ref `smncozbuzwdkaixmghwl`)의 `public.words` 테이블에 직접 접근한다. 코드에는 publishable 키만 들어 있고, service role 키는 절대 넣지 않는다.
- **`words` 테이블:** `id`(identity PK), `word`, `meaning`(둘 다 `btrim` 길이 > 0 체크), `memorized`(boolean, 기본 false), `created_at`. RLS가 켜져 있고 anon 역할에 select/insert/update/delete 정책이 열려 있다. 스키마 변경은 마이그레이션으로 하며 `index.html`과 함께 맞춘다.
- **상태 흐름:** 모듈 변수 `allWords`(서버에서 받은 전체 목록)와 `currentTab`(`all` | `todo`)이 상태다. 모든 쓰기(추가·외움 토글·삭제) 후 `loadWords()`로 서버에서 다시 받아 `render()`가 `allWords`를 탭으로 거르고, "N개 중 M개 외움"은 탭과 무관하게 `allWords` 전체로 계산한다. 탭 전환만 서버 호출 없이 `render()`를 다시 부른다.

## 규칙

- 사용자 입력(단어·뜻)은 항상 `textContent`로만 DOM에 넣는다. `innerHTML`은 쓰지 않는다(입력한 HTML 태그가 글자 그대로 보여야 한다).
- 추가 시 앞뒤 공백을 trim하고, 단어나 뜻이 비면 저장하지 않는다.
- 디자인은 휴대폰 세로 화면 우선(폭 375px에서 가로 스크롤 없음), 색은 `:root` CSS 변수로 관리하고 다크 모드를 지원한다.
