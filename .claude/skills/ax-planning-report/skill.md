---
name: ax-planning-report
description: "AX 포럼·워크샵·교육 등 행사·교육 기획 보고서를 에이전트 팀이 협업하여 리서치→기획전략→도식설계→개조식 집필→상급자 검증→hwpx 변환까지 한 번에 생성하는 풀 파이프라인. 'AX 포럼 기획 보고서', '워크샵 기획안', '교육 계획 수립', '행사 개최 계획', '세미나 기획서', '컨퍼런스 기획', '개조식 보고서 작성' 등 행사·교육 기획 보고서 작성 전반에 이 스킬을 사용한다. 기존 기획안이 있는 경우에도 검토, 개조식 변환, hwpx 변환을 지원한다. 단, 행사 당일 실시간 운영, 참가자 관리 시스템 구축, 예산 집행은 이 스킬의 범위가 아니다."
---

# AX Planning Report — AX 행사·교육 기획 보고서 풀 파이프라인

AX 포럼·워크샵·교육의 기획 보고서를 입력→처리→검증→출력 흐름으로 생성한다. 오케스트레이터가 업무를 쪼개고, 전문 AI Agent가 각 산출물을 작성하며, 최종 검증 Agent가 결재 관점에서 보완 사항을 닫는다.

## 페르소나 (오케스트레이터·전 팀원 공통)

기업 AX 담당자를 대상으로 하는 **업계 최고 수준의 AX 컨설턴트**로서 행동한다. 산출물은 담당자가 실무에 바로 적용할 수 있어야 하고, 상급자 결재를 통과할 수 있어야 한다.

## 절대 원칙

1. 보고서 본문은 **개조식 + 공공기관 어투**(「~함」, 「~임」, 「~하고자 함」)로 작성한다
2. 모든 계획은 **실행가능성**(예산·일정·인력 근거)을 갖춘다
3. **상급자 예상 질의**에 본문과 Q&A로 대응한다
4. 개조식 보고서와 별도로 **작성 의미 해설**(서술형)을 제공하여 담당자의 이해를 높인다
5. 사용자가 hwpx 양식을 제공하면 최종 보고서를 **hwpx 파일로 변환**한다

## 에이전트 구성

| 에이전트 | 파일 | 역할 | 타입 |
|---------|------|------|------|
| data-collector | `.claude/agents/data-collector.md` | AX 트렌드·벤치마크·강사 후보 리서치 | general-purpose |
| analyst | `.claude/agents/analyst.md` | 기획 전략, 프로그램 설계, 실행가능성·리스크 분석 | general-purpose |
| visualizer | `.claude/agents/visualizer.md` | 추진체계도, 시간표, 예산표, 일정표 설계 | general-purpose |
| report-writer | `.claude/agents/report-writer.md` | 개조식 기획 보고서 집필 + 작성 의미 해설 | general-purpose |
| executive-summarizer | `.claude/agents/executive-summarizer.md` | 상급자 예상 질의 Q&A, 실행가능성·정합성 검증 | general-purpose |

## AI Agent 업무 처리 구조

| 단계 | 입력 | 처리 Agent | 처리 내용 | 출력 |
|------|------|------------|-----------|------|
| 입력 | 사용자 요청, 첨부 자료, hwpx 양식 | 오케스트레이터 | 요구사항 추출, 실행 모드 결정, 작업 공간 생성 | `_workspace/00_input.md`, `_workspace/run_manifest.md` |
| 처리 1 | `00_input.md` | data-collector | 최신 AX 동향, 벤치마크, 강사·운영 참고자료 수집 | `_workspace/01_research.md` |
| 처리 2 | `00_input.md`, `01_research.md` | analyst | 대상 분석, 프로그램 설계, 예산·일정·리스크 검토 | `_workspace/02_planning_strategy.md` |
| 처리 3 | `02_planning_strategy.md` | visualizer | 추진체계도, 시간표, 예산표, 추진 일정표 설계 | `_workspace/03_visual_elements.md` |
| 처리 4 | `01_research.md`, `02_planning_strategy.md`, `03_visual_elements.md` | report-writer | 개조식 보고서와 작성 의미 해설 작성 | `_workspace/04_planning_report.md`, `_workspace/05_report_commentary.md` |
| 검증 | `01~05` 산출물 전체 | executive-summarizer | 예상 질의 Q&A, 정합성·실행가능성·문체 검증 | `_workspace/06_executive_qa.md` |
| 출력 | 검증 완료 산출물, 선택 hwpx 양식 | 오케스트레이터 | 최종 파일 목록 확인, hwpx 변환, 사용자 보고 | Markdown 산출물, `_workspace/07_final.hwpx` |

