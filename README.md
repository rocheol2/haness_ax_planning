# AX 행사·교육 기획 보고서 하네스

AX(AI Transformation) 포럼, 워크샵, 교육, 세미나 등 **행사·교육 프로그램 기획 보고서 작성 업무를 AI Agent 팀이 분업 처리하도록 설계한 Claude Code 하네스**이다.

핵심은 사용자의 간단한 요청을 받아 `입력 → 처리 → 검증 → 출력` 흐름으로 업무를 자동 분해하고, 리서치·기획·도식·집필·검증 Agent가 협업하여 상급자 보고용 기획 보고서를 생성하는 것이다.

GitHub 저장소: <https://github.com/rocheol2/haness_ax_planning>

---

## 1. 하네스 주제

**주제**: AX 행사·교육 기획 보고서 자동 작성 하네스

**대상 사용자**: 기업 또는 공공기관에서 AX 추진, AI 교육, 디지털 전환 프로그램을 기획하는 실무자

**해결하는 문제**:

- AX 포럼·워크샵·교육 기획 시 상급자 결재를 통과할 수 있는 보고서 초안이 필요함
- 리서치, 프로그램 구성, 예산 산출, 일정 계획, 예상 질의 대응을 각각 따로 작성해야 해 시간이 오래 걸림
- 공공기관·대기업 보고서 문체인 개조식, 「~함」체, 표 중심 구성이 익숙하지 않은 실무자가 많음

**최종 산출물**:

- AX 행사·교육 기획 보고서
- 작성 의미 해설
- 상급자 예상 질의 Q&A
- 검증·수정 이력
- 한글(hwpx) 양식 제공 시 최종 hwpx 파일

## 2. 구성 목적

| 목적 | 구현 방식 |
|------|-----------|
| AI Agent 업무 처리 구조 구현 | 오케스트레이터가 업무를 나누고 5개 Agent가 단계별 산출물을 작성 |
| 입력→처리→검증→출력 흐름 명확화 | `_workspace/00_input.md`부터 `06_executive_qa.md`, 선택 `07_final.hwpx`까지 산출물 흐름 고정 |
| 기획 품질 확보 | 리서치 → 전략 → 도식 → 집필 → 검증의 순차 파이프라인 적용 |
| 결재 대응력 강화 | executive-summarizer가 상급자 예상 질의와 실행가능성을 별도 검증 |
| 문체 일관성 확보 | `public-report-style` 스킬로 개조식·공공기관 어투 표준화 |
| 반복 실행 가능성 확보 | `run_manifest.md`로 실행 모드, 파일 상태, 수정요청 ID, 최종 출력 목록 추적 |
| 실무 문서 포맷 대응 | `hwpx-autofill-conversion` 스킬로 한글 양식 자동 채움 지원 |

## 3. AI Agent 업무 처리 구조

```text
입력
  사용자 요청, 기존 자료, hwpx 양식
  ↓
처리
  data-collector → analyst → visualizer → report-writer
  ↓
검증
  executive-summarizer가 Q&A, 실행가능성, 정합성, 문체 검증
  ↓
출력
  Markdown 보고서 묶음 + 선택 hwpx 최종본
```

| 단계 | 담당 | 입력 | 처리 내용 | 출력 |
|------|------|------|-----------|------|
| 입력 | 오케스트레이터 | 사용자 요청, 첨부 자료 | 요구사항 정리, 실행 모드 결정 | `00_input.md`, `run_manifest.md` |
| 처리 1 | data-collector | 입력 요약 | AX 트렌드, 벤치마크, 강사 후보 조사 | `01_research.md` |
| 처리 2 | analyst | 리서치 | 대상 분석, 프로그램 설계, 예산·일정·리스크 검토 | `02_planning_strategy.md` |
| 처리 3 | visualizer | 기획 전략 | 시간표, 예산표, 추진 일정표, 추진체계도 작성 | `03_visual_elements.md` |
| 처리 4 | report-writer | 리서치·전략·도식 | 개조식 보고서와 작성 해설 집필 | `04_planning_report.md`, `05_report_commentary.md` |
| 검증 | executive-summarizer | 전체 산출물 | 예상 질의 Q&A, 정합성, 실행가능성, 문체 점검 | `06_executive_qa.md` |
| 출력 | 오케스트레이터 | 검증 완료 산출물 | 최종 파일 확인, 선택 시 hwpx 변환 | Markdown 산출물, `07_final.hwpx` |

