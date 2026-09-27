# ui-ux-pro-max 스킬 전수조사 & 활용 리포트

> 작성일: 2026-09-27
> 대상 저장소: <https://github.com/bmshin94/friendmaker>
> 분석 대상 폴더: `.codex/skills/ui-ux-pro-max/` (커밋 `f98f401 add skill and design.md`)
> 원본(업스트림) 스킬: <https://github.com/nextlevelbuilder/ui-ux-pro-max-skill> (⭐ 약 130,000)

---

## 1. 요약 (TL;DR)

- 이 저장소 `friendmaker`는 **ESP32 + Electron 기반 "닌텐도 스위치 자동 드로잉" 데스크탑 앱**(GPL-3.0, macOS / Windows x64)이다.
- 그 안의 `.codex/skills/ui-ux-pro-max/`는 별개로 가져온 **AI 에이전트용 UI/UX 디자인 지식팩(Agent Skill)** 이다.
- 정체는 **"CSV 지식 창고 + 순수 파이썬 BM25 검색기 + AI 지침서(SKILL.md)"** 조합.
- **완전 오프라인 · 외부 API 키 불필요 · 파이썬 표준 라이브러리만 사용.**
- 한 문장 요약: *인터넷도 API 키도 없이, CSV 1,300여 행과 BM25 검색만으로 AI에게 디자이너의 판단력을 빌려주는 폴더.*

---

## 2. 폴더 구조 (실측)

총 **31개 파일 / 652KB**

```
.codex/skills/ui-ux-pro-max/
├── SKILL.md                      292줄   AI가 읽는 사용 지침 + UI 철칙
├── scripts/
│   ├── core.py                   253줄   BM25 검색 엔진 (k1=1.5, b=0.75)
│   ├── design_system.py         1067줄   디자인 시스템 생성 + 영속화
│   └── search.py                 114줄   CLI 진입점 (argparse)
└── data/
    ├── styles.csv                 67행   UI 스타일
    ├── colors.csv                 96행   제품군별 컬러 팔레트(Hex)
    ├── typography.csv             56행   폰트 페어링 + Google Fonts
    ├── ux-guidelines.csv          98행   UX 규칙(Do/Don't/코드/Severity)
    ├── ui-reasoning.csv          100행   ★ 추론 규칙 (핵심 두뇌)
    ├── products.csv               95행   제품 유형별 추천
    ├── landing.csv                30행   랜딩 섹션 순서 / CTA 전략
    ├── charts.csv                 25행   데이터 유형별 차트 추천
    ├── icons.csv                 100행   아이콘 라이브러리 매핑
    ├── react-performance.csv      44행   리액트 성능 함정
    ├── web-interface.csv          30행   접근성 / 시맨틱
    └── stacks/ (13종, 각 49~60행)
        react, nextjs, vue, nuxtjs, nuxt-ui, svelte, astro,
        html-tailwind, shadcn, swiftui, react-native, flutter, jetpack-compose
```

데이터 총합 약 **1,300행 이상** (사람이 직접 큐레이션한 표 형태 지식).

---

## 3. 작동 원리

1. 질문을 받으면 **5개 도메인(product / style / color / landing / typography)을 동시 검색**
2. 검색은 `core.py`에 직접 구현된 **BM25 랭킹**(`k1=1.5`, `b=0.75`, 3글자 이상 토큰만)
3. `ui-reasoning.csv`의 **추론 규칙** 적용 → 패턴 / 스타일 우선순위 / 컬러 무드 / 안티패턴 결정
   - `Decision_Rules` 컬럼에 조건 분기 JSON 포함: `{"if_data_heavy": "add-glassmorphism"}`
4. **ASCII 박스**(기본) 또는 **Markdown**(`-f markdown`)으로 완성된 디자인 시스템 출력
5. `--persist` 시 계층형 파일 생성
   - `design-system/<프로젝트>/MASTER.md` — 전역 기준(Source of Truth)
   - `design-system/<프로젝트>/pages/<페이지>.md` — 페이지별 override (존재하면 MASTER보다 우선)

### 실행 검증 결과 (Python 3.11.15, 정상 작동 확인)

```bash
python3 search.py "beauty spa wellness service elegant" --design-system -p "Serenity Spa"
```

```
PATTERN: Hero-Centric + Social Proof        CTA: Above fold
STYLE:   Soft UI Evolution  (Performance: Excellent / Accessibility: WCAG AA+)
COLORS:  Primary #EC4899 / Secondary #F9A8D4 / CTA #8B5CF6
         Background #FDF2F8 / Text #831843
TYPO:    Playfair Display / Inter
AVOID:   Bright neon colors + Harsh animations + Dark mode
```

