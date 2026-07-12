---
name: hwpx-autofill-conversion
description: "hwpx 양식을 첨부하면 첨부 양식에 맞춰 주제를 작성. 한글(hwpx) 양식 파일을 분석하여 서식을 유지한 채 내용을 채우고, 최종 hwpx 파일로 재조립한다. '한글 파일로 만들어줘', 'hwpx로 변환' 요청 시 사용."
---

# HWPX Autofill Conversion — 한글 양식 자동 채움 변환

첨부된 hwpx 양식 파일을 분석하여, 양식의 서식을 유지한 채 작성된 보고서 내용을 채우고 최종 hwpx 파일로 산출한다.

## 처리 절차

1. **첨부된 hwpx 파일을 xml로 분류**: hwpx는 ZIP 컨테이너이므로 압축을 해제하여 내부 XML 파일들을 확인한다
2. **페이지·구역별로 세부 분류**: `Contents/section*.xml` 전체를 XML parser로 파싱하여 문단·표·입력 위치를 파악한다
3. **placeholder 우선 매핑**: 양식에 `{{제목}}`, `[행사명]`, `○○○` 같은 입력 표시가 있으면 해당 위치를 우선 교체한다
4. **구조 기반 매핑**: placeholder가 없으면 제목, 개요 표, 본문 영역, 붙임 영역 등 양식 구조를 기준으로 보고서 내용을 배치한다
5. **서식 보존 편집**: 양식의 서식 ID(header.xml 참조)를 유지하면서 텍스트 노드 중심으로 교체·삽입한다
6. **최종 결과는 hwpx 파일로 작업 (필수)**: 수정된 XML을 원래 구조 그대로 재압축하여 `.hwpx` 파일로 산출한다

## hwpx ZIP 파일 구조

### 1. 최상위 구조 및 메타 정보

- **mimetype**: 이 파일이 HWPX 패키지임을 명시하는 짧은 텍스트 파일 (`application/hwp+zip`이 기록됨)
- **META-INF/ 폴더**
  - `manifest.xml`: 압축 파일 내부에 포함된 모든 파일의 경로와 각각의 미디어 타입(MIME Type)을 리스트 형태로 정의하는 문서
- **BinData/ 폴더**: 문서에 삽입된 이미지 파일, 도형, OLE 객체 등 바이너리 데이터가 원본 형태로 저장됨
- **Preview/ 폴더**: 문서를 열지 않고도 내용을 미리 볼 수 있도록 썸네일 이미지(`PrvImage.png`)나 텍스트(`PrvText.txt`)가 저장됨

### 2. Contents/ 폴더 (핵심 데이터)

문서의 실제 본문 내용, 서식, 메타데이터 등 문서 분석 시 가장 중요하게 다뤄지는 XML 파일들이 모여 있는 곳.

- **content.hpf**: 패키지 전체의 명세서 역할. 문서의 메타데이터(작성자, 제목 등), 파일 구성 요소(Manifest 아이템), 파일을 읽는 순서(Spine)를 정의
- **header.xml**: 문서 전반에 적용되는 서식 정보. 글꼴, 문단 스타일, 글자 모양, 표의 테두리 및 배경, 개요 번호 매기기 등이 선언됨. 본문(section)에서는 이곳에 부여된 ID 값을 참조하여 스타일을 적용
- **section0.xml, section1.xml, ...**: 실제 본문 텍스트와 객체가 담긴 핵심 파일. 구역(Section)을 나눌 때마다 파일 숫자가 증가. 텍스트, 표, 그림의 위치 정보 등이 모두 XML 태그 형태로 기록됨 (예: 문단은 `<hp:p>`, 실제 텍스트는 `<hp:t>` 태그 내부에 위치)
- **settings.xml**: 문서의 보기 배율, 호환성 설정, 기본 언어 등 편집기가 문서를 열 때 필요한 환경 설정 정보

## 작업 시 준수 사항

- **서식 보존**: `charPrIDRef`, `paraPrIDRef` 등 header.xml의 서식 참조 ID를 변경하지 않는다
- **구조 보존**: 양식의 표(`<hp:tbl>`), 문단 순서, 구역 구성을 임의로 재배치하지 않는다
- **다중 section 처리**: 본문이 `section0.xml` 하나에만 있다고 가정하지 않고 `Contents/section*.xml` 전체를 확인한다
- **텍스트 run 분할 주의**: HWPX는 하나의 문장이 여러 `<hp:t>` 노드로 나뉠 수 있으므로 단순 정규식 일괄 치환에 의존하지 않는다
- **양식 우선 원칙**: 양식 구조가 보고서 구조와 다르면 양식을 우선하고, 들어가지 못한 내용은 별도 붙임으로 정리한다
- **재압축 규칙**: `mimetype` 파일은 압축하지 않고(Stored) ZIP의 첫 번째 엔트리로 배치한다. 나머지 파일은 Deflate 압축한다
- **XML 인코딩**: UTF-8을 유지하고, 특수문자(`&`, `<`, `>`)는 XML 엔티티로 이스케이프한다
- **검증**: 재압축 후 ZIP 정상 열림, `Contents/section*.xml` well-formed, 필수 placeholder 잔존 여부를 확인한다
- **산출 경로**: 최종 파일은 `_workspace/07_final.hwpx`로 저장한다

## 권장 구현 (Python)

```python
import zipfile
from pathlib import Path
from xml.etree import ElementTree as ET

# 1. 압축 해제
with zipfile.ZipFile("template.hwpx") as z:
    z.extractall("hwpx_work")

# 2. Contents/section*.xml 전체를 parser로 확인
for section_path in Path("hwpx_work/Contents").glob("section*.xml"):
    tree = ET.parse(section_path)
    root = tree.getroot()
    # placeholder 또는 입력 위치를 찾아 텍스트 노드 중심으로 수정
    tree.write(section_path, encoding="utf-8", xml_declaration=True)

# 3. 재압축: mimetype을 무압축 첫 엔트리로
with zipfile.ZipFile("_workspace/07_final.hwpx", "w") as z:
    z.write("hwpx_work/mimetype", "mimetype", compress_type=zipfile.ZIP_STORED)
    # 이후 나머지 파일을 ZIP_DEFLATED로 추가 (mimetype 제외)
```

## 검증 체크리스트

| 항목 | 기준 |
|------|------|
| ZIP 구조 | 재압축 파일이 ZIP으로 열림 |
| mimetype | 첫 번째 엔트리이며 무압축 저장 |
| XML 정합성 | `Contents/section*.xml` 전체가 well-formed |
| placeholder | 필수 입력 표시가 남아 있지 않음 |
| 서식 참조 | `charPrIDRef`, `paraPrIDRef` 등 기존 ID 훼손 없음 |
| 대체 산출 | 변환 실패 시 실패 사유와 Markdown 최종본 제공 |

## 에러 핸들링

- 양식 파일이 hwp(구버전 바이너리)인 경우: hwpx로 다시 저장하여 제공해 달라고 안내한다
- 암호화·배포용 문서인 경우: 분석 불가 사유를 보고하고 마크다운 산출물로 대체한다
- 양식 구조가 보고서 구조와 크게 다른 경우: 양식 구조를 우선하고, 들어가지 못한 내용은 별도 붙임으로 정리한다