## 4. 전체 구조

```text
.claude/
├── CLAUDE.md
│   └── 하네스 공통 페르소나, 절대 원칙, 입력→처리→검증→출력 규칙
├── agents/
│   ├── data-collector.md
│   │   └── AX 트렌드·벤치마크 리서처
│   ├── analyst.md
│   │   └── 기획 전략가
│   ├── visualizer.md
│   │   └── 표·도식 설계자
│   ├── report-writer.md
│   │   └── 개조식 보고서 집필자
│   └── executive-summarizer.md
│       └── 상급자 관점 검증자
└── skills/
    ├── ax-planning-report/
    │   └── skill.md
    │       └── 전체 워크플로우를 조율하는 오케스트레이터 스킬
    ├── public-report-style/
    │   └── skill.md
    │       └── 개조식·공공기관 어투 작성 규칙
    ├── executive-qa-patterns/
    │   └── skill.md
    │       └── 상급자 예상 질의와 실행가능성 점검 패턴
    └── hwpx-autofill-conversion/
        └── skill.md
            └── hwpx ZIP·XML 구조 분석 및 양식 자동 채움 절차
```

## 5. 워크플로우와 산출물

```text
Phase 1. 입력 정리
  사용자 요청 → _workspace/00_input.md
  실행 추적 → _workspace/run_manifest.md

Phase 2. Agent 처리
  data-collector → 01_research.md
  analyst → 02_planning_strategy.md
  visualizer → 03_visual_elements.md
  report-writer → 04_planning_report.md, 05_report_commentary.md

Phase 3. 검증
  executive-summarizer → 06_executive_qa.md
  필수 수정 발견 시 FIX-001 형식으로 수정요청 추적

Phase 4. 출력
  Markdown 산출물 목록 보고
  hwpx 양식 제공 시 07_final.hwpx 생성
```

| 파일 | 역할 |
|------|------|
| `_workspace/00_input.md` | 사용자 요구사항 정리 |
| `_workspace/run_manifest.md` | 실행 모드, 산출물 상태, 수정요청 ID, 최종 출력 목록 추적 |
| `_workspace/01_research.md` | AX 트렌드, 벤치마크, 강사 후보, 운영 참고자료 |
| `_workspace/02_planning_strategy.md` | 프로그램 설계, 예산·일정·인력 실행가능성, 리스크 |
| `_workspace/03_visual_elements.md` | 추진체계도, 프로그램 시간표, 예산표, 추진 일정표 |
| `_workspace/04_planning_report.md` | 최종 기획 보고서 본문 |
| `_workspace/05_report_commentary.md` | 작성 의미 해설 |
| `_workspace/06_executive_qa.md` | 상급자 예상 질의 Q&A와 검증 보고서 |
| `_workspace/07_final.hwpx` | 한글 양식 적용 최종 파일 |

## 6. 구현 방식

이 하네스는 바이브 코딩 방식으로 구조를 설계하고 Claude Code 하네스 파일로 구현했다.

1. 범용 보고서 생성 하네스 구조를 참고하여 AX 행사·교육 기획 도메인으로 재설계
2. 업무를 리서치, 전략, 도식, 집필, 검증으로 분해
3. 각 업무를 전담하는 Agent 지시문 작성
4. 공공기관 보고서 문체와 상급자 예상 질의 패턴을 별도 Skill로 분리
5. 검증 단계에서 `FIX-001` 형식의 수정요청 ID를 남기도록 개선
6. 최종 결과물이 GitHub 저장소에서 확인 가능하도록 `README.md`와 `.claude/` 하네스 구조 정리