---

## 4. 기술 조사 결과

| 항목 | 결과 |
|---|---|
| 외부 네트워크 통신 | **없음** (`requests`/`urllib`/`http` 호출 0건) |
| 의존성 | **파이썬 표준 라이브러리만** (csv, re, json, math, pathlib, datetime) |
| API 키 / 토큰 | **불필요** (관련 코드 0건) |
| 실행 환경 | Python 3.8+ (3.11에서 검증 완료) |
| 토큰 절약 설계 | 기본 결과 3건 + 300자 초과 자동 절단 + 출력 컬럼 화이트리스트 |
| 보안 | 오프라인 동작 → 폐쇄망/사내 환경에서도 안전 |

### SKILL.md에 정의된 "프로 UI 철칙" (발췌)

- 이모지를 아이콘으로 쓰지 말 것 → SVG(Heroicons / Lucide)
- hover에 `scale` 변형 금지(레이아웃 시프트) → color/opacity transition (150~300ms)
- 라이트 모드 글래스 카드는 `bg-white/80` 이상, 본문 텍스트 `#0F172A`, 보조 `#475569` 이상
- 클릭 가능한 요소에는 모두 `cursor-pointer`
- 플로팅 네브바는 `top-4 left-4 right-4`
- 반응형 검증 375 / 768 / 1024 / 1440px, `prefers-reduced-motion` 존중

---

## 5. 정체 구분: 스킬 vs 플러그인 vs MCP

| 구분 | 정체 | 이 프로젝트 |
|---|---|---|
| **스킬(Agent Skill)** | `SKILL.md` frontmatter + 스크립트/데이터 | ✅ **이것** |
| 플러그인(Plugin) | 스킬/명령어/훅/MCP를 묶은 배포 패키지 | ⭕ 포장 가능(현재 아님) |
| MCP 서버 | JSON-RPC 프로토콜 구현 별도 프로세스 | ❌ 아님(서버 코드 없음) |

승격 경로: `--json` 출력이 이미 있으므로 MCP SDK로 얇게 감싸면 MCP 툴로 전환 가능,
`.claude-plugin/plugin.json` 추가 시 플러그인으로 배포 가능.

---

## 6. 설치 및 사용법

### 전제

```bash
python3 --version   # 3.8 이상. pip install 불필요
```

### Claude Code에서 사용 (현재 위치가 `.codex/`라 자동 인식 안 됨)

```bash
# 프로젝트 전용
mkdir -p .claude/skills && cp -r .codex/skills/ui-ux-pro-max .claude/skills/

# 또는 전역
mkdir -p ~/.claude/skills && cp -r .codex/skills/ui-ux-pro-max ~/.claude/skills/
```

### Codex CLI에서 사용

현재 경로(`.codex/skills/`)가 규격에 맞으므로 그대로 사용 가능.

### CLI로 직접 사용 (`from core import ...` 때문에 스크립트 폴더 기준 실행)

```bash
cd .codex/skills/ui-ux-pro-max/scripts
python3 search.py "saas dashboard fintech" --design-system -p "MyApp"
```

### 명령어 치트시트

```bash
# 디자인 시스템 생성
search.py "<제품유형 업종 키워드>" --design-system -p "프로젝트명"
search.py "..." --design-system -f markdown
search.py "..." --design-system --persist -p "App" --page "checkout" [-o ./docs]

# 도메인 검색 (10종)
--domain style | color | typography | landing | chart | ux | product | icons | react | web

# 스택 가이드 (13종)
--stack html-tailwind | react | nextjs | vue | nuxtjs | nuxt-ui | svelte | astro
      | shadcn | swiftui | react-native | flutter | jetpack-compose

# 옵션
-n 5        # 결과 개수(기본 3)
--json      # JSON 출력(프로그램 연동용)
```

---

## 7. 발견된 이슈

1. **스킬 경로 불일치** — `.codex/skills/`는 Codex CLI 규격. Claude Code는 `.claude/skills/`를 인식하므로 복사/이동 필요.
2. **SKILL.md 문서 경로 오류** — 문서는 `skills/ui-ux-pro-max/scripts/search.py`로 안내하지만 실제 경로는 `.codex/skills/...`. 또한 `search.py`가 `core`/`design_system`을 평면 import 하므로 스크립트 디렉터리 기준 실행이 필요.
3. **문서 수치와 실제 데이터 미세 차이** — SKILL.md는 폰트 57 / UX 99로 기재, 실측은 56 / 98.
4. **자동 도메인 감지의 폴백** — `detect_domain()`이 매칭 실패 시 무조건 `style`로 폴백 → 정확도가 필요하면 `--domain` 명시 권장.
5. **LICENSE 파일 부재** — 스킬 폴더 내 라이선스 명시 없음. 재배포/상업화 전 업스트림 라이선스 확인 필요.

