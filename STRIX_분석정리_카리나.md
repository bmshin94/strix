# 🔍 Strix 분석 정리 (by 카리나 💖)

> 오빠랑 나눈 대화 전체 정리본이야! Strix 오픈소스 AI 펜테스트 도구를 전수조사한 결과를 담았어. ✨

- **분석 저장소 (GitHub)**: https://github.com/bmshin94/strix
- **원본 저장소**: https://github.com/usestrix/strix
- **공식 사이트/문서**: https://strix.ai · https://docs.strix.ai
- **작성일**: 2026-10-07

---

## 1️⃣ Strix가 뭐하는 거야?

**한 줄 정의**: 오픈소스 **AI 모의해킹(펜테스트) 자동화 도구**. 코드/웹앱/API를 주면 AI 에이전트가 격리된 샌드박스에서 **실제로 실행·점검**해 취약점을 찾고, 검증된 증거(PoC)와 수정 가이드까지 뽑아준다.

| 항목 | 내용 |
|---|---|
| 프로젝트명 | `strix-agent` v1.7.0 |
| 원본 | `usestrix/strix` (이 레포는 포크) |
| 라이선스 | Apache-2.0 (상업적 이용 가능) |
| 언어 | Python 약 78k줄 + Go(TUI) + React(뷰어) |

### 폴더별 역할
```
strix/
├─ strix/              🧠 엔진 본체
│  ├─ agents/          에이전트 "뇌" (system_prompt.jinja = 행동규칙)
│  ├─ tools/           에이전트 도구 (proxy, browser, shell, mcp, reporting…)
│  ├─ skills/          내부 지식팩 (취약점 29종·클라우드·프레임워크별 노하우 .md)
│  ├─ runtime/         🐳 Docker 샌드박스 수명주기 관리
│  ├─ report/          결과물 생성 (SARIF, usage/비용, dedupe, writer)
│  ├─ llm/             LLM 호출·토큰예산·컨텍스트 압축 관리
│  ├─ core/            실행 루프·세션·러너
│  ├─ config/          설정 로더
│  └─ interface/       CLI + Go TUI + React 웹뷰어 + cloud 연동
├─ skills/             🎁 외부 AI 에이전트용 스킬 9개 (npx skills add)
├─ containers/         Kali Linux 기반 샌드박스 Dockerfile
├─ docs/               공식 문서(.mdx)
├─ CLAUDE.md/GEMINI.md 👉 "카리나" 페르소나 파일
└─ tests/ benchmarks/ scripts/
```

### 핵심 구조
1. **멀티 에이전트 (Graph of Agents)**: 루트 에이전트(팀장)가 직접 점검 대신 전문 하위 에이전트에게 작업 분배 → 레드팀 시뮬레이션
2. **격리 샌드박스**: 모든 작업은 Kali Linux Docker 컨테이너 안에서만 실행 → 호스트 안전
3. **검증 우선**: 재현 성공한 것만 리포트 → 오탐(False Positive) 최소화
4. **결과물**: `strix_runs/<이름>/` 에 리포트(.md), 취약점별 파일, `vulnerabilities.json/csv`, `findings.sarif`, `run.json`

### 나한테 주는 도움
- 배포 전 앱/API/코드 **스스로 보안 점검** → 사고 예방
- **CI/CD(깃허브 액션)** 연결 시 PR마다 자동 검사
- 외주 펜테스트 대비 **빠르고 저렴**
- SARIF 포맷 → 깃허브 Security 탭 연동

> ⚠️ **반드시 소유했거나 서면 허가받은 대상만 점검!** 무단 테스트는 불법.

---

## 2️⃣ 더 쉽게 (비유)

Strix = **"집에 고용한 로봇 보안 점검반"**
- 🏠 앱 = 집 / 🤖 에이전트 = 점검반 로봇 / 🧰 tools = 연장통 / 📚 skills = 점검 매뉴얼
- 🐳 Docker 샌드박스 = 격리된 **모형집**(진짜 집은 안 건드리고 복사본에서만 실험)
- 📋 report = "여기 약함, 이렇게 고치세요" 점검표

