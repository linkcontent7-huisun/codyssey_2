# 환경 구축 기록 — Higgsfield · scroll-world (2026-08-13)

> 랜딩페이지 장면을 만들기 위해 깐 도구들의 설치 기록.
> PC 를 바꾸거나 설정이 깨졌을 때 이 문서만 보고 그대로 복구할 수 있도록 남긴다.
> 순서가 중요하다 — 아래 순서를 바꾸면 중간에 막힌다.

---

## 1. 무엇을 왜 깔았나

| 도구 | 역할 | 없으면 |
| --- | --- | --- |
| **Higgsfield MCP** | 대화창에서 바로 이미지·영상 생성 | 웹사이트에 매번 들어가야 함 |
| **Higgsfield CLI** | scroll-world 스킬이 요구하는 백엔드 | 스킬이 Step 0 에서 멈춤 |
| **scroll-world 스킬** | 스크롤 랜딩페이지 제작 절차·프롬프트·스크럽 엔진 | 파이프라인을 직접 짜야 함 |
| ffmpeg · ffprobe | 프레임 추출·인코딩 | 클립 연결 불가 |
| Python 3 + Pillow | 배경 제거(선택) | 떠 있는 장면 연출 불가 |

**MCP 와 CLI 는 별개다.** MCP 를 연결해도 scroll-world 는 동작하지 않는다 —
스킬이 쓰는 것은 CLI 쪽이다. 이걸 몰라 한참 헤맸다.

---

## 2. 설치 순서

### ① Higgsfield MCP 등록

```bash
claude mcp add --transport http --scope user higgsfield https://mcp.higgsfield.ai/mcp
```

`--scope user` 로 넣으면 `~/.claude.json` 에 저장되어 모든 프로젝트에서 쓰인다.
프로젝트 저장소에는 들어가지 않는다.

등록만으로는 `! Needs authentication` 상태다. **승인은 대화형 터미널에서만 된다.**

```
claude          # 프로젝트 폴더에서 실행
/mcp            # 슬래시 포함. "mcp" 만 치면 작업 지시로 알아듣는다
```

목록에서 `higgsfield` → **Authenticate** → 브라우저에서 승인.
끝나면 반드시 `/exit` 로 정상 종료한다 — 창이 그냥 닫히면 토큰이 저장되지 않는다.

확인:

```bash
claude mcp list
# higgsfield: https://mcp.higgsfield.ai/mcp (HTTP) - √ Connected
```

### ② Higgsfield CLI

```bash
npm install -g @higgsfield/cli
higgsfield auth login          # 브라우저 OAuth
higgsfield workspace list      # 확인
```

`npm warn allow-scripts` 경고가 뜨지만 설치는 정상이다.

### ③ scroll-world 스킬

```bash
claude plugin marketplace add oso95/scroll-world
claude plugin install scroll-world@scroll-world
```

확인:

```bash
claude plugin list
# scroll-world@scroll-world  Version: 0.8.0  Scope: user  Status: √ enabled
```

설치 위치는 `~/.claude/plugins/cache/scroll-world/scroll-world/0.8.0/` 이고
`skills/scroll-world/` 아래에 `SKILL.md` 와 `references/`
(`prompts.md` · `pipeline.md` · `scrub-engine.js` · `index-template.html` · `knockout.py`)가 들어 있다.

---

## 3. 겪은 문제와 해결

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| `Failed to enable MCP server 'higgsfield': higgsfield was disabled while enabling` | `/mcp` 화면에서 실수로 Disable 을 눌러 `~/.claude.json` 의 `disabledMcpServers` 에 남아 있었다 | 해당 배열에서 이름을 지운다. **파일을 UTF-8 로 읽고 써야 한다** — 인코딩이 깨지면 Claude Code 가 설정을 통째로 초기화한다 |
| 설정이 저장되지 않음 | VS Code 업데이트로 터미널 세션이 강제 종료됨 | 작업 후 `/exit` 로 정상 종료 |
| `MCP` 라고 쳤더니 저장소를 뒤지기 시작 | 슬래시를 빼면 작업 지시로 해석된다 | `/mcp` |
| 홈 폴더에서 `claude` 실행 시 신뢰 여부를 물음 | 계정 폴더 전체 접근 권한을 요구하는 것 | Esc 후 프로젝트 폴더에서 실행 |

---

## 4. 크레딧 — 이게 실제 제약이다

무료 플랜은 **10크레딧**으로 시작한다. 이미지 1장 = **2크레딧**.

| 항목 | 크레딧 |
| --- | --- |
| 장면 스틸 5장 | 약 10 |
| 영상 9개 (장면 5 + 연결 4) | 약 225 |
| **한 번 완성에 필요** | **약 235** |

**무료 10크레딧으로는 이미지 5장이 한계이고 영상은 한 편도 못 만든다.**
일회성 크레딧 팩은 이 계정에 나오지 않아 구독밖에 선택지가 없다
(Plus 월 $49 = 1,000크레딧).

> 플랜의 "7-day unlimited" 혜택은 **MCP/CLI 에서 동작하지 않는다.**
> 공식 안내에 "Unlimited models and Free Generations ... are not accessible on
> MCP/CLI, Canvas or Supercomputer" 로 명시되어 있다. 플랜을 고를 때
> 그 혜택은 계산에서 빼야 한다 — 오직 크레딧 수량만 의미가 있다.

### 실제 사용 내역 (10크레딧 전량)

| 회차 | 결과 | 크레딧 |
| --- | --- | --- |
| 1 | 아이소메트릭 디오라마 · 새벽 안개 → **폐기** | 2 |
| 2 | 디오라마 · 초록 숲 → **폐기**(인물이 어린아이처럼) | 2 |
| 3 | 실사 · 아침 숲길 → 채택 | 2 |
| 4 | 실사 · 계곡 돌다리 (3번을 참조 이미지로) | 2 |
| 5 | 실사 · 언덕 위 성당 (3번을 참조 이미지로) | 2 |

**4크레딧을 버린 이유**: 과제 2 `storyboard.md` 에 이미 확정해 둔
`cinematic still-cut realism` 과 순례자 규격을 읽지 않고 새로 화풍을 잡았다.
**기존 규격 문서를 먼저 확인하는 것이 새로 만드는 것보다 빠르다.**

---

## 5. 인물 일관성 — 이번에 확인된 방법

과제 2 때는 시드 지정이 안 되어 프롬프트 고정으로만 버텼다.
이번에는 **참조 이미지 방식**이 훨씬 안정적이라는 것을 확인했다.

1. 1번 장면을 마음에 들 때까지 반복해 확정한다
2. 2번부터는 1번을 `medias: [{role: "image", value: <job_id>}]` 로 넣고
   프롬프트를 `Keep the exact same woman from the reference image` 로 시작한다
3. 바뀌면 안 되는 요소를 명시적으로 나열한다 —
   `same short bob-length hair, same glasses, same checked shirt, same brown trekking pants, same backpack`

이렇게 뽑은 3장에서 단발·안경·체크셔츠·배낭이 모두 유지됐다.
참조 없이 뽑으면 다른 사람이 된다.

---

## 6. 관련 위치

| 무엇 | 어디 |
| --- | --- |
| 생성된 장면 3장 | 이 폴더 `landing-scenes/` |
| 같은 이미지 사본 | `nohhuisun/visitholykorea` → `data/brand/landing-scenes/` |
| 인물·톤 규격 원본 | 이 저장소 `storyboard.md` |
| 오늘 작업 전체 정리 | `nohhuisun/visitholykorea` → `docs/00-overview/2026-08-13-작업정리.md` |