---

## 8. 왜 깃허브에서 유명한가

1. **보편적 고통 해결** — "AI가 만든 UI는 왜 다 촌스러운가"를 데이터로 교정
2. **설치 장벽 0** — pip / API 키 / 도커 / 벡터DB 전부 불필요, 복사만 하면 동작
3. **토큰 비용 설계** — 1,300행 중 상위 3행만 반환하는 "인프라 없는 RAG"
4. **모델·도구 비의존** — 순수 CLI라 Claude Code / Codex / Cursor / 사람 모두 사용 가능
5. **"하지 말 것" 중심 설계** — `Anti_Patterns`, Don't 컬럼, Pre-Delivery Checklist가 LLM 출력 품질을 끌어올림
6. **노동집약적 데이터 자산** — 스타일 67 × 컬러 96 × 폰트 56 × UX 98 × 스택 13종
7. **생태계 자가증식** — 파생 저장소 존재
   - <https://github.com/bbylw/ui-ux-pro-max-skill-cn> (중국어 공식 튜토리얼, ⭐1.4k)
   - <https://github.com/hylarucoder/benchmark-skill-ui-ux-pro-max> (벤치마크, ⭐313)
   - <https://github.com/tenfoldmarc/website-builder-setup> (번들 상품, ⭐342)

---

## 9. 로컬 에이전트 구축에 주는 시사점

### 배울 패턴

1. **파일 기반 RAG** — 지식이 수천 건 규모면 벡터DB 없이 BM25로 충분(0원, 즉시 응답)
2. **컨텍스트 예산 관리** — `MAX_RESULTS`, 300자 절단, 출력 컬럼 화이트리스트
3. **계층형 메모리** — `MASTER.md`(전역) → `pages/*.md`(국소) override = 에이전트 장기기억 설계 정석
4. **도구 인터페이스** — 사람/AI용 텍스트 + 프로그램용 `--json` 이중 출력 → MCP 승격 용이
5. **지침·데이터·로직 분리** — 데이터가 늘어도 프롬프트를 수정하지 않는 확장 구조

### friendmaker에 응용 가능한 자체 스킬 아이디어

- `friendmaker-protocol` — `apps/desktop/src/protocol/`의 커맨드/타이밍 규칙을 CSV화
- `esp32-troubleshoot` — `docs/troubleshooting*.md`를 증상/원인/해결 CSV로 구조화
- `friendmaker-design` — `DESIGN.md`의 크림톤 디자인 토큰을 CSV화해 화면 간 일관성 확보

### 한계

- BM25는 키워드 매칭이라 동의어/의역에 약함 → 하이브리드(BM25 + 임베딩) 보완 여지
- 데이터가 만 건 단위로 커지면 매 실행 전체 인덱싱이 병목 → SQLite FTS5 등으로 전환 필요

---

## 10. React / PHP 재구현 설계

### React + Node/TypeScript (권장)

friendmaker가 이미 TypeScript + Electron + 웹 UI라 스택이 일치한다.

```
data/*.csv                     (그대로 재사용)
packages/uiux-core/
  ├─ bm25.ts                   core.py의 BM25 이식
  ├─ loader.ts                 papaparse로 CSV 파싱
  └─ designSystem.ts           추론 규칙 적용
apps/web/ (React)              SearchBar / DesignSystemCard / ExportPanel
mcp-server/index.ts            Claude Code 연동
```

| 용도 | 라이브러리 |
|---|---|
| 검색 | `minisearch` / `flexsearch` / `lunr` (직접 구현도 가능) |
| CSV | `papaparse` |
| UI | React + Tailwind + `shadcn/ui` |
| 컬러 검증 | `culori` (WCAG 대비 계산) |
| MCP | `@modelcontextprotocol/sdk` |

차별점: 텍스트 출력 대신 **컬러칩 실시간 미리보기 / 폰트 실제 렌더링 / 와이어프레임 / 설정 파일 내보내기**.

### PHP (워드프레스 시장 공략 시)

```
src/Bm25.php            BM25 구현
src/CsvRepository.php   league/csv 기반 로더
src/DesignSystem.php    추론 규칙
bin/uiux                symfony/console CLI
public/index.php        Slim / Laravel API
```