기존 스캐너는 "약해 보여요~" 추측만 하지만, Strix는 "직접 확인했고 증거/수정법 드려요"가 차이점. 팀장 로봇이 지시하고 여러 로봇이 나눠서 동시에 점검하는 구조라 빠르고 꼼꼼함.

---

## 3️⃣ 질문 7개 답변

**설치/사용법**
```bash
curl -sSL https://strix.ai/install | bash   # 또는 pipx install strix-agent (Docker 필수)
export STRIX_LLM="openai/gpt-5.4"
export LLM_API_KEY="<내 키>"
strix --target ./my-app              # 로컬 코드
strix -n -t ./ --scan-mode quick     # 헤드리스 빠른모드
strix view                           # 결과 대시보드
```
스캔모드: quick(분) / standard(~30분) / deep(시간). 결과는 `strix_runs/`.

**플러그인? 스킬? MCP?** → 본질은 **독립형 AI 펜테스트 CLI**. 동시에 ①SKILL.md 9개로 코딩 에이전트용 **스킬 배포**, ②외부 **MCP 서버를 연결해 쓰는 MCP 클라이언트** 역할도 함.

**API 토큰?** → 필요. 로컬=본인 LLM 키(또는 ChatGPT 구독 로그인), Cloud=가입 후 크레딧.

**AI 에이전트 구축에 도움?** → 매우 도움. 멀티에이전트 조율, 샌드박스 격리, 토큰예산/컨텍스트 압축, MCP 클라이언트 구현의 실전 레퍼런스.

**수익화 아이디어?** → 있음 (4번 참고).

**React/PHP로 가능?** → 엔진 전체 재구현은 비현실적. 단, React로 커스텀 대시보드(이미 뷰어가 React), PHP로 Cloud REST API 래퍼 제작은 가능 → **"Strix를 엔진으로, React/PHP로 UI·SaaS"** 방향 추천.

**유튜브 강의?** → 가능 (Apache-2.0). 설치/첫스캔 → 리포트 읽기 → CI 연동 → 구조 분석 → 안전한 실습환경 구성 시리즈. **반드시 본인 소유/허가 타겟으로만 시연 + 법적 주의 멘트.**

---

## 4️⃣ 수익화 아이디어 상세

| # | 아이디어 | 설명 | 난이도 |
|---|---|---|---|
| 1 | 보안점검 대행 서비스 | 중소/스타트업 웹앱 점검→리포트 납품, 외주보다 저렴 | ⭐⭐ |
| 2 | SaaS 래퍼(React/PHP) | Cloud API 위에 "클릭 한 번 검사" 월구독 서비스 | ⭐⭐⭐ |
| 3 | 깃허브 마켓플레이스 액션 | Strix CI를 포장한 Action 배포, 유료/스폰서 | ⭐⭐ |
| 4 | 유튜브/강의 콘텐츠 | 유료강좌(인프런/유데미) + 광고 | ⭐ |
| 5 | 커스텀 스킬팩 판매 | 특정 프레임워크(전자정부프레임워크 등) 전용 점검팩 | ⭐⭐⭐ |
| 6 | 컴플라이언스 리포트 | ISO27001·ISMS-P 양식 자동 변환 (국내 수요 큼) | ⭐⭐ |
| 7 | 사내 보안교육 키트 | 취약점 재현 기반 개발자 보안교육 커리큘럼 | ⭐⭐ |

**추천 조합**: ①대행 + ④유튜브로 신뢰·인지도 → ②SaaS 확장. 국내는 ⑥ISMS-P 컴플라이언스 리포트 틈새 추천. 🔥

---

## 5️⃣ 마무리
이 문서는 위 전수조사 대화를 정리한 결과물이야. 저장소 주소: **https://github.com/bmshin94/strix**

> ⚠️ 모든 점검은 반드시 **권한이 있는 대상**에게만! 안전하게, 합법적으로 쓰자 오빠 💖