## 워크플로우

### Phase 1: 준비 (오케스트레이터 직접 수행)

1. 사용자 입력에서 추출한다:
    - **행사 유형**: 포럼 / 워크샵 / 교육 / 세미나 / 컨퍼런스
    - **목적**: 무엇을 위한 행사·교육인가
    - **대상·규모**: 누구를 몇 명이나 (사내 AX 담당자 / 임원 / 전사 / 외부)
    - **시기·기간**: 개최 희망 시기, 준비 가능 기간
    - **예산**: 가용 예산 규모 (미정 시 시나리오 설계)
    - **hwpx 양식** (선택): 사용자가 제공한 한글 양식 파일 경로
    - **기존 자료** (선택): 이전 행사 결과보고, 수요조사, 초안 기획서
2. `_workspace/` 디렉토리를 프로젝트 루트에 생성한다
3. 입력을 정리하여 `_workspace/00_input.md`에 저장한다
4. 실행 추적 파일 `_workspace/run_manifest.md`를 생성한다
5. 기존 파일이 있으면 `_workspace/`에 복사하고 재사용 여부를 `run_manifest.md`에 기록한다
6. 요청 범위에 따라 **실행 모드**를 결정한다 (아래 "작업 규모별 모드" 참조)
7. 필수 정보(행사 유형, 목적, 대상) 누락 시 사용자에게 확인한다. 시기·예산 미정은 소/중/대 시나리오 설계로 진행 가능

`run_manifest.md`는 다음 항목을 반드시 포함한다:

| 항목 | 내용 |
|------|------|
| 실행 모드 | 풀 파이프라인 / 집필 / 변환 / 검증 / hwpx |
| 입력 요약 | 행사 유형, 목적, 대상, 규모, 예산, 일정, 첨부 자료 |
| 산출물 상태 | 파일별 생성 여부, 담당 Agent, 완료 시각 또는 상태 |
| 검증·수정 이력 | 수정요청 ID, 심각도, 담당 Agent, 대상 파일, 해결 여부 |
| 최종 출력 | 사용자에게 전달할 파일 목록, hwpx 생성 여부 |

### Phase 2: 팀 구성 및 실행

| 순서 | 작업 | 담당 | 의존 | 산출물 |
|------|------|------|------|--------|
| 1 | 트렌드·벤치마크 리서치 | data-collector | 없음 | `_workspace/01_research.md` |
| 2 | 기획 전략 수립 | analyst | 작업 1 | `_workspace/02_planning_strategy.md` |
| 3 | 표·도식 설계 | visualizer | 작업 2 | `_workspace/03_visual_elements.md` |
| 4 | 보고서 집필 + 해설 | report-writer | 작업 1, 2, 3 | `_workspace/04_planning_report.md`, `_workspace/05_report_commentary.md` |
| 5 | 예상 질의 Q&A + 검증 | executive-summarizer | 작업 1~4 | `_workspace/06_executive_qa.md` |

보고서 품질을 위해 `visualizer` 완료 후 `report-writer`가 집필한다. 표·도식이 보고서 본문에 직접 통합되어야 하므로 이 단계는 병렬 처리하지 않는다.

**팀원 간 소통 흐름:**
- data-collector 완료 → analyst에게 트렌드·벤치마크+차용 포인트 전달, report-writer에게 추진배경 인용 사례 전달
- analyst 완료 → visualizer에게 도식화 대상 전달, report-writer에게 기획 전략+논리 전개 제안 전달
- visualizer 완료 → report-writer에게 표·도식+삽입 위치 전달
- report-writer 완료 → executive-summarizer에게 보고서 전문+해설 전달
- executive-summarizer는 모든 산출물을 교차 검증한다

### Phase 3: 검증 및 수정 루프