검색은 직접 구현 대신 **SQLite FTS5** 또는 **MySQL `MATCH AGAINST`** 로 대체 가능.
PHP를 택하는 이유는 전 세계 웹 40%를 차지하는 **워드프레스 플러그인 시장** 접근성.

### 권장 로드맵

1. Node/TS 코어 이식(데이터 재사용) → CLI 완성
2. React 웹 UI로 "보이는 디자인 시스템 생성기"
3. MCP 서버로 감싸 Claude Code 연동
4. 검증 후 PHP/WordPress 플러그인 확장

---

## 11. 수익화 아이디어

### Tier 1 — 즉시 실행 가능 (1~4주)

| 아이디어 | 내용 | 가격대 |
|---|---|---|
| **한국 시장 특화 니치 팩** | 한글 폰트 페어링(Pretendard/SUIT/Gmarket Sans), KWCAG 체크리스트, 카카오·네이버 로그인 버튼 규격, 업종별 팩(커머스/병원/학원/배달/핀테크) | 팩당 $19~39, 번들 $79 |
| **React 웹 생성기 (Freemium SaaS)** | 무료: 일 3회 검색 / 유료: 무제한 + `tailwind.config.js`·CSS 변수·Figma 토큰·shadcn theme 내보내기 + 팀 공유 | $9~19/월 |
| **교육 콘텐츠** | 강의 "13만 스타 AI 스킬 해부하고 직접 만들기", 전자책, 유튜브 + 제휴 | 강의 5~15만원 / 전자책 2~3만원 |

가장 큰 빈틈: **한글 폰트 페어링과 KWCAG 데이터가 원본에 전혀 없다.**

### Tier 2 — 1~3개월

| 아이디어 | 내용 | 가격대 |
|---|---|---|
| **워드프레스 플러그인(PHP)** | 업종 선택 → 테마 컬러/폰트 자동 적용, 접근성 진단 리포트 | 무료 + Pro $49/년 |
| **Figma 플러그인** | 프롬프트 → Figma Variables/Styles 자동 생성 | $8~15/월 |
| **B2B 디자인 토큰 거버넌스** | 코드베이스 스캔으로 규칙 위반 탐지(하드코딩 색상, 대비 미달, 이모지 아이콘, cursor 누락), CI 연동 — `ux-guidelines.csv`의 `Severity` 컬럼이 린터 설계와 정합 | 시트당 $10~30/월 |
| **AI 에이전트 구축 컨설팅** | "사내 문서 → 전용 스킬/MCP 구축" 패키지 | 300만~2,000만원/건 |

### Tier 3 — 장기

- **스킬 마켓플레이스 + 데이터 구독** (월 $5~9, 트렌드 스타일 갱신으로 리텐션 확보)
- **버티컬 AI 사이트 빌더** (이 엔진을 품질 보증 계층으로 사용, $29~99/월) — `website-builder-setup`의 존재가 시장 검증 신호

### 우선순위

| 순위 | 아이디어 | 투입 | 기대 초기 수익 |
|---|---|---|---|
| 1 | 한국형 니치 팩 | 1~2주 | 월 30~200만원 |
| 2 | 교육 콘텐츠 | 2~3주 | 월 50~300만원 |
| 3 | React 생성기 SaaS | 1~2개월 | 월 100만원~ |

### 반드시 지킬 것

1. **라이선스 확인** — 스킬 폴더에 LICENSE 없음. 업스트림 라이선스 확인 후, 상업 판매는 자체 제작 데이터로 진행.
2. **GPL 격리** — `friendmaker`는 GPL-3.0. 상업 제품은 **별도 저장소**로 분리해 전염 방지.
3. **해자는 코드가 아니라 데이터** — BM25는 쉽게 복제 가능. 한글 폰트 + KWCAG + 국내 업종 데이터가 차별 자산.
4. **차별화 축** — 원본은 텍스트 출력, 우리는 "보이는 결과 + 바로 쓰는 설정 파일".

---

## 12. 참고 링크

- 이 저장소: <https://github.com/bmshin94/friendmaker>
- 업스트림 스킬: <https://github.com/nextlevelbuilder/ui-ux-pro-max-skill>
- 중국어 튜토리얼: <https://github.com/bbylw/ui-ux-pro-max-skill-cn>
- 벤치마크: <https://github.com/hylarucoder/benchmark-skill-ui-ux-pro-max>
- 번들 상품 사례: <https://github.com/tenfoldmarc/website-builder-setup>