## 7. 사용 방법

### 설치

```bash
git clone https://github.com/rocheol2/haness_ax_planning.git
cd haness_ax_planning
claude
```

VSCode를 사용하는 경우 저장소 폴더를 열고 Claude Code를 실행하면 `.claude/`의 Agent와 Skill이 자동 인식된다.

### 실행 방법

Claude Code에서 자연어로 요청한다.

```text
사내 AX 담당자 대상 생성형 AI 활용 워크샵 기획 보고서 만들어줘.
11월 개최, 50명 규모, 예산 2천만원이야.
```

hwpx 양식이 있는 경우:

```text
AX 포럼 기획 보고서를 만들고 template.hwpx 양식에 맞춰 한글 파일로 변환해줘.
```

부분 실행도 가능하다.

| 요청 예시 | 실행 모드 |
|----------|-----------|
| "이 초안으로 기획서 완성해줘" | 집필 모드 |
| "이 기획서 개조식으로 바꿔줘" | 변환 모드 |
| "이 기획서 검토하고 예상 질문 뽑아줘" | 검증 모드 |
| "이 보고서를 hwpx로 만들어줘" | hwpx 변환 모드 |

## 8. 실행 예시와 결과 예시

### 예시 입력

```text
사내 AX 담당자 대상 생성형 AI 활용 워크샵 기획 보고서를 만들어줘.
11월 개최, 50명 규모, 예산 2천만원이야.
```

### 예상 처리 흐름

```text
입력 정리
→ AX 교육 트렌드와 유사 워크샵 벤치마크 조사
→ 50명 규모 워크샵 프로그램, 예산, 일정, 리스크 설계
→ 시간표·예산표·추진 일정표 작성
→ 개조식 보고서 작성
→ 상급자 예상 질의와 실행가능성 검증
→ 최종 산출물 목록 보고
```

### 예상 결과 파일

```text
_workspace/
├── 00_input.md
├── run_manifest.md
├── 01_research.md
├── 02_planning_strategy.md
├── 03_visual_elements.md
├── 04_planning_report.md
├── 05_report_commentary.md
└── 06_executive_qa.md
```

hwpx 양식 제공 시:

```text
_workspace/07_final.hwpx
```

### 결과 예시 요약

```text
# 생성형 AI 활용 워크샵 개최 계획(안)

## Ⅰ. 추진 배경
□ 전사 AX 실행력 제고를 위한 실습형 교육 필요
 ○ 생성형 AI 활용 수준의 부서 간 편차 해소 필요
  - AX 담당자 중심의 공통 활용 역량 확보 요구

## Ⅵ. 소요 예산
| 항목 | 산출 근거 | 금액(천원) |
|------|-----------|------------|
| 강사비 | 2명 × 3,000천원 | 6,000 |
| 장소·운영 | 50명 × 운영 단가 | 8,000 |
| 교재·홍보 | 교재 제작 및 안내물 | 3,000 |
| 예비비 | 총액의 약 15% | 3,000 |
| 합계 | | 20,000 |
```

## 9. GitHub 업로드 기준

저장소에는 다음 항목이 포함되어야 한다.

- `README.md`: 하네스 주제, 구성 목적, 전체 구조, 사용 방법, 실행 예시, 결과 예시
- `.claude/CLAUDE.md`: 하네스 공통 원칙
- `.claude/agents/`: 업무별 AI Agent 지시문
- `.claude/skills/`: 오케스트레이터와 보조 Skill 지시문

실행 중 생성되는 `_workspace/` 결과물과 배포용 임시 압축 파일은 `.gitignore`로 제외한다.