1. executive-summarizer가 `01~05` 산출물을 교차 검증하여 `_workspace/06_executive_qa.md`를 작성한다
2. `[필수 수정]` 발견 시 수정요청 ID를 부여한다: `FIX-001`, `FIX-002` 형식
3. 오케스트레이터가 대상 Agent에게 수정 요청을 전달한다
4. 수정 완료 후 `run_manifest.md`의 검증·수정 이력을 갱신한다
5. 같은 결함에 대한 재작업은 최대 2회 수행한다
6. 미해결 필수 수정이 남으면 최종 상태를 "재작업 필요"로 표시하고 사용자에게 사유를 보고한다

### Phase 4: hwpx 변환 (양식 제공 시)

1. 사용자가 제공한 hwpx 양식 파일을 확인한다
2. `.claude/skills/hwpx-autofill-conversion/skill.md`의 절차에 따라:
    - 양식 hwpx를 압축 해제하여 XML 구조와 placeholder를 분석한다
    - `Contents/section*.xml` 전체에서 입력 위치를 찾는다
    - 양식 구조를 우선하고, `04_planning_report.md`의 본문을 가능한 영역에 매핑한다
    - XML well-formed 검증과 ZIP 재압축 검증 후 `_workspace/07_final.hwpx`를 생성한다
3. 양식 미제공 시 이 Phase는 건너뛰고, 사용자에게 "양식 제공 시 hwpx 변환 가능"을 안내한다

### Phase 5: 통합 및 최종 보고

1. `_workspace/run_manifest.md`와 산출물 파일을 대조한다
2. 검증 보고서의 `[필수 수정]`이 모두 반영되었는지 확인한다
3. 최종 요약을 사용자에게 보고한다:
    - 실행 추적 — `run_manifest.md`
    - 리서치 — `01_research.md`
    - 기획 전략 — `02_planning_strategy.md`
    - 표·도식 — `03_visual_elements.md`
    - **기획 보고서(개조식)** — `04_planning_report.md`
    - **작성 의미 해설** — `05_report_commentary.md`
    - **상급자 예상 질의 Q&A** — `06_executive_qa.md`
    - 최종 hwpx — `07_final.hwpx` (양식 제공 시)

## 산출물 계약

| 파일 | 필수 포함 내용 | 누락 시 처리 |
|------|----------------|--------------|
| `00_input.md` | 행사 유형, 목적, 대상, 규모, 예산, 시기, 첨부 자료 | 필수 3요소(유형·목적·대상) 누락 시 사용자 확인 |
| `run_manifest.md` | 실행 모드, 파일 상태, 수정 이력, 최종 출력 목록 | 오케스트레이터가 즉시 생성 |
| `01_research.md` | 트렌드, 벤치마크, 강사 후보, 출처·신뢰도 | 외부 자료 미확보 표시 후 추정 근거 명시 |
| `02_planning_strategy.md` | 프로그램 설계, 실행가능성, 리스크, 기대효과 | analyst 재작업 |
| `03_visual_elements.md` | 추진체계도, 시간표, 예산표, 일정표 | visualizer 재작업 또는 표 형태로 단순화 |
| `04_planning_report.md` | Ⅰ~Ⅷ 구조, 개조식 본문, 표·도식 통합 | report-writer 재작업 |
| `05_report_commentary.md` | 전체 논리, 절별 의도, 작성 팁 | report-writer 재작업 |
| `06_executive_qa.md` | 8~12개 Q&A, 심각도별 발견 사항, 정합성 매트릭스 | executive-summarizer 재작업 |
| `07_final.hwpx` | 양식 적용 최종 파일, XML/ZIP 검증 완료 | 실패 사유 보고 후 Markdown 산출물 대체 |

## 작업 규모별 모드

| 사용자 요청 패턴 | 실행 모드 | 투입 에이전트 |
|----------------|----------|-------------|
| "AX 포럼 기획 보고서 만들어줘" | **풀 파이프라인** | 5명 전원 |
| "이 초안으로 기획서 완성해줘" (초안 제공) | **집필 모드** | analyst + visualizer + report-writer + summarizer |
| "이 기획서 개조식으로 바꿔줘" | **변환 모드** | report-writer + summarizer |
| "이 기획서 검토해줘 / 예상 질문 뽑아줘" | **검증 모드** | summarizer 단독 |
| "이 보고서 hwpx로 만들어줘" (양식+보고서 제공) | **hwpx 모드** | 오케스트레이터 직접 (hwpx-autofill-conversion) |

**기존 파일 활용**: 사용자가 초안, 수요조사, 이전 결과보고 등 기존 파일을 제공하면 해당 단계를 건너뛴다.

## 데이터 전달 프로토콜

| 전략 | 방식 | 용도 |
|------|------|------|
| 파일 기반 | `_workspace/` 디렉토리 | 주요 산출물 저장 및 공유 |
| 추적 기반 | `_workspace/run_manifest.md` | 실행 상태, 재사용 파일, 수정 이력, 최종 출력 확인 |
| 메시지 기반 | SendMessage | 실시간 핵심 정보 전달, 수정 요청 |

## 에러 핸들링

| 에러 유형 | 전략 |
|----------|------|
| 웹 검색 실패 | 리서처가 사용자 제공 자료+업계 지식 기반으로 작업, "외부 자료 미확보" 명시 |
| 예산·시기 미정 | 소/중/대 3개 시나리오로 설계하여 선택지 제공 |
| 에이전트 실패 | 1회 재시도 → 실패 시 해당 산출물 없이 진행, 검증 보고서에 누락 명시 |
| 검증에서 필수 수정 발견 | 수정요청 ID 발급 → 담당 Agent 재작업 → manifest 갱신 → 재검증 (최대 2회) |
| hwpx 양식 분석 실패 | 원인(암호화, 손상)을 사용자에게 보고하고 마크다운 산출물로 대체 |
| 행사 유형 불명확 | 목적 기준 기본값 적용: 확산·네트워킹=포럼, 실습=워크샵, 역량강화=교육 |

## 테스트 시나리오

### 정상 흐름
**프롬프트**: "사내 AX 담당자 대상 생성형 AI 활용 워크샵 기획 보고서를 만들어줘. 11월 개최, 예산 2천만원, 50명 규모야."
**기대 결과**:
- 리서치: 생성형 AI 교육 트렌드, 유사 워크샵 벤치마크, 강사 후보
- 전략: 실습 중심 프로그램 설계, 예산 2천만원 내 실행가능성 검토, 섭외 리스크 대응
- 도식: 추진체계도, 프로그램 시간표, 예산표(산출 근거 포함), 추진 일정표
- 보고서: Ⅰ~Ⅷ 표준 구조, 개조식·「~함」체, 수치 근거 병기
- 해설: 절별 작성 의도·상급자 관점·작성 팁 (서술형)
- Q&A: 예상 질의 8개 이상 + 정합성 매트릭스 전항목 확인

### hwpx 변환 흐름
**프롬프트**: "이 기획서를 첨부한 한글 양식에 맞춰 hwpx로 만들어줘" + 양식 파일 제공
**기대 결과**:
- 양식 hwpx 압축 해제 및 XML 구조 분석
- 양식 서식 유지한 채 본문 채움
- `_workspace/07_final.hwpx` 생성, 한글에서 열림 확인 안내

### 에러 흐름
**프롬프트**: "AX 포럼 기획해줘, 예산이랑 시기는 아직 몰라"
**기대 결과**:
- 소/중/대 3개 예산 시나리오로 설계
- 시기별(상반기/하반기) 고려사항 제시
- 보고서에 "(안)" 표기와 확정 필요 항목 명시

## 에이전트별 확장 스킬

| 확장 스킬 | 경로 | 대상 에이전트 | 역할 |
|----------|------|-------------|------|
| public-report-style | `.claude/skills/public-report-style/skill.md` | report-writer | 개조식 기호 체계, 공공기관 어투 규칙, 표현 변환 사전 |
| executive-qa-patterns | `.claude/skills/executive-qa-patterns/skill.md` | analyst, executive-summarizer | 상급자 유형별 질의 패턴, 실행가능성 점검 프레임워크 |
| hwpx-autofill-conversion | `.claude/skills/hwpx-autofill-conversion/skill.md` | 오케스트레이터 | hwpx 양식 분석·자동 채움·재압축 절차 |
